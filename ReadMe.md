# CoronaSampleApp #

_Gradle Edition_

## Layout / Files of Interest ##

The root directory roughly follows the default Android Studio [project layout](https://developer.android.com/tools/studio/index.html#project-structure), containing the IDE's project configuration directory, project modules in subdirectories, the Gradle wrapper scripts and project build/settings scripts.

The `app/` directory contains the main application [module](https://www.jetbrains.com/idea/help/module.html). The  `com.mycompany.app` and `plugin.library` packages live in here.  The most interesting of the build scripts is the `build.gradle` for this module.

The `coronaLib/` directory contains Corona's Android library version **2015.2571** from `/Applications/CoronaEnterprise/Corona/android/lib/Corona/` ported to an Android Studio library module.  This admittedly makes updating the Corona Enterprise version for the project somewhat of a pain.
