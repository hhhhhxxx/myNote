# **JAVA基础**



### exception和error区别

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210603143107974.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81MjE4ODM3NA==,size_16,color_FFFFFF,t_70#pic_center)

检查性异常: 不处理编译不能通过
非检查性异常:不处理编译可以通过，如果有抛出直接抛到控制台。

**(1) 非检查异常:**

ArrayIndexOutOfBoundsException //用非法索引访问数组时抛出的异常。如果索引为负或大于等于数组大小，则该索引为非法索引
ArithmeticException //当出现异常的运算条件时，抛出此异常。（ 例如，一个整数“除以零”时，抛出此类的一个实例）
IllegaArguementException //抛出的异常表明向方法传递了一个不合法或不正确的参数
NullPointerException //空指针异常（调用 null 对象的实例方法等）
ClassCastException //类转换异常
ArrayStoreException //数据存储异常，操作数组时类型不一致
**(2) 检查异常**

ClassNotFoundException // 找不到具有指定名称的类的定义
DataFormatException //数据格式异常
IOException //输入输出异常
SQLException //提供有关数据库访问错误或其他错误的信息的异常
FileNotFoundException //当试图打开指定路径名表示的文件失败时，抛出此异常
EOFException //当输入过程中意外到达文件或流的末尾时，抛出此异常

### NoClassDefFoundError 和 ClassNotFoundException异常



1、StringBuffer 与 StringBuilder 中的方法和功能完全是等价的，
2、只是 StringBuffer 中的方法大都采用了 synchronized 关键字进行修饰，因此是线程安全的，而
StringBuilder 没有这个修饰，可以被认为是线程不安全的。
3、在单线程程序下，StringBuilder 效率更快，因为它不需要加锁，不具备多线程安全而 StringBuffer 则每次都需要判断锁，效率相对更低
