[![Build Status](https://travis-ci.org/IBM-Swift/Package-Builder.svg?branch=master)](https://travis-ci.org/IBM-Swift/Package-Builder)

# Package-Builder

This repository contains build and utility scripts used for continuous integration builds on the Travis CI environment. It offers many extension points for customizing builds and tests.

## Build prerequisites

Package-Builder is intended to be used as part of Travis CI tests, and will operate on both Ubuntu 14.04 and macOS.  At a minimum, the `.travis.yml` file of your application will look something like this:

```
$ cat .travis.yml

matrix:
  include:
    - os: linux
      dist: trusty
      sudo: required
    - os: osx
      osx_image: xcode8.2
      sudo: required

before_install:
  - git clone https://github.com/IBM-Swift/Package-Builder.git

script:
  - ./Package-Builder/build-package.sh -projectDir $TRAVIS_BUILD_DIR
```

You should install any required system-level dependencies in the `before_install` section of the `.travis.yml` file so that the Travis CI build environment is ready for compilation and testing of your Swift package.

## Codecov
[Codecov](https://codecov.io/) is used in Package-Builder to determine how much test coverage exists in your code. Codecov allows us to determine which methods and statements in our code are not currently covered by the automated test cases included in the project. Codecov performs its analysis by generating an Xcode project. For example, see the current test coverage for the [Swift-cfenv](https://github.com/IBM-Swift/Swift-cfenv) package [here](https://codecov.io/gh/IBM-Swift/Swift-cfenv).



### Custom Xcode project generation
If you need custom commands to create an Xcode project, a `.swift-xcodeproj-linux` and `.swift-xcodeproj-macOS` file can be provided that contain a custom `swift package generate-xcodeproj` command.

## Custom SwiftLint
[SwiftLint](https://github.com/realm/SwiftLint) is a tool to enforce Swift style and conventions. Ensure that your team's coding standard conventions are being met by providing your own `.swiftlint.yml` in the root directory with the specified rules to be run by Package-Builder.  For now each project should provide their own `.swiftlint.yml` file to adhere to your preferences.  A default may be used in the future, but as of now no SwiftLint operations are performed unless a `.swiftlint.yml` file exists.

## Using different Swift versions and snapshots
Package-Builder uses the most recent release version of Swift, at the time of writing `3.0.2`.  If you need a specific version of Swift to build and compile your repo, you should specify that version in a `.swift-version` file in the root level of your repository.  Valid contents of this file include release and development snapshots from [Swift.org](https://swift.org/).

```
$ cat .swift-version

swift-DEVELOPMENT-SNAPSHOT-2017-02-14-a
```

## Custom build and test commands
If you need a custom command for compiling your Swift package, you should include a `.swift-build-linux` or `.swift-build-macOS` file in the root level of your repository and specify in it the exact compilation command for the corresponding platform. If you need a custom command for testing your Swift package, you should include a `.swift-test-linux` or `.swift-test-macOS` file in the root level of your repository and specify in it the exact testing command for the corresponding platform.
```
$ cat .swift-build-linux

swift build -Xcc -I/usr/include/postgresql
```

### Custom configuration for executing tests
Sometimes, a dependency must be set up in order for the testing process to be complete.  In order to leverage this extension point, include a `before_tests.sh` or `after_tests.sh` file containing the commands to be executed.

Put the relevant commands in a directory in the root, either `linux`, `osx` or `common`.  The linux or macOS `before_tests.sh` will be executed first, followed by `common/before_tests.sh`.  After the tests are performed, `common/after_tests.sh` is executed first, followed by `linux/after_tests.sh` or `osx/after_tests.sh`.

## Troubleshooting
If there is a crash during the execution of test cases, Package-Builder will perform a log dump to provide meaningful diagnosis of where the failure has occurred.
