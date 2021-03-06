---
title: JUnit_4.xAtTestWithExpected
---
{% include nav prev="JUnit_4.x#AtTestExpected" %}

## @Test(expected``=``SomeException.class)
Prior to JUnit 4, if I wanted to write a test that verified a certain exception was thrown, I'd write something like the following:
{% highlight terminal %}
public void testSomething() {
   try {
      someObject.shouldThrowException();
      fail("Expected FooException to be thrown but was not");
   } catch (FooException e) {
      // success for this test case
   }
}
{% endhighlight %}
This is a bit awkward so in JUnit 4, you can now tell the annotation that when this test runs, you expect the result to be an exception. Here's a snippet taken from [Example 2](JUnit_4.x#example2):
{% highlight terminal %}
51:     @Test(expected = ObjectExists.class)
52:     public void createVehicleTypeWhenNameAlreadyExists() {
53:         component.createVehicleType("PreExisting1", ValidState.valid);
54: 
55:     }
{% endhighlight %}
When 53 executes, I expect the exception ObjectExists to be thrown. If it is not, JUnit will let me know that my test did not pass. 

ObjectExists is a subclass of RuntimeException and is therefore an unchecked exception. This means I do not need to surround line 53 with a try/catch or add "throws ObjectExists" to the method signature on line 52. If ObjectExists were a checked exception (a subclass of exception but not of RuntimeException), here is what I'd have to change to make this example compile:
{% highlight terminal %}
52:     public void createVehicleTypeWhenNameAlreadyExists() throws ObjectExists {
{% endhighlight %}
I mention this to make it clear that @Test(expected=...) does not allow you to ignore checked exceptions. It just makes writing test that expect to generate an exception easier to write. Here is what I'd have to write using JUnit before version 4.x:
{% highlight terminal %}
52:     public void testCreateVehicleTypeWhenNameAlreadyExists() {
            try {
53:            component.createVehicleType("PreExisting1", ValidState.valid);
               fail("Expected " + ObjectExists.class + " to be thrown but it was not");
            } catch(ObjectExists e) {
               // success
            }
54: 
55:     }
{% endhighlight %}
The use of @Test(expected=...) removes the need for several lines of boilerplate code.

{% include nav prev="JUnit_4.x#AtTestExpected" %}
