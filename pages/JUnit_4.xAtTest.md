---
title: JUnit_4.xAtTest
---
[[JUnit 4.x#AtTest|<--Back]]
# @Test
To denote a method as a test method you use the @Test annotation. The following example demonstrates nearly everything you need to know to get your first test running in Eclipse 2.x - 3.x:
```
     03: import static org.junit.Assert.assertEquals;
     07: import org.junit.Test;
     14: public class TestVehicle {
     26:     @Test
     27:     public void createSimpleVehicle() {
```
The only thing you'd need to add to make this work in Eclipse is a [[JUnit 4.xSuite|suite method]]. Adding that, you'd be able to run this class as a JUnit test and it would give you the familiar JUnit execution dialog.

## Interesting Lines
||Line||Description||
||7||This is the import for the @Test annotation. You won't need to memorize that, simply enter @Test and then ctrl-space and Eclipse will add the import for you.||
||26||The @Test annotation applies to the next method. In this case, we've said that createSimpleVehicle() is a test method. The JUnit infrastructure will reflectively look at all the methods in a given class. Any method that has the @Test annotation will be added to a test suite for execution.||
||27||This line is only interesting in the fact that it is just a regular Java method. It's not JUnit aware.||

In JUnit prior to 4.0, this would have been written as follows:
```
     14: public class TestVehicle extends TestCase {
     27:     public void testCreateSimpleVehicle() {
```

[[JUnit 4.x#AtTest|<--Back]]