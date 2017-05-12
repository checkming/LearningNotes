#Android关于OOM的解决方案
##OOM
* 内存溢出（Out Of Memory）
* 也就是说内存占有量超过了VM所分配的最大


##出现OOM的原因
1. 加载对象过大
2. 相应资源过多，来不及释放
3. 图片过大导致OOM
4. 界面切换导致OOM

##如何解决
1. 在内存引用上做些处理，常用的有软引用、强化引用、弱引用
2. 在内存中加载图片时直接在内存中作处理，如边界压缩
BitmapFactory.Options opts = new BitmapFactory.Options()
opts.inSamp();
3. 动态回收内存
4. 优化Dalvik虚拟机的堆内存分配
4. 去除xml 中的相关设置,改在程序中设置背景图
Drawable dr = getReSoute.
5. 自定义堆内存大小


