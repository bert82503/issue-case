### Java线上Cases

#### Memory
##### OOM-OutOfMemoryError: unable to create new native thread
* [问题分析：java.lang.OutOfMemoryError: unable to create new native thread](http://blog.csdn.net/ado1986/article/details/48286513)
 * 根本原因：线程池对象持有的线程泄漏引发OOM，进而无法创建本地线程。
 * 建议：线程池对象必须声明为类变量
* [记一次线上OOM的问题](http://blog.csdn.net/ado1986/article/details/49491597)
 * 根本原因：静态全局的ConcurrentHashMap未释放缓存的对象，数量一直增长，从而引发内存溢出。


