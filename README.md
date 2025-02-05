[bgfx android activity](https://github.com/krisp3t/bgfx-android-activity) - Android glue for bgfx
================================================================================================

A minimal Android Activity using `NativeActivity` class which allows to run [bgfx](https://github.com/bkaradzic/bgfx)'s examples onto Android platforms. 

Very useful to quickly debug bgfx issues on devices, such as setting up OpenGL ES with EGL context. Simple setup and quick compilation times.

![Screenshot_20250205_131036_BGFX Examples](https://github.com/user-attachments/assets/30020f58-481c-4b6e-9bf6-4c1107c3f442)



# Table of Contents
1. [Prerequisites](#prerequisites)
   - [Determine minimum target](#determine-minimum-target)
   - [Android Studio](#android-studio)
   - [Environment variables](#environment-variables)
2. [Setup project](#setup-project)
   - [Clone repositories](#clone-repositories)
   - [Compile](#compile)
3. [Build APK](#build-apk)
   - [Modify application ID](#modify-application-id)
   - [Define the library name](#define-the-library-name)
   - [Resource files](#resource-files)
   - [Packaging](#packaging)
4. [Debugging (Android Studio)](#debugging-android-studio)
5. [Licence](#licence)


# Prerequisites


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

This will build `libexamplesDebug.so` / `libexamplesRelease.so` in `bgfx-android-activity\external\bgfx\.build\android-arm64\bin` (for `android-arm64` ABI), which we will later copy


Also modify `build.gradle` for chosen ABIs (default enabled is `arm64-v8a`).
# Build APK

## Modify application ID

Import the project in Android Studio, and edit `bgfx-android-activity/app/build.gradle`. Replace `applicationId` with your own application id and set appropriate SDK numbers that you're targeting.
```gradle
android {
    namespace 'com.krisp3t.bgfx.examples'
    defaultConfig {
        applicationId 'com.krisp3t.bgfx.examples'
        compileSdk 33
        minSdk 23
        targetSdk 26
        versionCode 100 // Application version, 3 digits, major/minor/revision
        versionName "1.0.0"
    }
```

Edit `bgfx-android-activity/app/src/java/...` and edit directory path of `BgfxAndroidActivity` so that it will match `applicationId`. Then also change the package import of `BgfxAndroidActivity`:

```java
package com.krisp3t.bgfx.examples;
```

Edit `bgfx-android-activity/app/src/main/res/values/strings.xml`, and replace `app_name` value with your application name.
```xml
<resources>
    <string name="app_name">BGFX Examples</string>
</resources>
```

## Define the library name

To define the `.so` file to load by the native activity, you have to edit `bgfx-android-activity/app/src/main/AndroidManifest.xml`.
```xml
<activity android:name="android.app.NativeActivity"
    <!-- Tell NativeActivity the name of our .so (strip 'lib' and '.so') -->
    <meta-data android:name="android.app.lib_name"
               android:value="examplesDebug" />
</activity>
```

Modify the .so to load in `BgfxAndroidActivity.java` too:
```java
public class BgfxAndroidActivity extends android.app.NativeActivity
{
    static
    {
        System.loadLibrary("c++_shared");
        System.loadLibrary("examplesDebug");
    }
}
```

## Resource files

Some examples requires resource files, you will need to copy them to the Android device (physical or emulator) SDCard using `adb`:
```shell
~/android/sdk/platform-tools/adb push external/bgfx/examples/runtime /sdcard/bgfx/examples/runtime
```
> Of course, please don't do this in production and properly bundle runtime assets in your APK. The reason why we're doing this is because bgfx has this path hardcoded in `entry_android.cpp`, requiring the least work on our side.

To read any files from /sdcard, we need to require `android.permission.MANAGE_EXTERNAL_STORAGE` (from API level 33) and `android.permission.READ_EXTERNAL_STORAGE` (below API level 33). Yet again, please don't do this in production.

## Launch & packaging

Launch Android Studio, and import the project. Select `Build` menu, and generate APK using `Make Project` entry.

> If you change the build variant to release, you'll need to sign your APK before deployment, this is off this tutorial's scope.*

To deploy to your target device, go to the `Run` menu and either choose `Run 'app'` or `Debug 'app'` entry. If they are greyed out, run `Sync Project with Gradle Files` first.

> Note: Generated APKs go to `bgfx-android-activity/app/build/outputs/apk` directory.*


# Debugging (Android Studio)
When debugging app through Android Studio (either by launching in debug or attaching with debugger), you should be able to set breakpoints in native (C++) code.

> Make sure that you have native debugging turned on in Run/Debug configurations, not just Java.

You don't really need to give `.so` location to the debugger, as it should pick it up itself. It's important though that we disable debug symbols stripping (in `build.gradle`)

```gradle
buildTypes {
	release { ... }
	debug {
		...
		packagingOptions {
			jniLibs.keepDebugSymbols += '**/*.so'
		}
	}
}
```

You can either set symbolic breakpoints (with symbol names such as `GlContext::Create`) or line breakpoints, and the appropriate `.cpp` file should open up in your IDE.

![Screenshot 2025-02-05 183827](https://github.com/user-attachments/assets/48f576cb-0a38-43e3-aa50-fd2d14e035dd)


# Licence
This repository is a fork of Nodrev's [bgfx-android-activity](https://github.com/Nodrev/bgfx-android-activity). I've bumped it up to modern Android, improved build system & instructions and added debugging capabilities.

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
