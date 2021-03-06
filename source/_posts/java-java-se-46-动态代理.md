---
title: Java Java SE 46.动态代理
date: 2022-07-05T08:10:09.362Z
tags: [javase]
---
# 1.代理模式【Proxy Pattern】

为什么要有“代理”？生活中就有很多例子，例如委托业务等等，**代理就是被代理者没有能力或者不愿意去完成某件事情，需要找个人代替自己去完成这件事**，这才是“代理”存在的原因。例如，我现在需要出国，但是我不愿意自己去办签证、预定机票和酒店（觉得麻烦 ，那么就可以找旅行社去帮我办，这时候旅行社就是代理，而我自己就是被代理了。

# 2.动态代理概述

动态代理简单来说是：**拦截对真实对象方法的直接访问，增强真实对象方法的功能**

动态代理详细来说是：代理类在程序运行时创建的代理对象被称为动态代理，也就是说，这种情况下，代理类并不是在Java代码中定义的，而是在运行时根据我们在Java代码中的“指示”动态生成的。也就是说你想获取哪个对象的代理，动态代理就会动态的为你生成这个对象的代理对象。**动态代理可以对被代理对象的方法进行增强**，**可以在不修改方法源码的情况下，增强被代理对象方法的功能**，**在方法执行前后做任何你想做的事情**。动态代理技术都是在框架中使用居多，例如：Struts1、Struts2、Spring和Hibernate等后期学的一些主流框架技术中都使用了动态代理技术。

# 3.案例引出

现在，假设我们要实现这样的需求：在企业的大型系统中，每个业务层方法的执行，都需要有对应的日志记录，比如这个方法什么时候调用完成的，耗时多久等信息，根据这些日志信息，我们可以看到系统执行的情况，尤其是在系统出现错误的时候，这些日志信息就显得尤为重要了。现在有一种实现思路是这样的，业务层实现类代码如下：

```java
public interface SchoolService{
     String login(String loginName, String passWord);
   	 String getAllClazzs()；
}


public class SchoolServiceImpl implements SchoolService {

	@Override
	public String login(String loginName, String passWord) {
		
		// 方法执行的开始时间点
		long startTimer = System.currentTimeMillis();
		
		try {
			Thread.sleep(500);
			if("admin".equals(loginName) && "123456".equals(passWord)){
				return "success";
			}
		} catch (Exception e) {
			throw new RuntimeException("登录异常");
		}
		long endTimer = System.currentTimeMillis();
		// 在什么时刻执行完，花费了多长时间完成
		SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
		System.out.println("login方法执行->"+sdf.format(endTimer)+"，耗时："+(endTimer - startTimer));
		
		return "登录名称或者密码不正确";
	}

	@Override
	public String getAllClazzs() {
		// 时间点
		long startTimer = System.currentTimeMillis();
		try {
			Thread.sleep(1000);
			return "返回了所有的班级(1班，2班，3班)";
		} catch (Exception e) {
			throw new RuntimeException("查询班级异常");
		}finally{
			long endTimer = System.currentTimeMillis();
			// 在什么时刻执行完，花费了多长时间完成
			SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
			System.out.println("getAllClazzs方法执行->"+sdf.format(endTimer)+"，耗时："+(endTimer - startTimer));
		}
	}
}
```

我们在业务层的方法每次执行的时候都去记录了开始时间和结束时间，这样虽然可以记录每个方法运行的截止时间和耗时情况，但是代码显得很臃肿，这些日记记录代码本身也是与业务功能无关的代码。

# 4.使用动态代理优化代码

那么有没有一种方式可以解决这个问题呢？

动态代理就是解决此类问题非常好的实现手段，通过动态代理我们可以为该业务层实现类对象提供一个动态的代理对象。该代理对象，可以为实现类的所有方法进行代理，并对代理的方法功能进行增强。也就是说只要调用了被代理实现类对象的方法，该方法的执行会先进入到代理对象中去，代理对象可以在该方法执行前记录开始时间，然后去触发该方法的执行，在方法执行完成以后再由代理对象去记录结束时间然后计算时间差作为日志记录，因为方法的日记记录由代理完成了，所以被代理对象的方法就无需自己单独记录日志操作了。这样就产生了一种非常好的设计模型。

现在我们来使用动态代理写一个日志记录的代理类：

代码如下：

