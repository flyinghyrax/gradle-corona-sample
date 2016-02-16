# Corona Enterprise Android Library #

_Gradle Edition_

This is the Corona Android library from `/Applications/CoronaEnterprise/Corona/android/lib/Corona`, after being imported into Android Studio and turned into a module.

Ported from build **2015.2571**

## Why

For old-style (Eclipse/Ant) builds, Corona's Android library continues to live in `/Applications/CoronaEnterprise`. Your project usually symlinks to the library so that it can be incorporated into the build, while still allowing it to be updated along with the rest of the Corona Enterprise installation.

Sadly, although there may be a way in Gradle to build an Ant project and integrate it as a library - I don't know what it is.  So it was much easier to simply run the Corona library through Android Studio's import feature, then make it a module dependency for the main app module.

## Updating

This means that the Corona library must be updated manually as follows:

1. JAR files from `/Applications/CoronaEnterprise/Corona/android/lib/Corona/libs/` should be copied to `<project>/coronaLib/libs/`
2. Contents of `/Applications/CoronaEnterprise/Corona/android/lib/Corona/libs/armeabi-v7a` should be copied to `<project>/coronaLib/src/main/jniLibs/armeabi-v7a/`

That should usually be all that is necessary, though you may also want to compare the `AndroidManifest.xml` from the two different projects for changes, as well as making sure that `<project>/coronaLib/build.gradle` reflects the correct `compileSdkVersion` and lists all the JAR files as dependencies.
