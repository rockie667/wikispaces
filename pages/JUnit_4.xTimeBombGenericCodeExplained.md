---
title: JUnit_4.xTimeBombGenericCodeExplained
---
[[JUnit 4.x|<--Back]]

# Time Bomb Generic Code Explained
Here's a use of TimeBomb (for context) taken from [[JUnit 4.x#example2|Example 2]]:
```
92:     @Test(expected = ObjectInUse.class)
93:     public void removeUsedValidType() {
94:         TimeBomb.throwUntil(16, 6, 2006, new ObjectInUse(VehicleType.class, "key"));
95:     }
```

Here is my first attempt at writing the throwUntil method in the TimeBomb class:
```
06:     public static void throwUntil(int d, int m, int y, Exception e) throws Exception {
07:         Calendar now = Calendar.getInstance();
08:         Calendar dateToThrowUntil = Calendar.getInstance();
09:         dateToThrowUntil.set(y, m - 1, d);
10:
11:         if(dateToThrowUntil.after(now)) {
12:             throw e;
13:         }
14:     }
```
This version did not "work". Why? If you look at line 93 just above, you'll notice that the method signature is:
```
public void removeUsedValidType()
```
If you look at line 6 just above, you'll notice that the method signature is:
```
public static void throwUntil(int d, int m, int y, Exception e) throws Exception
```
Java will complain on line 93 that you must either catch "Exception" or surround with a try/catch block. By complain, I mean "it won't compile."

My first vision wasn't up to my expectations. Sure, I could have just added "throws exception" to line 93, but I didn't want to do that. I'm not actually throwing a checked exception so I should not have to declare the test method that way. The problem was the signature of my "throwUntil" method. It claims it throws Exception but in fact it throws whatever is given to it. In the case of line 94, I'm passing in a RuntimeException. Prior to Java 5 I was out of luck. With Java 5 I have another option.

I made throwUntil a generic method. Doing so allowed Java to determine at compile time that while in general the method "throwUntil" can throw any exception, in this //**particular use**// it was throwing an ObjectInUse exception, which happens to be a RuntimeException and therefore does not require a try/catch or a throws clause on the method signature.

This is what I ended up using:
[[#TimeBombCode]]
## TimeBomb.java
```java
01: package vehicle.util;
02:
03: import java.util.Calendar;
04:
05: public class TimeBomb {
06:     public static<T extends Exception> void throwUntil(int d, int m, int y, T e) throws T {
07:         Calendar now = Calendar.getInstance();
08:         Calendar dateToThrowUntil = Calendar.getInstance();
09:         dateToThrowUntil.set(y, m - 1, d);
10:
11:         if(dateToThrowUntil.after(now)) {
12:             throw e;
13:         }
14:     }
15: }
```

Here are the essential lines:
```
06:     public static<T extends Exception> void throwUntil(int d, int m, int y, T e) throws T {
12:             throw e;
```

Why are these lines important?

Line 6 says this is a generic method. We know that because of the <T extends Exception> part. If you have used generics a little bit, you've probably seen classes and methods that look more like just this: "<T>", so what is the purpose of the "extends Exception" part? It tells the compiler that when this method is used, only allow types to be provided that inherit from Exception. For example, the following use of this method would fail:
```
94:         TimeBomb.throwUntil(16, 6, 2006, "Just passing in a String");
```
Java looks at actual parameters in this example. We have 4:
* 16, an int 
* 6, an int 
* 2006, an int 
* "Just passing in a String", a string
It matches the first three int parameters to the first three parameters to throwUntil. Next, it notices that the 4th parameter is a generic parameter. So Java looks at the compile-time type of the 4th parameter (a String in this case) and, knowing that String does not inherit from Exception, provides an appropriate compiler error.

Using <T extends Exception> allows for two more important things. First, notice this method "throws T". A throws clause must be followed by a class that extends Throwable. If you don't tell Java that "T extends Exception" then it would be possible for this method to be used in a way that the 4th parameter is not an Exception type. Java won't allow this potential to happen. However, because we told Java that we only want to allow types that are Exceptions, it can safely allow T to be used in a throws clause.

The second thing "extends Exception" allows amounts to the same thing. In line 12 we attempt to "throw e". Since e is of type T, which we know is Exception or some sub-class of Exception, this line is always safe. Without the "throws Exception", it is not safe and won't compile.

Back to the original use:
```
94:         TimeBomb.throwUntil(16, 6, 2006, new ObjectInUse(VehicleType.class, "key"));
```
Java matches the first three int parameters as before. Next, Java notices that the 4th parameter's compile-time type is "ObjectInUse". As already mentioned, ObjectInUse is a RuntimeException, so this line compiles and we do not need either a try/catch or a throws clause.

[[JUnit 4.x|<--Back]]