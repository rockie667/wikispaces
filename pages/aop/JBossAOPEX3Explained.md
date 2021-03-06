---
title: JBossAOPEX3Explained
---
{% include nav prev="JBossAOPEX3SoWhatIsHappening" next="JBossAOPEX3ApplyYourself" %}

## Example 3 Explained
In this example, we add Introductions (adding an interface + implementation of that interface to an existing class) and field setting as well as method execution. In the Field Setting example we simply reported existing and new values. Now we use that information and track whether an object has changed.

First, we introduce an interface, ITrackedObject, to Address. The implementation of this interface provides a single boolean field, changed, and the methods to maintain that field. Then, as fields are set, we check the existing value and and the new value. If there is a change, we set the changed state. Finally, we have a simulated Dao (Data Access Object) that can save the Address object. We intercept calls to the save() method and do not actually save the Dao if it is changed. We also change the state back to changed=false after a call to Dao.save().

### Main.java
This is the driver class that starts everything. Looking at this code, it does not seem to do too much...and it doesn't. 
{% highlight java %}
01: package ex3;
02: 
03: public class Main {
04: 	public static void main(String args[]) {
05: 		Address a = new Address();
06: 		Dao.save(a);
07: 		a.setZip(null);
08: 		Dao.save(a);
09: 		a.setZip("75001");
10: 		Dao.save(a);
11: 		Dao.save(a);
12: 	}
13: }
{% endhighlight %}
#### Interesting Lines
There are no interesting lines here. Something to point out about this example is the significant change that happens without making changes to existing Java classes.

### Dao.java
This Dao is simulated. The point of this example is that we can intercept calls to some thing, a DAO in this case, and change the path of execution based on any condition. This class is unaware of any introductions. 
{% highlight java %}
01: package ex3;
02: 
03: public class Dao {
04: 	public static void save(Object o) {
05: 		if (o != null) {
06: 			System.out.printf("Saving: %s\n", o.getClass());
07: 		}
08: 	}
09: }
{% endhighlight %}

#### Interesting Lines
Again, there are no interesting lines. The client, Main.main(), did not change. The Dao.save() method also did not change. However, we are tracking whether or not objects changed and not calling Dao.save() if the object is not changed.

### Address.java
The thing to notice is that it is unaware of whether it is changed or not. This is a simple java bean style class with attributes, setters and getters and a no-argument constructor (in this case a default constructor). 
{% highlight java %}
01: package ex3;
02: 
03: import java.io.Serializable;
04: 
05: @SuppressWarnings("serial")
06: public class Address implements Serializable {
07: 	private String addressLine1;
08: 	private String addressLine2;
09: 	private String city;
10: 	private String state;
11: 	private String zip;
12: 
13: 	public String getAddressLine1() {
14: 		return addressLine1;
15: 	}
16: 
17: 	public void setAddressLine1(String addressLine1) {
18: 		this.addressLine1 = addressLine1;
19: 	}
20: 
21: 	public String getAddressLine2() {
22: 		return addressLine2;
23: 	}
24: 
25: 	public void setAddressLine2(String addressLine2) {
26: 		this.addressLine2 = addressLine2;
27: 	}
28: 
29: 	public String getCity() {
30: 		return city;
31: 	}
32: 
33: 	public void setCity(String city) {
34: 		this.city = city;
35: 	}
36: 
37: 	public String getState() {
38: 		return state;
39: 	}
40: 
41: 	public void setState(String state) {
42: 		this.state = state;
43: 	}
44: 
45: 	public String getZip() {
46: 		return zip;
47: 	}
48: 
49: 	public void setZip(String zip) {
50: 		this.zip = zip;
51: 	}
54: }
{% endhighlight %}
#### Interesting Lines
Again, doesn't really apply. However, it *is* interesting that we know whether or not this object has changed even though looking at the class it's hard to see how.

### The Introduction
This is only part of the jboss-aop.xml file. It's the part that says to add an interface, ITrackedObject, to Address and provides an implementation of that interface in the form of TrackedObjectMixin: 
{% highlight xml %}
01: <introduction class="ex3.Address">
02: 	<mixin>
03: 		<interfaces>
04: 			ex3.ITrackedObject
05: 		</interfaces>
06: 		<class>ex3.TrackedObjectMixin</class>
07: 		<construction>
08: 			new ex3.TrackedObjectMixin()
09: 		</construction>
10: 	</mixin>
11: </introduction>
{% endhighlight %}
#### Interesting Lines

