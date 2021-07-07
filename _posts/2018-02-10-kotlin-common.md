---
layout:     post
title:      "kotlin-common"
subtitle:   " \"kotlin快速开发集成框架\""
date:       2018-02-10 12:01:25
author:     "zhouhaoo"
header-img: "img/banner/kotlin.jpg"
catalog: true
tags:
    - Android
    - Kotlin
---

[kotlin-common](https://github.com/zhouhaoo/kotlin-common)
本项目是熟悉kotlin的语法和编程方式，另外做一个common包，已备后续使用,已上传至` Jcenter `仓库。
<br/>此外，正准备开发[**Gank app**](https://github.com/zhouhaoo/Gank)，使用此[kotlin-common](https://github.com/zhouhaoo/kotlin-common#%E4%BE%9D%E8%B5%96)依赖。
## 依赖
1. 在项目的 `build.gradle` 中添加：

```
apply from: "dependencies.gradle"//将dependencies.gradle复制到根目录，并引入gradle文件
buildscript {
    ext.kotlin_version = '1.2.21'
    repositories {
        google()
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.0.1'
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
    }
}

allprojects {
    repositories {
        google()
        jcenter()
        mavenCentral()
        maven { url "https://jitpack.io" }
    }

task clean(type: Delete) {
    delete rootProject.buildDir
}  
```

2. app下`build.gradle`添加依赖

```
apply plugin: 'com.android.application'
apply plugin: 'kotlin-android'
apply plugin: 'kotlin-android-extensions'
apply plugin: 'kotlin-kapt'//注解编译器
...
dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jre7:$kotlin_version"
    implementation "com.zhouhaoo:kotlin-common:1.1.2"
    kapt annotationProcessorLibs //依赖注解
}
```

## 用法

1. 实现AppConfig类，为common配置依赖信息

```
class AppConfigImpl : AppConfig {
    override fun applyOptions(context: Context, module: ConfigModule) {
        module.apply {
            baseUrl = "http://gank.io/api/"
            gsonBuilder = { }//gson解析配置
            retrofitBuilder = { }//retroifit配置
            okhttpBuilder = { }//okhttp配置
            httplogBuilder = {//网络日志log打印配置
                loggable(BuildConfig.DEBUG).setLevel(Level.BODY).log(Platform.INFO)
                        .request("Request").response("Response")
            }
            globalHttpHandler = GlobalHttpHandlerImpl(context)//全局请求响应配置
//            addInterceptor()//添加okhttp拦截器
        }
    }

    override fun injectAppLifecycle(context: Context, appLifecycles: ArrayList<AppLifecycle>) {
        appLifecycles.add(AppLifecycleImpl())//注入application生命周期
    }

    override fun injectActivityLifecycle(context: Context, actLifecycles: ArrayList<Application.ActivityLifecycleCallbacks>) {
        actLifecycles.add(ActivityLifecycle())
    }

    override fun injectFragmentLifecycle(context: Context, fragLifecycles: ArrayList<FragmentManager.FragmentLifecycleCallbacks>) {

    }
}
```
2. 在app模块中，新建AppComponent

```
@AppScope
@Component( dependencies = [(CoreComponent::class)])
interface AppComponent {
    fun inject(baseApp: BaseApp)
}
```

3. 在app模块中，新建BaseApp继承BaseApplicationApp

```
class BaseApp : BaseApplication() {
    override fun onCreate() {
        super.onCreate()
        var appComponent = DaggerAppComponent
                .builder()
                .coreComponent(getCoreComponent())
                .build()
        appComponent.inject(this)
    }
}
```
4. 配置`AndroidManifest.xml ` 更改Application为app中的BaseApp，`application`节点下加入`meta-data`,加入第一步的AppConfigImpl路径，这样框架才能解析到配置（类似glide配置）。

```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          package="com.zhouhaoo.sample">

    <uses-permission android:name="android.permission.INTERNET"/>
    <application
         android:name=".base.BaseApp"//步骤3新建的BaseApp
       ...
          <meta-data
            android:name="com.zhouhaoo.sample.app.AppConfigImpl"
            android:value="CommonConfig"/>
    </application>

</manifest>
```

4. 同步`application`的生命周期,进行需要加载的框架的初始化，例如日志打印控制

```
class AppLifecycleImpl : AppLifecycle {
    override fun attachBaseContext(context: Context) {
        if (BuildConfig.DEBUG) {
            Timber.plant(Timber.DebugTree())
        }
    }
    override fun onCreate(application: Application) {
    }
    override fun onTerminate(application: Application) {

    }
}
```


>
详情见sample代码

## 感谢开源

1. [`MVPArms`](https://github.com/JessYanCoding/MVPArms)
2. [`Retrofit`](http://square.github.io/retrofit/)
3. [`Okhttp`](http://square.github.io/okhttp/)
4. [`Glide`](https://github.com/bumptech/glide)
5. [`Gson`](https://github.com/google/gson)
6. [`Timber`](https://github.com/JakeWharton/timber)
7. [`dagger2`](https://github.com/google/dagger)
8. [`rxjava2`](https://github.com/ReactiveX/RxJava)
9. [`rxandroid`](https://github.com/ReactiveX/RxAndroid)
10. [`rxkotlin`](https://github.com/ReactiveX/RxKotlin)
11. [`rxlifecycle`](https://github.com/trello/RxLifecycle) 
12. [`rxpermissions`](https://github.com/tbruyelle/RxPermissions) 
13. [`LoggingInterceptor`](https://github.com/ihsanbal/LoggingInterceptor)

...

