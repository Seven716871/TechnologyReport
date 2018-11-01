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

#### 1. Gerrit Trigger > Add New Server > Named like `XXGerrit`

 Fillup the basic information and `SSH Keyfile`.


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

* Check on `Recursively update submodules` in `Additional Behaviours` if you used `git submodule`
  
* Add `Additional Behaviours` in `Source Code`: `Gerrit Trigger`

#### 3. Build Triggers

* Check on `Gerrit event`
   
* Gerrit Trigger
	* Choose your gerrit server added above: `XXGerrit`

#### 4. Build Environment

We use java property files to configure Jenkins build environment with injecting its content as variables during build process.

The sample ci environment configuration file is [here](ci_env.properties).

**0x1. Inject ci_env.properties**: 

Go to `Build Environment` > Check on `Inject environment variables to the build process` > Set `Properties File Path` with `${WORKSPACE}/ci_env.properties`.

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

Goto `Add post-build action` > `Editable Email Notification Templates`

Choose your code review templates here.



## Daily build CI job


## CI build CD job

* Gerrit Trigger
	* Add `Trigger on`: `Change Merged`


## Reference