```java
public class LogProxy {
	// 提供一个方法，用于生产需要被代理对象的代理对象。
	public static Object getProxy(Object obj) {
		return Proxy.newProxyInstance(obj.getClass().getClassLoader(), obj
				.getClass().getInterfaces(), new InvocationHandler() {

			@Override
			public Object invoke(Object proxy, Method method, Object[] args)
					throws Throwable {
				// 先记录开始时间点
				long startTimer = System.currentTimeMillis();
				try {
                       // 真正去触发被代理对象中该方法的执行
					return method.invoke(obj, args);
				} catch (Exception e) {
					throw new RuntimeException(e);
				} finally {
					long endTimer = System.currentTimeMillis();
					// 在什么时刻执行完，花费了多长时间完成
					SimpleDateFormat sdf = new SimpleDateFormat(
							"yyyy-MM-dd HH:mm:ss");
					System.out.println(method.getName() + "方法执行->"
							+ sdf.format(endTimer) + "，耗时："
							+ (endTimer - startTimer));
				}
			}
		});
	}

}
```

# 5.重点类和方法

在上述代码中 getProxy 方法即是用于获取某个实现类对象的一个代理对象。在该代码中，如果要了解 Java 动态代理的机制，首先需要了解以下相关的类或接口：

`java.lang.reflect.Proxy`：这是 Java 动态代理机制的主类，它提供了一个静态方法来为一组接口的实现类动态地生成代理类及其对象。

**`newProxyInstance()`参数解析**

该方法用于为指定类装载器、一组接口及调用处理器生成动态代理类实例

`public static Object newProxyInstance(ClassLoader loader, Class[] interfaces, InvocationHandler h)`

1. `obj.getClass().getClassLoader()`目标对象通过getClass方法获取类的所有信息后，调用getClassLoader() 方法来获取类加载器。获取类加载器后，可以通过这个类型的加载器，在程序运行时，将生成的代理类加载到JVM即Java虚拟机中，以便运行时需要！ 
2. `obj.getClass().getInterfaces()`获取被代理类的所有接口信息，以便于生成的代理类可以具有代理类接口中的所有方法。
3. `InvocationHandler` 这是调用处理器接口，它自定义了一个 invoke 方法，用于集中处理在动态代理类对象上的方法调用，通常在该方法中实现对委托类方法的处理以及访问.

**invoke方法的参数**

`public Object invoke(Object proxy, Method method, Object[] args)`

1. Object proxy生成的代理对象，在这里不是特别的理解这个对象，但是个人认为是已经在内存中生成的proxy对象。
2. Method method：被代理的对象中被代理的方法的一个抽象。
3. Object[] args：被代理方法中的参数。这里因为参数个数不定，所以用一个对象数组来表示。



代理类定义完成以后业务层实现类的方法就无需再自己声明日记记录的代码了，因为代理对象会帮助做日志记录，修改后实现类代码如下：

```java
public class SchoolServiceImpl implements SchoolService {

	@Override
	public String login(String loginName, String passWord) {
		try {
			Thread.sleep(500);
			if("admin".equals(loginName) && "123456".equals(passWord)){
				return "success";
			}
		} catch (Exception e) {
			throw new RuntimeException("登录异常");
		}
		return "登录名称或者密码不正确";
	}

	@Override
	public String getAllClazzs() {
		try {
			Thread.sleep(1000);
			return "返回了所有的班级(1班，2班，3班)";
		} catch (Exception e) {
			throw new RuntimeException("查询班级异常");
		}
	}
}
```

开始使用动态代理去访问方法：

```java
public class TestMain {
	public static void main(String[] args) {
		// 获取业务层实现类对象的代理对象，那么业务层实现类对象就会被代理了
		SchoolService schoolService = (SchoolService) LogProxy.getProxy(new SchoolServiceImpl());
		
		System.out.println(schoolService.login("admin", "1234256"));
		
		System.out.println(schoolService.getAllClazzs());
	
	}
}
```

此代码中业务层对象已经是被代理的了，那么以后调用业务层对象的方法时，方法的调用会先被代理对象处理，代理会先记录方法执行的开始时间，然后通过`method.invoke(obj, args)`去真正触发该方法的执行，接下来代理对象进行方法结束时间的记录和日志的输出即可。这样整个过程就通过代理完美的实现了。

# 6.总结

动态代理非常的灵活，可以为任意的接口实现类对象做代理

动态代理可以为被代理对象的所有接口的所有方法做代理，动态代理可以在不改变方法源码的情况下，实现对方法功能的增强，

动态代理类的字节码在程序运行时由Java反射机制动态生成，无需程序员手工编写它的源代码。 

动态代理类不仅简化了编程工作，而且提高了软件系统的可扩展性，因为Java 反射机制可以生成任意类型的动态代理类。

动态代理同时也提高了开发效率。

缺点：只能针对接口的实现类做代理对象，普通类是不能做代理对象的。