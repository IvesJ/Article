> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/L_bgLX8s2XNmTaWDPah5cg)

![](https://mmbiz.qpic.cn/mmbiz_png/IYsHOGmoKN8KNvsh8ovyCaicb7SaHibtcnRoQlUIJWWO8E2PkQ1jzDnqj8Zgia1DBFhkDwAEEqHmqAcs3rKD3fhpg/640?wx_fmt=png&from=appmsg)

大家好，我是牢鹅！最近我在创建新项目的时候，发现新版 Android Studio 的 gradle 文件已经是默认使用 Kotlin 脚本 (KTS) 了。说来惭愧，由于太久没有接触过新项目和新版 AS，竟一时有些手足无措，因为有些配置代码老想着 copy 老项目的快一些，结果竟然一堆红色 error。

后来网上一通查找，完成 Groovy 到 KTS 迁移之余，顺便把差异都记录下来整理成文档，方便下次项目继续 copy。

![](https://mmbiz.qpic.cn/mmbiz_png/IYsHOGmoKN8KNvsh8ovyCaicb7SaHibtcnic2icdcIPrSIPd15xBfbTcTYkAVgjLBNuNWAPiaolDiba2qeMLBJDBnVdw/640?wx_fmt=png&from=appmsg)

KTS 的优势

首先 KTS 是什么？

KTS：是指 Kotlin 脚本，这是 Gradle 在 build 配置文件中使用的一种 Kotlin 语言形式。Kotlin 脚本是可从命令行运行的 Kotlin 代码。

他是 Gradle 官方推荐的构建脚本语言？相较于传统的 Groovy 语法，具有以下优势：

*   类型安全：Kotlin 的静态类型检查能减少构建脚本中的潜在错误。
    
*   IDE 支持：Android Studio 对 KTS 提供更好的语法高亮、自动补全和错误提示。
    
*   代码复用性：支持将通用配置抽取为独立 Kotlin 文件（如 `dependencies.kt` ）。
    
*   性能优化：Gradle 7.0+ 对 KTS 构建缓存机制有专项优化。
    

脚本文件命名

脚本文件扩展名取决于编写 build 文件所用的语言：

*   用 Groovy 编写的 Gradle build 文件使用 `.gradle` 文件扩展名。
    
*   用 Kotlin 编写的 Gradle build 文件使用 `.gradle.kts` 文件扩展名。
    

常见语法差异处理

<table><tbody><tr><td data-colwidth="174"><section>Groovy 语法</section></td><td data-colwidth="228"><section>KTS 语法</section></td><td data-colwidth="196"><section>说明</section></td></tr><tr><td data-colwidth="174"><section>implementation '...'</section></td><td data-colwidth="228"><section>implementation("...")</section></td><td data-colwidth="196"><section>依赖声明必须使用双引号和括号</section></td></tr><tr><td data-colwidth="174"><section>ext.versionCode &nbsp;= 1</section></td><td data-colwidth="228"><section>val versionCode by extra(1)</section></td><td data-colwidth="196"><section>扩展属性需使用委托语法</section></td></tr><tr><td data-colwidth="174"><section>fileTree(dir: 'libs')</section></td><td data-colwidth="228"><section>fileTree("dir" to "libs")</section></td><td data-colwidth="196"><section>Map 参数需显式声明键值对</section></td></tr><tr><td data-colwidth="174"><section>version = "1.0.0"</section></td><td data-colwidth="228"><section>version = "1.0.0"</section></td><td data-colwidth="196"><section>字符串赋值语法保持不变 &nbsp;</section></td></tr></tbody></table>

文件变化具体差异

1. settings.gradle

```
// Groovy 
  pluginManagement {
      repositories {
          gradlePluginPortal()
          google()
          mavenCentral()
          maven { url 'https://maven.aliyun.com/repository/central/' }
          maven { url 'https://maven.aliyun.com/repository/public/' }
          maven { url 'https://maven.aliyun.com/repository/google/' }
          maven { url 'https://maven.aliyun.com/repository/jcenter' }
          maven { url 'https://maven.aliyun.com/repository/releases' }
          maven { url 'https://jitpack.io' }
      }
  }
  include ':app'
  project(':app').projectDir = new File(rootDir, 'app-module')
  // KTS 
  pluginManagement {
      repositories {
          google {
              content {
                  includeGroupByRegex("com\\.android.*")
                  includeGroupByRegex("com\\.google.*")
                  includeGroupByRegex("androidx.*")
              }
          }
          mavenCentral()
          gradlePluginPortal()
          maven(url = "https://maven.aliyun.com/repository/central")
          maven(url = "https://maven.aliyun.com/repository/public")
          maven(url = "https://maven.aliyun.com/repository/jcenter")
          maven(url = "https://maven.aliyun.com/repository/google")
          maven(url = "https://maven.aliyun.com/repository/releases")
          maven(url = "https://jitpack.io")
      }
  }
  include(":app")
  project(":app").projectDir = File(rootDir, "app-module")

```

2. 项目级 build.gradle

```
// Groovy 
  plugins {
      id 'com.android.application' version 'x.x.x' apply false
      id 'com.android.library' version 'x.x.x' apply false
      id 'org.jetbrains.kotlin.android' version '1.9.0' apply false
  }
  // KTS (引用libs.versions.toml的配置文件)
  plugins {
      alias(libs.plugins.android.application) apply false
      alias(libs.plugins.kotlin.android) apply false
  }
  //实际相当于
  plugins {
      id("com.android.application") version "x.x.x" apply false
      id("org.jetbrains.kotlin.android") version "x.x.x" apply false
  }

```

3. 模块级 build.gradle

*   plugins
    

```
// Groovy 
    plugins {
        id 'com.android.application'
        id 'org.jetbrains.kotlin.android'
        id 'kotlin-android'
        id 'kotlin-kapt'
    }
    // KTS 
    plugins {
        alias(libs.plugins.android.application)
        alias(libs.plugins.kotlin.android)
    }

```

*   android
    

```
// Groovy 
    android {
        namespace = "com.xxx"
        compileSdk 35
        defaultConfig {
            applicationId "com.xxx"
            minSdk 22
            targetSdk 35
            versionCode 1
            versionName "1.0.0"
            testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
        }
        signingConfigs {
            release {
                storeFile file('xxx.jks')
                storePassword 'xxx'
                keyPassword 'xxx'
                keyAlias 'xxx'
                v2SigningEnabled true
            }
        }
        buildTypes {
            release {
                minifyEnabled true
                zipAlignEnabled true
                shrinkResources true
                proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
                signingConfig signingConfigs.release
            }
        }
        compileOptions {
            sourceCompatibility JavaVersion.VERSION_1_8
            targetCompatibility JavaVersion.VERSION_1_8
        }
        kotlinOptions {
            jvmTarget = '1.8'
        }
        sourceSets {
            main {
                jniLibs.srcDirs = ['libs']
            }
        }
    }
    // KTS
    android {
        namespace = "com.xxx"
        compileSdk = 35
        defaultConfig {
            applicationId = "com.xxx"
            minSdk = 24
            targetSdk = 35
            versionCode = 1
            versionName = "1.0.0"
            testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
        }
        signingConfigs {
            create("release") {
                keyAlias = "xxx"
                keyPassword = "xxx"
                storeFile = file("xxx.jks")
                storePassword = "xxx"
                enableV2Signing = true
            }
         }
        buildTypes {
            release {
                signingConfig = signingConfigs.getByName("release")
                isMinifyEnabled = true
                proguardFiles(
                    getDefaultProguardFile("proguard-android-optimize.txt"),
                    "proguard-rules.pro"
                )
            }
        }
        compileOptions {
            sourceCompatibility = JavaVersion.VERSION_11
            targetCompatibility = JavaVersion.VERSION_11
        }
        kotlinOptions {
            jvmTarget = "11"
        }
    }

```

4. 其余常见差异

*   动态版本号处理
    

```
// Groovy 
def appcompat_version = "1.6.1"
implementation "androidx.appcompat:appcompat:$appcompat_version" 
// KTS 
val appcompat_version by extra("1.6.1")
implementation("androidx.appcompat:appcompat:$appcompat_version") 

```

*   自定义任务
    

```
// Groovy 
task cleanAll(type: Delete) {
    delete rootProject.buildDir  
}
// KTS 
tasks.register("cleanAll",  Delete::class) {
    delete(rootProject.buildDir) 
}

```

配置修改后验证

1. 命令行验证

```
./gradlew clean assembleDebug --dry-run 

```

2. IDE 验证

*   检查 Android Studio 的 Build Output 窗口是否有语法错误
    
*   验证所有 Build Variants 的编译结果
    

注意事项

*   逐步修改：建议按模块逐个修改，避免全量修改导致构建失败。
    
*   版本控制：使用 Git 等工具做好版本备份，每次修改后立即提交。
    
*   兼容性检查：某些三方插件可能暂不支持 KTS，需关注插件文档
    

结语

以上，就是本次的 Gradle 从 Groovy 到 KTS 变化差异展示。牢鹅还是建议大家新项目尽量使用 KTS，毕竟 Google 官方都在推荐。老项目呢，能迁移到 KTS 也可以系统性地平稳迁移。但是不必太过执着，确保项目稳定性才是最重要的。

上面的文章也只是一少部分的参考示例，如果你有其他推荐或者补充，欢迎在评论区与我们交流。添加牢鹅的微信：kris_wuii，加入我的 GP 出海交流群，一起交流学习。（此群主要面向交流谷歌政策、账号和上架问题的朋友，同时分享行业信息资源）

**公告区🔈**

🔗**[邀你一起共建谷歌封号申诉共享库](http://mp.weixin.qq.com/s?__biz=MzkzMTcwNjI5Nw==&mid=2247485370&idx=1&sn=fbb1729cf136993c758c831ce6b7b682&chksm=c267a36cf5102a7a57c467333193d7dc7df02ebf4956f81d83ff1d536c0b4f85c15903881d79&scene=21#wechat_redirect)**

此库的数据源基于我本人以及 GP 出海交流群的群友的过往封号申诉案例，至此已收录百余例申诉案例，其中最多违规理由为高风险，其次是谷歌 8.3 或 10.3，剩下的封号理由为屡次违规、违反金融条例、恶意软件等。记录了信息包括：应用类型、账号信息、封号原因、处理过程以及最终结果。  

如果你也想将封号申诉的案例同步到此库，或者寻求封号申诉帮助，请私信我。

腾讯在线文档链接如下或点击文末「阅读原文」：  
https://docs.qq.com/sheet/DZldVTnBqeGxFRlBO?tab=BB08J2

GitHub 链接如下：  
https://github.com/AndroidGODev/Bad-Google-Play

**优质合作伙伴，助您出海无忧****👇**

![](https://mmbiz.qpic.cn/mmbiz_jpg/IYsHOGmoKNicAgxRI5sHLlIL06peYxDbPFmIEE83OcBsWqswTAoveHrOd8wg6YULYAcMib6XRyxVZ1RoGVkOZicfQ/640?wx_fmt=jpeg&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=wxpic)

全球收款结汇我选易宝支付，USD、CNY 均可，资金灵活！