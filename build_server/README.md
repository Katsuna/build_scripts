# Documentation - Build scripts

## Setting up the Katsuna Build Server & related scripts

The available commands are:

| Name                                                  | Result                                                                      |
|:------------------------------------------------------|:----------------------------------------------------------------------------|
| [katsuna-build](#katsuna-build)                       | start a Katsuna build on the server                                         |
| [katsuna-build-gradle](#katsuna-build-gradle)         | start a gradle build of Katsuna Apps on the server                          |
| [katsuna-build-init](#katsuna-build-init)             | initialise the Katsuna build server                                         |
| [katsuna-build-local](#katsuna-build-local)           | start a local Katsuna build on your development machine                     |
| [katsuna-build-local-init](#katsuna-build-local-init) | initialise your local Katsuna build environment on your machine             |
| katsuna-deploy                                        | deploy (upload) the build into the updater-server                           |
| katsuna-deploy-gradle                                 | deploy (upload) the Katsuna Apps into the updater-server (nightlies)        |
| [katsuna-monthly-merge](#katsuna-monthly-merge)       | automate the AOSP code merge step for monthly security releases             |
| [katsuna-monthly-updates](#katsuna-monthly-updates)   | automate the firmware & blobs extraction step for monthly security releases |
| ~~[katsuna-nightly](#katsuna-nightly)~~                  | ~~build a nightly (latest development code) build~~                             |
| katsuna-notify                                        | update the public json used by the client app                               |
| [katsuna-publish-release](#katsuna-publish-release)   | publish a Katsuna ROM public release                                        |
| [katsuna-release](#katsuna-release)                   | create a Katsuna ROM public release                                         |
| [katsuna-sync](#katsuna-sync)                         | customized 'repo sync' wrapper for faster syncs                             |

#### katsuna-build

1. Type into terminal:
```
katsuna-build "DEVICE_NAME" "TYPE_NAME" "ANDROID_VERSION" "JENKINS_INSTANCE"
```
e.g.:
```
katsuna-build "bullhead" "dev" "oreo" "true"
```

#### katsuna-build-gradle

1. Type into terminal:
```
katsuna-build-gradle "APP_NAME" "TYPE_NAME" "ANDROID_VERSION"
```
e.g.:
```
katsuna-build-gradle "KatsunaContacts" "dev" "oreo"
```

* One can also specify **all** as *APP_NAME* and all apps will be built.   
```
katsuna-build-gradle "all" "dev" "oreo"
```

**INFO**: Because the scripts checks latest HEAD commits to avoid unnecessary building, there is a `force (true|false)` argument one can use to force build the app:    
e.g.:
```
katsuna-build-gradle "KatsunaContacts" "dev" "oreo" "true"
```

#### katsuna-build-init

**IMPORTANT!**

*The script is configured to download the **Nougat** and **Oreo** source automatically. If you want to modify that, edit the script and toggle the **GET_XX** variables to true/false **BEFORE** running the script for the first time!*

1. Type into terminal:
```
katsuna-build-init
```

2. Wait for the script to finish. The ROM source should be located under **/root/android/**

* One can also execute specific functions by directly using their name, e.g:
```
katsuna-build-init "fixPerlWarning"
```

The allowed functions are:

|       Name          | Result                                                                                     |
|:--------------------|:-------------------------------------------------------------------------------------------|
| fixPerlWarning      | fixes a Debian apt-get install warning                                                     |
| backportOpenJDK8    | backports OpenJDK 8.x into Debian Jessie                                                   |
| depInstall          | installs all necessary dependencies                                                        |
| setJDK **$JAVASDK** | sets the JAVA variables to specified variable                                              |
| installRepo         | installs [Google's repo tool](https://source.android.com/setup/using-repo)                 |
| installGitLFS       | installs [Github's LFS tool](https://github.com/git-lfs/git-lfs/blob/master/INSTALLING.md) |
| setGitVars          | set gitconfig --global variables                                                           |
| preAddGithub        | adds Github to the SSH known_hosts file                                                    |
| downloadROM         | repo init, repo sync the ROM                                                               |
| enableCcache        | enable the CCACHE for the ROM builds                                                       |
| installAndroidSDK   | setup a headless Android SDK installation                                                  |

* Lastly, one can trigger a _DEBUG_ run:
```
katsuna-build-init "DEBUG"
katsuna-build-init "setJDK" "8" "DEBUG"
```

**INFO**: _DEBUG_ argument should **always be the last one**.       
When _DEBUG_ is set:

|       Name                  | Result                                         |
|:----------------------------|:-----------------------------------------------|
| depInstall _DEBUG_          | installs **debugging only** dependencies       |
| setJDK **$JAVASDK** _DEBUG_ | only prints the specified JAVA configuration   |
| downloadROM _DEBUG_         | doesn't start the **repo sync** command        |

#### katsuna-build-local

The script is interactive.

1. Type into terminal:
```
katsuna-build-local "DEVICE_NAME" "ANDROID_VERSION"
```

**INFO**: _DEVICE_NAME_ argument for the emulator is **x86**.  

e.g.:
```
katsuna-build-local "bullhead" "oreo"
```

Once started, it will ask the developer questions about the build setup, like:
* if a `katsuna-sync` should happen before building
* if the build will be a clean or a dirty one
* if the whole image/otapackage will be built or a specific package (and which one)

and only then it will start building. If a specific package name is entered, the script will try to sync it to the device/emulator with `adb sync` and kill/restart the specified app/process with `pkill`.

e.g. One wants to sync remote changes, make a dirty build and build SystemUI for the emulator, using the Oreo source:
```
$ katsuna-build-local x86 oreo

- Do you want to sync with the remote Katsuna servers first?
- y

- Should we try a clean build?
- n

- Are we building everything?
- n

- Which package should we build?
- SystemUI
```

The `SystemUI` package should be built, synced to the emulator device and the process should be killed.

#### katsuna-build-local-init

**IMPORTANT!**
  * **DON'T FORGET TO SETUP ROOT DIRECTORY** as described below in Step 1.

  * *The script is configured to download ONLY the **Oreo** source automatically. If you want to modify that, edit the script and toggle the **GET_XX** variables to true/false **BEFORE** running the script for the first time!*


1. Change into the directory you wish to use for your local Katsuna source installation.
e.g.:
```
cd ~/
mkdir katsuna
cd ~/katsuna
```

2. Type into terminal:
```
katsuna-build-local-init
```

3. Wait for the script to finish. The ROM source should be located under **/DIRECTORY-OF-CHOICE/**
e.g.: `~/katsuna/oreo/dev`

* One can also execute specific functions by directly using their name, e.g:
```
katsuna-build--local-init "depInstall"
```

The allowed functions are:

|       Name          | Result                                         |
|:--------------------|:-----------------------------------------------|
| depInstall          | installs all necessary dependencies            |
| setJDK **$JAVASDK** | sets the JAVA variables to specified variable  |
| installRepo         | installs Google's repo tool                    |
| preAddGithub        | adds Github to the SSH known_hosts file        |
| downloadROM         | repo init, repo sync the ROM                   |
| enableCcache        | enable the CCACHE for the ROM builds           |

* Lastly, one can trigger a _DEBUG_ run:
```
katsuna-build-local-init "DEBUG"
katsuna-build-local-init "setJDK" "8" "DEBUG"
```

**INFO**: _DEBUG_ argument should **always be the last one**.       
When _DEBUG_ is set:

|       Name                  | Result                                         |
|:----------------------------|:-----------------------------------------------|
| depInstall _DEBUG_          | installs **debugging only** dependencies       |
| setJDK **$JAVASDK** _DEBUG_ | only prints the specified JAVA configuration   |
| downloadROM _DEBUG_         | doesn't start the **repo sync** command        |

#### katsuna-monthly-merge

Merges the specified Android Release tag into the specified Katsuna source version, and pushes to Github.

1. Type into terminal:
```
katsuna-monthly-updates "TAG" "ANDROID_VERSION"
```
e.g.:
```
katsuna-monthly-merge android-8.1.0_r46 oreo
```

Which you can find from the [Codenames, Tags and Build Numbers](https://source.android.com/setup/start/build-numbers) site.

#### katsuna-monthly-updates

Downloads and extracts the necessary bootloader, radio and vendor image files from the specified security update image url. Besides that, it extracts and uploads the binary blobs changes into the
correspondent vendor_MANUFACTURER repository. Lastly, it updates the fingerprint values in [vendor_katsuna](https://github.com/Katsuna/vendor_katsuna) repository.

1. Type into terminal:
```
katsuna-monthly-updates "DOWNLOAD_URL" "AOSP_MERGE"
```
e.g.:
```
katsuna-monthly-updates https://dl.google.com/dl/android/aosp/bullhead-opm3.171019.016-factory-6e4c89cb.zip true
```

Which you can find from the [Factory Images for Nexus and Pixel Devices](https://developers.google.com/android/images) site. When the script is finished, the new extracted files will be found under hi.kitchuna.com/files/DEVICE_NAME/extras.

#### ~~katsuna-nightly~~

* **DEPRECATED**

You can use the main `katsuna-build` command to produce the same output.

1. Type into terminal:
```
katsuna-build "DEVICE_NAME" "dev" "ANDROID_VERSION" "JENKINS_INSTANCE" "false"
```
e.g.:
```
katsuna-build "bullhead" "dev" "oreo" "false" "false"
```

* One can also specify **all** as *DEVICE_NAME* and updates for all devices will be built and published.
```
katsuna-build "all" "dev" "oreo" "false" "false"
```

#### katsuna-publish-release

**IMPORTANT!**

* **DON'T RUN THIS AS A TEST!**    
Real users will be affected!

This script is only run by [katsuna-release](#katsuna-release) if *PUBLISH_BOOL* is set to **true**, or manually.

1. Type into terminal:
```
katsuna-publish-release "DEVICE_NAME"
```
e.g.:
```
katsuna-publish-release "bullhead"

```

#### katsuna-release

**IMPORTANT!**

* **DON'T RUN THIS AS A TEST!**    
Real users will be affected!

* *The script is configured to build using the **Oreo** source **ONLY***. If one wants to modify that, edit the script and change the `version` variable!

**BE CAREFUL with the [*PUBLISH_BOOL*](#katsuna-release) variable!**

1. Type into terminal:
```
katsuna-release "DEVICE_NAME" "JENKINS_INSTANCE" "PUBLISH_BOOL"
```
e.g.:
```
katsuna-release "bullhead" "false" "true"
```

* One can also specify **all** as *DEVICE_NAME* and updates for all devices will be built and published.
```
katsuna-release "all" "false" "true"
```

#### katsuna-sync

Wraps around the stock `repo sync` command to reduce download size and sync times. Accepts all common `repo sync` arguments, including specific project and modifiers like `--force-sync`

1. Type into terminal:
```
katsuna-sync
katsuna-sync --force-sync
katsuna-sync --force-sync packages/apps/KatsunaContacts
```
