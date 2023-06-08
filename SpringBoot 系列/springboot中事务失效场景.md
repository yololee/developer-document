# spring中事务失效场景

## 一、权限访问问题

> 如果方法不是public的就事务不生效

## 二、方法用final修饰

因为spring事务底层实现使用了代理，aop，通过jdk的动态代理或者cglib，生成了代理类，在代理类中实现了事务功能，如果方法被final修饰，无法重写该方法，也就无法添加事务的功能

![在这里插入图片描述](https://img-blog.csdnimg.cn/a7965e13a71a4b96ab1a17c44aaeff15.png#pic_center)


## 三、无事务嵌套有事务的方法

```java
    public void testPrivate() {
        testFinal();
    }

    @Transactional
    public  void testFinal(){
        TbUser build = TbUser.builder().userId(1).userAccount("admin").userPassword("123456")
                .blogUrl("www.baidu.com").remark("欢迎访问").build();
        userMapper.insertUser(build);
        int i = 1 / 0;
    }
```

这里情况下，数据库插入一条数据，数据没有回滚

## 四、没有被spring管理

在使用spring事务的时候，对象要被spring进行管理，也就是需要创建bean，一般我们都会加@Controller、@Service、@Component、@Repository等注解，可以自动实现bean实例化和依赖注入的功能。，如果忘记加了，也会导致，事务的失效

## 五、设计的表不支持事务

也可能是数据库不支持事务，比如 MySQL 使用的 MyISAM 存储引擎

## 六、没有开启事务

springboot项目有自动装配的类DataSourceTransactionManagerAutoConfiguration，已经默认开启了事务，配置spring.datasource参数就行，如果是spring项目，需要在applicationContext.xml配置的，不然事务不会生效

## 七、错误的事务传播

在使用@Transactional注解时，是可以指定propagation参数的

spring目前支持7种传播特性

* REQUIRED 如果当前上下文中存在事务，那么加入该事务，如果不存在事务，创建一个事务，这是默认的传播属性值
* SUPPORTS 如果当前上下文存在事务，则支持事务加入事务，如果不存在事务，则使用非事务的方式执行
* MANDATORY 如果当前上下文中存在事务，否则抛出异常
* REQUIRES_NEW 每次都会新建一个事务，并且同时将上下文中的事务挂起，执行当前新建事务完成以后，上下文事务恢复再执行
* NOT_SUPPORTED 如果当前上下文中存在事务，则挂起当前事务，然后新的方法在没有事务的环境中执行
* NEVER 如果当前上下文中存在事务，则抛出异常，否则在无事务环境上执行代码
* NESTED 如果当前上下文中存在事务，则嵌套事务执行，如果不存在事务，则新建事务

```java
    @Transactional(propagation = Propagation.NEVER)
    public  void testFinal(){
        TbUser build = TbUser.builder().userId(1).userAccount("admin").userPassword("123456")
                .blogUrl("www.baidu.com").remark("欢迎访问").build();
        userMapper.insertUser(build);
        int i = 1 / 0;
    }
```

正常来说这里出现错误，数据库不会插入一条数据，可是这里的事务传播行为是不支持事务的，最终结果就是数据插入了一条数据

> 前支持事务的三种传播特性为：REQUIRED，REQUIRES_NEW，NESTED

## 八、自己捕获了异常

```java
    @Transactional
    public  void testFinal(){

        TbUser build = null;
        try {
            build = TbUser.builder().userId(1).userAccount("admin").userPassword("123456")
                    .blogUrl("www.baidu.com").remark("欢迎访问").build();
            userMapper.insertUser(build);
            int i = 1 / 0;
        } catch (Exception e) {
            System.out.printf(e.toString());
//            throw new RuntimeException(e);
        }
    }
```

写代码的时候我们使用try...catch手动捕获了异常，然后没有手动抛出异常，所以这里事务不生效，数据库中会插入一条数据，如果要让事务生效，必须抛出它能处理的异常，如果没有抛出异常，spring会认为程序没有问题

## 九、手动抛出别的异常

```java
    @Transactional
    public  void testFinal() throws Exception {

        TbUser build = null;
        try {
            build = TbUser.builder().userId(1).userAccount("admin").userPassword("123456")
                    .blogUrl("www.baidu.com").remark("欢迎访问").build();
            userMapper.insertUser(build);
            int i = 1 / 0;
        } catch (Exception e) {
            throw new Exception(e);
//            throw new RuntimeException(e);
        }
    }
```

上面我们捕获了异常，然后手动抛出Exception，事务同样不会回滚，因为spring事务，默认情况下不会回滚Exception（非运行时的异常），只会回滚RuntimeException(运行时异常）和Error（错误）

## 十、自定义回滚异常

rollbackFor 默认是 RuntimeException 和 Error 及子类抛出，就会回滚；如果 rollbackFor 指定了异常类型，那只有指定的异常及子类发生才会回滚

```java
  @Transactional(rollbackFor = BusinessException.class)
  public  void query(Demo demo) throws Exception{
          save(demo);
  }
```

BusinessException 这个就是我们自定义异常，只有当这个异常发生的时候才会回滚