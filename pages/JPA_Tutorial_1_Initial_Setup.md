---
title: JPA_Tutorial_1_Initial_Setup
---
This example requires Java 5 (JDK 1.5) or later and Eclipse 3.2 or later. This page gives you a link to all of the downloads you'll need to get to get started. While I might mention specific version numbers, it's a good bet that newer versions should work as well... of course that's not always the case.

**Note:** We need the jar file that contains javax.persistence and the various annotations associated with JPA. You can either download JEE or get it from somewhere else. For this series of tutorials, we eventually use the JBoss EJB3 Embeddable container, so we'll use that to avoid an extra 150+ meg download for one jar file.

### Download Everything
First the basic stuff:
* Download [JSE 5](http://java.sun.com/javase/downloads/index_jdk5.jsp) (choose just JDK 5.0 (there will be an update after it))
* Download [Eclipse](http://www.eclipse.org/downloads/)

Next, the libraries:
* Download [Hibernate 3.3.1 GA](http://sourceforge.net/project/showfiles.php?group_id=40712&package_id=127784&release_id=625684)
* Download [Hibernate Annotations 3.4.0 GA](http://sourceforge.net/project/showfiles.php?group_id=40712&package_id=139933)
* Download [Hibernate Entity Manager 3.4.0.GA](http://sourceforge.net/project/showfiles.php?group_id=40712&package_id=156160)
* Download [HSQL Database Engine 1.8.0.10](http://sourceforge.net/project/platformdownload.php?group_id=23316&sel_platform=14519)
* Download [SLF4J 1.5.8](http://www.slf4j.org/download.html)
* Download [Open EJB 3.1.1](http://openejb.apache.org/download.html)

### Setup the JDK & Eclipse
* Install the JSE 5.
* Extract the eclipse download somewhere. For all examples I use C:/eclipse.

### Extract Jar Files
Extract each of the libraries to some location. In my case I extracted everything to C:/libs, so I have the following directories
{% highlight terminal %}
> C:/libs/hibernate-annotations-3.4.0.GA
> C:/libs/hibernate-entitymanager-3.4.0.GA
> C:/libs/hibernate-distribution-3.3.1.GA
> C:/libs/hsqldb-1.9.0-beta3
> C:/libs/openejb-3.1.1
> C:/libs/slf4j-1.5.8
{% endhighlight %}
