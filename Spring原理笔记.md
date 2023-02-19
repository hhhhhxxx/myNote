# Spring原理笔记



## spring ioc



spring ioc底层实现

![image-20221104150822355](D:\Code\myNote\Spring原理笔记.assets\image-20221104150822355.png)



```java
ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");

Student student = context.getBean("student", Student.class);


// AbstractApplicationContext

@Override
public <T> T getBean(String name, Class<T> requiredType) throws BeansException {
		assertBeanFactoryActive();
		return getBeanFactory().getBean(name, requiredType);
}
```

doGetBean

1. 从缓存中获取bean，如果获取到了则直接获取bean实例返回
2. 没有获取到就去创建
3. 如果有父bean，就先创建父bean
4. 合并bean
5. 检查合并后的bean是否依赖其他的bean，若有先创建依赖的bean
6. 创建不通生命周期的bean

```java
protected <T> T doGetBean(String name, @Nullable Class<T> requiredType, @Nullable Object[] args, boolean typeCheckOnly) throws BeansException {
        //解析bean的名字 可能有别名
        String beanName = this.transformedBeanName(name);
        // Eagerly check singleton cache for manually registered singletons.
		// 获取bean  一二三级缓存
		// 1.singletonObjects：单例对象的cache当中获取
		// 2.earlySingletonObjects：提前曝光单例对象的Cache
		// 3.singletonFactories：单例对象工厂的cache
    	Object sharedInstance = this.getSingleton(beanName);
        Object bean;
    
        
        if (sharedInstance != null && args == null) {
            if (this.logger.isTraceEnabled()) {
                if (this.isSingletonCurrentlyInCreation(beanName)) {
                    this.logger.trace("Returning eagerly cached instance of singleton bean '" + beanName + "' that is not fully initialized yet - a consequence of a circular reference");
                } else {
                    this.logger.trace("Returning cached instance of singleton bean '" + beanName + "'");
                }
            }
            //获取指定bean的示实例对象，主要用于FactoryBean的特殊处理，普通Bean会直接返回sharedInstance本身
			//下面会经常用到这个方法
            bean = this.getObjectForBeanInstance(sharedInstance, name, beanName, (RootBeanDefinition)null);
        } else {
            if (this.isPrototypeCurrentlyInCreation(beanName)) {
                throw new BeanCurrentlyInCreationException(beanName);
            }
			// Check if bean definition exists in this factory.
			// 获取父容器的bean 然后创建父bean
            BeanFactory parentBeanFactory = this.getParentBeanFactory();
            if (parentBeanFactory != null && !this.containsBeanDefinition(beanName)) {
                // Not found -> check parent.
                String nameToLookup = this.originalBeanName(name);
                if (parentBeanFactory instanceof AbstractBeanFactory) {
                    return ((AbstractBeanFactory)parentBeanFactory).doGetBean(nameToLookup, requiredType, args, typeCheckOnly);
                }

                if (args != null) {
                    return parentBeanFactory.getBean(nameToLookup, args);
                }

                if (requiredType != null) {
                    return parentBeanFactory.getBean(nameToLookup, requiredType);
                }

                return parentBeanFactory.getBean(nameToLookup);
            }

            if (!typeCheckOnly) {
                this.markBeanAsCreated(beanName);
            }

            try {
                //合并bean
                RootBeanDefinition mbd = this.getMergedLocalBeanDefinition(beanName);
                // 检查是否是抽象类
                this.checkMergedBeanDefinition(mbd, beanName, args);
                // 是否有当前bean以来的bean，先创建依赖的bean
                String[] dependsOn = mbd.getDependsOn();
                String[] var11;
                if (dependsOn != null) {
                    var11 = dependsOn;
                    int var12 = dependsOn.length;

                    for(int var13 = 0; var13 < var12; ++var13) {
                        String dep = var11[var13];
                        if (this.isDependent(beanName, dep)) {
                            throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Circular depends-on relationship between '" + beanName + "' and '" + dep + "'");
                        }

                        this.registerDependentBean(dep, beanName);

                        try {
                            this.getBean(dep);
                        } catch (NoSuchBeanDefinitionException var24) {
                            throw new BeanCreationException(mbd.getResourceDescription(), beanName, "'" + beanName + "' depends on missing bean '" + dep + "'", var24);
                        }
                    }
                }
				// 单例对象走这里
                if (mbd.isSingleton()) {
                    sharedInstance = this.getSingleton(beanName, () -> {
                        try {
                            // 创建bean的逻辑
                            return this.createBean(beanName, mbd, args);
                        } catch (BeansException var5) {
                            this.destroySingleton(beanName);
                            throw var5;
                        }
                    });
                    bean = this.getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
                } else if (mbd.isPrototype()) {
                    var11 = null;

                    Object prototypeInstance;
                    try {
                        this.beforePrototypeCreation(beanName);
                        prototypeInstance = this.createBean(beanName, mbd, args);
                    } finally {
                        this.afterPrototypeCreation(beanName);
                    }

                    bean = this.getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);
                } else {
                    String scopeName = mbd.getScope();
                    Scope scope = (Scope)this.scopes.get(scopeName);
                    if (scope == null) {
                        throw new IllegalStateException("No Scope registered for scope name '" + scopeName + "'");
                    }

                    try {
                        Object scopedInstance = scope.get(beanName, () -> {
                            this.beforePrototypeCreation(beanName);

                            Object var4;
                            try {
                                var4 = this.createBean(beanName, mbd, args);
                            } finally {
                                this.afterPrototypeCreation(beanName);
                            }

                            return var4;
                        });
                        bean = this.getObjectForBeanInstance(scopedInstance, name, beanName, mbd);
                    } catch (IllegalStateException var23) {
                        throw new BeanCreationException(beanName, "Scope '" + scopeName + "' is not active for the current thread; consider defining a scoped proxy for this bean if you intend to refer to it from a singleton", var23);
                    }
                }
            } catch (BeansException var26) {
                this.cleanupAfterBeanCreationFailure(beanName);
                throw var26;
            }
        }

        if (requiredType != null && !requiredType.isInstance(bean)) {
            try {
                T convertedBean = this.getTypeConverter().convertIfNecessary(bean, requiredType);
                if (convertedBean == null) {
                    throw new BeanNotOfRequiredTypeException(name, requiredType, bean.getClass());
                } else {
                    return convertedBean;
                }
            } catch (TypeMismatchException var25) {
                if (this.logger.isTraceEnabled()) {
                    this.logger.trace("Failed to convert bean '" + name + "' to required type '" + ClassUtils.getQualifiedName(requiredType) + "'", var25);
                }

                throw new BeanNotOfRequiredTypeException(name, requiredType, bean.getClass());
            }
        } else {
            return bean;
        }
    }
```