|Line|Description|
|1|Indicates to which classes this introduction applies. Wild-cards are OK.|
|2 - 10|Add this mixin into the Address class. This mixin consists of three parts, an interface to add to Address, an implementation of that interface and a constructor to call for that mixin class.|
|4|ITrackedObject is the interface to add to Address.|
|6|TrackedObjectMixin implements ITrackedObject and will provide the implementation for Address. Essentially, all of the methods described in ITrackedObject will be added to Address. The implementation of those methods will be to call a the same method on instance variable added to an address.|
|8|When creating a new instance of Address, call this constructor on TrackedObjectMixin.|

### ITrackedObject.java
This is simply an interface that has the methods for a Java-bean style boolean interface.
{% highlight java %}
01 package ex3;
02 
03 public interface ITrackedObject {
04    boolean isChanged();
05    void setChanged(boolean changed);
06 }
{% endhighlight %}

### TrackedObjectMixin.java
This is an implementation of ITrackedObject. Our goal is to augment Address with this interface/implementation without Address' knowledge. Furthermore, we want to augment Dao.save(..) to not save unnecessarily; we do this without its knowledge as well.
{% highlight java %}
01 package ex3;
02 
03 public class TrackedObjectMixin implements ITrackedObject {
04    private boolean changed = false;
05 
06    public TrackedObjectMixin() {
07    }
08 
09    public boolean isChanged() {
10       return changed;
11    }
12 
13    public void setChanged(boolean changed) {
14       this.changed = changed;
15    }
16 }
{% endhighlight %}

