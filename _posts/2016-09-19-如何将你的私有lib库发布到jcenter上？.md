---
title: "如何将你的私有lib库发布到jcenter上？"
layout: post
category: post
tags: 
---

众所周知，将lib发布到jcenter的途径五花八门，八仙过海各显神通，各有各的插件，自己喜欢就好。我也在众多的方法中迷失过，有复杂也有简单，遇到的问题也奇奇怪怪，人都有选择综合征，一旦选择多了就无从下手。还好我找到了一种相对简单的上传方法。

首先你需要在[https://bintray.com/](https://bintray.com/)上注册一个账号，这是必须的一步，后面在上传的时候会用到账号的一些信息。有了账号之后我们就可以开始在Android Studio中对build.gradle文件进行配置，以实现将我们的lib库上传到jcenter中。下面是build.gradle文件的配置步骤。

### 1.在项目根目录下面的build.gradle中添加插件的依赖路径，以及通过ext配置私有lib信息，包括库的包名，库名，版本号，库的源代码地址等等...

插件路径依赖

```java
dependencies {
    classpath 'com.novoda:bintray-release:0.3.4'
}
```

库信息的描述:

```java
def libVersion = "1.0.8"
ext {
    userOrg = "iknow"          //bintray.com用户名
    groupId = "com.github.iknow4"   //jcenter上的路径
    publishVersion = libVersion //版本号
    description = "It is a android utils Library"//类库的描述
    website = "https://github.com/iknow4/Android-utils"//该库在github上对应的链接
    uploadName = "AndroidUtils" //上传在bintray的文件夹
    licences = ["Apache-2.0"]
}
```
完整的code如下：

```java
buildscript {
    repositories {
       jcenter()
 }
dependencies {
      classpath 'com.android.tools.build:gradle:2.0.0'
      classpath 'com.novoda:bintray-release:0.3.4'
  }
}
def libVersion = "1.0.8"
ext {    userOrg = "iknow"          //bintray.com用户名  
      groupId = "com.github.iknow4"   //jcenter上的路径
      publishVersion = libVersion //版本号
      description = "It is a android utils Library"//类库的描述
      website = "https://github.com/iknow4/Android-utils"//该库在github上对应的链接
      uploadName = "AndroidUtils" //上传在bintray的文件夹
      licences = ["Apache-2.0"]
}
allprojects {
    repositories {
         jcenter()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```

### 2.在主工程目录下面的build.gradle文件中，将插件'com.novoda.bintray-release'，apply到build.gradle中。

添加插件依赖：

```java
apply plugin: 'com.novoda.bintray-release' //添加插件依赖
```

增加发布模块：

```java

//添加发布模块
publish {
        artifactId = 'android-utils-sdk'//模块名称
        userOrg = rootProject.userOrg
        groupId =   rootProject.groupId
        uploadName = rootProject.uploadName //模块上传后所在的文件夹名称
        publishVersion = rootProject.publishVersion//模块版本号
        desc = rootProject.description//模块的描述
        website = rootProject.website //模块的网站
        licences = rootProject.licences //模块的licences
}
```

完整的代码如下:

```java
apply plugin: 'com.android.library'
apply plugin: 'com.novoda.bintray-release' //添加插件依赖
android {
  compileSdkVersion 23
  buildToolsVersion "24.0.0"

  defaultConfig {
      minSdkVersion 14
      targetSdkVersion 23
      versionCode 1
      versionName "1.0"
  }
  buildTypes {
        release {
              minifyEnabled false
              proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
      }
 }
lintOptions {
        abortOnError false
   }
}
dependencies {
      compile fileTree(dir: 'libs', include: ['*.jar'])
}
//添加发布模块
publish {
        artifactId = 'android-utils-sdk'//模块名称
        userOrg = rootProject.userOrg
        groupId =   rootProject.groupId
        uploadName = rootProject.uploadName //模块上传后所在的文件夹名称
        publishVersion = rootProject.publishVersion//模块版本号
        desc = rootProject.description//模块的描述
        website = rootProject.website //模块的网站
        licences = rootProject.licences //模块的licences
}
```

3.在终端执行./gradlew clean build bintrayUpload -PbintrayUser=xxx -PbintrayKey=xxx -PdryRun=false命令
其中PbintrayUser是你在[https://bintray.com/](https://bintray.com/)注册的用户名，PbintrayKey是账户设置页面下的key。
如果命令执行成功，你的库就上传到bintray 网站上了，但是还无法被依赖使用，需要将库发布到jcenter 上，发布有时候需要等待一天时间，如果成功，你会收到发布成功的邮件。这时候，恭喜你，你和其他人就可以在Android Studio中依赖使用了，是不是觉得很不错。
以上是我选择发布库的一种方式，自己觉得还是蛮简单的，从配置到最后发布命令，只需要三步，简称:发布三部曲。需要注意的是，发布是不能将相同版本的库覆盖的，所以每次发布的版本号要求不一样。

可以参考我github上的一个项目[Android-utils](https://link.jianshu.com/?t=https://github.com/iknow4/Android-utils)
该项目是一个开发工具库，如果想使用可以直接在build.gradle中进行依赖

```java
dependencies {
      compile 'com.github.iknow4:android-utils-sdk:1.0.8'
}
```