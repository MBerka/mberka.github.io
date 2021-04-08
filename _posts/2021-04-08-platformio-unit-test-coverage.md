---
layout: post
title: "PlatformIO native unit test coverage"
date: 2021-04-08 15:30:20 -0700
categories: software, electronics
---

The PlatformIO IDE allows [unit testing with the Unity framework](https://docs.platformio.org/en/latest/plus/unit-testing.html). A [basic project template for unit testing](https://github.com/thingforward/unit-testing-with-platformio) is already available.

As shown on [this GitHub feature request](https://github.com/platformio/platformio-core/issues/882), the IDE does not currently make coverage data the easiest to obtain.

First, I used the lcov Linux test coverage project (the options and process for installing them on other systems may vary).

`sudo apt-get install lcov`

This project can summarize C++ code coverage data, which is produced when gcc is called with the `--coverage -lgcov` flags. In PlatformIO, this can be combined with your existing unit test environment. Here is one for PC-side unit testing:

```ini
[env:native]
platform = native
lib_compat_mode = off
src_filter = +<*> -<*dirs*> -<to/> -<exclude/>
; basically lib and test folders
build_flags =
  -lgcov
  --coverage
extra_scripts = test-coverage.py
```

The extra_scripts setting triggers a Python script that executes the generated code, collects an initial coverage report, then removes irrelevant library/test files:

```python
import os

Import("env")

def generateCoverageInfo(source, target, env):
    for file in os.listdir("test"):
        os.system(".pio/build/native/program test/"+file)
    os.system("lcov -d .pio/build/native/ -c -o lcov.info")
    os.system("lcov --remove lcov.info '*/tool-unity/*' '*/test/*' '*/MockArduino/*' -o filtered_lcov.info")
    os.system("genhtml -o cov/ --demangle-cpp filtered_lcov.info")

env.AddPostAction(".pio/build/native/program", generateCoverageInfo)
```
*Mostly the [work](https://github.com/platformio/platformio-core/issues/882#issuecomment-682558855) of GitHub user [jcw](https://github.com/jcw), with some cleanup.*

The test suite, stored in the test/ subdirectory and structured like

```cpp
void setup() {
    // NOTE!!! Wait for >2 secs if board doesn't support software reset via Serial.DTR/RTS
    // delay(2000);

    UNITY_BEGIN();    // IMPORTANT LINE!
    RUN_TEST(testPreQueueSetup);
    RUN_TEST(testFlagsInit);
}

void loop() {
    UNITY_BEGIN(); // stop unit testing

    RUN_TEST(testFunctionName);
    // Many more of these

    UNITY_END(); // stop unit testing
}
```

, can then be run with `pio test -e native`. Output is generated in the cov/ subdirectory of your project, accessible at `cov/index.html`.

<img alt="Browser view of genhtml report based on lcov output for unit test" src="/assets/lcov-screenshot.png">

Clicking the individual links allows you to view the per-line execution details. Best of luck!
