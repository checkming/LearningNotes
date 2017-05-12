#Android关于OOM的解决方案
##OOM
每个Android应用程序运行时都有一定的内存限制,各手机开发厂商的标准都不一样,因此在开发中需要注重到内存的使用量,否则就会容易出现内存溢出.
* 内存溢出（Out Of Memory）
* 也就是说内存占有量超过了JVM所分配的最大限值


##出现OOM的原因
1. 加载对象过大
2. 相应资源过多，来不及释放

##如何解决
1. 在内存引用上做些处理，常用的有软引用、强化引用、弱引用
2. 在内存中加载图片时直接在内存中作处理，如边界压缩
3. 动态回收内存
4. 优化Dalvik虚拟机的堆内存分配
5. 自定义堆内存大小

##最经典导致OOM异常的原因  --- 加载图片
* 分析:imageView 在底层创建图片的时候,会占用很大的内存空间
* 解决方式:
1. 在从网络或本地加载图片的时候,只加载缩略图
  * 致命缺点:图片失真
2. 运用Java的软应用,进行图片缓存,将经常需要加载的图片存放在缓存里,避免反复加载(过时)
2. 1.缓存图片到内存中
 一个LruCache 类专门用于处理图片缓存
##LruCache介绍
 * 核心的类是LruCache (此类在android-support-v4的包中提供) 。这个类非常适合用来缓存图片，它的主要算法原理是把最近使用的对象用强引用存储在 LinkedHashMap 中，并且把最近最少使用的对象在缓存值达到预设定值之前从内存中移除。
 当缓存的图片达到预先设定的值,则该类会根据最近最少使用放入算法,移除最近使用次数最少的图片,以防达到OOM溢出的目的.
** 预先设定的值:允许当前内存分配给图片最大的内存空间,根据项目实际需求自行定义的,一般设置为期内存的1/8.
 >获取应用内存的大小的具体代码:
 >int MAXMEMONRY = (int)(Runtime.getRuntime().maxMemory()/1024)

* 在过去，我们经常会使用一种非常流行的内存缓存技术的实现，即软引用或弱引用 (SoftReference or WeakReference)。但是现在已经不再推荐使用这种方式了，因为从 Android 2.3 (API Level 9)开始，垃圾回收器会更倾向于回收持有软引用或弱引用的对象，这让软引用和弱引用变得不再可靠。另外，Android 3.0 (API Level 11)中，图片的数据会存储在本地的内存当中，因而无法用一种可预见的方式将其释放，这就有潜在的风险造成应用程序的内存溢出并崩溃。
 

3. 及时销毁不再使用的Bitmap 对象,即及时回收不用的图片资源.
> 回收Bitmap对象具体代码:
>public void releaseImage(){
  if(null !=bitmap){
    bitmapp.recyle();
    bitmap = null;
  }
}
4. 读取图片时注意方法的调用,适当压缩,即调整图片的大小,利用手机屏幕宽高和该图片的宽高缩放比例进行缩放.
   通过设置Options的inJustDecodeBounds 属性为true,将图片的width 和height属性读出来,然后我们可以利用这些属性对Bitmap 进行压缩,同时通过
   Options.inSampleSize 属性设置图片的压缩比.
 尽量不要使用setImageBitmap或setImageResource或BitmapFactory.decodeResource来设置一张大图，因为这些函数在完成decode后，
 最终都是通过java层的createBitmap来完成的，需要消耗更多内存。 因此，改用先通过BitmapFactory.decodeStream方法，创建出一个bitmap，
 再将其设为ImageView的  source，decodeStream最大的秘密在于其直接调用JNI>>nativeDecodeAsset()来完成decode，无需再使用java层的createBitmap，
 从而节省了java层的空间







