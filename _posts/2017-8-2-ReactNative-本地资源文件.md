---
layout: post
title: ReactNative 本地资源文件
category: ReactNative
tags: [ReactNative]
---



## iOS资源文件

**RN项目目录结构图**

![RN项目目录结构图](https://ws1.sinaimg.cn/large/962a6dfegy1fi5dzr5lvlj20b40bjq50.jpg)

**iOS打包后app内部目录结构图**

![iOS打包后app内部目录结构图](https://ws1.sinaimg.cn/large/962a6dfegy1fi5dzrbvlmj20b407rwgw.jpg)

从上两图可以看出来，iOS打包后，会把RN项目中(通过`require`)引用的资源文件(图片，html等文件)放到**assets**文件夹中。**assets**中的资源路径和**RN项目**中的是一样的。可以简单理解为`ui-mobile`改名为`assets`内部文件路径一样。

## android资源文件

和iOS不同的是RN打包apk以后，会把资源整理到不同的**drawable**下，而不会放进**assets**文件夹里面。而[drawable](https://developer.android.com/guide/topics/resources/drawable-resource.html?hl=zh-cn#Bitmap)目录下只存放绘制文件(jpg、png、gif、xml等)所以，如果android项目中需要用到html，ttf等资源文件，还是需要手动加进`android/app/src/main/assets`路径下。

## 加载本地html问题

**RN目录结构**

~~~
// []代表文件夹

- [rn_project_name]
    - app.js
    - [htmls]
        - about.html
        - [imgs]
            - bg.png
    - [components]
~~~

**about.html**

~~~
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title></title>
</head>
<body>
<div style="alignment: center">
    <img src="./imgs/bg.png" width="100%" />
</div>
</body>
</html>
~~~

**app.js**

~~~
...
<WebView source={require('./htmls/about.html')}/>
...
~~~

现在我们需要用WebView组件加载并显示本地的about.html。在模拟器上，android和iOS都可以正常显示。**但是，打包成apk和app以后，android会提示找不到about.html文件，而iOS可以打开*about.html*但找不到*bg.png*。其实这和RN打包资源文件方式有关(上面有提到)。**

### iOS解决方法

RN打包iOS的时候，会把(通过`require`)引用的资源文件打包到**assets**目录下，所以WebView可以正确找到*about.html*，而找不到*bg.png*。

**解决步骤：**

1. 在iOS目录下创建**assets**文件夹
2. 按照RN目录结构在**assets**中加入缺失的文件/目录
3. 把**assets**文件拖进Xcode中，并勾选 `Create folder reference`

**iOS目录**

![](https://ws1.sinaimg.cn/mw690/962a6dfegy1fi5ieydk3dj21jf08c0vb.jpg)

**Xcode Project**

![](https://ws1.sinaimg.cn/mw690/962a6dfegy1fi5ieyatkyj20ay08cwg8.jpg)

> **注意：**
> 一定要勾选`Create folder reference`，成功以后在Xcode中显示蓝色的文件夹。
> 
> 在Xcode中`group`是存放编译的文件,如：.m .h。(黄色文件夹)
> 而`folder reference`是不会编译的，会原封不动的打包进app里面。(蓝色文件夹)

### android解决方法

上面已经提到，RN会把引用的文件打包进**drawable**目录下，因为html文件不是可绘制文件，所以会自动被删除。

**解决步骤：**

1. 创建`android/app/src/main/assets`目录
2. 把需要的文件放进**assets**中
3. RN中利用`{ uri: 'file:///android_asset/***' }`访问

**androi 目录**

![](https://ws1.sinaimg.cn/mw690/962a6dfegy1fi5ius8dv7j20bv08c0ty.jpg)

**app.js**

~~~
...
<WebView source={{ uri: 'file:///android_asset/htmls/about.html' }}/>
...
~~~

>**注意：**
>assets文件夹会原封不动的保存在apk包中，不会被编译成二进制。

### 完美兼顾方法

**普通方案**

~~~
let source;
if (__DEV__) {
  source = require('./index.html');
} else {
  source = Platform.OS === 'ios' ? require('./htmls/about.html') : { uri: 'file:///android_asset/htmls/about.html' };
}
~~~

**可传query方案**：path/to/file.html?page=1&page_size=10

~~~
import resolveAssetSource from 'react-native/Libraries/Image/resolveAssetSource';
let source;
const _source = resolveAssetSource(require('./index.html'));
if (__DEV__) {
  source = { uri: `${_source.uri}&id=${article.id}` };
} else {
  const sourceAndroid = { uri: `file:///android_asset/index.html?id=${article.id}` };
  const sourceIOS = { uri: `file://${_source.uri}?id=${article.id}` };
  source = Platform.OS === 'ios' ? sourceIOS : sourceAndroid;
}
~~~

> **引用：**[react native webview加载本地html资源文件的解决方案](https://blog.jeepeng.com/2017/03/16/react-native-webview-load-from-device-local-file-system/)

