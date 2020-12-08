# 什么是Bean

# IoC容器初始化
这是第一步，包括配置解析、beanDefinition加载、注册等。
## 容器初始化入口
包括经典的ClassPathXmlApplicationContext、AnnotationConfigApplicationContext等，类似new ***ApplicationContext的方式，很少直接用于开发环境，一般是通过Spring MVC 中 ServletContext 为 Spring 的 IoC容器提供宿主环境，从宿主环境开始进行容器的初始化工作。初始化，或者说启动容器，实际上指的就是实例化ApplicationContext的这个动作。只是在不同情况下可能有不同的表现形式。
以ClassPathXmlApplicationContext为例，容器初始化主要包括：
1. BeanDefinition 的 Resource 定位
这里的Resource定位 是通过继承ResourceLoader获得的，ResourceLoader代表了加载资源的一种方式，正是策略模式的实现；
2. 从 Resource中解析、载入BeanDefinition；
3. BeanDefinition 在IoC 容器中的注册；

实例化ApplicationContext这个上下文，就是在启动 IoC 容器，从它的构造函数入手：
```java
public ClassPathXmlApplicationContext(String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)
		throws BeansException {

	super(parent);
	setConfigLocations(configLocations);
	if (refresh) {
		refresh();
	}
}
```
入参中的configLocations在这里就是XML配置文件的classpath。setConfigLocations(configLocations)就是把一些带有占位符的地址解析成实际的地址。再之后就是refresh()，我们说的容器初始化，就是在这里面进行的，这里取名为refresh，是因为容器启动之后，再调用refresh()会刷新IoC容器。


# Bean的生命周期
Spring容器中Bean的生命周期由多个特定的阶段组成，每个阶段都允许外界对Bean加以控制。在Spring中可以从两个层面定义Bean的生命周期：Bean的作用范围；实例化Bean时所经理的一系列阶段。

## 文字描述
1. Bean容器在配置文件中找到Spring Bean的定义。
2. Bean容器使用Java Reflection API创建Bean的实例。
3. 如果声明了任何属性，声明的属性会被设置。如果属性本身是Bean，则将对其进行解析和设置。
以上步骤见“spring IoC 源码解析”。

4. 如果Bean类实现BeanNameAware接口，则将通过传递Bean的名称来调用setBeanName()方法。
如果Bean类实现BeanClassLoaderAware接口，则将通过传递加载此Bean的ClassLoader对象的实例来调用setBeanClassLoader()方法。
如果Bean类实现BeanFactoryAware接口，则将通过传递BeanFactory对象的实例来调用setBeanFactory()方法。
如果有任何与BeanFactory关联的BeanPostProcessors对象已加载Bean，则将在设置Bean属性之前调用postProcessBeforeInitialization()方法。
如果Bean类实现了InitializingBean接口，则在设置了配置文件中定义的所有Bean属性后，将调用afterPropertiesSet()方法。
如果配置文件中的Bean定义包含init-method属性，则该属性的值将解析为Bean类中的方法名称，并将调用该方法。
如果为Bean Factory对象附加了任何Bean 后置处理器，则将调用postProcessAfterInitialization()方法。
如果Bean类实现DisposableBean接口，则当Application不再需要Bean引用时，将调用destroy()方法。
如果配置文件中的Bean定义包含destroy-method属性，那么将调用Bean类中的相应方法定义。


