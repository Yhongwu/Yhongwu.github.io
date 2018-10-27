---
title: java中的代理模式（Proxy）
categories: 设计模式
tags: 
	- 代理
	- 动态代理
	- cglib
grammar_cjkRuby: true
---


### 代理模式介绍
#### 什么是代理模式
代理模式：为其他对象提供一种代理以控制对这个对象的访问；
代理模式的好处：在目标对象的基础上，去添加额外的功能操作，而不修改原先的业务方法。让业务方法去专注于自己的业务逻辑。代理模式让我们可以去扩展目标对象的功能。
通俗点，就是有个代理人去帮我们处理琐碎的事情。
![enter description here][1]

#### 代理模式的分类
代理模式主要分为:
1、静态代理：由工具或者开发者手动生成代理源代码，并对其进行编译成.class，这种方式，在运行前.class文件就已经存在了。不够灵活。
2、动态代理：在程序运行时，通过运用反射机制动态生成.class。
（1） jdk动态代理：代理所有“实现的有接口”的目标类
（2） cglib动态代理：代理任意一个目标类，但对final类和方法无法代理

#### 静态代理模式的uml图
典型的静态的代理模式uml图如下：
![enter description here][2]
主要的角色：
Subject：抽象主题角色，负责定义RealSubject(目标对象)和Proxy(代理对象)角色应该实现的接口
RealSubject：具体主题角色，即被代理对象，用来真正完成业务服务功能
Proxy：代理主题角色，委托类，负责将自身的Request请求，调用realsubject 对应的request功能来实现业务功能，自己不真正做业务。
### 静态代理
先来定义两个类：UserService和UserServiceImpl

``` java
/**
 * 接口
 * @author hongwu
 *
 */
public interface UserService {
	void save();
}

```
``` java
/**
 * 对象
 * 代理目标对象
 * @author hongwu
 *
 */
public class UserServiceImpl implements UserService{

	@Override
	public void save() {
		System.out.println("保存操作...");
	}

}
```
下面的几个例子都将使用这两个类来举例。
先来看看静态代理的例子：

``` java
package com.howard.demo.proxy.staticproxy;

import com.howard.demo.proxy.UserService;

/**
 * 静态代理
 * @author hongwu
 *
 */
public class UserServiceProxy implements UserService{
	
	private UserService target;
	
	public UserServiceProxy(UserService target) {
		super();
		this.target = target;
	}

	@Override
	public void save() {
		System.out.println("操作前的预处理...");
		target.save();
		System.out.println("操作后的处理...");
		
	}

}

```
``` java
/**
 * 静态代理测试类
 * @author hongwu
 *
 */
public class Test {
	public static void main(String[] args) {
		UserService target = new UserServiceImpl();
		UserServiceProxy proxy = new UserServiceProxy(target);
		proxy.save();
	}
}

```
运行结果：
>操作前的预处理...
>保存操作...
>操作后的处理...

静态代理可以在不修改目标对象代码的前提下，对目标对象功能进拓展，但是，由于静态代理对每一个被代理对象都要编写一个代理类，一旦有很多需要被代理的对象时，就要写很多代理类，所以就有了动态代理的方式。
### 动态代理
动态代理是指在运行时，动态生成代理类。即，代理类的字节码将在运行时生成并载入当前的ClassLoader。动态代理的实现方式主要有两种，一种是使用jdk自带的对于动态代理的支持，一种是使用cglib的方式。
#### jdk对于动态代理的支持
jdk的动态代理是基于接口的。主要使用Proxy的静态方法和创建一个类实现 InvocationHandler接口，通过重写`invoke)()`方法来完成。

``` java
    public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h)
										  
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable
```



主要做了以下几个工作：
1、获取RealSubject上的所有接口；
2、确定要生成的代理类的类名，默认为com.sun.proxy.$ProxyXXX;
3、根据需要实现的接口信息，在代码中动态创建该Proxy类的字节码；
4、将对应的字节码转换为对应的class对象；
5、创建InvocationHandler实例实现handler，处理Proxy的所有方法调用。
6、Proxy的class对象以创建的handler对象为参数，实例化一个proxy对象。
例子：

