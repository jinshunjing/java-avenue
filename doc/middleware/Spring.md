# Spring

## 面试问题
- Spring IOC/AOP/MVC的工作原理
- Bean的加载过程，循环依赖如何解决
- A和B如何按顺序加载？Order(1)


## IoC
- 1996年，Michael Mattson提出IoC控制反转，解决对象之间耦合度过高的问题
- 2004年，Martin Fowler提出DI依赖注入，获得依赖对象的过程被反转了
- 通过引入IoC容器，利于依赖注入的方式，实现对象之间的解耦
- 通过Java反射机制来实现，在运行期，实例化对象，注入依赖，执行初始化

### singleton加载过程
- createBeanInstance 实例化bean，并且把BeanFactory放入singletonFactories里面
- populateBean 填充属性
- initializeBean 初始化

### 循环依赖
- 构造方法不能循环依赖
- prototype不能循环依赖

### singleton循环依赖
- 三级缓存：singletonObjects, earlySingletonObjects, singletonFactories
- singletonObjects: 存放完全初始化好的bean
- earlySingletonObjects: 存放原始的bean，尚未填充属性
- singletonFactories: 存放bean工厂对象


## AOP
- AspectJ
- Javasist
- Runtime aspect weaving

### Java代理
- Java代理：通过反射和InvocationHandler回调接口实现 Proxy.newProxyInstance(ClassLoader, Class[], InvocationHandler) 
1. 生成并加载代理类：ProxyGenerator.generateProxyClass()，继承Proxy，实现接口，所有的方法都要通过InvocationHandler
2. 获取带有InvocationHandler参数的构造方法
3. 创建代理实例


### CGLIB
- Java代理只能代理接口，CGLIB可以代理实现类，不能代理声明为final的方法。使用字节码生成框架。通过拦截器织入横切逻辑。
- 拦截器：MethodInterceptor.intercept(Object, Method, Object[], MethodProxy)
- 调用被代理对象的方法：MethodProxy.invokeSuper(Method, Object[])
- 创建代理对象：Enhancer.setSuper(Class), Enhancer.setCallback(MethodInterceptor), Enhancer.create()

### Javassit
- 动态代码创建，生成字节码
- 能代理实现类



## Spring 5 新特性
### 响应式编程




