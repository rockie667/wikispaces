---
title: JAR_Files_for_JBoss_AOP
---
{% include nav prev="Environment_Configuration_for_JBOSS_AOP" %}

Every project using JBoss AOP needs several JAR files. You defined a classpath variable in [Eclipse_Classpath_Variable_for_JBoss_AOP](Eclipse_Classpath_Variable_for_JBoss_AOP). We now need to use that classpath variable:

* Select your project, right-click and select **Properties** (Alt-Enter on PC)
* Select **Java Build Path**
* Select the **Libraries** tab
* Click on **Add Variable**
* Select the variable you created (called JBOSS_AOP_LIB)
* Click **Extend**
* Select all JAR files listed (Ctrl-A on PC)
* Click **OK**
* Click **OK**

You'll need to do this for each project you create. If you don't you'll see a long stack trace that resembles the following:
{% highlight terminal %}
java.lang.reflect.InvocationTargetException
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:39)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:25)
	at java.lang.reflect.Method.invoke(Method.java:585)
	at sun.instrument.InstrumentationImpl.loadClassAndCallPremain(InstrumentationImpl.java:141)
Caused by: java.lang.NoClassDefFoundError: javassist/ClassPool
	at java.lang.ClassLoader.defineClass1(Native Method)
	at java.lang.ClassLoader.defineClass(ClassLoader.java:620)FATAL ERROR in native method: processing of -javaagent failed

	at java.security.SecureClassLoader.defineClass(SecureClassLoader.java:124)
	at java.net.URLClassLoader.defineClass(URLClassLoader.java:260)
	at java.net.URLClassLoader.access$100(URLClassLoader.java:56)
	at java.net.URLClassLoader$1.run(URLClassLoader.java:195)
	at java.security.AccessController.doPrivileged(Native Method)
	at java.net.URLClassLoader.findClass(URLClassLoader.java:188)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:306)
	at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:268)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:251)
	at java.lang.ClassLoader.loadClassInternal(ClassLoader.java:319)
	at org.jboss.aop.standalone.Agent.premain(Agent.java:41)
	... 5 more
Exception in thread "main" 
{% endhighlight %}
{% include nav prev="Environment_Configuration_for_JBOSS_AOP" %}
