---
title: JBoss_AOP_Example_1
---
[[JBoss AOP Example 1|<--Back]] [[JBossEX1ExpectedVersusActualOutput|Next-->]]

Source files are here: [[file:JBossEX1Source.jar]]. If you need instructions on what do with these files, try [[ExtractingSourceFilesIntoProject|here]].

# Predict the Output
This is the first of several exercise. If you'd like to get a feel for the general outline for each exercise, take a look [[JBossAOPGeneralExerciseOutline|here]].

## Code to Review
Examine the following 2 Java files. Your assignment is to predict the output based on these two files.
----
### Main.java
```java
package ex1;

public class Main {
	public static void main(String[] args) {
		MethodExecutionExample me = new MethodExecutionExample();

		me.noArgMethod();
		System.out.println("-------------");

		me.methodWithStringParam("Brett");
		System.out.println("-------------");

		me.methodCallingOtherMethod();
		System.out.println("-------------");

		MethodExecutionExample.staticMethod();
	}
}
```
----
### MethodExecutionExample.java
```java
package ex1;


public class MethodExecutionExample {
	public void noArgMethod() {
		System.out.printf("\tnoArgMethod\n");
	}

	public void methodWithStringParam(String param) {
		System.out.printf("\tmethodWithStringParam: %s\n", param);
	}

	public static void staticMethod() {
		System.out.printf("\tstaticMethod\n");
	}

	public void methodCallingOtherMethod() {
		System.out.printf("\tmethodCallingOtherMethod\n");
		noArgMethod();
	}
}
```

## Predict the output
Before continuing on, please predict the output from these two files.

[[JBoss AOP Example 1|<--Back]] [[JBossEX1ExpectedVersusActualOutput|Next-->]]