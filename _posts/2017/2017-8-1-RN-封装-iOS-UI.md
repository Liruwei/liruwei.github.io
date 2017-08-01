---
layout: post
title: RN 封装 iOS UI
category: RN
tags: [RN,iOS]
---

# RN 封装 iOS UI

最近项目需要实现一个通讯录的功能，一次性显示2000到3000条数据。RN提供的List Component实现起来性能不理想，后来就改用原生实现。为了以后少踩坑，简单写个教程(实现一个CustomView)。效果如下：

![](https://ws1.sinaimg.cn/mw690/962a6dfegy1fi40l3ecexg20al0c6b2a.gif)

> **注意：**本文章是在 react-native@0.44 下编写的，低版本或者高版本的可能不合适。
> [demo下载地址](https://github.com/Liruwei/Demos/tree/master/RN%E5%B0%81%E8%A3%85iOS_UI_Demo)

## 步骤

封装一个iOS的View，步骤十分简单

1. 首先创建一个子类
2. 添加RCT_EXPORT_MODULE()标记宏
3. 实现`- (UIView *)view`方法
4. RN中获取/配置Component

**创建子类**

~~~objectivec
/**CustomViewManager.h*/

#import <React/RCTViewManager.h>
@interface CustomViewManager : RCTViewManager
@end
~~~

**添加RCT_EXPORT_MODULE()标记宏以及实现`- (UIView *)view`方法**

~~~objectivec
/**CustomViewManager.m*/

@implementation CustomViewManager

RCT_EXPORT_MODULE(RCTCustomView)

- (UIView *)view {
  //这里的CustomView是自定义的一个View
  return [[CustomView alloc] init];
}
@end
~~~

**RN中获取/配置Component**

~~~
/**xxx.js*/
import { requireNativeComponent } from 'react-native';
const CustomView =  requireNativeComponent('RCTCustomView', null);

...
render() {
    return (
        <View style={styles.contain}>
            <CustomView style={{flex:1}}/>
        </View>
    )
}
...
~~~

其中`requireNativeComponent(viewName,componentInterface,extraConfig)`有三个参数

* viewName: RCT_EXPORT_MODULE 设置的名称(不会自动删除RCT前缀)
* componentInterface: 配置component，这里可以传两种形式的值，下面会给出
* extraConfig: 这东西我还没用过，自己查看文档吧[[中文](https://reactnative.cn/docs/0.44/native-component-ios.html)][[英文](http://facebook.github.io/react-native/releases/0.44/docs/native-components-ios.html)]

当我们设置了Native UI的[属性](#iOS属性)/[事件](#iOS事件回调)，必须提供componentInterface，不然就会红屏。componentInterface的值有两种形式。

第一种:

~~~
import React, {PropTypes} from 'react';
import {View, requireNativeComponent} from 'react-native';

let iface = {
  name: 'ImageView',
  propTypes: {
    src: PropTypes.string,
    borderRadius: PropTypes.number,
    resizeMode: PropTypes.oneOf(['cover', 'contain', 'stretch']),
    ...View.propTypes // 包含默认的View的属性
  },
};
const ImageView = requireNativeComponent('RCTImageView', iface);
~~~

第二种:

~~~
export class MapView extends Component {
  static propTypes = {
    pitchEnabled: PropTypes.bool,
  };
  render() {
    return <RNTMap {...this.props} />;
  }
}

const RNTMap = requireNativeComponent('RNTMap', MapView);
~~~

> [一般来说我喜欢第二种写法，原因在添加方法的时候会说到。](#设置componentInterface)


## 属性

~~~objectivec
/**CustomView.h*/

@property (nonatomic, copy) NSString *title;

/**CustomViewManager.m*/

RCT_EXPORT_VIEW_PROPERTY(title, NSString)
~~~

~~~
/**xxx.js*/

export class CustomView extends Component {
  static propTypes = {
    title: PropTypes.string,
  };
  render() {
    return <CustomView {...this.props} />;
  }
}

const RCTCustomView = requireNativeComponent('RCTCustomView', RCTCustomView);
~~~

> **注意：** 用RCT_EXPORT_VIEW_PROPERTY标记的属性一定要在componentInterface中的propTypes设置好，不然会红屏

## 事件回调

iOS的事件回调其实和属性一样写法，只不过是一个block类型的属性。

~~~
/**CustomView.h*/
#import <React/RCTComponent.h>

@interface CustomView : UIView
@property (nonatomic, copy) RCTBubblingEventBlock onClickButton;
@end

/**CustomView.m*/

- (IBAction)click:(id)sender {
    if (self.onClickButton) {
        self.onClickButton(@{@"title":@"this is title value"});
    }
}

/**CustomViewManager.m*/

RCT_EXPORT_VIEW_PROPERTY(onClickButton, RCTBubblingEventBlock)
~~~

~~~
/**xxx.js*/

export class CustomView extends Component {
  static propTypes = {
    title: PropTypes.string,
    onClickButton: PropTypes.func,
  };
  render() {
    return <RCTCustomView {...this.props} />;
  }
}

const RCTCustomView = requireNativeComponent('RCTCustomView', CustomView);


...
render() {
    return (
        <CustomView 
            onClickButton={e=>{
                let result = e.nativeEvent;
                console.log(result.title);
            }}
        />
    )
}
...
~~~

> **注意：** 所有回调方法都需要通过 **nativeEvent** 获取返回值

## 方法

**RCTViewManager**实现了**RCTBridgeModule**协议,这说明我们可以通过`NativeModules.moduleName`获取到对应的XXXViewManager。利用宏**RCT_EXPORT_METHOD**导出方法。

导出步骤如下:

1. 重写`-uiBlockToAmendWithShadowView:`方法
2. 获取**viewRegistry**对象 
3. 导出带有 _viewTag_ 参数的方法

~~~
@interface CustomViewManager ()
@property (nonatomic, strong) NSDictionary *viewRegistry;
@end

...

RCT_EXPORT_METHOD(changeColor:(NSInteger)viewTag) {
  CustomView *view = (CustomView *)self.viewRegistry[@(tag)];
  //确保是在主线程上执行
  [view performSelectorOnMainThread:@selector(changeColor) 
                         withObject:nil 
                      waitUntilDone:YES];
}

- (RCTViewManagerUIBlock)uiBlockToAmendWithShadowView:(RCTShadowView *)shadowView {
  __weak __typeof__(self) weakSelf = self;
  return ^(RCTUIManager *uiManager, NSDictionary<NSNumber *, UIView *> *viewRegistry){
    weakSelf.viewRegistry = viewRegistry;
  };
}
@end
~~~

重要方法`- (RCTViewManagerUIBlock)uiBlockToAmendWithShadowView:`，每当RN布局完成后都会回调这个方法。实现该方法需要返回一个**RCTViewManagerUIBlock**，block中有个重要参数: viewRegistry。viewRegistry中包含了所有RN中所有正在显示的Component,输出来看一下：

~~~
// xcode consoler
{
   ...
    4 = "<RCTView: 0x7fe44c4d71c0; reactTag: 4; frame = (0 0; 414 736); layer = <CALayer: 0x7fe44c4dc320>>";
    23 = "<RCTView: 0x7fe44c549690; reactTag: 23; frame = (0 0; 0 0); layer = <CALayer: 0x7fe44e84c370>>";
    12 = "<RCTText: 0x7fe44ea18c30; reactTag: 12; text:  frame = (0 0; 414 34.6667); opaque = NO; layer = <CALayer: 0x7fe44ea18710>>";
    1 = "<RCTRootContentView: 0x7fe44eb05ed0; reactTag: 1; frame = (0 0; 414 736); gestureRecognizers = <NSArray: 0x7fe44e9e9da0>; layer = <CALayer: 0x7fe44e9f04a0>>";
    22 = "<CustomView: 0x7fe44eb14110; frame = (0 84; 414 406.667); autoresize = W+H; layer = <CALayer: 0x7fe44e9303e0>>";
   ...
}
~~~

其实就是 _viewTag_:View 的键值对，只要我们知道 _viewTag_ 就可以获取到对应的View。

上述代码已经把一个`changeColor:`的方法导出到RN中，调用该方法和使用[Native 模块]()一样。

使用步骤:

1. 通过NativeModules获取XXXViewManager对象
2. 通过`_rootNodeID`属性获取 _viewTag_
3. 使用XXXViewManager调用方法

~~~
import {requireNativeComponent,NativeModules} from 'react-native';

export class CustomView extends Component {
  static propTypes = {
    title: PropTypes.string,
    onClickButton: PropTypes.func,
  };
  
  render() {
    return <RCTCustomView {...this.props} ref={r=>{this._UI = r;}}/>;
  }
  
  changeColor() {
    NativeModules.CustomView.changeColor(this._UI._rootNodeID);
  }
}

const RCTCustomView = requireNativeComponent('RCTCustomView', CustomView);
~~~
<a id="设置componentInterface"></a>
使用很简单：`NativeModules.CustomView.changeColor(this._UI._rootNodeID);`当然，为了美观，我一般是在CustomView的自定义方法里面调用，这样子，外部使用CustomView.changeColor就不需要传入 _viewTag_ 了。这也是我为什么喜欢用第二种写法来设置componentInterface的原因。

> **注意：**
> 
> 1. 通过输出ViewManager的地址，可以看到地址是不变的，所以我猜测不同类型的ViewManager是一个单例。
> 2. 调用Native方法的时候，如果需要更新UI必须要保证在主线程上执行。