``` java
/**
 * jdk 动态代理
 * @author hongwu
 *
 */
public class JDKProxy implements InvocationHandler{
	
	//被代理类
	private Object target;
	
	//Object 可以接受不同的代理类
	public Object getInstance(Object target) {
		this.target = target;
		
		//Object java.lang.reflect.Proxy.newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h) 
		//注意这里创建代理对象时传入的第二个参数是代理对象的接口对象 jdk的proxy代理只能代理接口对象，所以被代理类都必须有实现接口
		return Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(), this);
	}

	@Override
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		Object obj = null;
		System.out.println("操作前的预处理...");
		obj = method.invoke(target, args);
		System.out.println("操作后的处理...");
		return obj;
	}

}

```
测试：

``` java
public class Test {
	public static void main(String[] args) {
		//打开这个配置  可以把生成$Proxy0的class文件保存下来
		//System.getProperties().put("sun.misc.ProxyGenerator.saveGeneratedFiles", "true"); 
		
		//只能代理接口对象
		UserService service = (UserService) new JDKProxy().getInstance(new UserServiceImpl());
		System.out.println(service.getClass().getName()); //com.sun.proxy.$Proxy0
		service.save();
	}
}
```
运行后可以看到：
>com.sun.proxy.$Proxy0
操作前的预处理...
保存操作...
操作后的处理...

可以看到，返回的UserService是一个代理对象：$Proxy0
为了可以看看这个代理对象究竟是怎样的一个类，可以打开注释掉的那个配置，是生成的class文件保存到本地：

``` java
System.getProperties().put("sun.misc.ProxyGenerator.saveGeneratedFiles", "true"); 
```
运行后会发现生成的$Proxy0的class文件，使用反编译工具进行查看：

``` java
package com.sun.proxy;

import com.howard.demo.proxy.UserService;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.lang.reflect.UndeclaredThrowableException;

public final class $Proxy0
  extends Proxy
  implements UserService
{
  private static Method m1;
  private static Method m3;
  private static Method m2;
  private static Method m0;
  
  public $Proxy0(InvocationHandler paramInvocationHandler)
    throws 
  {
    super(paramInvocationHandler);
  }
  
  public final boolean equals(Object paramObject)
    throws 
  {
    try
    {
      return ((Boolean)this.h.invoke(this, m1, new Object[] { paramObject })).booleanValue();
    }
    catch (Error|RuntimeException localError)
    {
      throw localError;
    }
    catch (Throwable localThrowable)
    {
      throw new UndeclaredThrowableException(localThrowable);
    }
  }
  
  public final void save()
    throws 
  {
    try
    {
      this.h.invoke(this, m3, null);
      return;
    }
    catch (Error|RuntimeException localError)
    {
      throw localError;
    }
    catch (Throwable localThrowable)
    {
      throw new UndeclaredThrowableException(localThrowable);
    }
  }
  
  public final String toString()
    throws 
  {
    try
    {
      return (String)this.h.invoke(this, m2, null);
    }
    catch (Error|RuntimeException localError)
    {
      throw localError;
    }
    catch (Throwable localThrowable)
    {
      throw new UndeclaredThrowableException(localThrowable);
    }
  }
  
  public final int hashCode()
    throws 
  {
    try
    {
      return ((Integer)this.h.invoke(this, m0, null)).intValue();
    }
    catch (Error|RuntimeException localError)
    {
      throw localError;
    }
    catch (Throwable localThrowable)
    {
      throw new UndeclaredThrowableException(localThrowable);
    }
  }
  
  static
  {
    try
    {
      m1 = Class.forName("java.lang.Object").getMethod("equals", new Class[] { Class.forName("java.lang.Object") });
      m3 = Class.forName("com.howard.demo.proxy.UserService").getMethod("save", new Class[0]);
      m2 = Class.forName("java.lang.Object").getMethod("toString", new Class[0]);
      m0 = Class.forName("java.lang.Object").getMethod("hashCode", new Class[0]);
      return;
    }
    catch (NoSuchMethodException localNoSuchMethodException)
    {
      throw new NoSuchMethodError(localNoSuchMethodException.getMessage());
    }
    catch (ClassNotFoundException localClassNotFoundException)
    {
      throw new NoClassDefFoundError(localClassNotFoundException.getMessage());
    }
  }
}

```
可以看到该类继承自 java.lang.reflect.Proxy，里面所有的方法都是final的，并且方法内部功能的实现都统一调用了InvocationHandler的invoke()方法。所以动态代理与反射是有很大关系的。
但是jdk实现的动态代理有一个缺点，那就是被代理的对象都必须有实现的接口，所以生成的代理类也只能代理该接口定义的方法。而cglib动态代理就不会有这个问题。
#### cglib实现动态代理
cglib(Code Generation Library)实现动态代理是基于ASM框架实现的，需要cglib和asm两个jar包。maven下导入下面的依赖默认asm也会加入：

