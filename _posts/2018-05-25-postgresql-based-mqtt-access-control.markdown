---
layout: post
title:  "PostgreSQL-based MQTT access control"
date:   2018-05-25 15:15:35 -0600
categories: web
---
[MQTT](http://mqtt.org)'s lightweight, pub-sub model is a natural choice for IoT systems, but takes some fiddling to secure. There is less documentation than for popular protocols like HTTP, especially for specific brokers ([Mosquitto(http://mosquitto.org) vs Apache, anyone?). Having already chosen Mosquitto due to its broad hardware support, I needed scalable per-device authentication beyond the built-in <a href="www.steves-internet-guide.com/topic-restriction-mosquitto-configuration/">Access Control List</a>, which worked fine in testing, but required a service restart to update access. A PostgreSQL installation was already configured, and was the natural place for device-related data storage.

[mosquitto-auth-plug][auth-plug] to the rescue. This Mosquitto plugin extends authentication to include a variety of databses. As can be seen from a [three-year-old guide for Mosquitto 1.3.5 and MySQL](http://my-classes.com/2015/02/05/acl-mosquitto-mqtt-broker-auth-plugin/), the setup is slightly involved. This post builds on that for PostgreSQL (fully supported by the plugin) and Mosquitto 1.5. It assumes that the database, git, and compilation libraries (make, gcc, g++) are already installed. Debian-style commands are used.

# Initial installation and build

Since the plugin must be built with a path to the source, I had to uninstall Mosquitto first (you may want to back up any complex configurations from /etc/mosquitto first, and fully purge the installation to avoid problems on re-install). The initial steps to build the fresh Mosquitto consisted of:

{% highlight bash %}
sudo apt-get install libc-ares-dev libssl-dev uuid uuid-dev
wget http://mosquitto.org/files/source/mosquitto-1.5.tar.gz
tar xvzf mosquitto-1.5.tar.gz
cd mosquitto-1.5
make mosquitto
sudo make install
{% endhighlight %}

The setup for the plugin remained mercifully unchanged:
{% highlight bash %}
git clone https://github.com/jpmens/mosquitto-auth-plug.git
cd mosquitto-auth-plug
cp config.mk.in config.mk
{% endhighlight %}

Editing config.mk, to account for the different database and folder,
{% highlight markdown %}
BACKEND_MYQSL ?= no
# ...
BACKEND_POSTGRES ?= yes
# ...
# Specify the path to the Mosquitto sources here
MOSQUITTO_SRC = [path]/mosquitto-1.5
{% endhighlight %}
([path] will vary depending on the earlier tar)

Making mosquitto-auth-plug consisted of:
{% highlight bash %}
make clean #recommended by the make script since configuration changed
sudo apt-get install libpq-dev #The server was already installed and working, and this was supposed to install without it, but otherwise I hit a missing header error
make
sudo mv auth-plug.so /etc/mosquitto/ #As in the guide
{% endhighlight %}

# Configuration and database

It was now time to configure Mosquitto. My prior mosquitto.conf had some persistence and log settings, so I left those in place instead o overwriting everything with the example. I appended a modified version of the guide's text, fortunately [shorter for PostgreSQL][postgres-params]:
{% highlight plaintext %}
auth_plugin /etc/mosquitto/auth-plug.so
auth_opt_backends postgres
auth_opt_dbname [dbname] %[use your own]
auth_opt_user [user]
auth_opt_pass [password]
auth_opt_userquery SELECT pw FROM account WHERE username = $1 limit 1
auth_opt_superquery SELECT COALESCE(COUNT(*),0) FROM account WHERE username = $1 AND superuser = 1
auth_opt_aclquery SELECT topic FROM acls WHERE (username = $1) AND (rw & $2) > 0
{% endhighlight %}
The last three are default PostgreSQL calls (on principle, I recommend using your own schema). Per the [documentation][auth-plug],
* userquery: mandatory query returning a 1x1 result with the PBKDF2 password hash for a given user
* superquery: query for superusers who are exempt from access control restrictions, returning 1x1 entry with 0/1 indicating whether th user is a superuser (useful since I needed a global user to read from all the channels)
* aclquery: query returning a single column with any number of rows, each containing an [MQTT topic string][mqtt-topics].

It was time to test the np password generation utility, and set up the database.

{% highlight bash %}
./np #Enter a password twice
Enter password:
Re-enter same password:
PBKDF2$sha256$901$gjOcwLF+xs92U5Mf$wHdnuPJTAdhTsy7bMOco+9vtO2W86K8h
sudo psql -U [dbuser]
\connect [database]
#'You are now connected to database "[database]" as user "[dbuser]"'
{% endhighlight %}
{% highlight sql %}
CREATE TABLE account (
id serial PRIMARY KEY,
username varchar(20) NOT NULL,
pw varchar(100) NOT NULL,
superuser smallint NOT NULL DEFAULT 0,
CONSTRAINT superuser CHECK (superuser = 0 OR superuser = 1)
);
-- CREATE TABLE
INSERT INTO account(username, pw)
VALUES
('testuser', 'PBKDF2$sha256$901$gjOcwLF+xs92U5Mf$wHdnuPJTAdhTsy7bMOco+9vtO2W86K8h');
-- INSERT 0 1
SELECT * FROM account;
-- Should show new user

CREATE TABLE acls (
id serial PRIMARY KEY,
username varchar(20) NOT NULL,
topic varchar(40) NOT NULL,
rw smallint NOT NULL DEFAULT 1,
CONSTRAINT rw CHECK (rw >= 1 AND rw <= 4)
);
-- CREATE TABLE
INSERT INTO acls(username, topic, rw)
VALUES
('testuser', 'testuser/#', 2);
-- INSERT 0 1
SELECT * FROM acls;
-- Should show new user
{% endhighlight %}

# Testing

{% highlight bash %}
cd /usr/local/sbin
./mosquitto -c /etc/mosquitto/mosquitto.conf
{% endhighlight %}
After a few false starts (failing silently with the -c argument, yet running normally without it), the logs revealed the [error described by Rex Xia](http://rexpie.github.io/2015/08/25/mosquitto-troubleshooting.html), with the same solution:
{% highlight bash %}
sudo ln -s /usr/local/lib/libmosquitto.so.1 /usr/lib/libmosquitto.so.1
{% endhighlight %}
From here, Mosquitto and the plugin ran perfectly. Note that any flaw in the PostgreSQL authentication for the user configured in mosquitto.conf, or in that user's access to the newly created tables, reveals itself here, bringing authentication/authorization checks to a premature halt.

Success looked like
{% highlight plaintext %}
1527399177: |-- mosquitto_auth_unpwd_check(testuser)
1527399177: |-- ** checking backend postgres
1527399177: |-- GETTING USERS: testuser
1527399177: |-- getuser(testuser) AUTHENTICATED=1 by Postgres
{% endhighlight %}
on subscribe with, e.g., [MQTT.fx](http://mqttfx.jensd.de/) (any credentials besides testuser/password fail to connect), and:
{% highlight plaintext %}
USERNAME: testuser, TOPIC: testuser/test, acc: 2
1527400680: |--   postgres: topic_matches(testuser/#, testuser/#) == 1
1527400680: |-- aclcheck(testuser, testuser/test, 2) trying to acl with postgres
1527400680: |-- aclcheck(testuser, testuser/test, 2) AUTHORIZED=1 by postgres
{% endhighlight %}
when publishing to testuser/test. Oddly enough, though an rw value of 3 should grant both read and write, changing this value was not enough for testuser. Since the following appeared whenever I attempted a subscription to the same topic,
{% highlight plaintext %}
1527401125: |-- USERNAME: testuser, TOPIC: testuser/#, acc: 4
{% endhighlight %}
I set a permission of 4 and was able to both publish a message and receive it with the same connection. Attempting to subscribe to any unpermitted topic still caused the connection to terminate.

# Next steps

1. Consider the schema - does the default table structure make sense? If you will only ever need one topic string per user, you may wish to combine the account and acl tables.
2. Personalize the schema - rename tables/columns to make your database a bit less obvious.
3. Set up tools for managing the new permission tables. For me, the main stumbling block was reproducing the np script's encryption in [Node-RED](https://nodered.org/).

[auth-plug]: https://github.com/jpmens/mosquitto-auth-plug/
[postgres-params]: https://github.com/jpmens/mosquitto-auth-plug#postgresql
[mqtt-topics]: https://www.eclipse.org/paho/files/mqttdoc/MQTTClient/html/wildcard.html
