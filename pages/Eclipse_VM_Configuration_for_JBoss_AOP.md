---
title: Eclipse_VM_Configuration_for_JBoss_AOP
---
[[Environment Configuration for JBOSS AOP|<--Back]] [[Eclipse Classpath Variable for JBoss AOP|Next-->]]

Your goal is to configure the Java VM to include one additional parameter:
> -javaagent:/AOP/jboss-aop_1.5.0.GA/lib-50/jboss-aop-jdk50.jar

In Eclipse:
# Pull down **Windows**
# Select **Preferences**
# Select **Java**
# Select **Installed JREs**
# Double-click on **Java 1.5.x JDK** (you'll see the below image)
# In **Default VM Arguments:**
# Adjust the directory to where your version of JBoss AOP is installed

> [[image:JBossAOPJREConfiguration.jpg]]

[[Environment Configuration for JBOSS AOP|<--Back]] [[Eclipse Classpath Variable for JBoss AOP|Next-->]]