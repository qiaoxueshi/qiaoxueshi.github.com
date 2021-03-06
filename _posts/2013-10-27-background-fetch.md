---
layout: post
title: "Background Fetch"
image-width: ""
image-height: ""
description: ""
category: iOS
tags: [iOS]
---
{% include JB/setup %}

Background Fetch 是iOS7带来的非常Cool的新特性，开启Background Fetch的App会被系统在合适的时机执行后台任务的代码。比如这个场景：你每天晚上10点会通过自己的RSS阅读器App来阅读，系统可能会在10点之前执行App中设定的下载RSS最新资源的任务，当你打开RSS阅读器App的时候就显示出最新的内容。实现Background Fetch的步骤也是非常的简单，下面就来看一下。

###1、开启Background Fetch
给一个App开启Background Fetch非常的简单，可以总结为三个步骤：
####Step 1 
进入`Project`设置 -> `Capabilities` -> 设置`Background Modes`为ON -> 选中`Background Fetch`

![BG_Fetch01](/assets/resources/BG_Fetch01.png)

####Step 2
在ApplicationDelegate类的
{% highlight objc %}
-(BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{% endhighlight %}
方法中，添加下面的代码：

{% highlight objc %}
[[UIApplication sharedApplication] setMinimumBackgroundFetchInterval:UIApplicationBackgroundFetchIntervalMinimum];
{% endhighlight %}

`MinimumBackgroundFetchInterval`参数值是时间间隔的数值，系统保证两次Fetch的时间间隔不会小于这个值，不能保证每隔这个时间间隔都会调用。这里设置为`UIApplicationBackgroundFetchIntervalMinimum`，意思是告诉系统，尽可能频繁的调用我们的Fetch方法。

####Step 3
开始实现我们的Fetch方法，在ApplicationDelegate类中加入下面这个方法：

{% highlight objc %}
- (void)application:(UIApplication *)application performFetchWithCompletionHandler:(void (^)(UIBackgroundFetchResult))completionHandler {
    SSViewController *ssVC = (SSViewController*)self.window.rootViewController;
    if ([ssVC isKindOfClass:[SSViewController class]]) {
        NSLog(@"is SSViewController");
        ssVC.indexValue ++;
        completionHandler(UIBackgroundFetchResultNewData);
    } else {
        NSLog(@"is not SSViewController");
        completionHandler(UIBackgroundFetchResultFailed);
    }
    
}
{% endhighlight %}

这个方法每次系统执行Background Fetch时都会被调用，可以在这里下载网络数据等。执行完下载任务之后，需要立即调用`completionHandler`block。文档中提到系统用耗时来估算这次fetch的电量消耗和数据消耗，如果耗时比较长，未来可能减少被调用的机会。`completionHandler`block可以用的参数值有下面三个：

* UIBackgroundFetchResultNewData 拉取数据OK
* UIBackgroundFetchResultNoData  没有新数据
* UIBackgroundFetchResultFailed  拉取数据失败或者超时

文档中也提到，当这个方法被调用后，App有30s的时间来执行下载操作，然后马上执行`completionHandler`block，就是说最好能把下载任务的耗时限制在30s内，超过30s的，App会被系统挂起。

在刚才给出的方法中，为了方便测试只是更新了ViewController的一个参数值，这个参数值会直接反应到界面上，方面测试。

有个小细节是假如Background Fetch方法更新了UI的话，系统会刷新Home键切换App界面中的缩略图。

###2、模拟Background Fetch
创建了Background Fetch后，怎么来方面的模拟和测试呢？有两种方式，一种是在App被挂起后，系统执行Background Fetch，另外一种是App没有在运行，被系统唤醒执行Background Fetch方法。

#### 情况1
直接运行程序，在Xcode的菜单中，选择"Debug" -> "Simulate Background Fetch"，你会发现会先打开App，然后后台挂起，接着执行`(void)application: performFetchWithCompletionHandler`方法。

![BG_Fetch02](/assets/resources/BG_Fetch02.png)

#### 情况2
复制（Duplicate）一份当前的Schema，在新的Schema的Options下，选中"Launch due to a background fetch event"，运行这个Schema。

![BG_Fetch03](/assets/resources/BG_Fetch03.png)

![BG_Fetch04](/assets/resources/BG_Fetch04.png)

###3、Remote Notifications & Background Transfer Service
Background Fetch适用于定期检查更新数据，如果想从服务端推送一条消息告诉客户端来执行某些操作的话，可以使用Remote Notifications，它和普通的Push Notification很相似，不同的是推送时的Payload不太一样以及客户端收到通知之后会执行一个的方法，和Background Fetch一样有30s的时间来做事情。你看到这里一定有个疑问，如果任务在30s内不能完成怎么破？比如下载音视频文件。Background Transfer Service闪亮出场了，感兴趣的话可以参考Ref里的第3、4条链接里的内容。

###Ref

* [iOS 7 SDK: Working with Background Fetch](http://mobile.tutsplus.com/tutorials/iphone/ios-7-sdk-working-with-background-fetch/)
* iOS 7 by Tutorials
* [iOS 7 SDK: Multitasking Enhancements](http://mobile.tutsplus.com/tutorials/iphone/ios-7-sdk-mutlitasking-enhancements/) 延伸阅读 : About Remote Notifications & Background Transfer Service
* [Multitasking in iOS 7](http://www.objc.io/issue-5/multitasking.html) （推荐，objc.io出品）

完鸟，如果有写的不对的地方，欢迎小伙伴们指正，Have fun~



