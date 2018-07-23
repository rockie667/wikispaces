---
title: AspectJEX1ExpectedVersusActualOutput
---
[[AspectJ Example 1|<--Back]] [[AspectJEX1FormTheory|Next-->]]

## Expected Output
Did you guess the output would look something like this?
```
   noArgMethod
-------------
   methodWithStringParam: Brett
-------------
   methodCallingOtherMethod
   noArgMethod
-------------
   staticMethod
```

## Actual Output
What if I told you the output was actually this (ignoring line wrapping)?
```
Entering: void ex1.MethodExecutionExample.noArgMethod()
	noArgMethod
Leaving void ex1.MethodExecutionExample.noArgMethod()
-------------
Entering: void ex1.MethodExecutionExample.methodWithStringParam(String)
	methodWithStringParam: Brett
Leaving void ex1.MethodExecutionExample.methodWithStringParam(String)
-------------
Entering: void ex1.MethodExecutionExample.methodCallingOtherMethod()
	methodCallingOtherMethod
Entering: void ex1.MethodExecutionExample.noArgMethod()
	noArgMethod
Leaving void ex1.MethodExecutionExample.noArgMethod()
Leaving void ex1.MethodExecutionExample.methodCallingOtherMethod()
-------------
Entering: void ex1.MethodExecutionExample.staticMethod()
	staticMethod
Leaving void ex1.MethodExecutionExample.staticMethod()
```

[[AspectJ Example 1|<--Back]] [[AspectJEX1FormTheory|Next-->]]


