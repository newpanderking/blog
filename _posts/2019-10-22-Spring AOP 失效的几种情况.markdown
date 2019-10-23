---
layout: post
title:  "Spring AOP 失效的几种情况"
date: 2019-09-24
categories: 技术
tags: spring
description: Spring-AOP-失效的几种情况总结
---

## Spring AOP 代理原理
```
  Spring AOP使用JDK动态代理或者CGLIB来为目标对象创建代理。（建议优先使用JDK的动态代理）
  如果被代理的目标对象实现了至少一个接口，则会使用JDK动态代理。所有该目标类型实现的接口都将被代理。若该目标对象没有实现任何接口，则创建一个CGLIB代理。如果你希望强制使用CGLIB代理，（例如：希望代理目标对象的所有方法，而不只是实现自接口的方法） 那也可以。但是需要考虑以下问题:
  1、无法通知（advise）final方法，因为他们不能被覆写。
  2、代理对象的构造器会被调用两次。因为在CGLIB代理模式下每一个代理对象都会 产生一个子类。每一个代理实例会生成两个对象：实际代理对象和它的一个实现了通知的子类实例。而是用JDK代理时不会出现这样的行为。通常情况下，调用代理类型的构造器两次并不是问题，因为除了会发生指派外没有任何真正的逻辑被实现。
  3、强制使用CGLIB代理需要将<aop:config>的proxy-target-class属性设为true:
```


## 1、private, protected方法无效。
> Spring AOP使用JDK动态代理或者CGLIB来为目标对象创建代理。 

- 1、使用原生的Java的代理是无法代理protected和private类型的方法。
- 2、CGLIB的代理虽然在技术上可以代理protected和private类型的方法，但是用于AOP的时候不推荐代理protected和private类型的方法。
- 参考: https://stackoverflow.com/questions/15093894/aspectj-pointcut-for-annotated-private-methods

## 2、同一个class中public方法无效(inner public)。
> Spring AOP 切面原理，一旦调用通过代理对象触达目标对象调用，目标对象再对于自身方法的调用时是通过`this`引用调用，而不是通过代理调用，因此代理切面将会失效。解决这种问题时，可以将对象自身作为一个对象引用注入到成员变量中，引用时通过成员变量调用，而非直接调用。

```java
@Component
public class Demo{

@Autowired
private Demo demo;

public void method1(){
	method3();
}

public void method2(){
	demo.method3();
}

@aspect
public void method3(){
}

}
```

> 如上代码所示，外部直接调用`method1()`时，通过`this`调用`method3()`时，注解`@aspect`切面不生效；而通过`method2()`调用时，内部通过对象代理调用，注解`@aspect`切面才会生效。
 

## 3、注解写在父类抽象方法上。

|| 自定义注解继承@Inherited | 自定义注解未继承@Inherited |
| --- | --- | --- |
| 子类的类上能否继承到父类的类上的注解 | N | Y |
| 子类方法，实现了父类上的抽象方法，这个方法能否继承到注解 | N | N |
| 子类方法，继承了父类上的方法，这个方法能否继承到注解 | Y | Y |
| 子类方法，覆盖了父类上的方法，这个方法能否继承到注解 | N | N |

- 注解继承的规则如上表格所示，其中，@Inherited注解指明自定义注解是否可以被继承。








