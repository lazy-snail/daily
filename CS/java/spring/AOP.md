# AOP
AOP是一种编程思想。  
日志、安全、事务管理、缓存等，在软件系统中都是非常重要的功能，但它们与软件本身所关注的“功能”即业务逻辑，从概念上讲（应该）是分离的，然而它们散布嵌入在业务逻辑之中，需要在业务逻辑功能执行的过程中被动地触发。这些功能通常被称为横切关注点（cross-cutting concern）。把这些横切关注点与业务逻辑相分离就是面向切面编程（AOP，Aspect Oriented Programming），实现横切关注点与它们所影响的对象之间的解耦。  
重用通用功能的方案一般为继承或委托，而切面是另一种实现该目标的方案。在使用面向切面编程时，仍然在一个地方定义通用功能，但是可以通过声明的方式定义这个功能要以合何种方式在何处应用，而无需修改受影响的类。横切关注点可以被模块化为特殊的类，这些类被称为切面（aspect）。有两个好处：
1. 每个关注点都集中在一个地方，而不是分散在多处代码中；
2. 服务模块更加简洁，因为它们都只包含主要关注点（核心功能）的代码，而次要关注的代码被转移到切面中。

## 相关术语
### 增强 advice
切面的工作被称为增强。  
增强定义了切面是什么以及何时使用（what and when）。除了描述切面要完成的工作，增强还解决了何时执行这个工作的问题：它应该应用在某个方法被调用之前、之后还是只在方法抛出异常时，等。  
Spring切面支持5种类型的增强：
1. 前置增强（Before）：在目标方法被调用之前调用增强功能；
2. 后置增强（After）：在目标方法完成之后调用增强，此时不会关心方法的输出是什么；
3. 环绕增强（Around）：在被增强方法调用前后都执行自定义的行为；
4. 返回增强（After-returning）：在目标方法成功执行之后调用增强；
5. 异常增强（After-throwing）：在目标方法抛出异常之后调用增强。

### 连接点 join point
应用增强的时机，被称为连接点。连接点是在应用执行过程中能够插入切面的一个点，这个点可以是调用方法时、抛出异常时、甚至修改一个字段时，切面代码可以利用这些点插入到应用的正常流程之中，并添加新的行为。

### 切点 pointcut
增强定义了切面的“what”和“when”，切点定义了切面的“where”，切点的定义会匹配增强所要织入的一个或多个连接点。通常使用明确的类和方法名，或利用正则表达式定义所匹配的类和方法名来指定这些切点。

### 切面 aspect
切面是增强和切点的结合。增强和切点共同定义了切面的全部内容——它是什么，在何时何处完成其功能。 

### 引入 introduction
引入允许向现有的类添加新方法或属性，从而在无需修改现有类的情况下，让这些类具有新的行为和状态。是一种特殊的增强。

### 织入 weaving
把切面应用到目标对象并创建新的代理对象的过程。切面在指定的连接点被织入到目标对象中，在目标对象的生命周期里有多个点可以进行织入：
* 编译期：切面在目标类编译时被织入。这种方式需要特殊的编译器。AspectJ的织入编译器就是以这种方式织入切面的。
* 类加载期：切面在目标类加载到JVM时被织入。这种方式需要特殊的类加载器，它可以在目标类被引入应用之前增强该目标类的字节码。AspectJ 5的加载时织入（load-time weaving，LTW）就支持以这种方式织入切面。
* 运行期：切面在应用运行的某个时刻被织入。一般，在织入切面时，AOP容器会为目标对象动态地创建一个代理对象。Spring AOP就是以这种方式织入切面的。

## Spring种的AOP
AOP框架在连接点模型上有强弱之分，比如有些允许在字段修饰符级别应用增强，有些只支持与方法调用相关的连接点。此外，框架织入切面的方式和时机也有所不同。但无论如何，**创建切点来定义切面所织入的连接点是AOP框架的基本功能。**

Spring提供4种类型的AOP支持：
* 基于代理的经典Spring AOP（现在来看笨重且复杂）；
* 纯POJO切面（需要xml配置）；
* @AspectJ注解驱动的切面（本质上依然是基于代理的AOP）；
* 注入式AspectJ切面（基本的方法调用级别切面满足不了需求时）。

前三种都是Spring AOP实现的变体。Spring AOP构建在**动态代理**基础之上，因此，Spring对AOP的支持局限于方法拦截。

## 运行时增强对象
通过在代理类中包裹切面，Spring在运行期把切面织入到Spring管理的bean中。代理类封装了目标类，拦截被增强方法的调用，再把调用转发给真正的目标bean。即代理类处理方法的调用，执行额外的切面逻辑，并调用目标方法。  
![](https://img-blog.csdn.net/20160505123413155)  
Spring运行时才创建代理对象。

## 切点类型
Spring提供6种类型的切点：
* 静态方法切点：org.springframework.aop.support.StaticMethodMatcherPointcut是静态方法切点的抽象基类，默认情况下它可以匹配所有的类。包含两个主要的子类：NameMatchMethodPointcut & AbstractRegexpMethodPointcut，前者提供简单的字符串匹配方法签名，后者使用正则表达式匹配方法签名；
* 动态方法切点：org.springframework.aop.support.DynamicMethodMatcherPointcut是动态方法切点的抽象基类，默认情况下匹配所有的类；
* 注解切点：org.springframework.aop.support.annotation.AnnotationMatchingPointcut的实现类表示注解切点，支持在Bean中直接通过java5.0注解标签定义的切点；
* 表达式切点：org.springframework.aop.support.ExpressionPointcut接口主要是为了支持AspectJ切点表达式语法而定义的接口；
* 流程切点：org.springframework.aop.support.ControlFlowPointcut实现类表示控制流程切点。ControlFlowPointcut是一类特殊的切点，它根据程序执行堆栈的信息查看目标方法