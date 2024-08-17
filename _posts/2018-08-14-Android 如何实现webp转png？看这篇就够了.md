---
title: 'Android 如何实现webp转png？看这篇就够了'
layout: post

categories: post
tags:
- Android
---

> Webp是什么，请移步[What‘s Webp?](https://developers.google.com/speed/webp/)

由于webp与生俱有优点，webp如今使用已经很广泛了，包括客户端。比如今日头条客户端页面的图片都是webp格式的。客户端碰到与webp有关的一些问题，大概有:
* webp图片的显示
* webp转成其他格式图片
* 其他格式图片转成webp
比如我们有一个这样的需求：
> 客户端将webp格式的图片转成png后上传到服务端(由于服务端暂时还不支持webp图片的上传)
如何将webp转成png？我们需要依赖Google的webp源码的支持，但是webp的源码都是用C来实现的，需要将它编译成.so库，同时还需要提供几个供Java层调用的API，用于实现webp与png相互转换。在这里我们参考开源项目[webp-android-backport](https://github.com/alexey-pelykh/webp-android-backport)

[![webp-android-backport](/static/post-image/webp1.png)](/static/post-image/webp1.png)

下面详细说说如何编译出我们想要的webp.so库
## 1. 将[webp-android-backport](https://github.com/alexey-pelykh/webp-android-backport)项目源码下载至本地后用AS打开
## 2.下载webp源码
因为这个开源项目对于webp的源码依赖方式是通过git submodule来管理的，如果没有Google 开发者账号不方便下载。
还可以通过下面的链接获取源码。

* [libwebp](https://github.com/webmproject/libwebp/)
* [Googleapis](https://storage.googleapis.com/downloads.webmproject.org/releases/webp/index.html)
[![webp-android-backport](/static/post-image/webp2.png)](/static/post-image/webp2.png)
[![webp-android-backport](/static/post-image/webp3.png)](/static/post-image/webp3.png)

## 3. 编译
下载源码后放置到工程目录/jni，进行编译，如图

[![webp-android-backport](/static/post-image/webp4.png)](/static/post-image/webp4.png)

> 认真看了开源项目源码后发现，这样编译出来的aar库其实是有bug的,尴尬😓

如图：

[![webp-android-backport](/static/post-image/webp5.png)](/static/post-image/webp5.png)

为了解决该bug，我们需要将接口类中私有API定义成public，方便上层使用。这里的修改涉及到jni的编译了，期间我也碰到了一些比较坑的问题。(这里我会用另一篇博客来详细说明)
在编译so库的时候，可以修改webp-android-backport-library包下面的Application.mk文件中APP_ABI 参数，来指定项目需要支持的cpu架构平台。

```java
#APP_ABI := armeabi-v7a x86 armeabi
#APP_ABI := armeabi
APP_ABI := armeabi-v7a
APP_CPPFLAGS := -fno-rtti -fno-exceptions
APP_PLATFORM := android-8

ifndef WEBP_BACKPORT_DEBUG_NATIVE
# Force release compilation in release optimizations, even if application is debuggable by manifest
APP_OPTIM := release
endif
```

编译完成会在webp-android-backport-library/build/outputs目录下生成一个aar文件，这样我们就完成了webp库的编译了。
> 如何使用编译生成的库来实现webp转png呢？

直接看代码吧:
```java
/**
 * author : J.Chou
 * e-mail : who_know_me@163.com
 * time   : 2018/08/13 10:53 PM
 * version: 1.0
 * description:
 */
public class Webp2PngUtil {

  public static boolean isWebpImage(File imageFile) {
    BitmapFactory.Options options = new BitmapFactory.Options();
    options.inJustDecodeBounds = true;
    BitmapFactory.decodeFile(imageFile.getAbsolutePath(), options);
    String type = options.outMimeType;
    return type.equals("image/webp");
  }

  public static Bitmap webp2Bitmap(String path) {
    Bitmap bitmap = null;
    try {
      bitmap = WebPFactory.nativeDecodeFile(path, null);
    } catch (Throwable t) {
      t.printStackTrace();
    }
    return bitmap;
  }

  @SuppressWarnings("ResultOfMethodCallIgnored")
  public static String bitmap2Png(Bitmap bitmap) {
    if(bitmap == null) return null;
    String pngImagePath = null;
    File pngFile = new File(FileUtil.getCacheDir() + File.separator + "webp2png" + System.currentTimeMillis() + ".png");
    if(pngFile.exists()){
      pngFile.delete();
    }
    FileOutputStream out;
    try{
      out = new FileOutputStream(pngFile);
      if(bitmap.compress(Bitmap.CompressFormat.PNG, 90, out)) {
        out.flush();
        out.close();
      }
      pngImagePath = pngFile.getAbsolutePath();
    } catch (FileNotFoundException e) {
      e.printStackTrace();
    }catch (IOException e) {
      e.printStackTrace();
    }
    return pngImagePath;
  }
}
```
这个工具类使用到aar库中的接口类WebPFactory.
这里简单说一下上传图片的逻辑
先判断上传文件(webpFile)是否是webp格式(不能判断图片后缀名)，如果是，则将该文件转成png图片文件(webpFile ---> pngFile)，然后将pngFile上传到服务端。

使用工具类Webp2PngUtil来实现webp转png的实例代码

```java
if (Webp2PngUtil.isWebpImage(new File(originalImagePath))) {
  String pngImagePath = Webp2PngUtil.bitmap2Png(Webp2PngUtil.webp2Bitmap(originalImagePath));
  if (!TextUtils.isEmpty(pngImagePath)) {
    originalImagePath = pngImagePath;
  }
}
```

实现这样的需求还是比较容易，主要的精力可以会消耗在aar库的编译上。
由于aar中包含了webp的so库，而且这个so库其实不小(随着webp的源码越来越多，编译后的so库也在不断的增大)，考虑到APP包的size，支持的cpu架构也不能太多。

#### **极好的参考资料**
##### 1. [http://www.geekince.com/android/2017/07/31/android-webp-build/](http://www.geekince.com/android/2017/07/31/android-webp-build/)

##### 2. [http://www.hahack.com/wiki/sundries-webp.html](http://www.hahack.com/wiki/sundries-webp.html)


