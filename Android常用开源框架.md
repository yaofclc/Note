## RXJava 

### RxJava四要素 
1.被观察者  
2.观察者  
3.订阅  
4.事件  


## Retrofit

### 网络通信八个步骤
1.创建Retrofit实例  
2.定义一个网络请求接口并为接口中的方法添加注解  
3.通过动态代理生成网络请求对象  
4.通过网络请求适配器将网络请求对象进行平台适配  
5.通过网络请求执行器发送网络请求  
6.通过数据转换器解析数据  
7.通过回调执行器切换线程  
8.用户在主线程处理返回结果  


## Leakcanary 原理
### 原理 
1.Activity Destroy之后会将它放在一个WeakReference  
2.这个WeakReference关联到一个ReferenceQueue  
3.查看ReferenceQueue是否存在Activity的引用  
4.如果该Activity泄漏了，Dump出heap信息，然后再去分析泄漏路径  


**总结一**  
1.首先创建一个refwatcher，启动一个ActivityRefWatcher  
2.通过ActivityLifecycleCallbacks把Activity的onDestroy生命周期关联  
3.最后在线程池中去开始分析我们的内存泄漏  


**总结checkForLeak**  
1.把.hprof转为Snapshot  
2.优化gcroots  
3.找出泄漏的对象/找出泄漏对象的最短路径  


1.LeakCanary是如何使用ObjectWatcher 监控生命周期的？  
LeakCanary使用了Application的ActivityLifecycleCallbacks和FragmentManager的FragmentLifecycleCallbacks方法进行Activity和Fragment的生命周期检测，当Activity和Fragment被回调onDestroy以后就会被ObjectWatcher生成KeyedReference来检测，然后借助HeapDumpTrigger的轮询和触发gc的操作找到弹出提醒的时机。

2.LeakCanary如何dump和分析.hprof文件的？  
使用Android平台自带的Debug.dumpHprofData方法获取到hprof文件，使用自建的Shark库进行解析，获取到LeakTrace  

## EventBus 
- Subscribe：注解，包含ThreadMode sticky = false，priority = 0
- 初始化：getDefault()获取单例实例对象，通过构建者模式创建  
- 注册：SubscriberMethodFinder的findSubscriberMethods(subscriberClass)带有Subscriber的注解，返回List<SubscriberMethod>，遍历集合调用subscribe(subscriber,subscriberMethod)注册  
- findSubscriberMethods():现在缓存中找，没有则调用findUsingReflection(subscriber)或者findUsingInfo()返回List<SubscriberMethod>放入缓存，并返回  
- new Subscription(subscriber,subscriberMethod)，检查是否注册过，没注册过就注册(Map<Class<?>,CopyOnWriteArrayList<Subscription>> subscriptionsByEventType)，已经注册了抛异常，根据优先级排序，后面还有判断粘性事件  
- post():从subscriptionsByEventType获取CopyOnWriteArrayList<Subscription>并遍历调用postToSubscription发送    
- ThreadMode: POSTING 发送消息的当前线程处理，MAIN:HandlerPoster处理，BACKGROUND:BackgroundPoster处理，取出所有消息处理，最多1000，ASYNC：AsyncPoster处理，只处理一个  
- 取消注册：


## Glide  
- with(context):返回RequestManager，包含在RequestManagerFragment里，Glide通过RequestManagerFragment监听生命周期  
- load():返回DrawableTypeRequest，它是DrawableRequestBuilder的子类，用于构建参数，placeHolder等，  
- into():真正请求图片加载  
- 缓存：内存缓存和硬盘缓存，其中内存缓存会先调用loadFromCache，使用LruCache算法，如果没有调用loadFromActiveResources，使用了弱引用，如果都没有就回开启自线程去网络请求图片，内存缓存主要是防止同样的图片重复读取到内存中来，节约JVM的内存空间，硬盘缓存也是使用LruCache算法，主要是防止同一张网络图片重复从网络中读取和下载  
- 四种缓存策略：NONE：不缓存文件，SOURCE：只缓存原图，RESULT：只缓存最终加载的图(默认的缓存策略),ALL：同时缓存原图和结果图