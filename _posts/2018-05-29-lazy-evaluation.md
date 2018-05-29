---
layout: post
title: Lazy evaluation in Java (applicable to other languages Node...)
date: 2018-05-29
author: e.caceres
categories: lazy java lambdas
description: Lazy evaluation in Java using Lambdas
---

## Definition but no evaluation
  __Lazy evaluation is an evaluation strategy which delays the evaluation of an expression until its value is needed. The opposite of this is eager evaluation, where an expression is evaluated as soon as it is bound to a variable.__

Java like most imperative programming languages evaluates methods arguments eagerly but we should consider a lazy alternative for scenarios where it can boost performance, for example avoiding a needless expensive computation. In the case of Java, Lambdas allow it to be lazy.

Have a look to the following process:

```java
public class NonLazyCodeExample {
    public static void main(String args[]) {
        final int number = 1;
        final boolean computeProcess = computeProcess(number);
        final boolean longProcess = longProcess(number);
        if (computeProcess && longProcess) {
            System.out.println("TRUE");
        } else {
            System.out.println("FALSE");
        }
    }
    public static boolean computeProcess (final int number) {
        System.out.println("Computing: " + number);
        return (number > 10);
    }
    public static boolean longProcess (final int number) {
        System.out.println("Processing: " + number);
        return (number < 10);
    }
}
```



__Output:__
```
Computing: 4 
Processing: 4 
FALSE 
```

In this case both methods are called but longprocess is not needed so is a waste of resources. Of course, we can improve this code delaying the invocation of the methods into the if. 
```java
if (computeProcess (number) && longProcess (number)) { ...
```

But if we have more than a more complex logic we can lose performance and we might fall in a lack of readability. 
```java
if (computeProcess (number) && longProcess(number) || (computeProcess(number) || mycondition && mycondition2)) {
	if(computeProcess (number) && mycondition) { ...
```

Using lambdas combine readability and lazy evaluation. ComputeProcess and LongProcess are added to Supplier. Theses expressions are not evaluated until the use of the method get().

```java
public class LazyCodeExample {
    public static void main(String args[]) {
        final int number = 1;
	// This expressions are defined but not evaluated
        final Supplier <Boolean> computeProcess = () - > computeProcess(number);
        final Supplier <Boolean> longProcess = () - > longProcess(number);
        // Now are evaluated. But thanks to short-circuit in && since computeProcess.get() is false longProcess.get() is not evaluated
	  if (computeProcess.get() && longProcess.get()) {
            System.out.println("TRUE");
        } else {
            System.out.println("FALSE");
        }
    }
    public static boolean computeProcess (final int number) {
        System.out.println("Computing: " + number);
        return (number > 10); 
    }
    public static boolean longProcess (final int number) {
        System.out.println("Processing: " + number);
        return (number < 10);
    }
}
```

```
Output:
Computing: 4
FALSE
```

Lazy evaluation is just another tool in our stuff. Is suitable to improve the performance when there are possibilities of to avoid some logic that need to be defined but not evaluated eagerly in the same expression.

**So, be lazy my friend, when it needed.**


Have you tried lazy evaluation in your favourite language? 

 * https://hackernoon.com/lazy-evaluation-in-javascript-84f7072631b7  
 * https://blogs.msdn.microsoft.com/pedram/2007/06/02/lazy-evaluation-in-c/ 
