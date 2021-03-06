#Spring Boot中的事物管理
###什么是事物？
开发企业应用时，对于业务人员的一个操作实际是对数据读写的多步操作的结合。由于数据操作在顺序执行的过程中，
任何一步操作都有可能发生异常，异常会导致后续操作无法完成，此时由于业务逻辑并未正确的完成，之前操作数据
并不可靠，需要在这种情况下回退(回滚)。

事务的作用就是为了保证每一个操作都是可靠的，事务中的每一步操作都必须成功执行，只要有异常发生就回退到
事务开始未进行操作的状态。

事务管理是Spring框架中最为常用的功能之一，Spring Boot开发应用时，大部分情况也都需要使用事务。
###快速入门
在Spring Boot中，当我们使用了spring-boot-starter-jdbc或spring-boot-starter-data-jpa依赖的时候，
框架会自动默认分别注入DataSourceTransactionManager或JpaTransactionManager。
所以我们不需要任何额外配置就可以用@Transactional注解进行事务的使用。
即在@Transactional注解之后，如果发生异常的时候，事务会实现回退。
为保证测试之间的数据独立，会使用@Rollback注解让每个单元测试都能在结束时回滚。
###事务详解
快速入门只满足了基本的事物需求，但是当我们项目较大较复杂时(多个数据源等)，这时候需要在声明食物时，
指定不同的事务管理器。如@Transactional(value="transactionManagerPrimary")

除了指定不同的事务管理器之后，还能对事务进行隔离级别和传播行为的控制，分别解释：

####隔离级别

隔离级别是指若干个并发的事务之间的隔离程度，与我们开发时候主要相关的场景包括：
脏读取、重复读、幻读。

我们可以看org.springframework.transaction.annotation.Isolation枚举类中定义了五个表示隔离级别的值：
```java
public enum Isolation {
    DEFAULT(-1),
    READ_UNCOMMITTED(1),
    READ_COMMITTED(2),
    REPEATABLE_READ(4),
    SERIALIZABLE(8);
}
```
* DEFAULT：这是默认值，表示使用底层数据库的默认隔离级别。对大部分数据库而言，通常这值就是：READ_COMMITTED。
* READ_UNCOMMITTED：该隔离级别表示一个事务可以读取另一个事务修改但还没有提交的数据。
该级别不能防止脏读和不可重复读，因此很少使用该隔离级别。
* READ_COMMITTED：该隔离级别表示一个事务只能读取另一个事务已经提交的数据。该级别可以防止脏读，这也是大多数情况下的推荐值。
* REPEATABLE_READ：该隔离级别表示一个事务在整个过程中可以多次重复执行某个查询，并且每次返回的记录都相同。
即使在多次查询之间有新增的数据满足该查询，这些新增的记录也会被忽略。该级别可以防止脏读和不可重复读。
* SERIALIZABLE：所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。
但是这将严重影响程序的性能。通常情况下也不会用到该级别。

指定方法：通过使用isolation属性设置，例如:
@Transactional(isolation = Isolation.DEFAULT)
####传播行为
所谓事务的传播行为是指，如果在开始当前事务之前，一个事务上下文已经存在，
此时有若干选项可以指定一个事务性方法的执行行为。

我们可以看org.springframework.transaction.annotation.Propagation枚举类中定义了6个表示传播行为的枚举值：
```java
public enum Propagation {
    REQUIRED(0),
    SUPPORTS(1),
    MANDATORY(2),
    REQUIRES_NEW(3),
    NOT_SUPPORTED(4),
    NEVER(5),
    NESTED(6);
}
```
* REQUIRED：如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。
* SUPPORTS：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
* MANDATORY：如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。
* REQUIRES_NEW：创建一个新的事务，如果当前存在事务，则把当前事务挂起。
* NOT_SUPPORTED：以非事务方式运行，如果当前存在事务，则把当前事务挂起。
* NEVER：以非事务方式运行，如果当前存在事务，则抛出异常。
* NESTED：如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；
如果当前没有事务，则该取值等价于REQUIRED。

指定方法：通过使用propagation属性设置，例如：
@Transactional(propagation = Propagation.REQUIRED)