{: #SetInterceptor}
### SetInterceptor.java
The SetInterceptor first intercepts each assignment to a field (as we have seen in other examples) but now it gets the current value and checks to see if the new value is different. If the new value is different, the SetInterceptor sets the underlying target object's changed state to true. There are three things worth noting:
* This interceptor assumes the underlying target object is in fact an ITrackedObject. We know it is because of the jboss-aop.xml, but we could check here if we wanted. 
* The underlying object is an instance of Address, which does not implement ITrackedObject directly. That interface and its implementation, TrackedObjectMixin, were introduced with the snippet of jboss-aop.xml shown above. 
* We could check the state first and if it is changed, we could avoid some reflection. This is a viable optimization but is left as an exercise.
{% highlight java %}
01 package ex3;
02 
03 import java.lang.reflect.Field;
04 
05 import org.jboss.aop.advice.Interceptor;
06 import org.jboss.aop.joinpoint.FieldWriteInvocation;
07 import org.jboss.aop.joinpoint.Invocation;
08 
09 public class SetInterceptor implements Interceptor {
00    public String getName() {
11       return "SetInterceptor";
12    }
13 
14    public Object invoke(Invocation invocation) throws Throwable {
15       if (!(invocation instanceof FieldWriteInvocation))
16          return invocation.invokeNext();
17 
18       FieldWriteInvocation mi = (FieldWriteInvocation) invocation;
19       Object newValue = mi.getValue();
20       Object target = mi.getTargetObject();
21       Field field = mi.getField();
22       Object currentValue = field.get(target);
23 
24       if (equals(currentValue, newValue)) {
25          return null;
26       } else {
27          ((ITrackedObject) target).setChanged(true);
28          return invocation.invokeNext();
39       }
30    }
31 
32    private boolean equals(Object lhs, Object rhs) {
33       if (lhs == null && rhs == null) {
34          return true;
35       }
36       if (lhs == null && rhs != null) {
37          return false;
38       }
39       if (lhs != null && rhs == null) {
40          return false;
41       }
42       return lhs.equals(rhs);
43    }
44 }
{% endhighlight %}
#### Interesting Lines

|Line|Description|
|15|If this is not a set (FieldWriteInvocation), just continue what you were doing.|
|18 – 19|Get the new value (what appears on the Right Hand Side of "=").|
|20 – 22|Get the current value of the field (what appears on the Left Hand Side of "=").|
|25|If the values are already equal, do not perform the assignment and return.|
|27 – 28|The values are different, set changed to true on the introduced mixin then perform the assignment.|

### SaveMethodInterceptor.java
The SaveMethodInterceptor surrounds all calls to Dao.save(..). When called, it checks to see if the object passed into Dao.save(..) is or is not changed. If it is not changed, the call to Dao.save(..) never happens. Before completing, the changed state is set to false since either it was already false or it was true but then saved. As with the SetInterceptor, SaveMethodInterceptor is using behavior that has been introduced. Specifically, it uses isChanged() and setChanged(). 
{% highlight java %}
01: package ex3;
02: 
03: import org.jboss.aop.advice.Interceptor;
04: import org.jboss.aop.joinpoint.Invocation;
05: import org.jboss.aop.joinpoint.MethodInvocation;
06: 
07: public class SaveMethodInterceptor implements Interceptor {
08: 	public String getName() {
09: 		return "SaveMethodInterceptor";
10: 	}
11: 
12: 	public Object invoke(Invocation invocation) throws Throwable {
13: 		MethodInvocation mi = (MethodInvocation) invocation;
14: 		Object param = mi.getArguments()[0];
15: 		ITrackedObject tracked = (ITrackedObject) param;
16: 
17: 		try {
18: 			if (tracked.isChanged()) {
19: 				return invocation.invokeNext();
20: 			} else {
21: 				System.out.printf("Not saving: %s, it is unchanged\n", param
22: 						.getClass());
23: 				return null;
24: 			}
25: 		} finally {
26: 			tracked.setChanged(false);
27: 		}
28: 	}
29: }
{% endhighlight %}
#### Interesting Lines

|Line|Description|
|18 - 20|If the state is changed, allow the save() to happen.|
|20 - 24|The state is not changed, do not call save().|
|26|Regardless of what happened, always set the state to unchanged.|

### jboss-aop.xml
Here is the full aspect configuration:
{% highlight xml %}
01: <?xml version="1.0" encoding="UTF-8"?>
02: <aop>
03: 	<introduction class="ex3.Address">
04: 		<mixin>
05: 			<interfaces>
06: 				ex3.ITrackedObject
07: 			</interfaces>
08: 			<class>ex3.TrackedObjectMixin</class>
09: 			<construction>
10: 				new ex3.TrackedObjectMixin()
11: 			</construction>
12: 		</mixin>
13: 	</introduction>
14: 	
15: 	<pointcut name="SkipTrackedObject" expr="!set(* *.TrackedObjectMixin->*)"/>
16: 	<pointcut name="SetImplementorFields" expr="set(* $instanceof{*.ITrackedObject}->*)"/>
17: 	
18: 	<bind pointcut="SetImplementorFields AND SkipTrackedObject"  >
19: 		<interceptor class="ex3.SetInterceptor" />
20: 	</bind>
21: 
22: 	<pointcut name="DaoSaveMethod" expr="execution(public * *.Dao->save(java.lang.Object))"/>
23: 	
24: 	<bind pointcut="DaoSaveMethod">
25: 		<interceptor class="ex3.SaveMethodInterceptor" />
26: 	</bind>
27: </aop>
{% endhighlight %}
#### Interesting Lines

|Line|Description|
|15|Ignore changes to all fields defined in TrackedObjectMixin. This is absloutely necessary. If you forget this, when any field is changed, the SetInterceptor will set changed=true. When that happens, the SetInterceptor will notice that the changed field is changing from false to true and will set the changed state to true. When that happens, the SetInterceptor will notice that the changed field is changing from false to truen and will set the changed state to true. Repeat ad nausem (or until you get a stack overflow error).|
|16|Capture sets of fields in all classes that implement ITrackedObject.|
|18 – 20|When any field in any class that implements ITrackedObject is set (other than TrackedObjectMixin), e,g. this.name = "Brett", intercept that set with the class SetInterceptor.|
|22 – 26|Intercept Dao->save(...) and allow the SaveMethodInterceptor to have a look first.|

{% include nav prev="JBossAOPEX3SoWhatIsHappening" next="JBossAOPEX3ApplyYourself" %}
