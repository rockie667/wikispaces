---
title: Eclipse_VM_Configuration_for_JBoss_AOP
---
{% include nav prev="Environment_Configuration_for_JBOSS_AOP" next="Eclipse_Classpath_Variable_for_JBoss_AOP" %}

Your goal is to configure the Java VM to include one additional parameter:
{% highlight terminal %}
-javaagent:/AOP/jboss-aop_1.5.0.GA/lib-50/jboss-aop-jdk50.jar
{% endhighlight %}

In Eclipse:
* Pull down **Windows**
* Select **Preferences**
* Select **Java**
* Select **Installed JREs**
* Double-click on **Java 1.5.x JDK** (you'll see the below image)
* In **Default VM Arguments:**
* Adjust the directory to where your version of JBoss AOP is installed

![](../images/JBossAOPJREConfiguration.jpg)

{% include nav prev="Environment_Configuration_for_JBOSS_AOP" next="Eclipse_Classpath_Variable_for_JBoss_AOP" %}
