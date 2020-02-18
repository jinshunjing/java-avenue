

## Spring 5 新特性
### 响应式编程


## 问题
- Spring IOC/AOP/MVC的工作原理
- Bean的加载过程，循环依赖如何解决
- A和B如何按顺序加载？Order(1)


## 循环依赖
- 构造方法不能循环依赖
- prototype不能循环依赖

## singleton循环依赖
- 三级缓存：singletonObjects, earlySingletonObjects, singletonFactories
- singletonObjects: 存放完全初始化好的bean
- earlySingletonObjects: 存放原始的bean，尚未填充属性
- singletonFactories: 存放bean工厂对象

## singleton加载过程
- createBeanInstance 实例化bean，并且把BeanFactory放入singletonFactories里面
- populateBean 填充属性
- initializeBean 初始化



## Spring AOP
- AspectJ
- Javasist
- JDK proxy
- CGLIB
- Runtime aspect weaving

