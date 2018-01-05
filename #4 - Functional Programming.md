# Using Variables in Lambda
In Chapter one we covered the ide of "effectively final". This means this means that if you could add the final modifier to a local variable, it is "effectively final".
Lambda expressions can access static variables, instance variables, effectively final method parameters and effectively final local variables. Let's see how many of those we can find in the example below:
```java
interface Gorilla{ String move();}
class GorillaFamily{
  String walk = "walk";
  void everyonePlay(boolean baby){
    String aproach = "amble";
    //aproach = "run";
    
    play(() -> walk);
    play(() -> baby ? "hitch a ride" : "run");
    play(() -> approach);
  }
  
  void play(Gorilla g){
    System.out.println(g.move());
  }

}

```
The first play lambda uses an instance variable in the lambda. The secon play lambda uses a method parameter and we know it is effectively final because there are no reassignments to that variable. The last play lambda uses an effectively final local variable. However if we try uncommenting line 6 will no longer be effectively final. This will cause a compile error on line 10 which is the last play() method. Bear in mind that a normal rules for access control still apply. For example, a lambda can't access private variables in another class. Remember that lambdas can access a subset of variables that are accessible, but never more tha that.

# Working with Built-In Functional Interfaces
As you remember, a functional interface has only one abstract method. All of the funtional interfaces in the table below were introduced in Java 8 and are provided in the ``` java.util.function ``` package. The convention here is to use generic type T for type param. If a second type param is needed, the next letter U, is used. If a distinct return type is needed, R for return is used for generic type.

![Common Functional Interface](img/commonFunctionalInterface.png)

There are many other functional interfaces defined in ```java.util.funtion``` for working with primitives. You do need to memorize the table above.

Now let's see how these functional interfaces are implemented.

## Implementing Supplier
## Implementing Consumer and BiConsumer
## Implementing Predicate and BiPredicate
## Implementing Function and BiFunction
## Implementing UnaryOperator and BinaryOperator
## Checking Functional Interfaces

# Returning an Optional

# Using Streams
## Creating Stream Sources
## Using Common Terminal Operations
### count()
### min() and max()
### findAny() and findFirst()
### allMatch(), anyMatch() and noneMatch()
### forEach()
### reduce()
### collect()
## Using Common Intermediate Operations
### filter()
### distict()
### limit() and skip()
### map()
### flatMap()
### sorted()
### peek()
## Putting Together the Pipeline
## Printing a Stream

# Working with Primitives
## Creating Primitive Streams
## Using Optional with Primitive Streams
## Summararizing Statistics
## Learning the Functional Interfaces for Primitives
### Functional Interfaces for boolean
### Functional Interfaces for double, int and long

# Working with Advanced Stream Pipeline Concepts
## Linking Streams to the Underlying Data
## Chaining Optional 
## Collecting Results
### Collecting Using Basic Collectors 
### Collecting into Maps
### Collecting Using Grouping, Partitioning and Mapping 
