[bgfx android activity](https://github.com/nodrev/bgfx-android-activity) - Android glue for bgfx
================================================================================================

A minimal Android Activity using `NativeActivity` class which allows to run [bgfx](https://github.com/bkaradzic/bgfx)'s examples onto Android platforms.
![Android emulator with helloworld example](https://github.com/nodrev/bgfx-android-activity/raw/master/app/src/main/screenshot.png)

Forked from: https://github.com/Nodrev/bgfx-android-activity

# Prerequisites

**Remark**: Although those build instructions assume a linux platform to generate the APKs, the used tools are available for OSX and MSWindows. As a result, the specified commands should be easily adapted to work for those platforms.

## Determine minimum target

By default, `bgfx` targets Android API 24 (Android Nougat 7.0). So, if you have a phone or a tablet which runs an older Android version, you'll need to target the corresponding API number.

Determine the minimal Android version you wish to support, and get the corresponding API level using this page: https://source.android.com/source/build-numbers

## Android Studio

This project uses [Android Studio](http://developer.android.com/sdk/index.html) build system, Gradle, to generate Android's APK files, so you need to download it and install it properly. Android Studio comes with Android SDK, so no need to install it separately.

## Environment variables

Add `ANDROID_NDK_ROOT` as an environment variable. Should contain `toolchains` folder.

You can install Android NDK through Android Studio's SDK manager.

You can also extend the `PATH` variable to be able to access Android platform tools (`adb`, `dmtracedump`, etc) from the shell.

# Setup project

## Clone repositories

Make sure to clone the repository with included submodules: `git clone --recursive`.

## Compile

First, modify the file `external/bgfx/makefile` and modify the `projgen` to provide the minimal supported Android platform (add `--with-android=xx`):
```makefile
	$(GENIE) --with-android=23 --with-combined-examples --gcc=android-arm gmake
	$(GENIE) --with-android=23 --with-combined-examples --gcc=android-x86 gmake
```

Then, compile BGFX samples for every Android ABI we want to support:
```shell
cd external/bgfx
make projgen
make android-arm & make android-arm64 & make android-x86
```
Also modify `build.gradle` for chosen ABIs (default enabled are `armeabi-v7a` and `arm64-v8a`).
# Build APK

## Modify application ID

Import the project in Android Studio, and edit `bgfx-android-activity/app/build.gradle`. Replace `applicationId` with your own application id. Set `compileSdkVersion` and `targetSdkVersion` to your android platform number:
```
    compileSdkVersion 26
    defaultConfig {
        applicationId 'com.nodrev.bgfx.examples'
        minSdkVersion 23 // The version we use to compile bgfx
        targetSdkVersion 26
        versionCode 100 // Application version, 3 digits, major/minor/revision
        versionName "1.0.0"
    }
```

Edit `bgfx-android-activity/app/src/main/AndroidManifest.xml`, and set `package` value to the same application id:
```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.nodrev.bgfx.examples">
    [...]
</manifest>
```

## Modify application name

Edit `bgfx-android-activity/app/src/main/res/values/strings.xml`, and replace `app_name` value with your application name
```
<resources>
    <string name="app_name">BGFX Examples</string>
</resources>
```

## Define the library name

To define the .so file to load by the native activity, you have to edit `bgfx-android-activity/app/src/main/AndroidManifest.xml`
```
<activity android:name="android.app.NativeActivity"
    <!-- Tell NativeActivity the name of our .so (strip 'lib' and '.so') -->
    <meta-data android:name="android.app.lib_name"
               android:value="examplesRelease" />
</activity>
```

Modify the .so to load in `BgfxAndroidActivity.java` too:
```java
public class BgfxAndroidActivity extends android.app.NativeActivity
{
    static
    {
        System.loadLibrary("c++_shared");
        System.loadLibrary("examplesRelease");
    }
}
```

## Resource files

Some examples requires resource files, you will need to copy them to the Android device (physical or emulator) SDCard using `adb`:
```shell
~/android/sdk/platform-tools/adb push bgfx/examples/runtime /sdcard/bgfx/examples/runtime
```

**Remark:** This is not the official way to do for a real application, runtime files should be embedded into APK, but for bgfx examples, we go that way.

## Packaging

Launch android studio, and import the project. Select `Build` menu, and generate APK using `Make Project` entry.

*Note: If you change the build variant to release, you'll need to sign your APK before deployment, this is off this tutorial's scope*

To deploy to your target device, go to the `Run` menu and either choose `Run 'app'` or `Debug 'app'` entry.

*Note: Generated APKs goes to `bgfx-android-activity/app/build/outputs/apk` directory*

[License (BSD 2-clause)](https://github.com/nodrev/bgfx-android-activity/blob/master/LICENSE)
-----------------------------------------------------------------------

<a href="http://opensource.org/licenses/BSD-2-Clause" target="_blank">
<img align="right" src="http://opensource.org/trademarks/opensource/OSI-Approved-License-100x137.png">
</a>

	Copyright 2016-2018 Jean-François Verdon. All rights reserved.
	
	https://github.com/nodrev/bgfx-android-activity
	
	Redistribution and use in source and binary forms, with or without
	modification, are permitted provided that the following conditions are met:
	
	   1. Redistributions of source code must retain the above copyright notice,
	      this list of conditions and the following disclaimer.
	
	   2. Redistributions in binary form must reproduce the above copyright
	      notice, this list of conditions and the following disclaimer in the
	      documentation and/or other materials provided with the distribution.
	
	THIS SOFTWARE IS PROVIDED BY COPYRIGHT HOLDER ``AS IS'' AND ANY EXPRESS OR
	IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF
	MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO
	EVENT SHALL COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT,
	INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
	(INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
	LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND
	ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
	(INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF
	THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