``` xml
		<!-- cglib -->
		<dependency>
		    <groupId>cglib</groupId>
		    <artifactId>cglib</artifactId>
		    <version>2.2.2</version>
		</dependency>
```
cglib代理，,也叫作子类代理，它是在内存中构建一个子类对象并覆盖其中方法实现增强，从而实现对目标对象功能的扩展.，但因为采用的是继承，所以不能对final修饰的类进行代理。 
CGLIB创建某个类A的动态代理类的模式是：
1、查找A上的所有非final的public类型的方法定义
2、将这些方法的定义转换成字节码
3、将组成的字节码转换成相应的代理的class对象
4、实现MethodInterceptor接口，用来处理对代理类上所有方法的请求（这个接口和Jdk动态代理InvocationHandler的功能和角色是一样的）
cglib动态代理的核心类是：Enhancer，它负责代理对象的生命周期，提供很多方法来帮助我们完成代理实例的创建。其中最主要的两个方法是：

``` java
public void setSuperclass(Class superclass) ；//设置父类，即传入代理对象
//设置了回调。也就是将来对我们代理中方法的访问会转发到该回调中，所有自定义的回调类必须继承MethodInterceptor接口并实现其intercept方法，这一点和jdk的InvocationHandler类似。
public void setCallback(final Callback callback)；
```

MethodInterceptor接口的intercept方法：

``` java
public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable
```
其中，proxy指被代理的对象，method指调用的当前方法，args指方法的参数集合，methodProxy被调用方法的代理，它可以和method完成同样的事情，但是它使用FastClass机制非反射执行方法，效率比较高。
下面看看具体使用的例子：

``` java
/**
 * cglib动态代理
 * @author hongwu
 *
 */
public class CglibProxy implements MethodInterceptor{
	
	private Object target;
	
	public Object getInstance(Object target) {
		this.target = target;
		Enhancer enhancer = new Enhancer();
		enhancer.setSuperclass(target.getClass());
		enhancer.setCallback(this);
		Object proxy = enhancer.create();
		return proxy;
	}
	
	@Override
	public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
		Object obj = null;
		System.out.println("操作前的预处理...");
        obj = method.invoke(target, args);
        System.out.println("操作后的处理...");
        return obj;
	}

}

```
测试：

``` java
public class Test {
	public static void main(String[] args) {
		//可以代理对象
		UserServiceImpl service = (UserServiceImpl) new CglibProxy().getInstance(new UserServiceImpl());
		//获取到的是代理对象 com.howard.test.proxy.UserServiceImpl$$EnhancerByCGLIB$$6aa6fa4f
		System.out.println(service.getClass().getName());
		service.save();
	}
}
```
运行结果：
>com.howard.demo.proxy.UserServiceImpl$$EnhancerByCGLIB$$3bd67ffa
操作前的预处理...
保存操作...
操作后的处理...

sping aop的代理默认是使用jdk的代理方式，如果需要使用cglib，需加上配置：

``` xml
<aop:aspectj-autoproxy proxy-target-class="true">
```
该配置默认是false

[1]: /img/java中的代理模式（proxy）/1.PNG "1"
[2]: /img/java中的代理模式（proxy）/2.PNG "2"