singletonObjects  单例缓存池



```java
	protected Object getSingleton(String beanName, boolean allowEarlyReference) {
		Object singletonObject = this.singletonObjects.get(beanName);
		if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
			synchronized (this.singletonObjects) {
				singletonObject = this.earlySingletonObjects.get(beanName);
				if (singletonObject == null && allowEarlyReference) {
					ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
					if (singletonFactory != null) {
						singletonObject = singletonFactory.getObject();
						this.earlySingletonObjects.put(beanName, singletonObject);
						this.singletonFactories.remove(beanName);
					}
				}
			}
		}
		return singletonObject;
```



```java
	protected void addSingleton(String beanName, Object singletonObject) {
		synchronized (this.singletonObjects) {
			this.singletonObjects.put(beanName, singletonObject); // 加入到单例缓存池
			this.singletonFactories.remove(beanName); // 从三级缓存移出
			this.earlySingletonObjects.remove(beanName); // 从二级缓存移出（早期对象存在与二级缓存）
			this.registeredSingletons.add(beanName); // 记录保存已处理的对象名字
		}
	}
```



bean的生命周期

![image-20221022094543462](C:\Users\25800\AppData\Roaming\Typora\typora-user-images\image-20221022094543462.png)



 SpringIOC有两个核心思想就是IOC控制反转和DI依赖注入，IOC 控制反转的基本思想是，将原来的对象控制从使用者，有了spring之后可以把整个对象交给spring来帮我们进行管理，DI 依赖注入，就是把对应的属性的值注入到具体的对象中。spring提供<bean/>标签和@Autowired和@Resource注解等方式注入，注入方式本质上是AbstractAutowireCapableBeanFactory的populateBean() 方法先从beanDefinition 中取得设置的property值，例如autowireByName方法会根据bean的名字注入；autowireByType方法根据bean的类型注入，完成属性值的注入（涉及bean初始化过程）。对象会存储在map结构中，在spring使用Map结构的singletonObjects存放完整的bean对象（涉及三级缓存和循环依赖）。整个bean的生命周期，从创建到使用到销毁的过程全部都是由容器来管理（涉及bean的生命周期）。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513185033417.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNTY0NDEw,size_16,color_FFFFFF,t_70)



