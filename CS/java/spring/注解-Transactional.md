# @Transactional 
Spring事务管理分为编码式和声明式两种：编码式指通过编码方式实现事务；声明式事务基于AOP，将具体业务逻辑与事务处理解耦。声明式事务管理使得业务代码逻辑不不受污染，在实际场景中使用得更多。
 
两种声明式事务：配置文件（xml等）和基于@Transactional注解。一般采用注解配置。

## 简单使用
Spring boot默认配置下，只需要在方法上添加@Transactional注解就可以了，以mybatis为例，spring boot会自动配置一个DataSourceTransactionManager，添加注解后就自动纳入Spring的事务管理。  
默认情况下，该注解要注意两个问题：
* Spring只会回滚运行时、未检查异常（继承自RuntimeException的异常）或者Error；
* 只能应用到public方法上才有效。

## 属性
* value、transactionManager：具有相同含义，当配置了多个事务管理器时，可以使用该属性指定事务管理器；

* propagation：事务的传播行为，默认为Propagation.REQUIRED，可选值有：
  * Propagation.REQUIRED：如果当前存在事务，则加入该事务，如果不存在，则创建一个新的事务；
  * Propagation.SUPPORTS：如果当前存在事务，则加入该事务；如果当前不存在事务，则以非事务的方式继续运行；
  * Propagation.MANDATORY：如果当前存在事务，则加入该事务；如果当前不存在事务，则抛出异常；
  * Propagation.REQUIRES_NEW：重新创建一个新的事务，如果当前存在事务，暂停当前的事务；
  * Propagation.NOT_SUPPORTED：以非事务的方式运行，如果当前存在事务，暂停当前的事务；
  * Propagation.NEVER：以非事务的方式运行，如果当前存在事务，则抛出异常；
  * Propagation.NESTED：和Propagation.REQUIRED效果一样。

* isolation：事务的隔离级别，默认为Isolation.DEFAULT，可选值有：
  * Isolation.DEFAULT：使用底层数据库默认的隔离级别；
  * Isolation.READ_UNCOMMITTED；
  * Isolation.READ_COMMITTED；
  * Isolation.REPEATABLE_READ；
  * Isolation.SERIALIZABLE。

* timeout：事务超时时间，默认为-1，即不超时。如果超过该时间限制但事务还没有完成，则自动回滚事务。

* readOnly：指定事务是否为只读事务，默认值为false。为了忽略那些不需要事务的方法，比如读取数据，可以设置read-only为true。

* rollbackFor：用于指定能够触发事务回滚的异常类型，可以指定多个异常类型。

* noRollbackFor：抛出指定的异常类型，不回滚事务，也可以指定多个异常类型。

# spring中的@Transactional
Transactional 可以作用于接口、接口方法、类以及类方法上。当作用于类上时，该类的所有 public 方法将都具有该类型的事务属性，同时也可以在方法级别使用该标注来覆盖类级别的定义。
Spring 建议不要在接口或者接口方法上使用该注解，因为这只有在使用基于接口的代理时它才会生效；并且应该只被应用到 public 方法上，这是由 Spring AOP 的本质决定的。如果在 protected、private 或者默认可见性的方法上使用 @Transactional 注解，这将被忽略，也不会抛出任何异常。
不建议在Dao层使用事务注解，一般在Service实现层。
@Transactional 可以继承，并且使用方法级别优先类级别和最近原则，而且不会将所有的 @Transactional 结合使用。
在使用基于 ORM 的框架时，只读标志基本上毫无用处，在大多数情况下会被忽略。如果使用它，将传播模式设置为SUPPORTS，这样就不会启动事务。