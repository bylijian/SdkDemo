# SdkDemo
Android 组件化 学习 Demo
#参考资料：

#一、Android 组件化 基本 项目结构
以前的 Android 应用，复杂一点的多采用 单Application +多Module 处理
这种方法现在已经稍显落后
痛点：
* 每一个编译都要build 所有的依赖Module，非常缓慢
* Module无法单独编译运行，以至于Module无法单独开发，测试，牵一发动全身
* 所有的 业务 都集中在一个Application，如果像美团，既有外卖，又有机票，酒店，每个项目同时也有自己的 App，这种 单 Application 无法解决

所有我们需要组件化开发模式

这个模式

主Application + 多个Module Application +Base Lib Module

这个模式下

每一个 Module Application 都是 既可以是一个 Module，集合进 主Application。
同时，也可以是一个 独立的 Application 可以单独编译运行，
这样带来了巨大的好处，每一个Module Application 都能独立运行，如果只修改一个，只编译执行它非常快速。
最重要的是，开发 Module Application 对主Application影响减低到最低，不需要修改主Application的代码。每一次只需要测试Module Application就可以了。

Base LibModule 
这个是基础库，类似 Log 工具类，等等可以放在这里


#二、具体实现
##2.1 控制 Module 编译为 Application 还是 Library
组件化，最重要的一点是 组件 Module既可以是一个单独的 Application 也可以是 Library
Android Studio 中，区分 一个Module 是Application 还是 Library 

### Applicaton Module  下面的 build.gradle
```
apply plugin: 'com.android.application'
```
### Library Module  下面的 build.gradle
```
apply plugin: 'com.android.application'
```

我们需要一个地方，有一个标记，通过修改这个标记，判断到底是编译为 Application 还是Library
我的做法是：
在项目根目录下，有一个 gradle.properties 

增加一个 标记
```
isSdkAsLib=false
```

### 组件Module下的 build.gradle
看注释，需要在两个地方判断 注意 isSdkAsLib 是一个String类型，所以需要
```
//通过 isSdkAsLib 标记 编译为 library 还是 application
if (isSdkAsLib.toBoolean()) {
    apply plugin: 'com.android.library'
} else {
    apply plugin: 'com.android.application'
}

android {
    compileSdkVersion 26
    buildToolsVersion "26.0.2"

    defaultConfig {
        //注意只有 Application 模式才能配置 applicationId
        if(!isSdkAsLib.toBoolean()){
            applicationId "com.lijian.testsdk"
        }
        minSdkVersion 14
        targetSdkVersion 26
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"

    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    compile 'com.android.support:appcompat-v7:26.+'
    compile 'com.android.support.constraint:constraint-layout:1.0.2'
    testCompile 'junit:junit:4.12'
}

```

###处理组件 Module 的 AndroidManifest 冲突问题

很简单，判断编译模式，在Module下面的 build.gradle 
```
sourceSets {
    main {
        if (isSdkAsLib.toBoolean()) {
            manifest.srcFile 'src/main/release/AndroidManifest.xml'
        } else {
            manifest.srcFile 'src/main/debug/AndroidManifest.xml'
        }
    }
}
```
###处理 资源 res 命名冲突问题
其实很简单，就是在组件Module的build.gradle 

```
  resourcePrefix "sdk_"
```
### 处理主工程编译依赖问题
暂时没实现

### 处理Module 之间 跳转问题
因为组件的Module 可以随时增减，所以Module之间的Activity ，各种的跳转是不可以使用直接依赖具体实现的，
这样会带来强耦合。
为了解决这个问题，
一个可以使用系统的
隐式意图，或者自定义 Scheme

也可以使用第三方开源的 库

比如 Alibaba的  ARouter

还有就是 ActivityRouter

这些库应该都是通过注解实现解耦的。





 