三级缓存的作用

```java
// DefaultSingletonBeanRegistry#getSingleton()
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    // Quick check for existing instance without full singleton lock
    Object singletonObject = this.singletonObjects.get(beanName);
    // 一级缓存中没有 beanName, 且 beanName 正在创建中
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
        singletonObject = this.earlySingletonObjects.get(beanName);
        // bean 的早期引用中没有 beanName，且允许早期引用
        if (singletonObject == null && allowEarlyReference) {
            synchronized (this.singletonObjects) {
                // Consistent creation of early reference within full singleton lock
                singletonObject = this.singletonObjects.get(beanName);
                if (singletonObject == null) {
                    singletonObject = this.earlySingletonObjects.get(beanName);
                    if (singletonObject == null) {
                        ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                        if (singletonFactory != null) {
                            singletonObject = singletonFactory.getObject();
                            this.earlySingletonObjects.put(beanName, singletonObject);
                            this.singletonFactories.remove(beanName);
                        }
                    }
                }
            }
        }
    }
    return singletonObject;
}
```



```java
// 设置三级缓存 传入lamda表达式 即上面的singletonFactory.getObject()方法
addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
```

```java
protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
    Object exposedObject = bean;
    if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
        for (BeanPostProcessor bp : getBeanPostProcessors()) {
            
            // 这里会有一个进行aop的后置处理器 AbstractAutoProxyCreator
            if (bp instanceof SmartInstantiationAwareBeanPostProcessor) {
                SmartInstantiationAwareBeanPostProcessor ibp = (SmartInstantiationAwareBeanPostProcessor) bp;
                // 完成了将普通对象替换成代理对象的工作
                exposedObject = ibp.getEarlyBeanReference(exposedObject, beanName);
            }
        }
    }
    return exposedObject;
}
```





普通对象只需要二级缓存就可以，三级缓存是解决aop对象的循环依赖。



如果进行aop的对象的成员也进行aop，是通过获取普通对象，然后再获取成员对象，这里的成员对象是已经代理的。



## spring aop

aop: 动态代理

aop本身是一个扩展功能，所以在BeanPostProcessor的后置处理方法中来实现

1.代理对象的创建过程（advice,切面，切点）

2.通过jdk或者cglib的方式来生成代理对象

