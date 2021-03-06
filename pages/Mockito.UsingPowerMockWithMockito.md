---
title: Mockito.UsingPowerMockWithMockito
---
{% include toc %}
## Overview
Sometimes there's some low-level code, or badly written code, that makes verifying its behavior difficult. In this example, I'll show a few examples of mixing [PowerMock](https://code.google.com/p/powermock/) in with [Mockito](https://code.google.com/p/mockito/) to take control of:
* static methods
* calls to new

You can get a working copy of this code from my [spring_aop github repo](https://github.com/schuchert/spring_aop).

### Maven Dependencies
This example works with PowerMock 1.5:

**Code from: pom.xml**
{% highlight xml %}
<dependency>
    <groupId>org.powermock</groupId>
    <artifactId>powermock-module-junit4</artifactId>
    <version>1.5</version>
    <scope>test</scope>
</dependency>

<dependency>
    <groupId>org.powermock</groupId>
    <artifactId>powermock-api-mockito</artifactId>
    <version>1.5</version>
    <scope>test</scope>
</dependency>
{% endhighlight %}
            
And Mockito 1.9:
{% highlight xml %}
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-all</artifactId>
    <version>1.9.0</version>
    <scope>test</scope>
</dependency>
{% endhighlight %}

## The Basics
When you want to take control of something not controllable by Mockito, you need to identify the class where you want to take that control. For example, if you are taking control of a static method, you name the class that has the static method. **However**, if you want to change what new X returns,**you name the class that has the call to new**.

### Taking control of a static method
There are a few things you need to do according to the instructions and then one more thing currently (using PowerMock 1.5 and Mockito 1.9) if you happen to be using certain open-source libraries.

### The Annotations
Each test class wanting to use PowerMock needs 2 annotations, with a third optional one for handling current shortcoming (as I see it) in PowerMock:
**Code from: MetricsRecorderTest**

{% highlight java %}
@RunWith(PowerMockRunner.class)
@PowerMockIgnore( {"javax.management.*"})
@PrepareForTest({MetricsRecorder.class, SystemLoggerFactory.class})
public class MetricsRecorderTest {
{% endhighlight %}

|-|-|
| Annotation | Description |
| @RunWith| This is a JUnit 4.x annotation to control the runner used for this particular test. You name a JUnit runner class. Other kinds of JUnit runners you might have come across: Cucumber.class (cucumber-jvm), SpringJUnit4ClassRunner.class (Spring tester), MockitoJUnitRunner.class (Mockito without PowreMock). Note, that the PowerMockRunner gives you a super-set of Mockito runner's features.|
| @PowerMockIgnore | This is an optional annotation. We are using something that, underneath the covers, uses something that conflicts with how PowerMock replaces classes. This is not a problem with JMockIt (also had a problem with Emma and PowerMock leading me to prever JMockIt). If you happen to grab the code and run it, I recommend you comment this out and run the tests to see the exception that results, so you'll be aware of what to expect when it comes up somewhere else.|
| @PrepareForTest | This informs PowerMock that these classes need to be instrumented. This list includes the classes from two different examples (static methods and calls to new). For the purpose of changing the static method, only SystemLoggerFactory.class is required. You can confirm this by running this test with the other classes removed. One test will pass, the others will fail.|

#### Replacing one static method
Next, you need to actually replace the static method. I want to replace the return value of a static method to return a Mockito-created object. Here's the problem code that I want to control:

**Code taken from MetricsRecorder**
{% highlight java %}
SystemLogger logger = SystemLoggerFactory.get(className);
logger.info("start : %s-%s", methodName, correlationId.get());
{% endhighlight %}
The first line calls a static method. I want to control what that returns. Since I am targeting a static method, the code being changed is in the class of the static method. This is important, because the only class that needs special instrumentation is the SystemLoggerFactory class. To actually control the return value:

**Code from: MetricsRecorderTest**
{% highlight java %}
public void replaceLogger() {
  PowerMockito.mockStatic(SystemLoggerFactory.class);
  when(SystemLoggerFactory.get(anyString())).thenReturn(logger);
}
{% endhighlight %}

The first line informs PowerMock of your intention to actually change the class. The next line sets up the staticMethod get(...) to return logger, which is initialized using an @Mock annotation.

### Changing what is returned from new
What happens when you want to change a use of new? In Java, new is a keyword that does 2 things:
* Causes allocation of an object on the heap (at least logically, in moder JVMs this can be heavily optimized)
* Calls the constructor of the class being created

#### Taking Control
Note, you do not directly call the constructor, Java matches the constructor based on the parameters and does the invocation for you. Every place that uses new X does this. That is important, because if you want to take control of new X, you have to tell PowerMock to instrument the**class with the call to new**, not the class you're swapping out:

**Code from: MetricsRecorderTest**
{% highlight java %}
public void replaceCorrelationId() throws Exception {
  PowerMockito.whenNew(CorrelationId.class).withAnyArguments().thenReturn(id);
  when(id.get()).thenReturn("1.1");
}
{% endhighlight %}
This causes calls to new of the class CorrelationId, taking any arguments, to return a controlled value (id, which is Mockito-created object).

#### Annotation to Enable Control
**Code from: MetricsRecorderTest**
{% highlight java %}
@RunWith(PowerMockRunner.class)
@PowerMockIgnore( {"javax.management.*"})
@PrepareForTest({MetricsRecorder.class, SystemLoggerFactory.class})
public class MetricsRecorderTest {
{% endhighlight %}

In this case, the test calls the class MetricsRecorder, which calls new CorrelationId(), so MetricsRecorder is the class added to the @PrepareForTest annotation.

## Summary
It's nice that PowerMock integrates nicely with Mockito. There's a mismatch between using JMockIt and Mockito, but that could be an advantage. Using tools like PowerMock and JMockIt suggest that [maybe your design isn't so great](http://martinfowler.com/articles/modernMockingTools.html).

I had problems with PowerMock not playing nice with emma, a code coverage tool, when executed from Gradle. Emma does instrumentation and it's instrumentation mechanism conflicted with what was going on in JMX. 

The problems involving Emma and having to use the @PowerMockIgnore annotation do not occur with JMockIt. As a result, I prefer JMockIt.

Having said that, we're using PowerMock for my current project and so far we've managed to get around these problems:
* We switched from emma to Jacoco
* @PowerMockIgnore( {"javax.management.*"}) fixed the problem with JMX

The syntax being more integrated with Mockito I actually think of as unfortunate. To be clear, I prefer Mockito's syntax over JMockIt for a number of reasons. However, if I can do something with Mockito, it suggests something that I cannot say if I'm using JMockIt or PowerMock. The different approach (in terms of using anonymous inner classes with instance initialization) stands out, which is a good thing to me.

However, PowerMock seems to be a fine solution and if you prefer more integrated syntax, it wins.

## The Complete Source
### The Test File
{% highlight java %}
package shoe.example.metrics;

import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.Mock;
import org.powermock.api.mockito.PowerMockito;
import org.powermock.core.classloader.annotations.PowerMockIgnore;
import org.powermock.core.classloader.annotations.PrepareForTest;
import org.powermock.modules.junit4.PowerMockRunner;
import shoe.example.log.SystemLogger;
import shoe.example.log.SystemLoggerFactory;
import shoe.example.toggles.TrackMetrics;

import static org.mockito.Mockito.*;

@RunWith(PowerMockRunner.class)
@PowerMockIgnore( {"javax.management.*"})
@PrepareForTest({MetricsRecorder.class, SystemLoggerFactory.class})
public class MetricsRecorderTest {
  @Mock
  SystemLogger logger;

  @Mock
  CorrelationId id;

  @Before
  public void init() throws Exception {
    replaceCorrelationId();
    replaceLogger();
    executeMetricsLogger();
  }

  public void replaceCorrelationId() throws Exception {
    PowerMockito.whenNew(CorrelationId.class).withAnyArguments().thenReturn(id);
    when(id.get()).thenReturn("1.1");
  }

  public void replaceLogger() {
    PowerMockito.mockStatic(SystemLoggerFactory.class);
    when(SystemLoggerFactory.get(anyString())).thenReturn(logger);
  }

  private void executeMetricsLogger() {
    MetricsRecorder recorder = new MetricsRecorder(new TrackMetrics(), "className", "methodName");
    recorder.enter();
    recorder.exit(true);
  }

  @Test
  public void usesCorrelationIdCorrectly() {
    verify(id, times(1)).enter();
    verify(id, times(1)).exit();
  }

  @Test
  public void callsLoggerTwice() {
    verify(logger, times(1)).info(anyString(), anyObject(), anyObject());
    verify(logger, times(1)).info(anyString(), anyObject(), anyObject(), anyObject(), anyObject());
  }
}
{% endhighlight %}

### The Production Code
{% highlight java %}
package shoe.example.metrics;

import com.yammer.metrics.Metrics;
import com.yammer.metrics.core.MetricName;
import com.yammer.metrics.core.Timer;
import com.yammer.metrics.core.TimerContext;
import shoe.example.log.SystemLogger;
import shoe.example.log.SystemLoggerFactory;
import shoe.example.toggles.TrackMetrics;

import java.net.InetAddress;
import java.net.UnknownHostException;

import static java.util.concurrent.TimeUnit.MILLISECONDS;
import static java.util.concurrent.TimeUnit.SECONDS;

public class MetricsRecorder {
  public static final String APP_NAME_PROPERTY = "APPLICATION_NAME";
  public static final String PORT_NAME_PROPERTY = "PORT";

  private TimerContext context;
  private long startTime;
  private long stopTime;
  private final String className;
  private final String methodName;
  private final TrackMetrics trackMetrics;
  private CorrelationId correlationId;

  public MetricsRecorder(TrackMetrics trackMetrics, String className, String methodName) {
    this.className = className;
    this.methodName = methodName;
    this.trackMetrics = trackMetrics;
  }

  public void enter() {
    correlationId = new CorrelationId();
    correlationId.enter();
    startTime = System.currentTimeMillis();

    if (trackMetrics.isEnabled()) {
      MetricName name = new MetricName(group(), className, methodName);
      Timer responses = Metrics.newTimer(name, MILLISECONDS, SECONDS);
      context = responses.time();
    }

    SystemLogger logger = SystemLoggerFactory.get(className);
    logger.info("start : %s-%s", methodName, correlationId.get());
  }

  public void exit(boolean success) {
    if (context != null) {
      context.stop();
    }

    stopTime = System.currentTimeMillis();

    String result = success ? "finish" : "failure";
    SystemLogger logger = SystemLoggerFactory.get(className);
    logger.info("%7s: %s-%s(%dms)", result, methodName, correlationId.get(), duration());
    correlationId.exit();
  }

  private long duration() {
    return stopTime - startTime;
  }

  private String hostName() {
    try {
      return InetAddress.getLocalHost().getHostName();
    } catch (UnknownHostException e) {
      return "UnknownHost";
    }
  }

  private String group() {
    String app = System.getProperty(APP_NAME_PROPERTY);
    String port = System.getProperty(PORT_NAME_PROPERTY);
    return String.format("%s.%s.%s", app, hostName(), port);
  }
}
{% endhighlight %}

