---
title: 加上事务aop后项目启动报错解决方法参考
categories: 踩过的坑
date: 2016-05-04 10:32:02
tags:
	- j2ee
	- spring
	- aop
	- 异常
---

很多时候在进行ssm或者ssh整合时会遇到各种搞不清的报错

下面就是我在整合过程遇到的错误，费了好一番功夫才知道错在哪

希望对大家有帮助

在整合中的spring配置文件里

各项我都测试没问题，这时我就加上了aop事务管理

```xml
	<!-- 事务管理 -->
	<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource"/>
	</bean>
	
		<!-- 配置事务通知 -->
	<tx:advice id="advice" transaction-manager="transactionManager">
		<tx:attributes>
			<!-- 默认只处理运行时异常，可加rollback-for="Exception/Throwable"等处理所有异常或包括错误 -->
			<tx:method name="insert*" propagation="REQUIRED"
				rollback-for="Exception" />
			<tx:method name="update*" propagation="REQUIRED"
				rollback-for="Exception" />
			<tx:method name="delete*" propagation="REQUIRED"
				rollback-for="Exception" />
			<tx:method name="*" propagation="SUPPORTS" />
		</tx:attributes>
	</tx:advice>
	
	<!-- 配置切面织入的范围,后边要把事务边界定在service层 -->
    <aop:config>
		<aop:advisor advice-ref="advice"
			pointcut="execution(* com.hw.service.Imp.*.*(..))" />
	</aop:config> 
```

但是当下面这段代码加上去的时候就连启动都报了错

```xml
<!-- 配置切面织入的范围,后边要把事务边界定在service层 -->
<aop:config>
	<aop:advisor advice-ref="advice"
		pointcut="execution(* com.hw.service.Imp.*.*(..))" />
</aop:config> 
```
报错如下： 

