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