3.



 AOP，一般称为[面向切面编程](https://so.csdn.net/so/search?q=面向切面编程&spm=1001.2101.3001.7020)，作为面向对象的一种补充，用于将那些与业务无关，但却对多个对象产生影响的公共行为和逻辑，抽取并封装为一个可重用的模块，这个模块被命名为“切面”（Aspect），切面将那些与业务无关，却被业务模块共同调用的逻辑提取、并封装起来，减少了系统中的重复代码，降低了模块间的耦合度，同时提高了系统的可维护性。可用于权限认证、日志、事务处理等。



## spring 数据库

事务的传播

这个注解的效果是看的是调用者，当前指的是调用者，

比如controller调用service，controller没有开启事务，service开始事务，为required，则新建一个事务。



**REQUIRED**(Spring默认的事务传播类型 required：需要、依赖、依靠)：如果当前没有事务，则自己新建一个事务，如果当前存在事务则加入这个事务
当A调用B的时候：如果A中没有事务，B中有事务，那么B会新建一个事务；如果A中也有事务、B中也有事务，那么B会加入到A中去，变成一个事务，这时，要么都成功，要么都失败。（假如A中有2个SQL，B中有两个SQL，那么这四个SQL会变成一个SQL，要么都成功，要么都失败）

**SUPPORTS**（supports：支持;拥护）:当前存在事务，则加入当前事务，如果当前没有事务，就以非事务方法执行
如果A中有事务，则B方法的事务加入A事务中，成为一个事务（一起成功，一起失败），如果A中没有事务，那么B就以非事务方式运行（执行完直接提交）； （不抛异常）

**MANDATORY**（mandatory：强制性的）:当前存在事务，则加入当前事务，如果当前事务不存在，则抛出异常。
如果A中有事务，则B方法的事务加入A事务中，成为一个事务（一起成功，一起失败）；如果A中没有事务，B中有事务，那么B就直接抛异常了，意思是B必须要支持回滚的事务中运行。（抛异常）

**REQUIRES_NEW**（requires_new：需要新建）:创建一个新事务，如果存在当前事务，则挂起该事务。
B会新建一个事务，A和B事务互不干扰，他们出现问题回滚的时候，也都只回滚自己的事务；

**NOT_SUPPORTED**（not supported：不支持）:以非事务方式执行,如果当前存在事务，则挂起当前事务
被调用者B会以非事务方式运行（直接提交），如果当前有事务，也就是A中有事务，A会被挂起（不执行，等待B执行完，返回）；A和B出现异常需要回滚，互不影响

**NEVER**（never：从不）: 如果当前没有事务存在，就以非事务方式执行；如果有，就抛出异常。就是B从不以事务方式运行
A中不能有事务，如果没有，B就以非事务方式执行，如果A存在事务，那么直接抛异常

**NESTED**（nested：嵌套的）嵌套事务:如果当前事务存在，则在嵌套事务中执行，否则REQUIRED的操作一样(开启一个事务)
如果A中没有事务，那么B创建一个事务执行，如果A中也有事务，那么B会会把事务嵌套在里面。



**NESTED和REQUIRES_NEW的区别**

NESTED是为被嵌套的方法开启了一个子事务，这个事务与父类使用的是同一个连接。**父事务回滚，子事务要回滚，子事务回滚，父事务不一定回滚**

REQUIRES_NEW是使用一个全新的事务，这个事务属于另外一条全新的连接。**相当于相互独立，不影响。**

两者最重要的体现，就是在多数据源中，REQUIRES_NEW会再次触发一下数据源的获取，而NESTED则不会



## spring mvc

![img](https://img-blog.csdnimg.cn/20181228102711689)

1.用户发送请求至 前端控制器DispatcherServlet。

2.前端控制器DispatcherServlet收到请求后调用处理器映射器HandlerMapping。

3.处理器映射器HandlerMapping根据请求的Url找到具体的处理器，生成**处理器对象Handler**及**处理器拦截器HandlerIntercepter**（如果有则生成）一并返回给前端控制器DispatcherServlet。

4.前端控制器DispatcherServlet通过处理器适配器HandlerAdapter调用处理器Controller。

5.执行处理器（Controller，也叫后端控制器）

6.处理器Controller执行完后返回ModelAnView。

7.处理器映射器HandlerAdapter将处理器Controller执行返回的结果ModelAndView返回给前端控制器DispatcherServlet。

8.前端控制器DispatcherServlet将ModelAnView传给视图解析器ViewResolver。

9.视图解析器ViewResolver解析后返回具体的视图View。

10.前端控制器DispatcherServlet对视图View进行渲染视图（即：将模型数据填充至视图中）

11.前端控制器DispatcherServlet响应用户。



父子容器特点
父容器和子容器是相互隔离的，他们内部可以存在名称相同的bean
子容器可以访问父容器中的bean，而父容器不能访问子容器中的bean
调用子容器的getBean方法获取bean的时候，会沿着当前容器开始向上面的容器进行查找，直到找到对应的bean为止
子容器中可以通过任何注入方式注入父容器中的bean，而父容器中是无法注入子容器中的bean，原因是第2点

**问题1：springmvc中只使用一个容器是否可以？**

只使用一个容器是可以正常运行的。

**问题2：那么springmvc中为什么需要用到父子容器？**

通常我们使用springmvc的时候，采用3层结构，controller层，service层，dao层；父容器中会包含dao层和service层，而子容器中包含的只有controller层；这2个容器组成了父子容器的关系，controller层通常会注入service层的bean。

采用父子容器可以避免有些人在service层去注入controller层的bean，导致整个依赖层次是比较混乱的。

父容器和子容器的需求也是不一样的，比如父容器中需要有事务的支持，会注入一些支持事务的扩展组件，而子容器中controller完全用不到这些，对这些并不关心，子容器中需要注入一下springmvc相关的bean，而这些bean父容器中同样是不会用到的，也是不关心一些东西，将这些相互不关心的东西隔开，可以有效的避免一些不必要的错误，而父子容器加载的速度也会快一些。 



## spring boot



## spring cloud



多环境切换prod dev



### Feign的实现

Feign = [RestTemplate](https://so.csdn.net/so/search?q=RestTemplate&spm=1001.2101.3001.7020) + Ribbon + Hystrix

Feign是基于HTTP协议

Feign默认整合了Ribbon实现负载均衡

Feign默认整合了Hystrix的熔断机制