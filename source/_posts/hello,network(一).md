---
date:
  - 2016-8-20
tags:  
  - Android
  - 笔记
  - hello
title: hello，network(一)
---
网络请求项目经验总结
<!--more-->
# NetWorkUtils
　这篇博客的初衷是为了自己的一个开源项目而写的---[NetWorkUtis][1]，现在是初期版本，肯定会有有不少设计或者是代码方面的问题，希望大家能去看看，提点意见。

　看标题就知道，是关于Android 网络请求方面的问题。

　Android的网络请求技术一直以来发展迅速，官方，民间大神，特别是接下来，全世界的网络基础的快速部署和升级，而每个人又少不了升级，业务推动技术。

　根据官方的说法,是在Android2.3之前HttpUrlConnection类库是存在问题的，例如连接池的问题。所以，在Android2.3以前是推荐使用HttpClient类库,但是是Apache之前在JAVA平台上的类库，好用是好用，但是与Android系统，在性能优化和兼容方面的工作量大，谷歌的工程师们并不是推荐(因为他们不知道什么时候会去升级HttpClient)。

　后来，谷歌又推出了Volley,Volley就是在2.3以前调用HtttpClient，2.3以后调用的HttpUrlConnection，证明，谷歌方面还是比较推荐HttpUrlConnection的.那么问题来了，挖...不，Android网络层到底用什么？不是说谷歌在2.3以后调用了HttpUrlConnection么？

　为什么有这样的一个问题呢?因为Square公司推出了OkHttp.并且在短时间几乎漫延到了世界各地，因为Square公司开源的质量都是顶级的，推动了Android一步又一步发展的，经历了实际业务的考验的。Android在4.4之后使用okHttp来进行一个网络连接的替换。Volley也听说停止了维护。

那么，现在摆在我们面前的就有三个主流选择（因为主要的网络底层协议库是这些，大部分的网络库是应用层的）。

 1. HttpClient
 2. HttpUrlConnection
 3. OkHttp

　但是，现在的一个最佳选择，随着技术的发展，最终会产生他的替代品，可能需要漫长的时间，也可能短短几个月或者一年就会发生。那么，之前因为选择的技术残留在项目中，我们就不能很快的进行更换。

　举个栗子：Android6.0删除HttpClient对大家项目影响应该是蛮大的，不过因为6.0系统占有比例小，使用了HttpClient的大家现在不急着修改，而且很大一部分开发者应该会选择加入HttpClient的jar，这样比较丑陋的解决方案。

　所以，我就萌生(-_-||| )了这个项目的想法。

　而我的设计想法主要来源于两个点：

　　一个是一些网络图片加载库的可以替换网络层，或者自定义网络层的做法。

