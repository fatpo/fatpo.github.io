* **1、请描述下Spring bean的生命周期**
* **2、Spring bean的有哪几种方式**
* **3、谈谈对IOC和AOP的理解**
* **4、什么是JavaConfig**
* **5、Spring中@Component和@Bean的区别**



## **1、请描述下Spring bean的生命周期**

* 1、解析xml配置或注解配置的类，得到BeanDefinition；
* 2、通过BeanDefinition反射创建Bean对象；
* 3、对Bean对象进行属性的填充；
* 4、回调实现了Aware接口的方法，比如 BeanNameAware、ApplicationContextAware；
    * 注意时间点：`在完成bean配置后，在调用任何Bean生命周期回调方法前。`
    * 举例1：BeanNameAware，让bean对象获取自己在BeanFactory中配置的名字。
    * 举例2：ApplicationContextAware， 某些业务场景下，Bean需要实现某个功能，它要借助于Spring容器才能实现。此时必须让Bean先获取Spring容器，怎么获取：Bean创建时，回调ApplicationContextAware会自动把上下文环境对象 设置到bean对象中。
* 5、调用BeanPostProcessor的初始化前方法；
* 6、调用init初始化方法；
* 7、调用BeanPostProcessor的初始化后方法；
* 8、将创建的bean对象放到一个map中；
* 9、业务使用bean对象；
* 10、spring容器关闭时，调用DisposableBean的 destroy() 方法。

## **2、Spring bean的有哪几种方式**
* @Component和@ComponentScan
* @Bean，自己定制bean的生成规则
* @Import(User.class) 



## **3、谈谈对IOC和AOP的理解**
* 谈在前面：
    * IOC、AOP都不是Spring独有的，比spring更早之前就出现了这2个概念，是一种`思想`。只是Spring把这2个思想很好地执行下去。
* IOC：
    * IoC （Inversion of control ）控制反转/反转控制。它是一种思想不是一个技术实现。描述的是：`Java 开发领域对象的创建以及管理的问题`。
    * 举个栗子：
        * 现有类 A 依赖于类 B，往往是在类 A 中手动通过 new 关键字来 new 一个 B 的对象出来。
        * 控制反转：不通过 new 关键字来创建对象，而是通过 IoC 容器(Spring 框架) 来帮助我们实例化对象。我们需要哪个对象，直接从 IoC 容器里面拿过去即可。
    * **控制怎么理解**：
        * 指的是对象创建（实例化、管理）的权力
    * **反转怎么理解**：
        * 控制权交给外部环境（Spring 框架、IoC 容器）
    * IOC解决了什么问题：
        * 解耦！！！
        * 资源变得容易管理。
    * DI又是什么？
        * DI 是 Dependency Injection， 依赖注入，如果是IOC是思想，DI就是实现方式了。
        * IoC 容器是 Spring 用来实现 IoC 的载体， IoC 容器实际上就是个 Map（key，value）,Map 中存放的是各种对象。怎么把对象加入到IOC容器呢，DI!
* AOP: 
    * AOP是什么？
        * Aspect oriented programming 面向切面编程。算是OOP的延续。
        * OOP面向对象编程，已经可以缩减很多重复代码了，比如Animal和Dog、Cat，我们把公共的代码提取到Animal这个父类中，Dog和Cat子类就专心写业务代码。有一天我们需要监控函数耗时，就在Animal父类中加即可，但如果父类中有多个函数，OOP无能为力，只能一个个函数加过去，每个函数都写同样的代码，这些代码叫做`横切逻辑代码`。
    * AOP是为了解决什么问题？
        * 横切逻辑代码存在的问题：（比如统计函数耗时）
          * 代码重复问题
          * 横切逻辑代码和业务代码混杂在一起，代码臃肿，不变维护
        * AOP怎么解决：
            * 在不改变原有业务逻辑的情况下，增强横切逻辑代码，根本上解耦合，避免横切逻辑代码重复。
    * 举个例子：
        ```
          @Aspect // 声明一个切面
          @Component
          public class MyAspect {
              // 原业务方法执行前
              @Before("execution(public void com.rudecrab.test.service.*.doService())")
              public void methodBefore() {
                  System.out.println("===AspectJ 方法执行前===");
              }
          
              // 原业务方法执行后
              @AfterReturning("execution(* com.rudecrab.test.service..doService(..))")
              public void methodAddAfterReturning() {
                  System.out.println("===AspectJ 方法执行后===");
              }
          }
      ```

## **4、什么是JavaConfig**
* 历史：
    * Java5推出注解
    * Guice框架：号称史上最好用的依赖注入框架Google Guice.
    * Spring社区退出JavaConfig
    * Spring 3.0 JavaConfig代替xml配置

## **5、Spring中@Component和@Bean的区别**
* 作用对象：
    * @Component 作用于类
    * @Bean 作用于方法
* 使用方法：
    * @Component+@ComponentScan 通过注解
    * @Bean 自定义创建一个bean对象返回
* 实现方式：
    * @Component 通常需要通过类路径扫描来自动侦测以及自动装配到spring容器中
    * @Bean 通常需要在标有@Configuration注解的方法中定义产生这个bean，默认情况下，bean的id/name就是方法的名字，当然也能自定义指定。
* 灵活性不同：
    * @Component 对第三方的类无能为力，灵活性不强
    * @Bean 可以灵活注册第三方类库的bean