```java
五月 04, 2016 10:21:48 上午 org.apache.catalina.core.StandardContext listenerStart
严重: Exception sending context initialized event to listener instance of class org.springframework.web.context.ContextLoaderListener
org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'userServiceImp' defined in file [E:\javaee\.metadata\.plugins\org.eclipse.wst.server.core\tmp4\wtpwebapps\scm1.2\WEB-INF\classes\com\hw\service\Imp\UserServiceImp.class]: BeanPostProcessor before instantiation of bean failed; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'org.springframework.aop.support.DefaultBeanFactoryPointcutAdvisor#0': Initialization of bean failed; nested exception is java.lang.NoClassDefFoundError: org/aspectj/weaver/reflect/ReflectionWorld$ReflectionWorldException
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:452)
	at org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:293)
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:222)
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:290)
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:192)
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons(DefaultListableBeanFactory.java:585)
	at org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:895)
	at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:425)
	at org.springframework.web.context.ContextLoader.createWebApplicationContext(ContextLoader.java:282)
	at org.springframework.web.context.ContextLoader.initWebApplicationContext(ContextLoader.java:204)
	at org.springframework.web.context.ContextLoaderListener.contextInitialized(ContextLoaderListener.java:47)
	at org.apache.catalina.core.StandardContext.listenerStart(StandardContext.java:4729)
	at org.apache.catalina.core.StandardContext.startInternal(StandardContext.java:5167)
	at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:150)
	at org.apache.catalina.core.ContainerBase$StartChild.call(ContainerBase.java:1408)
	at org.apache.catalina.core.ContainerBase$StartChild.call(ContainerBase.java:1398)
	at java.util.concurrent.FutureTask.run(FutureTask.java:266)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
	at java.lang.Thread.run(Thread.java:745)
Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'org.springframework.aop.support.DefaultBeanFactoryPointcutAdvisor#0': Initialization of bean failed; nested exception is java.lang.NoClassDefFoundError: org/aspectj/weaver/reflect/ReflectionWorld$ReflectionWorldException
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:527)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:456)
	at org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:293)
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:222)
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:290)
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:196)
	at org.springframework.aop.framework.autoproxy.BeanFactoryAdvisorRetrievalHelper.findAdvisorBeans(BeanFactoryAdvisorRetrievalHelper.java:86)
	at org.springframework.aop.framework.autoproxy.AbstractAdvisorAutoProxyCreator.findCandidateAdvisors(AbstractAdvisorAutoProxyCreator.java:100)
	at org.springframework.aop.aspectj.autoproxy.AspectJAwareAdvisorAutoProxyCreator.shouldSkip(AspectJAwareAdvisorAutoProxyCreator.java:107)
	at org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator.postProcessBeforeInstantiation(AbstractAutoProxyCreator.java:278)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.applyBeanPostProcessorsBeforeInstantiation(AbstractAutowireCapableBeanFactory.java:848)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.resolveBeforeInstantiation(AbstractAutowireCapableBeanFactory.java:820)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:446)
	... 19 more
Caused by: java.lang.NoClassDefFoundError: org/aspectj/weaver/reflect/ReflectionWorld$ReflectionWorldException
	at java.lang.Class.getDeclaredConstructors0(Native Method)
	at java.lang.Class.privateGetDeclaredConstructors(Class.java:2663)
	at java.lang.Class.getDeclaredConstructors(Class.java:2012)
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.determineCandidateConstructors(AutowiredAnnotationBeanPostProcessor.java:230)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.determineConstructorsFromBeanPostProcessors(AbstractAutowireCapableBeanFactory.java:930)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBeanInstance(AbstractAutowireCapableBeanFactory.java:903)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:485)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:456)
	at org.springframework.beans.factory.support.BeanDefinitionValueResolver.resolveInnerBean(BeanDefinitionValueResolver.java:270)
	at org.springframework.beans.factory.support.BeanDefinitionValueResolver.resolveValueIfNecessary(BeanDefinitionValueResolver.java:125)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.applyPropertyValues(AbstractAutowireCapableBeanFactory.java:1325)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean(AbstractAutowireCapableBeanFactory.java:1086)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:517)
	... 31 more
Caused by: java.lang.ClassNotFoundException: org.aspectj.weaver.reflect.ReflectionWorld$ReflectionWorldException
	at org.apache.catalina.loader.WebappClassLoaderBase.loadClass(WebappClassLoaderBase.java:1313)
	at org.apache.catalina.loader.WebappClassLoaderBase.loadClass(WebappClassLoaderBase.java:1164)
	... 44 more
五月 04, 2016 10:21:48 上午 org.apache.catalina.core.StandardContext startInternal
严重: One or more listeners failed to start. Full details will be found in the appropriate container log file
五月 04, 2016 10:21:48 上午 org.apache.catalina.core.StandardContext startInternal
严重: Context [/scm1.2] startup failed due to previous errors
五月 04, 2016 10:21:48 上午 org.apache.catalina.core.ApplicationContext log
信息: Closing Spring root WebApplicationContext
五月 04, 2016 10:21:48 上午 org.apache.catalina.loader.WebappClassLoaderBase clearReferencesJdbc
警告: The web application [scm1.2] registered the JDBC driver [com.mysql.jdbc.Driver] but failed to unregister it when the web application was stopped. To prevent a memory leak, the JDBC Driver has been forcibly unregistered.
五月 04, 2016 10:21:48 上午 org.apache.catalina.loader.WebappClassLoaderBase clearReferencesThreads
警告: The web application [scm1.2] appears to have started a thread named [Timer-0] but has failed to stop it. This is very likely to create a memory leak. Stack trace of thread:
 java.lang.Object.wait(Native Method)
 java.util.TimerThread.mainLoop(Timer.java:552)
 java.util.TimerThread.run(Timer.java:505)
五月 04, 2016 10:21:48 上午 org.apache.catalina.loader.WebappClassLoaderBase clearReferencesThreads
警告: The web application [scm1.2] appears to have started a thread named [com.mchange.v2.async.ThreadPoolAsynchronousRunner$PoolThread-#0] but has failed to stop it. This is very likely to create a memory leak. Stack trace of thread:
 java.lang.Object.wait(Native Method)
 com.mchange.v2.async.ThreadPoolAsynchronousRunner$PoolThread.run(ThreadPoolAsynchronousRunner.java:534)
五月 04, 2016 10:21:48 上午 org.apache.catalina.loader.WebappClassLoaderBase clearReferencesThreads
警告: The web application [scm1.2] appears to have started a thread named [com.mchange.v2.async.ThreadPoolAsynchronousRunner$PoolThread-#1] but has failed to stop it. This is very likely to create a memory leak. Stack trace of thread:
 java.lang.Object.wait(Native Method)
 com.mchange.v2.async.ThreadPoolAsynchronousRunner$PoolThread.run(ThreadPoolAsynchronousRunner.java:534)
五月 04, 2016 10:21:48 上午 org.apache.catalina.loader.WebappClassLoaderBase clearReferencesThreads
警告: The web application [scm1.2] appears to have started a thread named [com.mchange.v2.async.ThreadPoolAsynchronousRunner$PoolThread-#2] but has failed to stop it. This is very likely to create a memory leak. Stack trace of thread:
 java.lang.Object.wait(Native Method)
 com.mchange.v2.async.ThreadPoolAsynchronousRunner$PoolThread.run(ThreadPoolAsynchronousRunner.java:534)
五月 04, 2016 10:21:48 上午 org.apache.catalina.loader.WebappClassLoaderBase clearReferencesThreads
警告: The web application [scm1.2] appears to have started a thread named [Abandoned connection cleanup thread] but has failed to stop it. This is very likely to create a memory leak. Stack trace of thread:
 java.lang.Object.wait(Native Method)
 java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:142)
 java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:158)
 com.mysql.jdbc.NonRegisteringDriver$1.run(NonRegisteringDriver.java:93)
五月 04, 2016 10:21:48 上午 org.apache.coyote.AbstractProtocol start
信息: Starting ProtocolHandler ["http-nio-8080"]
五月 04, 2016 10:21:48 上午 org.apache.coyote.AbstractProtocol start
信息: Starting ProtocolHandler ["ajp-nio-8009"]
五月 04, 2016 10:21:48 上午 org.apache.catalina.startup.Catalina start
信息: Server startup in 17888 ms
```

经过各种查资料：

发现是少了个jar包：

com.springsource.org.aspectj.weaver-1.6.8.RELEASE
有些版本也叫：
aspectj.weaver.jar