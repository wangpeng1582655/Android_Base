# Android_Base
android整个项目的基础仓库

## 1. 四大组件(生命周期，特性)
        注意：四大组件都是运行在主线程
        1. Activity:
           1. 生命周期
              onCreate->onStart->onResume->onPause->onStop->onDestory
              …->onStop->onRestart->onStart->…
        2. Service:
          前台服务，后台服务，一般使用的时后台服务，在系统内存不足的情况下，可能被杀死，需要使用前台服务时，使用startForeground(int id, Notification notification)
          id,通知的id
           在后台运行，没有ui界面，不需要与用户交互，且需要长期运行，不要在Service中做
           耗时操作，因为他也运行在主线程
           1.生命周期，2，区别，3.运行在主线程，4.在service里做耗时操作，需要创建子线程，
           1.startService：
            使用该方式启动服务之后，服务于调用者脱离关系，调用者退出，不影响服务运行
             onCreate->onStartCommond->onDestory
           2.bindService：
             使用该方式启动服务，service的生命周期与调用者绑定，调用者一旦退出，服务也
             退出
             onCreate->onBind->onUnbind->onDestory
        3. Broadcast Receiver:
           前台广播，后台广播
           1.动态注册：用于注册的组件一旦销毁，广播也就失效了
           2.静态注册:AndroidManifest文件注册，一直存在，直到进程消失
        4. Content Provider:
           应用程序间共享数据，如通讯录，图片媒体文件等都提供了相应的Content Provider
           供其他程序调用。
## 2. 存储
   1. SharePreferences(apply,commit):
      1.apply,commit区别
      2.apply之后，调用get能否获取值
   2. Sqlite(创建)  
      创建过程
      1.继承自SQLiteOpenHelper
      2.实现onCreate,onUpgrade方法，onCreate用于创建一些表，做一些初始化，onUpgrade
      方法用于数据库的更新，另外实现构造方法，需要传入3个参数，database_name,
      database_version,CursorFactory，CursorFactory一般不传，使用默认Factory

## 3. Looper,Thread,Handler,MessageQueue联系
     1.一个线程有几个Looper，通过什么保存
        一个线程有且只有一个，通过ThreadLocal保存在线程局部变量中
     2. Looper.prepare:通过ThreadLocal获取当前Looper，如果有，抛出异常，
        没有,则创建Looper，保存在当前线程的ThreadLocal里,并创建MessageQueue
        Looper.loop:无限for(;;)循环，从MessageQueue中取Message,并调用
        msg.target.dispatchMessage,而这个target就是Handler
     3.Handler:创建Handler,重写handleMessage方法，调用Handler.myLooper，从
       Threadlocal里获取当前线程的Looper，进而获取Looper的MessageQueue,post message时，将message放入MessageQueue中，并将Message.target赋值为当前handler，这样
       Looper中msg.target.dispatchMessage既是调用的当前Handler的方法，而该方法调用了重写的handleMessage方法

## 4.ANR(Application Not Responding)
    anr发生的情况
    1.程序在5秒内未响应用户的输入事件(屏幕触摸事件，键盘输入事件)
    2.Broadcast Recceiver 的onReceive事件，前台广播在10s内未执行完毕，后台广播在60s
      内未执行完毕，就会触发anr,一般我们发的时后台广播，发送前台广播，需要在Intent中，
      添加一个Flag,Intent.FLAG_RECEIVER_FOREGROUND
    3.Service:运行在主线程，前台服务20s,后台服务200s没有执行完毕，就会ANR
    4.ContentProvider:ContentProvider的publish在10s内没进行完。

## 5. Activity,Fragment生命周期
     Activity:onCreate->onStart->onResume->onPause->onStop->onDestory
              …->onStop->onRestart->onStart->…
    Fragment:onAttach->onCreate->onCreateView->onActivityCreated->onStart->onResume->onPause->onStop->onDestoryView->onDestory->onDetach
    …->onDestoryView ->onCreateView->…

    Activity在屏幕旋转时的生命周期
    onCreate -> onStart -> onResume -> onPause -> onSaveInstanceState -> onStop >onDestory  -> onCreate -> onStart -> onRestoreInstanceState -> onResume
    1、会走整个生命周期方法，而且会调用onSaveInstance和onRestoreInstancceState方法
    2、sdk版本 > 13,后，要想activity横竖屏切换，不重新创建，需要在Manifest的注册的Activity里添 android:configChanges="keyboardHidden|orientation|screenSize"而sdk<13,android:configChanges="keyboardHidden|orientation"
   3、配置了configChanges，生命周期(从Activity创建开始)：onCreate -> onStart -> onResume -> onConfigurationChanged

## 6.Activity启动模式：
   1. standard:每一次启动，都会生成一个新的实例，放入栈顶中
   2. singleTop:通过singelTop启动Activity时，如果发现有需要启动的实例正在栈顶，责直接重用，否则生成新的实例
   3. singleTask:通过singleTask启动Activity时，如果发现有需要启动的实例正在栈中，责直接移除它上边的实例并 重用该实例，否则生成新的实例
   4. singleInstance:通过singleTask启动Activity时，会启用一个新的栈结构，并将新生成的实例放入栈中。

## 7.IntentService，HandlerThread
    1. IntentService 是继承自 Service,内部通过HandlerThread启动一个新线程处理耗时操作么，可以看做是Service和HandlerThread的结合体，在完成了使命之后会自动停止，适合需要在工作线程处理UI无关任务的场景
    2. 如果启动 IntentService 多次，那么每一个耗时操作会以工作队列的方式在 IntentService 的 onHandleIntent 回调方法中执行，依次去执行，使用串行的方式，执行完自动结束。
    源码角度：HandlerThread，是一个thread，内部run方法，初始化Looper，调用Looper.prepare:创建Looper，MessageQueue,将Looper保存在当前线程的ThreadLocal中，Looper.loop方法，启动消息循环,IntentService：onCreate时，创建了一个HandlerThread，并用这个HandlerThread的Looper创建了一个ServiceHandler,让这个Handler负责这个Thread的消息循环，onStart方法，启动service，将启动的Intent作为Message的obj，传给当前ServiceHandler处理，进而调onHandleIntent处理消息

## 8. LocalBroadcast本地广播:
   1.LocalBroadcast是APP内部维护的一套广播机制，有很高的安全性和高效性。
   所以如果有APP内部发送、接收广播的需要应该使用LocalBroadcast。

   2.Receiver只允许动态注册，不允许在Manifest中注册。
   
   3.LocalBroadcastManager所发送的广播action，只能与注册到LocalBroadcastManager中BroadcastReceiver产生互动。
   使用：1.LocalBroadcastManager.getInstance(context)获取LocalBroadcastManager实例
        2.LocalBroadcastManager.getInstance(context).registerReceiver(receiver, filter);
	3.LocalBroadcastManager.getInstance(context).unregisterReceiver(receiver);
        4.LocalBroadcastManager.getInstance(context).sendBroadcast(intent); 

## 9. 自定义View
## 10. 事件分发
## 11. Glide原理，Lru cache算法
## 12. EventBus原理 
## 13. orm Sqlite创建，第三方框架应用，GreenDao,Realm应用，原理

