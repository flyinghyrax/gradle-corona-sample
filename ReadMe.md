# Corona Enterprise Android Studio Sample  #

Example of how to setup the Android portion of a Corona Enterprise project for Android Studio. See [this forum thread](https://forums.coronalabs.com/topic/54273-android-studio-integration-status/) for some background.

### *! This sample is outdated and unnecessary !*

Corona Enterprise now officially supports Android studio integration! ([announcement blog post](https://coronalabs.com/blog/2016/05/25/android-studio-templates-now-available-for-corona-enterprise/))

See the [official documentation](https://docs.coronalabs.com/native/android/index.html) for using Corona Enterprise on Android: specifically the [Android Studio integration guide](https://docs.coronalabs.com/native/android/androidStudio/index.html) and the [project structure guide](https://docs.coronalabs.com/native/android/androidProject.html). 

This repository will remain available for historical purposes.

--------------------------------------------------

## Project Structure

_(where to put these files)_

The project in this directory is a port of the _Android component_ of the Corona Enterprise template application - from `/Applications/CoronaEnterprise/ProjectTemplates/App/Android` - and is designed to be used _along with the rest of that sample project_. I recommend copying that sample project, then replacing the contents of the `android/` directory with this subproject, or creating a new `android-studio/` at the same level.  Example directory structure:

- < your project >
    + Corona
        * main.lua
        * config.lua
        * build.settings
        * ...
    + android-studio
        * < these files! >
    + ios
        * App.xcodeproj
        * Plugin.xcodeproj
        * AppCoronaDelegate.*
        * ...
    + ...

### Android sub-project structure

_(info about the files in this repository)_

The root directory roughly follows the default Android Studio [project layout](https://developer.android.com/tools/studio/index.html#project-structure), containing the IDE's project configuration directory, project modules in subdirectories, the Gradle wrapper scripts and project build/settings scripts.

#### app module

The `app/` directory contains the main application [module](https://www.jetbrains.com/idea/help/module.html). The `com.mycompany.app` and `plugin.library` packages live in here.  The most interesting of the build scripts is the `build.gradle` for this module.

- Lua plugin (Corona/native interface layer) source: `app/src/main/java/plugin/library/LuaLoader.java`
- Native application (CoronaApplication instance): `app/src/main/java/com/mycompany/app/`

#### coronaLib module

The `coronaLib/` directory contains Corona's Android library version **2015.2571** from `/Applications/CoronaEnterprise/Corona/android/lib/Corona/` ported to an Android Studio library module.  This admittedly makes updating the Corona Enterprise version for the project somewhat of a pain; see the [ReadMe in that directory](coronaLib/ReadMe.md) for more info.

## How-to's

### Setting the application package name

The package name will need to be adjusted in at least 3 places:

1. The "package" attribute of the "manifest" (root) tag in `app/src/main/AndroidManifest.xml`
2. The `android.defaultConfig.applicationId` key in `app/build.gradle`
3. The directory structure in `src/main/java` should be changed to reflect the package name.

### Setting up keystore for release builds

Release builds will not work without providing a keystore and setting its parameters in the `signingConfigs` section of `app/build.gradle`. The path to your release keystore (`storeFile`) and the alias of the key inside the keystore (`keyAlias`) can be written directly to the build file.  Placeholder keys for passwords are in the section so the script can write to them later, but normally you will not want to write your passwords there directly.

[This page in the Android documentation](https://developer.android.com/tools/publishing/app-signing.html#release-mode) describes several ways of handling release keystore passwords such as environment variables or prompting for the passwords at build time. In addition, there is also a way to keep the passwords in a separate file and have the Gradle script read them from there:

Create a file in the `app` directory with the following contents:

```
android.signingConfigs.release.storePassword = 'YourStorePassword'
android.signingConfigs.release.keyPassword = 'YourKeyPassword'
```

(For example, we will pretend the file is named "private.gradle". Be sure to add this file to `.gitignore` so that it is not included in version control!)

Then, in `app/build.gradle`, add the following line between the `android` configuration block and the `dependencies` configuration block:

```
apply from: 'private.gradle'
```

(This statement reads any Gradle instructions from the given file and applies them in the current script, thus setting your keystore passwords.)

### Updating the Corona library module

See the [coronaLib/ReadMe.md](coronaLib/ReadMe.md#updating)

## How it works

> I am not an expert with either Gradle in general or the Android Gradle plugin, so this was very ad-hoc. <br/>
> TL;DR - it's a hack.

I began by porting the custom Ant tasks into Gradle tasks as closely as I could.  There is one task to compile the Lua code and package assets, and another which certifies the Corona native components included in the build.  However, the new Android Gradle plugin build process is significantly different from the old Ant plugin process, such that there are not equivalent places to inject the custom tasks into the build. The certification task works by extracting the APK, writing a signature, then re-packing it.  Ideally, this task would run after the APK has been created but before it has been signed with jarsigner.  Unfortunately I was not able to find a way to inject a custom task into the build at this point. One step too early, and the APK did not exist yet for Corona's script to sign; one step later, and the APK which was available had already been signed/sealed with `jarsigner`.  At that stage running the Corona signing script on the APK invalidates the existing signature by extracting the APK and modifying its contents, leaving an APK that cannot be installed due to certificate errors.

As a workaround, I have created yet another custom task which _signs the APK again_ at the end of the build process, before zip-aligning. In summary:

1. The Android Gradle plugin produces a fully-signed (but not zipaligned) APK
2. The Corona CertifyBuild.sh script is run, which invalidates the signed APK by modifying its contents.
3. We inject another custom task before the end of the build to _re-sign the APK_.
4. The APK is zip aligned

## Limitations

Using Android Studio and Gradle provides several important benefits (like proper multidexing support and a better IDE), but because this integration is essentially a hack there are several notable limitations:

- Out-of-the-box, the script only supports two build types - debug and release. This aligns with Corona's Lua compiler, which only supports debug and release modes.  Additional build types could be added as long as they are mapped to either a debug or release configuration argument in the Corona Build Task.
- No support for product flavors.  It might be possible but I simply never investigated it.
- The "Build > Generate Signed APK" wizard does not work. This is due to there being a discrepancy between the save path given for the APK in the wizard and the actual build outputs path where the APK is temporarily stored. The variable we check to locate the APK is set from the wizard, but at the point in the build where we sign the APK, it has not yet been moved to that location.
- The Lua sources are compiled / Corona assets are copied on every single build, including the partial builds triggered when running "Build > Clean Project".  It definitely makes the build time longer for a larger project, but it would be potentially very complicated to solve.

