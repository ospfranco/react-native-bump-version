# react-native-bump-version

This is the script I use to automatically bump versions on the JavaScript package.json, iOS/macOS Plist and Android Gradle.

Add it to your package.json `scripts` under `yarn bump`

```json
'bump': './bump_version.sh'
```

```bash
#!/bin/bash

set -ex

npm --no-git-tag-version version patch

# DON'T BE SILLY REPLACE THIS FOR YOUR DIRECTORY
PROJECT_DIR="macos/cidemon-macOS"
INFOPLIST_FILE="Info.plist"
INFOPLIST_DIR="${PROJECT_DIR}/${INFOPLIST_FILE}"

PACKAGE_VERSION=$(cat package.json | grep version | head -1 | awk -F: '{ print $2 }' | sed 's/[\",]//g' | tr -d '[[:space:]]')

BUILD_NUMBER=$(/usr/libexec/PlistBuddy -c "Print CFBundleVersion" "${INFOPLIST_DIR}")
BUILD_NUMBER=$(($BUILD_NUMBER + 1))

# Update plist with new values
/usr/libexec/PlistBuddy -c "Set :CFBundleShortVersionString ${PACKAGE_VERSION}" "${INFOPLIST_DIR}"
/usr/libexec/PlistBuddy -c "Set :CFBundleVersion $BUILD_NUMBER" "${INFOPLIST_DIR}"

git add .

git commit -m "Bump version"

git tag $PACKAGE_VERSION

git push
```

# Android

On android you can add the following function on `app/build.gradle`, and the replace the version string with the function call:

```gradle
def getNpmVersion() {
    def inputFile = new File(projectDir.getPath() + "/../../package.json")
    def packageJson = new JsonSlurper().parseText(inputFile.text)
    return packageJson["version"]
}

// Later down in your config
def userVer = getNpmVersion()

defaultConfig {
    ... STUFF
    versionName userVer
    ... MORE STUFF
}
```
