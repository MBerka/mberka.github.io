---
layout: post
title:  "PostgreSQL-based MQTT access control"
date:   2018-05-25 15:15:35 -0600
categories: mqtt mosquitto postgresql authentication cloud
---
<a href="http://mqtt.org">MQTT</a>'s lightweight, pub-sub model is a natural choice for IoT systems, but takes some fiddling to secure. There is less documentation than for popular protocols like HTTP, especially for specific brokers (<a href="http://mosquitto.org">Mosquitto</a> vs Apache, anyone?). Having already chosen Mosquitto due to its broad hardware support, I needed scalable per-device authentication beyond the built-in <a href="www.steves-internet-guide.com/topic-restriction-mosquitto-configuration/">Access Control List</a>, which worked fine in testing, but required a service restart to update access. A PostgreSQL installation was already configured, and was the natural place for device-related data storage.

[mosquitto-auth-plug][auth-plug] to the rescue. This Mosquitto plugin extends authentication to include a variety of databses. As can be seen from a <a href="http://my-classes.com/2015/02/05/acl-mosquitto-mqtt-broker-auth-plugin/">three-year-old guide for Mosquitto 1.3.5 and MySQL</a>, the setup is slightly involved. This post builds on that for PostgreSQL (fully supported by the plugin) and Mosquitto 1.5. It assumes that the database, git, and compilation libraries (make, gcc, g++) are already installed. Debian-style commands are used.

Since Mosquitto was already installed, it was necessary to backup the configurations from /etc/mosquitto and uninstall. The initial steps to build the fresh Mosquitto consisted of:

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
It was now time to configure Mosquitto. My prior mosquitto.conf had some persistence and log settings, so I left those in place instead o overwriting everything with the example. I appended a modified version of the guide's text, fortunately [shorter for PostgreSQL][postgres-params]:
{% highlight plaintext %}
auth_plugin /etc/mosquitto/auth-plug.so
auth_opt_backends postgres
auth_opt_dbname [dbname] %[use your own]
auth_opt_user [user]
auth_opt_pass [password]
auth_opt_userquery SELECT pw FROM account WHERE username = $1 limit 1
auth_opt_superquery SELECT COALESCE(COUNT(*),0) FROM account WHERE username = $1 AND mosquitto_super = 1
auth_opt_aclquery SELECT topic FROM acls WHERE (username = $1) AND (rw & $2) > 0
{% endhighlight %}
The last three are default PostgreSQL calls (on principle, I recommend using your own schema). Per the [documentation][auth-plug], 
* userquery: mandatory query returning a 1x1 result with the PBKDF2 password hash for a given user
* superquery: query for superusers who are exempt from access control restrictions, returning 1x1 entry with 0/1 indicating whether th user is a superuser (useful since I needed a global user to read from all the channels)
* aclquery: query returning a single column with any number of rows, each containing an [MQTT topic string][mqtt-topics].

[auth-plug]: https://github.com/jpmens/mosquitto-auth-plug/ 
[postgres-params]: https://github.com/jpmens/mosquitto-auth-plug#postgresql
[mqtt-topics]: https://www.eclipse.org/paho/files/mqttdoc/MQTTClient/html/wildcard.html
