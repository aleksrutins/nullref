---
layout: docpage
title: Automated Builder Script
author: Aleks Rutins
---
# Automated Builder Script
This script (for Linux and macOS) builds, runs, and manages your FTC project for you. It has menu and CLI interfaces. Put it in a file called `builder.sh` in the root directory of your Android Studio project. A makefile to integrate with other IDEs (e.g. GNOME Builder) may be found further down the page.

Dependencies:
- Java (installed by option 1, with Homebrew)
- Gradle (installed by option 1, with Homebrew)
- tput
- Bash

```sh
#!/usr/bin/env bash
# MIT License

# Copyright (c) 2021 Aleks Rutins

# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:

# The above copyright notice and this permission notice shall be included in all
# copies or substantial portions of the Software.

# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.
has_homebrew=1
backlines() {
    for ((i=0;i<$1;i++)); do
        printf "\e[1A\e[K"
    done
}

filename() {
    path=$1
    path=${path##*/}
    path=${path%.*}
    printf $path
}

asdkloc=$(cat asdk_location.dat)
page() {
    clear
    printf "\e[1;32m - Automated macOS/Linux Builder - \e[0m\n"
}
key() {
    printf "\e[1;32m - Press any key to continue - \e[0m\n"
    read -n1 dummy
}
asdk() {
    page
    printf "\e[1;32mAndroid SDK location: \e[0m"
    read loc
    echo $loc > asdk_location.dat
    asdkloc=$(cat asdk_location.dat)
}
asdkmangr() {
    bash -c "$asdkloc/bin/sdkmanager --sdk_root=$(bash -c "cd $asdkloc && pwd") $*"
}
asdkman() {
    page
    asdkmangr "'platforms;android-29'"
    asdkmangr "'build-tools;29.0.2'"
}
menu() {
    page
    printf "
    \e[1;32mAndroid SDK location: ${asdkloc:-"Unset - please use item 5 to set"}\e[0m

    \e[1;32mWhat would you like to do?\e[0m
    \e[1;33m (1) \e[0mInstall dependencies with Homebrew
    \e[1;33m (2) \e[0mInstall Android SDK
    \e[1;33m (3) \e[0mInstall required Android SDK components
    \e[1;33m (4) \e[0m(First build) Remove signed FtcRobotController from smartphone
    \e[1;33m (5) \e[0mBuild project \e[2m(default)\e[0m
    \e[1;33m (6) \e[0mInstall on control hub or smartphone
    \e[1;33m (7) \e[0mReboot Control Hub (after installation)
    \e[1;33m (8) \e[0mBuild, install, run & watch logs
    \e[1;33m (9) \e[0mSet Android SDK location
    \e[1;33m (q) \e[0mQuit
    "
    printf "\e[33m> \e[0m"
    read -n1 choice
    case $choice in
        1) indeps; key; menu;;
        2) asdk_install_tools; key; menu;;
        3) asdkman; key; menu;;
        4) page; adb uninstall com.qualcomm.ftcrobotcontroller; key; menu;;
        5) build; key; menu;;
        6) install; key; menu;;
        7) adb reboot; menu;;
        8) run; menu;;
        9) asdk; menu;;
        q) clear; exit 0;;
        *) build; key; menu;;
    esac
}
indeps() {
    page
    brew install java
    brew install gradle
    printf "\e[1;31mYou will need to install the Android SDK by yourself from https://developer.android.com/studio#command-tools \e[0m\n"
}
build() {
    page
    sh -c "ANDROID_SDK_ROOT=$asdkloc ./gradlew assembleDebug"
}
install() {
    page
    sh -c "ANDROID_SDK_ROOT=$asdkloc ./gradlew installDebug"
}
asdk_install_tools() {
    page
    printf "\e[1;32mWhere would you like me to put the Android SDK? \e[1;33m[~/Android] \e[0;34m"
    read loc
    echo ${loc:="~/Android"} > asdk_location.dat
    printf "`backlines 1`\e[1;32mWhere would you like me to put the Android SDK? \e[0;36m~/Android\e[0m\n"
    asdkloc=$loc
    printf "\e[1;32mDownloading...\e[0m\n"
    if [[ "$OSTYPE" == "darwin"* ]]; then
        curl https://dl.google.com/android/repository/commandlinetools-mac-6858069_latest.zip > ~/cmdline-android-sdk.zip
    elif [[ "$OSTYPE" == "linux-gnu"* ]]; then
        curl https://dl.google.com/android/repository/commandlinetools-linux-6858069_latest.zip > ~/cmdline-android-sdk.zip
    else
        printf "`backlines 1`[1;32mDownloading... \e[1;31mERROR\nCould not detect OS or using Windows\n\e[0m"
        exit 1
    fi
    printf "$(backlines 4)\e[1;32mDownloading... \e[0;32mDone\e[0m\n"
    printf "\e[1;32mUnzipping...\e[0m\n"
    unzip -q ~/cmdline-android-sdk.zip -d ~/cmdline-android-sdk
    mv ~/cmdline-android-sdk/cmdline-tools $(bash -c "mkdir -p $asdkloc && cd $asdkloc && pwd && cd .. && rm -rf $asdkloc")
    rm -rf ~/cmdline-android-sdk
    printf "`backlines 1`\e[1;32mUnzipping... \e[0;32mDone\e[0m\n"
}
run() {
    install && adb shell monkey -p com.qualcomm.ftcrobotcontroller 1 && adb logcat
}
tput smcup
page
echo "Detecting Homebrew..."
brew -v > /dev/null 2>&1

if [ "$?" != "0" ]; then
    has_homebrew=0
fi

if [ "$1" == "" ]; then
    menu
else
    tput rmcup
    case $1 in
        install-deps) indeps;;
        build) build;;
        install) install;;
        run) run;;
        asdk-set-location) asdk;;
        asdk-install-components) asdkman;;
        *) clear; printf "
\e[1;32mUsage:\e[0m
\e[1;33m<no arguments>\e[0m - Show the menu system
\e[1;33minstall-deps\e[0m - Install dependencies via Homebrew
\e[1;33mbuild\e[0m - Build project
\e[1;33minstall\e[0m - Install project on control hub
\e[1;33mrun\e[0m - Build, install, run & watch logs
\e[1;33masdk-set-location\e[0m - Set Android SDK location
\e[1;33masdk-install-components\e[0m - Install Android SDK components
";;
esac
tput smcup
fi
tput rmcup
```

The Makefile:
```makefile
# MIT License

# Copyright (c) 2021 Aleks Rutins

# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:

# The above copyright notice and this permission notice shall be included in all
# copies or substantial portions of the Software.

# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.

# Hmm, the license is longer than the code.

.DEFAULT_TARGET := all
all:
        @./builder.sh build
install:
        @./builder.sh install
```