## BeanFactory中Bean的生命周期
![Bean生命周期](https://raw.githubusercontent.com/lazy-snail/ImageHosting/master/wiki/Spring-Bean-LifeCycle.png)
具体过程：
1. 当调用者通过getBean(beanName)向容器请求某一个Bean时，如果容器注册了org.springframework.beans.factory.config.InstantiationAwareBeanPostProcessor接口，则在实例化Bean之前，将调用接口的postProcessBeforeInstantiation()；
2. 根据配置情况调用Bean构造方法或工厂方法实例化Bean；
3. 如果容器注册了InstantiationAwareBeanPostProcessor接口，那么在实例化Bean之后，调用该接口的postProcessAfterInstantiation()，可在这里对已经实例化的对象进行一些“梳妆打扮”；
4. 如果Bean配置了属性信息，那么容器在这一步着手将配置值设置到Bean对应的属性中，不过在设置每个属性之前将先调用InstantiationAwareBeanPostProcessor接口的postProcessPropertyValues()；
5. 调用Bean的属性设置方法设置属性值；
6. 如果Bean实现了org.springframework.beans.factory.BeanNameAware接口，则将调用setBeanName()，将配置文件中该Bean对应的名称设置到Bean中；
7. 如果Bean实现了org.springframework.beans.factory.BeanFactoryAware接口，则将调用setBeanFactory()，将BeanFactory容器实例设置到Bean中；
8. 如果BeanFactory装配了org.springframework.beans.factory.config.BeanPostProcessor后处理器，则将调用BeanPostProcessor的Object postProcessBeforeInitialization(Object bean, String beanName)对Bean进行加工操作。其中，入参bean是当前正在处理的bean，beanName时当前bean的配置名，返回的对象为加工处理后的Bean。可以使用该方法对Bean进行处理，甚至改变Bean的行为。BeanPostProcessor在Spring框架中占有重要地位，为容器提供对Bean进行后续加工处理的切入点，Spring容器所提供的各种“神奇功能”，如AOP，动态代理等，都是通过它来实施的；
9. 如果Bean实现了InitializingBean接口，则将调用接口的afterPropertiesSet()；
10. 如果在\<bean>中通过init-method属性定义了初始化方法，则将执行这个方法；
11. BeanPostProcessor后处理器定义了两个方法：其一是postProcessBeforeInitialization()，在(8)步调用；其二是Object postProcessAfterInitialization(Object bean, String beanName)，这个方法在此时调用，容器再次获得对Bean进行加工处理的机会；
12. 如果在\<bean>中指定Bean的作用范围是scope="prototype"，则将Bean返回给调用者，调用者负责Bean后续生命周期的管理，Spring不再管理这个Bean的生命周期。如果将作用范围设置为scope="singleton"，则将Bean放入Spring IoC容器的缓存池中，并将Bean引用返回给调用者，Spring继续对这些Bean进行后续的生命管理；
13. 对于scope="singleton"的Bean（默认情况），当容器关闭时，将触发Spring对Bean后续生命周期的管理工作。如果Bean实现了DisposableBean接口，则将调用接口的destory()，可以在此编写释放资源、记录日志等操作；
14. 对于scope="singleton"的Bean，如果通过\<bean>的destory-method属性指定了Bean的销毁方法，那么Spring将执行Bean的这个方法，完成Bean资源的释放等操作。

Bean的完整生命周期从Spring容器着手实例化Bean开始，直到最终销毁Bean。其中经过了许多关键点，每个关键点都涉及特定的方法调用，可以将这些方法大致分为4类：
* Bean自身的方法：如调用Bean构造方法实例化Bean、调用setter设置Bean的属性值以及通过\<bean>的init-method和destory-method所指定的方法；
* Bean级生命周期接口方法：如BeanNameAware、BeanFactoryAware、InitializingBean和DisposableBean，这些接口方法由Bean类直接实现；
* 容器级生命周期接口方法：图示中⭐标识的步骤是由InstantiationAwareBeanPostProcessor和BeanPostProcessor这两个接口实现的，一般称它们的实现类为“后处理器”。后处理器接口一般不由Bean本身实现，它们独立于Bean，实现类以容器附加装置的形式注册到Spring容器中，并通过接口反射为Spring容器扫描识别。当Spring创建任何Bean的时候，这些后处理器都会发生作用，所以这些后处理器的影响是全局性的。也可以自行编写后处理器，让其仅对感兴趣的Bean进行加工处理；
* 工厂后处理器接口方法：包括AspectJWeavingEnabler、CustomAutowireConfigurer、ConfigurationClassPostProcessor等方法。工厂后处理器也是容器级的，在应用上下文装配配置文件后立即调用。

## ApplicationContext中Bean的生命周期
Bean在应用上下文中的生命周期和在BeanFactory中的生命周期类似，不同的是，如果Bean实现了org.springframework.context.ApplicationContextAware接口，则会增加一个调用接口方法setApplicationContext()的步骤。  
## 作用域
Spring定义了多种作用域：
* 单例（singleton），在整个应用中，只创建bean的一个实例。
* 原型（prototype），每次注入或者通过Spring应用上下文获取的时候，都会创建一个新的bean实例。
* 会话（session），在web应用中，为每个会话创建一个bean实例。
* 请求（request），在web应用中，为每个请求创建一个bean实例。

默认情况下，Spring的所有bean都是单例（singleton）的。使用@Scope注解来选择其它作用域，它可与@Component、@Bean一起使用。两种方式（推荐1）：
1. ```java
    @Component
    @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
    public class AClass{...}
   ```
2. ```java
    @Component
    @Scope("prototype")
    public class AClass{...}

   	@Override
	public Object getBean(String name) throws BeansException {
		return doGetBean(name, null, null, false);
	}
	
   ```
在xxxConfig类中使用@Bean注解声明bean时和上述方式相同。  
会话和请求作用域，使用场景是web应用，从命名理解：会话作用域是指该bean的生命周期和一个会话的起始保持一致，比如典型的电商场景，用户登录，搜索商品，加入购物车，购买，付费，结束，这个流程就是一个完整的会话；请求作用域是指用户从发起一个请求开始，到服务器相应该请求的过程。这两种作用域的bean就是在该场景下，随着会话/请求的生命周期实例化到销毁，典型的，如电商场景中的”购物车”。


```java

protected BeanWrapper createBeanInstance(String beanName, RootBeanDefinition mbd, Object[] args) {
	// Make sure bean class is actually resolved at this point.
	// 解析出 Class
	Class<?> beanClass = resolveBeanClass(mbd, beanName);

	if (beanClass != null && !Modifier.isPublic(beanClass.getModifiers()) && !mbd.isNonPublicAccessAllowed()) {
		throw new BeanCreationException(mbd.getResourceDescription(), beanName,
				"Bean class isn't public, and non-public access not allowed: " + beanClass.getName());
	}

	// 如果工厂方法不为空，则是用工厂方法初始化
	if (mbd.getFactoryMethodName() != null)  {
		// 相关知识点看另一篇文章关于FactoryBean的
		return instantiateUsingFactoryMethod(beanName, mbd, args);
	}

	// Shortcut when re-creating the same bean...
	// 如果不是第一次创建，比如第二次创建 prototype bean。
	// 这种情况下，我们可以从第一次创建知道，采用无参构造函数，还是构造函数依赖注入 来完成实例化
	// 所以注释说叫shortcut
	boolean resolved = false;
	boolean autowireNecessary = false;
	if (args == null) {
		synchronized (mbd.constructorArgumentLock) {
			if (mbd.resolvedConstructorOrFactoryMethod != null) {
				// 有已经解析过的构造方法
				resolved = true;
				autowireNecessary = mbd.constructorArgumentsResolved;
			}
		}
	}
	// 如果已经解析过则使用解析好的构造方法不需要再次锁定
	if (resolved) {
		if (autowireNecessary) {
			// 构造方法自动注入
			return autowireConstructor(beanName, mbd, null, null);
		}
		else {
			// 默认构造方法
			return instantiateBean(beanName, mbd);
		}
	}

	// Need to determine the constructor...
	// 判断是否采用有参构造函数
	// 构造器自动装配
	Constructor<?>[] ctors = determineConstructorsFromBeanPostProcessors(beanClass, beanName);
	if (ctors != null ||
			mbd.getResolvedAutowireMode() == RootBeanDefinition.AUTOWIRE_CONSTRUCTOR ||
			mbd.hasConstructorArgumentValues() || !ObjectUtils.isEmpty(args))  {
		return autowireConstructor(beanName, mbd, ctors, args);
	}

	// No special handling: simply use no-arg constructor.
	// 使用无参构造器
	return instantiateBean(beanName, mbd);
}
```