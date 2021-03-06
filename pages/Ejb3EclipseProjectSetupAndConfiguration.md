---
title: Ejb3EclipseProjectSetupAndConfiguration
---
### Create the Project
* Pull down the **File** menu and select **new:project**
* Select **Java Project**
* Click **Next**
* Enter **<project>**
* Under Project Layout select **create separate source and output folders**
* Click **Finish**
* Select **<project>**, right-click and select **new:Source Folder**
* Enter **conf** for the name
* Click on **Finish**
* Select **<project>**, right-click and select **new:Source Folder**
* Enter **test** for the name
* Click on **Finish**

### Edit your project properties
Now that we have created a user library, we can add that user library to our project:
* Select **<project>**, and press alt-enter or right-click and select properties.
* Select **Java Build Path**
* Select the **Libraries** tab
* Click on **Add Library**
* Select **User Library** and click **Next**
* Click on the box next to **EJB3_EMBEDDABLE** and click **Finish**
* Click **Add Library**
* Select **JUnit** and click **Next**
* In the pull-down list, select **JUnit 4** and click **Finish**
* Click on **OK** to make the change to your project's classpath

### Setup the configuration files
The JBoss Embeddable container looks for several files in the classpath. To keep all of these in one place, we'll add another source directory to our project and then import several files into that directory.
* Select the **conf** folder under **<project>**
* Pull down the **File** menu and select **Import**
* Expand **General**
* Select **File System** and click on **Next**
* Click on **Browse** and go to the following directory: **C:/libs/jboss-EJB-3.0_Embeddable_ALPHA_9/conf**
* Click on **OK**
* You'll see **conf** in the left pane, select it
* Verify that the **Into folder:** lists **<project>/conf** (if not, just enter it or browse to it)
* Click **Finish**
* Expand the **conf** directory and verify that the files are now there

### Add Resource Adapter Archive(RAR)
The Java Connector system defines Resource Adapter Archive files (RAR files). We need to add a few RAR files into the class path. We will import two more files into the **conf** directory:
* Select the **conf** folder
* Pull down the **File** menu and select **Import**
* Expand **General**
* Select **File System** and click on **Next**
* Click on **Browse** and go to the following directory: **C:/libs/jboss-EJB-3.0_Embeddable_ALPHA_9/lib**
* Select **jcainflow.rar** and **jms-ra.rar**
* Click **Finish**

### Create a jndi.properties file 
Note, depending on the version of the embeddable container you download, you might already have a file called **jndi.properties**. If you do, skip to the next section.
* Select the **conf** directory, right-click and select **new** then select **File**
* Enter the name **jndi.properties** and click **finish**
* Enter the following 2 lines then save and close the file:
{% highlight xml %}
java.naming.factory.initial=org.jnp.interfaces.LocalOnlyContextFactory
java.naming.factory.url.pkgs=org.jboss.naming:org.jnp.interfaces
{% endhighlight %}

### Create a persistence.xml
This example presents a utility class we'll be using later. The container needs a **persistence.xml** file to operate. This file must be found under a META-INF directory somewhere in the classpath or the embeddable container will not start. The file's name is **persistence.xml** with a lower-case 'p'. On a Unix system, this will make a difference. On a PC, this won't make a difference and it is one of those things that might work on your machine but not on the linux build box.

* Select your **src** directory
* Right-click, select **New:Folder**
* Enter **META-INF**
* Click **OK**
* Select **META-INF**
* Right-lick, select **New:File**
* Enter **persistence.xml**
* Click **Finish**
* Copy the following example into your new file then save it by pressing ctrl-s

### persistence.xml
{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<persistence>
   <persistence-unit name="custdb">
   
    <!-- This persistence unit uses the default data source that JBoss    -->
    <!-- defines called DefaultDS. If we wanted to use our own data       -->
    <!-- source, we'd need to define a custom data source somewhere.      -->
    <!-- That somewhere is vendor specific.                               -->
    
    <!-- In the case of JBoss, since we're using the embedded container,  -->
    <!-- we need to add two beans in a file called                        -->
    <!-- embedded-jboss-beans.xml. We name the first                      -->
    <!-- HypersonicLocalServerDSBootstrap and we name the second          -->
    <!-- HypersonicLocalServerDS. This two step process defines a data    -->
    <!-- source.                                                          -->
    
    <!-- In the first bean definition, we additionally bind it in Jndi    -->
    <!-- under some name. If we used the name                             -->
    <!-- java:/HypersonicLocalServerDS then we would use the following    -->
    <!-- entry to use that data source instead of the default one:        -->
    <!-- <jta-data-source>java:/HypersonicLocalServerDS</jta-data-source> -->
 
      <jta-data-source>java:/DefaultDS</jta-data-source>
      <properties>
         <property name="hibernate.hbm2ddl.auto" value="create-drop"/>
      </properties>
   </persistence-unit>
</persistence>
{% endhighlight %}
