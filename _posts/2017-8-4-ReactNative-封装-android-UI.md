---
layout: post
title: ReactNative 封装 android UI
category: ReactNative
tags: [ReactNative]
---


## 简介

前几天写了一篇[《ReactNative 封装 iOS UI》](https://liruwei.github.io/2017/08/01/ReactNative-%E5%B0%81%E8%A3%85-iOS-UI)的教程，今天总结一下 Android 的封装教程。

Demo效果如下：
 
![](https://ws1.sinaimg.cn/mw690/962a6dfegy1fi40l3ecexg20al0c6b2a.gif)

> **注意：**
>  
> * 本文章是在 react-native@0.44 下编写的，低版本或者高版本的可能不合适，请自行查看对应版本的文档。
> * [demo下载地址](https://github.com/Liruwei/Demos/tree/master/RN%E5%B0%81%E8%A3%85%20android_UI_Demo)

## 步骤 

1. [创建一个ViewManager的子类。](#bz_1)
2. [实现createViewInstance方法。](#bz_2)
3. [导出视图的属性设置器：使用@ReactProp（或@ReactPropGroup）注解。](#bz_3)
4. [把这个视图管理类注册到应用程序包的createViewManagers里。](#bz_4)
5. [实现JavaScript模块。](#bz_5)
6. [CustomView重写requestLayout](#)

**<a id="bz_1">1.</a> 创建一个ViewManager的子类。**

~~~
//CustomViewManager.java

//CustomView是自定义的LinearLayout的子类
public class CustomViewManager extends SimpleViewManager<CustomView> {

    public static final String REACT_CLASS = "RCTCustomView";

    /**
     * @return the name of this view manager. This will be the name used to reference this view
     * manager from JavaScript in createReactNativeComponentClass.
     */    
    @Override
    public String getName() {
        return REACT_CLASS;
    }
}
~~~

**<a id="bz_2">2.</a> 实现createViewInstance方法。**

~~~
//CustomViewManager.java

/**
 * Creates a view and installs event emitters on it.
 */
@Override
public CustomView createViewInstance(ThemedReactContext context) {
   return new CustomView(context);
}
~~~

**<a id="bz_3">3.</a> 导出视图的属性设置器：使用@ReactProp（或[@ReactPropGroup]()）注解。**

设置方法有两个参数。方法的 _第一个参数是要修改属性的视图实例_ ， _第二个参数是要设置的属性值_ 。方法的返回值类型必须为void，而且访问控制必须被声明为public。

~~~
//CustomViewPackage.java

@ReactProp(name = "title")
public void setTitle(CustomView view, String title) {
   view.setTitle(title);
}
~~~

属性类型对照表：
 
| JavaScript 类型 | Java 类型 |
| --- | --- |
| boolean | boolean, Boolean  |
| number | int, float,double,Integer |
| string | String |
| array | ReadableArray |
| object | ReadableMap |

>**注意：**
>
>* ReadableArray，ReadableMap和我们熟悉的Array以及Map不一样，要经过转换才能获取到里面的值。
>* ReadableArray转Object[] ：[ArrayUtil.java](https://github.com/Liruwei/Demos/tree/master/RN封装%20android_UI_Demo)
>* ReadableMap转Map ： [MapUtil.java](https://github.com/Liruwei/Demos/tree/master/RN封装%20android_UI_Demo)

**<a id="bz_4">4.</a> 把这个视图管理类注册到应用程序包的createViewManagers里。**

在Java这边要做的最后一件事就是注册这个视图控制器。我们需要在应用的Package类的`createViewManagers`方法中添加这个视图控制器。如果视图控制器没有被注册，它也无法在JavaScript中被访问到。

~~~
//CustomViewPackage.java

public class CustomViewPackage implements ReactPackage {

    /**
     * @param reactContext react application context that can be used to create modules
     * @return list of native modules to register with the newly created catalyst instance
     */    
    @Override
    public List<Class<? extends JavaScriptModule>> createJSModules() {
        return Collections.emptyList();
    }

    /**
     * @return list of JS modules to register with the newly created catalyst instance.
     *
     * IMPORTANT: Note that only modules that needs to be accessible from the native code should be
     * listed here. Also listing a native module here doesn't imply that the JS implementation of it
     * will be automatically included in the JS bundle.
     */
    @Override
    public List<ViewManager> createViewManagers(ReactApplicationContext reactContext) {
        List<ViewManager> managers = new ArrayList<>();
        managers.add(new CustomViewManager());
        return managers;
    }

    /**
     * @return a list of view managers that should be registered with {@link UIManagerModule}
     */
    @Override
    public List<NativeModule> createNativeModules(ReactApplicationContext reactContext) {
        return return Collections.emptyList();
    }
}
~~~

这个package需要在MainApplication.java文件的getPackages方法中提供。这个文件位于你的react-native应用文件夹的android目录中。具体路径是: `android/app/src/main/java/com/your-app-name/MainApplication.java.`

~~~
protected List<ReactPackage> getPackages() {
    return Arrays.<ReactPackage>asList(
            new MainReactPackage(),
            new AnExampleReactPackage()); // <-- 添加这一行，类名替换成你的Package类的名字.
}
~~~

**<a id="bz_5">5.</a> 实现JavaScript模块。**

整个过程的最后一步就是创建JavaScript模块并且定义Java和JavaScript之间的接口层。大部分过程都由React底层的Java和JavaScript代码来完成，你所需要做的就是通过propTypes来描述属性的类型。这一步有两种实现方式，和[《iOS 实现JavaScript模块》](https://liruwei.github.io/2017/08/01/ReactNative-封装-iOS-UI/#bz_4)一样。

~~~
// xxx.js

class NativeComponent extends React.Component{
  static propTypes =  {
    title: PropTypes.string,
    ...View.propTypes // 包含默认的View的属性
  }

  render() {
    return <RCTCustomView {...this.props}/>
  }
}

const RCTCustomView = requireNativeComponent('RCTCustomView',NativeComponent);
~~~

## 函数回调

视图组件可以通过`RCTEventEmitter`向Javascript发送事件的方法实现函数回调。这部分内容在[《封装原生模块》](https://reactnative.cn/docs/0.44/native-modules-ios.html#%E7%BB%99javascript%E5%8F%91%E9%80%81%E4%BA%8B%E4%BB%B6)中有提到。

~~~
//CustomView.java

public class CustomView extends LinearLayout {

...
public void onClick(View v) {
    
    //发送事件代码
    //以 key:value 的形式发送到Javascript中。
    WritableMap event = Arguments.createMap();
    event.putString("title", titleValue);
    
    ReactContext reactContext = (ReactContext)getContext();
    reactContext.getJSModule(RCTEventEmitter.class).receiveEvent(
         getId(),
         "topChange",
         event);
}
~~~

这个事件名`topChange`在JavaScript端映射到`onChange`回调属性上（这个映射关系在`UIManagerModuleConstants.java`文件里）

~~~
// xxx.js

class NativeComponent extends React.Component{
  static propTypes =  {
    title: PropTypes.string,
    ...View.propTypes
  }

  render() {
    return <RCTCustomView {...this.props} 
        ref={r=>{this._UI = r;}}     
        onChange={(e)=>{
            //在onChange接受事件处理
            //只能通过nativeEvent属性获取传递过来的值
            let obj = e.nativeEvent;
            console.log(obj.title)
        }}
    />
  }
}

const RCTCustomView = requireNativeComponent('RCTCustomView',NativeComponent);
~~~

## 方法

ViewManager和[ReactContextBaseJavaModule](https://reactnative.cn/docs/0.45/native-modules-android.html#%E6%B3%A8%E5%86%8C%E6%A8%A1%E5%9D%97)一样，可以通过Package类的`createNativeModules`注册。注册以后在JavaScript中通过`NativeModules.XXXViewManager`获取。

步骤：

1. 重写`onAfterUpdateTransaction`、`onDropViewInstance`方法
2. 保存CustomView
3. 导出带有 _viewID_ 的ReactMethod
4. 在`createNativeModules`注册
5. 通过`NativeModules.XXXViewManager`调用方法

~~~
// CustomViewManager.java

// 定义一个静态成员，用来保存CustomView的实例
static HashMap _views = new HashMap();

@ReactMethod
public void changeColor(Integer viewId) {
   final CustomView view = (CustomView) _views.get(viewId);
   if (view instanceof  CustomView) {
       view.post(new Runnable() {
           @Override
           public void run() {
               view.changeColor();
           }
       });
   }
}

@Override
protected void onAfterUpdateTransaction(CustomView view) {
   //RN更新layout的时候会调用这方法
   _views.put(view.getId(),view);
}

@Override
public void onDropViewInstance(CustomView view) {
   //view被移除后会调这方法
   _views.remove(view.getId());
}
~~~

~~~
// CustomViewPackage.java

@Override
public List<NativeModule> createNativeModules(ReactApplicationContext reactContext) {
   List<NativeModule> modules = new ArrayList<>();
   modules.add(new CustomViewManager());
   return modules;
}
~~~



~~~
// xxx.js

class NativeComponent extends React.Component{
  static propTypes =  {
    title: PropTypes.string,
    onClickButton: PropTypes.func,
    ...View.propTypes
  }

  changeColor() {
    //利用NativeModules调用方法
    NativeModules.CustomView.changeColor(this._UI._rootNodeID);
  }

  render() {
    let {onClickButton} = this.props;
    return <RCTCustomView {...this.props} ref={r=>{this._UI = r;}} onChange={(e)=>{
      onClickButton && onClickButton(e);
    }}/>
  }
}

const RCTCustomView = requireNativeComponent('RCTCustomView',NativeComponent);
~~~

>**注意：** 在JavaScript中可以通过_rootNodeID获取 _viewId_

## CustomView必须重写requestLayout

在封装ListView的时候，调用setAdapter或者adapter.notifyDataSetChanged后页面没有进行相应的更新，必须滚动页面后才会进行相应的刷新。原因在于ReactVIewGroup没用调用父类的requestLayout方法。requestLayout的作用就是告诉ViewRootImpl进行从新布局，而RN的view都是继承ReactVIewGroup的，ListView需要重新布局的时候ReactVIewGroup并没有把requestLayout事件传递给ViewRootImpl。

解决方法：

~~~
// CustomView.java 
// 在自定义的View中重写加入以下方法

private final Runnable measureAndLayout = new Runnable() {
   @Override
   public void run() {
       measure(
               MeasureSpec.makeMeasureSpec(getWidth(), MeasureSpec.EXACTLY),
               MeasureSpec.makeMeasureSpec(getHeight(), MeasureSpec.EXACTLY));
       layout(getLeft(), getTop(), getRight(), getBottom());
   }
};

@Override
public void requestLayout() {
   super.requestLayout();
   // The spinner relies on a measure + layout pass happening after it calls requestLayout().
   // Without this, the widget never actually changes the selection and doesn't call the
   // appropriate listeners. Since we override onLayout in our ViewGroups, a layout pass never
   // happens after a call to requestLayout, so we simulate one here.
   post(measureAndLayout);
}
~~~

> **参考：**
> 
> * [ReactNative（嵌入到android）调用android原生组件与原生模块（比如某个方法）](http://blog.csdn.net/mwp123555/article/details/72867466)
> * [Android View 深度分析requestLayout、invalidate与postInvalidate](http://blog.csdn.net/a553181867/article/details/51583060)