　　第二个是一些下拉刷新框架，例如[android-Ultra-Pull-To-Refresh](https://github.com/liaohuqiu/android-Ultra-Pull-To-Refresh)的写法。[廖祜秋大神](https://github.com/liaohuqiu)将实现上拉，松开等的操作抽象出来形成接口，这个样的话，当项目需要或者自己心血来潮，想实现一个新的下拉动画，就轻而易举了。
```java
public interface PtrUIHandler {

    /**
     * When the content view has reached top and refresh has been completed, view will be reset.
     *
     * @param frame
     */
    public void onUIReset(PtrFrameLayout frame);

    /**
     * prepare for loading
     *
     * @param frame
     */
    public void onUIRefreshPrepare(PtrFrameLayout frame);

    /**
     * perform refreshing UI
     */
    public void onUIRefreshBegin(PtrFrameLayout frame);

    /**
     * perform UI after refresh
     */
    public void onUIRefreshComplete(PtrFrameLayout frame);

    public void onUIPositionChange(PtrFrameLayout frame, boolean isUnderTouch, byte status, PtrIndicator ptrIndicator);
}

```
　我想，网络层应该也可以，抽象出一个通用的抽象类，内部实现由具体的子类实现，然后定义一个固定的网络方法类，来调用继承了抽象类的子类，或者进行内部的网络层的替换。
```java

public abstract class NetWork<R extends NetWorkCallback> {

    /**
     * 网络框架初始化
     *
     * @param application 应用上下文
     */
    public abstract void init(Application application);

    /**
     * 网络层的post请求方法，post一个json字符串到服务器
     *
     * @param paramsJSON   json
     * @param url          请求url
     * @param connTimeOut  连接超时时间
     * @param readTimeOut  读取超时时间
     * @param writeTimeOut 写入超时时间
     * @param tag          请求tag
     * @param callback     请求回调
     */
    public abstract void post(String url, String paramsJSON,
                              long connTimeOut, long readTimeOut, long writeTimeOut,
                              String tag, R callback);

    /**
     * 网络层的post请求方法,post
     *
     * @param paramsMap    请求需要的参数集
     * @param url          请求url
     * @param connTimeOut  连接超时时间
     * @param readTimeOut  读取超时时间
     * @param writeTimeOut 写入超时时间
     * @param tag          请求tag
     * @param callback     请求回调
     */
    public abstract void post(String url, HashMap<String, String> paramsMap,
                              long connTimeOut, long readTimeOut, long writeTimeOut,
                              String tag, R callback);

    /**
     * 网络层的get请求方法
     *
     * @param paramsMap    请求需要的参数集
     * @param url          请求url
     * @param connTimeOut  连接超时时间
     * @param readTimeOut  读取超时时间
     * @param writeTimeOut 写入超时时间
     * @param tag          请求tag
     * @param callback     请求回调
     */
    public abstract void get(String url, HashMap<String, String> paramsMap,
                             long connTimeOut, long readTimeOut, long writeTimeOut,
                             String tag, R callback);

    /**
     * 网络层上传参数和单个文件
     *
     * @param uploadUrl     上传url
     * @param paramsMap     上传同时需要带的参数集
     * @param fileKey       文件key
     * @param file          文件
     * @param connTimeOut   连接超时时间
     * @param readTimeOut   读取超时时间
     * @param writeTimeOut  写入超时时间
     * @param uploadFileTag 上传文件tag
     * @param callback      上传回调
     */
    public abstract void uploadFile(String uploadUrl,
                                    HashMap<String, String> paramsMap,
                                    String fileKey, File file,
                                    long connTimeOut, long readTimeOut, long writeTimeOut,
                                    String uploadFileTag, R callback);

    /**
     * 网络层上传参数和多个文件
     *
     * @param uploadUrl     上传url
     * @param paramsMap     上传同时需要带的参数集
     * @param fileKeys      文件keys
     * @param files         文件s
     * @param connTimeOut   连接超时时间
     * @param readTimeOut   读取超时时间
     * @param writeTimeOut  写入超时时间
     * @param uploadFileTag 上传文件tag
     * @param callback      上传回调
     */
    public abstract void uploadFile(String uploadUrl, HashMap<String, String> paramsMap,
                                    List<String> fileKeys, List<File> files,
                                    long connTimeOut, long readTimeOut, long writeTimeOut,
                                    String uploadFileTag, R callback);

    /**
     * 网络层下载文件
     *
     * @param downUrl      文件下载url
     * @param savePath     文件保存路径
     * @param saveFileName 文件保存名称
     * @param connTimeOut  下载链接超时时间
     * @param readTimeOut  读取超时时间
     * @param writeTimeOut 写超时时间
     * @param downFileTag  下载文件tag
     * @param callBack     下载回调
     */
    public abstract void downFile(String downUrl, String savePath, String saveFileName,
                                  long connTimeOut, long readTimeOut, long writeTimeOut,
                                  String downFileTag, DownCallback callBack);

    /**
     * 取消tag对应的请求
     *
     * @param tag 请求tag
     */
    public abstract void cancel(String tag);

    /**
     * 取消tag对应的下载请求
     *
     * @param tag 请求tag
     */
    public abstract void cancelDown(String tag);

    /**
     * 取消tag对应的上传请求
     *
     * @param tag 请求tag
     */
    public abstract void cancelUpload(String tag);

    /**
     * 取消所有请求
     */
    public abstract void cancelAll();
}
```
　上面大致就是我的想法了,实际项目中的一些总结。不过,现在很简单的，只是参数相关,例如okhttp的拦截器、http协议的请求头等特性.  
　
然后，我通过公共的HttpResponse来兼容其余网络库的HttpResponse
```java
public abstract class NetWorkResponse<T> {

    public abstract String getString();

    public abstract Object getResponse();
}
```
　当使用着需要某些网络层特定的功能的时候，就需要自己去获取Response或者Request了。这些都将由子类来暴露接口或者具体的实现。

　当需要扩展某个网络层的时候，也可以自定义Callback。当然，我还是不建议大家使用自定义Callback。

　之前说过，这个项目就是为了在替换网络库的时候的减少甚至没有代码修改，如果使用自定义callback，可能效果就没有那么好了。

　大概就是这样了，总结一下，将项目分为几层：

 - 供使用者直接调用一层，提供项目接口调用，我称呼为NetWorkUtils，改动应该是微乎其微的(为了项目的持续发展，当有更好的解决方案，是必不可少添加上去的)。

 - NetWorkUtils调用抽象网络请求层，我称为NetWork层。在NetWorkUtils层提供一个变量，供使用者初始化NetWork层。

 - Callback层，兼容不同的网络库中的回调或者监听。

　其余的都是些兼容的小细节了。写到这里，突然想起现在的一个趋势：用WebAPP来代替原生态开发，也是兼容的路子。我这个也是有一点点赶上时代的潮流么？

　其实感觉就想是一个VPN的模式，把okHttp、httpUrlConnection当做美国，法国等国家的网络，使用者就是'防火墙'里面的人，要出去看一看的话，总是得弄一个梯子，梯子来帮你兼容和打开各国的网络。

　不过，WebAPP是跨平台兼容，我只是兼容一下类库的使用，差之甚远啊。

　因为会存在只使用某一个网络库或者想自己扩展的，所以我将原型与扩展分开来了。

　原型设计:　[NetWorkUtils](https://github.com/zengcanxiang/NetWorkUtils)

---

  [1]: https://github.com/zengcanxiang/NetWorkUtils
  [2]: https://github.com/Y0LANDA/NoHttp
  [3]: http://blog.csdn.net/yanzhenjie1003
