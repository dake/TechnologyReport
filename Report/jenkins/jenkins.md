# macOS + Tomcat + Jenkins + fir.im

* iOS Continous integration with Jenkins

	* code review CI

	* daily build CI

	* ci build CD

* Distribute ipa files with fir.im

* Manage Jenkins server with ssh 



## Why Jenkins

* [iOS Continous integration: Xcode Server, Jenkins, Travis and fastlane](http://thebugcode.github.io/ios-continous-integration-choosing-a-build-server-and-tooling/)

* [IOS CONTINUOUS INTEGRATION WITH FASTLANE & JENKINS](https://apiumhub.com/tech-blog-barcelona/ios-continuous-integration/)


## Install Tomcat + java runtime

* [Installing Tomcat on macOS 10.14 Mojave](https://wolfpaulus.com/mac/tomcat/)

* [JDK management between versions](https://docs.oracle.com/javase/8/docs/technotes/guides/install/mac_jdk.html#A1096903)

We will use JDK for java 8 here, since java 11 has not widely supported by Jenkins and its plugins.
 

## Install Jenkins and launch in Tomcat

* [Download Jenkins .war package](https://jenkins.io)

* Copy `jenkins.war` into `webapp` directory in Tomcat's home directory

* Launch Tomcat server and visit [http://localhost:8080/jenkins](http://localhost:8080/jenkins)



## Configure Jenkins and Install plugins

1. [Continuous Integration and Delivery for iOS with Jenkins and Fastlane (Part 1)](https://medium.com/@cherrmann.com/continuous-integration-and-delivery-for-ios-with-jenkins-and-fastlane-part-1-3b17f1901a73)

2. Go to `Manage Jenkins` > `Configure Global Security` > set up your account of Jenkins.

3. Set system admin email in `Configure System`: which will be used to send mails.

* Configure `E-mail Notification`

* Configure `Extended E-mail Notification`
	
Configure SMTP server of your email and test connections.


### Jenkins plugins to install

Developer tool

* [Xcode integration](https://wiki.jenkins.io/display/JENKINS/Xcode+Plugin)

* [Keychains and Provisioning Profiles Management](http://wiki.jenkins-ci.org/display/JENKINS/Keychains+and+Provisioning+Profiles+Plugin)


Code repository

* [Gerrit Trigger](http://wiki.jenkins-ci.org/display/JENKINS/Gerrit+Trigger)

* [GitLab](https://wiki.jenkins-ci.org/display/JENKINS/GitLab+Plugin)

* [Gitlab Hook Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Gitlab+Hook+Plugin)

* [Gitlab Merge Request Builder](https://wiki.jenkins-ci.org/display/JENKINS/Gitlab+Merge+Request+Builder+Plugin)


Build control

* [Commit Message Trigger Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Commit+Message+Trigger+Plugin)

* [Email Extension Template](https://wiki.jenkins-ci.org/display/JENKINS/Email-ext+Template+Plugin)

* [build-name-setter](http://wiki.jenkins-ci.org/display/JENKINS/Build+Name+Setter+Plugin)

* [Environment Injector](https://wiki.jenkins-ci.org/display/JENKINS/EnvInject+Plugin)


### Configure Jenkins plugins

Go to `Manage Jenkins`

#### 1. Gerrit Trigger > Add New Server > Name it like `XXGerrit`

 Fill up basic information and `SSH Keyfile`.


#### 2. Editable Email Notification Templates > Add New Template

* iOS code review template

* iOS ci build template


Configure `Default Content`, `Advanced Settings` of your templates by different purpose.

*Plenty of variables will help your template contents configuration. You can even inject your own variables outside, such as git change logs.*


#### 3. Keychains and Provisioning Profiles Management

* Upload your keychain contains ipa signed certificates and private key here, and configure it.

* Upload your app provisioning profile files one by one.

*The first time you build an ipa generation job, the access of your keychain on your Jenkins server will request a trust with GUI prompt, and you must trust it manually.*


### Create your Jenkins jobs

#### 1. Set a display name for job if a human readable name is necessary: 
`Advanced Project Options` > `Advanced` > `Display Name`

#### 2. Add your git repo in `Source Code Management`

`Git` > `Additional Behaviours` > Add `Advanced sub-modules behaviours`:

Check on `Recursively update submodules`, if you used `git submodule`
  

`Git` > `Additional Behaviours` > Add `Strategy for choosing what to build`:

Choosing strategy: `Gerrit Trigger`


#### 3. Build Triggers

* Check on `Gerrit event`
   
* Gerrit Trigger
	* Choose your gerrit server added above: `XXGerrit`

#### 4. Build Environment

We use java property files to configure Jenkins build environment with injecting its content as variables during build process.

[ci_env.properties](ci_env.properties)
```bash
APP_DOWNLOAD_URL = https://fir.im/your-app-path
FIR_IM_SHORT = your-app-short-name-in-fir.im
FIR_IM_TOKEN = your fir.im token
TC_TARGET_NAME = Xcode target name to build 
TC_PROJECT_DIR = Xcode project file root directory
INFO_PLIST_PATH = Targe Info.plist path

# these environment variabls will be injected by Jenkins during buiding process.

FIR_IM_UPLOAD = CHANGE_NOTE=`printf "${PROJECT_DISPLAY_NAME}\\\\nV${IOS_BUILD_VERSION} - Production Environment:\\\\n${GIT_CHANGES_LOG_FIX}"`; \
fir p "${WORKSPACE}/output/release/${IOS_BUILD_VERSION}.ipa" -T "$FIR_IM_TOKEN" -c "$CHANGE_NOTE" -s "$FIR_IM_SHORT"; \
# \
# CHANGE_NOTE=`printf "${PROJECT_DISPLAY_NAME}\\nV${IOS_BUILD_VERSION} - Test Environment:\\\\n${GIT_CHANGES_LOG_FIX}"`; \
# fir p "${WORKSPACE}/output/debug/${IOS_BUILD_VERSION}.ipa" -T "$FIR_IM_TOKEN" -c "$CHANGE_NOTE" -s "$FIR_IM_SHORT"

# email list of tester who will receive emails: 1@mail.com, cc:2.com,
TESTER_EMAIL_LIST =

# email address for tester to reply to
TESTER_REPLY_EMAIL_LIST =

TESTER_EMAIL_CONTENT = V${IOS_BUILD_VERSION} Changes：\n \
${GIT_CHANGES_LOG} \

TESTER_EMAIL_THEME_SUCCESS = $PROJECT_DISPLAY_NAME V$IOS_BUILD_VERSION delivery for test

TESTER_EMAIL_CONTENT_SUCCESS = $PROJECT_DISPLAY_NAME V$IOS_BUILD_VERSION delivery for test \n \
\n \
${TESTER_EMAIL_CONTENT}\n \
\n \
APP dwonload:\n \
${APP_DOWNLOAD_URL}\n \
\n
```

**0x1. Inject ci_env.properties**: 

Go to `Build Environment` > Check on `Inject environment variables to the build process`

Properties File Path:

```bash
${WORKSPACE}/ci_env.properties
```

**0x2. Set a human readable build name: Go to `Build`**: 

Go to `Add build step` > `Execute shell`

```bash
rm -f env_project_build_version.properties
VERSION=`/usr/libexec/PlistBuddy -c "print CFBundleVersion" "${WORKSPACE}/${INFO_PLIST_PATH}"`
echo "IOS_BUILD_VERSION:$VERSION\nBUILD_NUMBER:$VERSION" > env_project_build_version.properties

```

Go to `Add build step` > `Update build name` > `Use macro` > Build name macro template

```bash
${PROPFILE,file="env_project_build_version.properties",property="IOS_BUILD_VERSION"}
```

**0x3. Inject project build variables for `Post-build Actions`**

Go to `Add build step` > `Inject environment variables`:

Properties File Path:

```bash
${WORKSPACE}/env_project_build_version.properties
```

#### 5. Post-build Actions

We are going to inject some environment variables for next step using.

`TC_GIT_CHANGES_LOG`, `GIT_CHANGES_LOG_FIX`

Goto `Add post-build action` > `Execute a set of scripts` > `Build steps`

`Add build step` > `Execute shell` > Command:

```bash
# https://cloud.tencent.com/developer/ask/137080

rm -f env_project_change_log.properties
TC_GIT_CHANGES_LOG=`git log -1 --max-count=1 --pretty=format:"%s%n%b" |sed -e ':a' -e 'N' -e '$!ba' -e 's/\n/\\\\\\\\n/g'`
if [ -z "$TC_GIT_CHANGES_LOG" ]; then TC_GIT_CHANGES_LOG=`git log -1 --max-count=1 --pretty=format:"%s%n%b"`; fi
if [ -z "$TC_GIT_CHANGES_LOG" ]; then TC_GIT_CHANGES_LOG="no changes"; fi
GIT_CHANGES_LOG_FIX=`git log -1 --max-count=1 --pretty=format:"%s%n%b" |sed -e ':a' -e 'N' -e '$!ba' -e 's/\n/\\\\\\\\\\\\\\\\n/g'`
if [ -z "$GIT_CHANGES_LOG_FIX" ]; then GIT_CHANGES_LOG_FIX=`git log -1 --max-count=1 --pretty=format:"%s%n%b"`; fi
if [ -z "$GIT_CHANGES_LOG_FIX" ]; then GIT_CHANGES_LOG_FIX="no changes"; fi

printf "GIT_CHANGES_LOG:$TC_GIT_CHANGES_LOG\nGIT_CHANGES_LOG_FIX:$GIT_CHANGES_LOG_FIX" > env_project_change_log.properties
```

`Add build step` > `Inject environment variables` > Properties File Path:

```bash
env_project_change_log.properties
```


## Code review CI job

### Xcode: `Build` > `Add build step` > `Xcode`

#### General build settings
  
Target:

```bash
${TC_TARGET_NAME}
```

Settings > Configuration: 

```bash
Publish
```

#### Advanced Xcode build options

SDK:

```bash
iphonesimulator
```

Xcode Project Directory:

```bash
${TC_PROJECT_DIR}
```

Xcode Project File:

```bash
${TC_PROJECT_FILE}
```


### Email

Goto `Post-build Actions` > `Add post-build action` > `Editable Email Notification Templates`

Select your code review templates here.


## Distribute ipa files with fir.im

Install [fir-cli](https://github.com/dake/fir-cli) command line tool

```bash
gem install fir-cli-dake
```

Upload ipa file to fir.im with fir-cli

```bash
fir p "${WORKSPACE}/output/release/${IOS_BUILD_VERSION}.ipa" -T "$FIR_IM_TOKEN" -c "$CHANGE_NOTE" -s "$FIR_IM_SHORT"
```


## CI build CD job

### Gerrit Trigger
	
* Add `Trigger on`: `Change Merged`


### Build Environment

#### Check on `Enable Commit Message Trigger`

Keyword:
```bash
build
```

#### Check on `Keychains and Code Signing Identities` > Add Keychain

* Select a keychain you uploaded in `Keychains and Provisioning Profiles Management`
   
* Select `Code Signing Identity` for develop environment

* Enter a `Variables Prefix` for develop environment, e.g. "DEV", "ADHOC"

then you get these Variables for next step using:

| Environment | Variables |
|:-------------|:-------------|
| develop | ${DEV_KEYCHAIN_PATH} ${DEV_KEYCHAIN_PASSWORD} ${DEV_CODE_SIGNING_IDENTITY} |
| adhoc | ${ADHOC_KEYCHAIN_PATH} ${ADHOC_KEYCHAIN_PASSWORD} ${ADHOC_CODE_SIGNING_IDENTITY} |


#### Check on `Mobile Provisioning Profiles` > Add Provisioning Profile

Add provisioning profiles for app with different environment (develop, adhoc, distribution).

*Upload your provisioning profiles in `Keychains and Provisioning Profiles Management`.*


### Build > Xcode

#### General build settings
  
Target:

```bash
${TC_TARGET_NAME}
```

Settings > Configuration: 

```bash
Release_CI
```

Check on `Pack application and build .ipa?`

.ipa filename pattern
```bash
${VERSION}
```

Output directory
```bash
${WORKSPACE}/output/release/
```

#### Code signing settings


| Environment | Code Signing Identity |
|:-------------|:-------------|
| develop | ${DEV_CODE_SIGNING_IDENTITY} |
| adhoc | ${ADHOC_CODE_SIGNING_IDENTITY} |


Check on `Sign IPA at build time`

Check on `Unlock Keychain`

| Environment | Keychain path | Keychain password |
|:-------------|:-------------|:-------------|
| develop | ${DEV_KEYCHAIN_PATH} | ${DEV_KEYCHAIN_PASSWORD} |
| adhoc | ${ADHOC_KEYCHAIN_PATH} | ${ADHOC_KEYCHAIN_PASSWORD} |


#### Advanced Xcode build options
		
Xcode Schema File: name of scheme used of your project

```bash
${TC_SCHEME_NAME}
```

SDK:

```bash
iphoneos
```

Custom xcodebuild arguments:

```bash
APP_PROFILE=${ADHOC_PROVISIONING_PROFILE}
TODAY_PROFILE=${TODAY_ADHOC_PROVISIONING_PROFILE}
```

*`APP_PROFILE` and `TODAY_PROFILE` specified in `Provisioning Profile` of your Xcode project settings.*


Xcode Project Directory:

```bash
${TC_PROJECT_DIR}
```

Xcode Project File:

```bash
${TC_PROJECT_FILE}
```


### Post-build Actions

Check on `Execute a set of scripts` > `Execute script only if build succeeds`

then
`Add build step` > `Execute shell`:

`FIR_IM_UPLOAD` was defined in [ci_env.properties](ci_env.properties)

```bash
eval $FIR_IM_UPLOAD
```


### Email

Goto `Add post-build action` > `Editable Email Notification Templates`

Select your ci build templates here.


## Daily build CI job


## Reference
