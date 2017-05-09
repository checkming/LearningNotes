#ANR


## 前提背景(补)
1.广播是否可以请求网络?
     由于网络请求是一个耗时的操作,而BroadcastReceiver 里不能做一些比较耗时的操作,否则会弹出ANR(application no response )的对话框,因此广播不可以请求网络.
     
2.广播引起ANR的时间限制? 
     在Android中,程序的响应被活动管理器和窗口管理器这两个系统服务所监视,当BroadcastReceiver 在10秒内没有执行完毕,Android会认为该程序无响应,所以在BroadcastReceiver 里不能做一些比较耗时的操作,否则会弹出ANR的对话框.
     
     
---

1、ANR排错一般有三种类型

1. KeyDispatchTimeout(5 seconds) --主要是类型按键或触摸事件在特定时间内无响应
2. BroadcastTimeout(10 seconds) --BroadcastReceiver在特定时间内无法处理完成
3. ServiceTimeout(20 secends) --小概率事件 Service在特定的时间内无法处理完成


2、哪些操作会导致ANR
在主线程执行以下操作：
1. 高耗时的操作，如图像变换
2. 磁盘读写，数据库读写操作
3. 大量的创建新对象


3、如何避免

1. UI线程尽量只做跟UI相关的工作
2. 耗时的操作(比如数据库操作，I/O，连接网络,发送或接收网络数据,进行大量计算,读写文件或者别的有可能阻塞UI线程的操作)把它放在单独的线程处理
3. 尽量用Handler来处理UIThread和别的Thread之间的交互
4. 在设计与代码编写阶段避免出现同步,死锁及其错误处理不恰当等情况


4、解决的逻辑
1. 使用AsyncTask
	1. 在doInBackground()方法中执行耗时操作
	2. 在onPostExecuted()更新UI
2. 使用Handler实现异步任务
	1. 在子线程中处理耗时操作
	2. 处理完成之后，通过handler.sendMessage()传递处理结果
	3. 在handler的handleMessage()方法中更新UI
	4. 或者使用handler.post()方法将消息放到Looper中
3.类型二的解决的措施: (补)
   如果需要完成一项比较耗时的工作,应该通过发送Intent给Service ,由Service来完成,而不是使用子线程的方法来解决,因为BroadcastReceiver 的生命周期很短,(在onCreate()执行后BroadcastReceiver 的实例就会被销毁),子线程可能还没有结束,BroadcastReceiver就先结束了.如果BroadcastReceiver 先结束了,它的宿主还在运行,那么子线程还会继续执行.但宿主进程此时很容易在系统需要内存时被优先杀死,因为根据activity的周期原理.(他是属于一个空没有任何活动组件的进程)	
	

5、如何排查

1. 首先分析log
2. 从trace.txt文件查看调用stack，adb pull data/anr/traces.txt ./mytraces.txt
3. 看代码
4. 仔细查看ANR的成因(iowait?block?memoryleak?)

6、监测ANR的Watchdog

最近出来一个叫LeakCanary

#FC(Force Close)
##什么时候会出现
1. Error
2. OOM，内存溢出
3. StackOverFlowError
4. Runtime,比如说空指针异常

##解决的办法
1. 注意内存的使用和管理
2. 使用Thread.UncaughtExceptionHandler接口
