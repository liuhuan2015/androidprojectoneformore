>学习目标文章[一个项目如何编译多个不同签名、包名、资源等，的apk？](https://mp.weixin.qq.com/s/OQtAVhQVPNVxo9zJc3NG9w)

#### 一、前言
如文章题目所示，本片文章就是为了解决这种问题。方便打包和运行的时候能够做到无需手动替换配置，即可打包想要的apk。

>需求如下：

1. 同一个项目
2. 不同的apk图标
3. 不同的服务器域名
4. 不同的包名
5. 不同的名称
6. 不同的签名
7. 不同的第三方key
8. 不同的版本号版本名

**解决思路：**

1. 最直接的方式是在每次打不同包的时候去替换对应的配置，比较麻烦。
2. 将所有配置、资源文件等都配置入项目中，打包的时候，根据选择渠道打包不同配置的apk（本篇文章讲述）
3. 相信还有其它的

#### 二、相关的几个要点
1. 使用productFlavors来配置渠道。
```java
    android{
        ...
        productFlavors {
            //测试
            sit {

            }
            //灰度
            prodark {

            }
            //生产
            product {

            }
        }
    }
```
注意：在defaultConfig里面需要配置一个  flavorDimensions "versionCode" 属性，否则 Sync 会出错。

2. 当选择了某一个渠道后，运行打包时会根据渠道名选择资源文件。可以在这里做不同渠道下资源文件的替换。
![根据渠道名选择资源文件](https://github.com/liuhuan2015/androidprojectoneformore/blob/master/images/2-2-%E6%A0%B9%E6%8D%AE%E6%B8%A0%E9%81%93%E5%90%8D%E9%80%89%E6%8B%A9%E8%B5%84%E6%BA%90%E6%96%87%E4%BB%B6.png)

3. 签名可在signingConfigs中配置多个（我将所有的签名文件放在了项目根目录的key文件夹中），
可通过signingConfigs指定预先设置好的签名配置
```java
    android{
        ...
         //签名配置
            signingConfigs {

                sitRelease {
                    storeFile file("../key/sit-key-20190127.jks")
                    storePassword "lh123456"
                    keyAlias "sit-key"
                    keyPassword "lh123456"
                }

                ...
            }

            productFlavors {
                //测试
                sit {
                    //指定签名配置
                    signingConfig signingConfigs.sitRelease
                }
                //灰度
                prodark {
                    //指定签名配置
                    signingConfig signingConfigs.prodarkRelease

                }
                //生产
                product {
                    //指定签名配置
                    signingConfig signingConfigs.productRelease

                }
            }

    }
```

4. 可在build.gradle中动态配置java代码调用的常量数据（如：通过该方式我们可根据不同渠道动态配置第三方appid，
或其它需要根据渠道而改变的数据）
```java
    defaultConfig {
        ...

        buildConfigField "String", "SERVER_URL", '"http://xx.xxx.com/"'
    }
```
代码中调用：
```java
    BuildConfig.SERVER_URL
```

查看BuildConfig源码，可以看到里面还包含当前的包名、版本号等信息
```java
public final class BuildConfig {
  public static final boolean DEBUG = Boolean.parseBoolean("true");
  public static final String APPLICATION_ID = "com.liuh.android_project_one_for_more";
  public static final String BUILD_TYPE = "debug";
  public static final String FLAVOR = "prodark";
  public static final int VERSION_CODE = 1;
  public static final String VERSION_NAME = "1.0";
  // Fields from default config.
  public static final String SERVER_URL = "http://xx.xxx.com/";
}
```
5. 在渠道配置那里可以配置对应的包名、版本名、签名等
```java
    productFlavors {
        //测试
        sit {
            applicationId "com.liuh.sit"
            //指定签名配置
            signingConfig signingConfigs.sitRelease
        }
        //灰度
        prodark {
            applicationId "com.liuh.prodark"
            versionCode 2
            versionName "2.0"
            //指定签名配置
            signingConfig signingConfigs.prodarkRelease
        }
        //生产
        product {
            applicationId "com.liuh.product"
            //指定签名配置
            signingConfig signingConfigs.productRelease

        }
    }
```

6. 打正式包的时候选好渠道，就可以打包不同配置的apk，当然也可以使用命令的方式

7. 其它配置

获取当前时间
```java
    static def releaseTime() {
        return new Date().format("yyyyMMddHHmm", TimeZone.getTimeZone("GMT+8"))
    }
```

打包的时候，修改文件名，方便区分渠道号和版本打包时间
```java
    applicationVariants.all {
        variant ->
            variant.outputs.all {
                outputFileName="${variant.productFlavors[0].name}-v${variant.productFlavors[0].versionName}-${releaseTime()}.apk"
            }
    }
```

如果在清单文件中，有那种以包名开头命名的那种，如果包名改变了，有些也是需要动态改变的，可以用${applicationId}代替。
在打包的时候，会自动替换成当前包名。

另外，在代码中我们也不能把包名写死，可以通过BuildConfig来得到当前包名。

>上面是gradle的一些关于多版本打包的简单的配置，作者在原文中提供了他项目的完整配置，也可以作为参考。感觉文章还是挺不错的。











