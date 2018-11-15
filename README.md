# Spring

##　切面编程AOP
- 方法级别的AOP框架
- 底层技术：动态代理
> - 切面(Aspect)
> - 切点(Pointcut) 
> - 连接点(Join point)
> - 通知(Advice)
> - 织入(weaving)
> - 引入(Introduction)

#### 基于`@AspectJ`
- 主流
> - 创建切面
```java
@Aspect
public class TestAspect{
    
    /*单独使用@Pointcut指定连接点*/
    @Pointcut("execution(* pro.onlyou.spring.aop.TestAspectDemo.test(..))")
    public void pointcuts(){
        
    }
    
    
    @Before("pointcuts()")
    public void before(){
        System.out.println("before");
    }
    
    @After("execution(* pro.onlyou.spring.aop.TestAspectDemo.test(..))")
    public void after(){
        System.out.println("after");
    }
    @AfterThrowing("execution(* pro.onlyou.spring.aop.TestAspectDemo.test(..))")
    public void afterThrowing(){
        System.out.println("afterThrowing......");
    }
    @AfterReturning("execution(* pro.onlyou.spring.aop.TestAspectDemo.test(..)) && args()")
    public void afterReturning(){
        System.out.println("afterReturning");
    }
    
    @Around("pointcuts()")
    public void around(ProceedingJoinPoint joinPoint){
        /*可以通过ProceedingJoinPoint实现更强大的功能*/
        joinPoint.proceed();
    }
    
}
```
> - 使用`@EnableAspectjAutoProxy`进行自动织入
```java
@Configuration
@EnableAspectjAutoProxy
@ComponentScan
public class AopConfig{
    
    @Bean
    public TestAspect testAspect(){
        return new TestAspect();
    }
    
}
```
> - 使用xml配置自动织入
```xml
<beans>
    <aop:aspectj-autoproxy />
    <bean id="testAspect" class="pro.onlyou.spring.aop.TestAspect"/>
</beans>

```

####　基于xml实现AOP
> xml配置
> - `aop:advisor`:定义通知器,现在很少用
> - `aop:aspect`:
> - `aop:before`:
> - `aop:after`:
> - `aop:around`:
> - `aop:after-returning`:
> - `aop:after-throwing`:
> - `aop:config`:顶层的aop配置元素，aop的配置是以他为开始的
> - `aop:declare-parents`:引入
> - `aop:pointcut`:
```xml
<aop:config>
    <!-- 切面 -->
    <aop:acpect ref="切面bean">
        
        <aop:pointcut id="wa" expression="execution(.....)"/>
    
        <aop:before method="" pointcut="execution()"/>
        <aop:around method="" pointcut-ref="wa"/>
    </aop:acpect>
</aop:config>
```
#### 给切面传递参数
- 基于注解`@Aspect`
> 在建议中的表达式中，可以有一下几种表达式,都是以`&&`或" and "连接的字符串
> > - `execution()`:连接点表达式
> > - `args()`:即可用来传递参数
> > - `within()`:
> > - `this()`:
```
- 基于xml的方式与上类似
@Before("execution(* com.ssm.chapterll.aop.service.impl.RoleServiceimpl.printRole(..))" + "&&" +  args(role,sort"）
```
#### 实现引入（即增加接口增强）
- 基于@Aspect
> - 将以下代码段放入切面中
```java
@DeclareParents(
        value = "pro.onlyou.spring.aop.TestAspectDemo+",
        defaultImpl = TestINtroductionImpl.class
)
private TestIntroduction testIntroduction;
```
- 基于xml
使用`<aop:declare-parents　/>`实现

####　多个切面的执行顺序
使用`@Order`可控制切面执行顺序

## 数据库事物管理
`PlatformTransactionManager`:顶级接口

#### 数据库事务ACID特性
> - 原子性(Atomicity):整个事务的不可分割性，要么全部完成，要么全部放弃
> - 一致性(Consistency):保持系统一致,不会凭空增减
> - 隔离性(Isolation):两个事务之间的相互隔离
> - 持久性(Durability):事务完成提交后，所有更改会被持久保存到数据库中

> - Mybatis -> `DataSourceTransactionManager`
> - Hibernate -> `HibernateTransactionManager`

####　使用Java配置的方式实现事务管理
- 在配置类中实现TransactionManagementConfigurer接口的annotationDrivenTransactionManager方法
```java
@Configuration
@EnableTransactionManagement
@ComponentScan
public class TransactionConfig implements TransactionManagementConfigurer{
    
    @Value("druidDataSource")
    private DataSource dataSource;
    
    @Override
    @Bean(name = "transactionManager")
    public PlatformTransactionManager annotationDrivenTransactionManager(){
        DataSourceTransactionManager transactionManager = 
                    new DataSourceTransactionManager();
        transactionManager.setDataSource(dataSource);
        return transactionManager;
    }
}
```
####　编程式事物（不推荐）
- `TransactionDefinition`
- `TransactionStatus`

###＃　声明式事物
- 基于`@Transactional`
> - `value`:@AliasFor("transactionManager"),设置事物管理器
> - `transactionManager`:同上
> - `isolation`:隔离级别
- - -
> **隔离级别**
- `Isolation`
- spring或者java按照SQL规范将隔离级别分成４个
- 脏读(dirty read):最低的隔离级别，可以读另一个事务未提交的数据
- 读/写提交(read commit):只能读另一个事务已经提交的数据(引发两个事务同时操作一个数据，造成错误)
- 可重复读(repeatable read):克服上一个隔离级别的错误
- 序列化(serializable):克服幻读(phantom read)
> 在spring中，默认隔离级别为`Isolation.DEFAULT`，即根据数据库默认隔离级别，MySQL支持四种,默认可重复读,Oracle支持读/写提交，序列化,默认为读/写提交
- - -
> - `propagation`:事物传播行为
- - - 
> **传播行为**
- `Propagation`
- `REQUIRED`:默认
- `SUPPORTS`
- `MANDATORY`
- `REQUIRES_NEW`
- `NOT_SUPPORTED`
- `NEVER`
- `NESTED`:嵌套事务，回滚自己的
- - - 
> - `timeout`:超时时间
> - `readOnly`:是否开启只读事物
> - `rollbackFor`:
> - `rollbackForClassName`:
> - `noRoolbackFor`:
> - `noRollbackForClassName`:
> - 在xml中
```xml
<tx:annotation-driven transaction-manager="transactionManager"/>
```
- 基于xml(不常用)
> - 配置事物拦截器:`TransactionInterceptor`

- 事物定义器
`TransactionDefinition`

#### `＠Transactional`下自调用事务失效问题
- 事务管理基于AOP
- static或非public方法会导致事务失效
- 自调用:一个类的一个方法去调用另一个方法
> 原因：自调用不会触发动态代理



