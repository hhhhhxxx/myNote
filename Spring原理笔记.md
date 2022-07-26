# Spring原理笔记



## spring ioc



spring ioc底层实现



bean的生命周期

![image-20221022094543462](C:\Users\25800\AppData\Roaming\Typora\typora-user-images\image-20221022094543462.png)



## spring aop

aop: 动态代理

aop本身是一个扩展功能，所以在BeanPostProcessor的后置处理方法中来实现

1.代理对象的创建过程（advice,切面，切点）

2.通过jdk或者cglib的方式来生成代理对象

3.



## spring 数据库

**REQUIRED**(Spring默认的事务传播类型 required：需要、依赖、依靠)：如果当前没有事务，则自己新建一个事务，如果当前存在事务则加入这个事务
当A调用B的时候：如果A中没有事务，B中有事务，那么B会新建一个事务；如果A中也有事务、B中也有事务，那么B会加入到A中去，变成一个事务，这时，要么都成功，要么都失败。（假如A中有2个SQL，B中有两个SQL，那么这四个SQL会变成一个SQL，要么都成功，要么都失败）

**SUPPORTS**（supports：支持;拥护）:当前存在事务，则加入当前事务，如果当前没有事务，就以非事务方法执行
如果A中有事务，则B方法的事务加入A事务中，成为一个事务（一起成功，一起失败），如果A中没有事务，那么B就以非事务方式运行（执行完直接提交）；

**MANDATORY**（mandatory：强制性的）:当前存在事务，则加入当前事务，如果当前事务不存在，则抛出异常。
如果A中有事务，则B方法的事务加入A事务中，成为一个事务（一起成功，一起失败）；如果A中没有事务，B中有事务，那么B就直接抛异常了，意思是B必须要支持回滚的事务中运行

**REQUIRES_NEW**（requires_new：需要新建）:创建一个新事务，如果存在当前事务，则挂起该事务。
B会新建一个事务，A和B事务互不干扰，他们出现问题回滚的时候，也都只回滚自己的事务；

**NOT_SUPPORTED**（not supported：不支持）:以非事务方式执行,如果当前存在事务，则挂起当前事务
被调用者B会以非事务方式运行（直接提交），如果当前有事务，也就是A中有事务，A会被挂起（不执行，等待B执行完，返回）；A和B出现异常需要回滚，互不影响

**NEVER**（never：从不）: 如果当前没有事务存在，就以非事务方式执行；如果有，就抛出异常。就是B从不以事务方式运行
A中不能有事务，如果没有，B就以非事务方式执行，如果A存在事务，那么直接抛异常

**NESTED**（nested：嵌套的）嵌套事务:如果当前事务存在，则在嵌套事务中执行，否则REQUIRED的操作一样(开启一个事务)
如果A中没有事务，那么B创建一个事务执行，如果A中也有事务，那么B会会把事务嵌套在里面。

## spring mvc



## spring boot



## spring cloud