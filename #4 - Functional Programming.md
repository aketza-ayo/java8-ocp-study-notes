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

There are many other functional interfaces defined in ```java.util.function``` for working with primitives. You do need to memorize the table above.

Now let's see how these functional interfaces are implemented.

## Implementing Supplier
A ```Supplier``` is used when you want to generate or supply values without taking any input. The interface is defined as:

```java
@FunctionalInterface
public interface Supplier<T>{
  public T get();
}
```
You can use the Supplier in the following manner:

```java
Supplier<LocalDate> s1 = LocalDate::now;
Supplier<LocalDate> s2 = () -> LocalDate.now();

LocalDate d1 = s1.get();
LocalDate d2 = s2.get();

System.out.println(d1);
System.out.println(d2);
```


This example prints a data suck as 2018-01-05 twice. It is also a good opportunity to review static method references. The ```LocalDate::now``` reference is used to create a Supplier to assign to an intermidiate variable s1. A Supplier is often used when constructing new objects. For example, we can print two empty StringBuilders:

```java
Supplier<StringBuilder> s1 = StringBuilder::new;
Supplier<StringBuilder> s2 = () -> new StringBuilder();

System.out.println(s1.get());
System.out.println(s2.get());
```

This time, we use a constructor reference to create the object. We've been using generics to declare what type of Suppliers we are using. 

Another example of Supplier is as follows:

```java 
 final AtomicInteger count = new AtomicInteger(10);
 Supplier<Integer> supplier = () -> count.incrementAndGet();
 System.out.println(supplier.get());
```

As you can see in the code above the first line creates an object of AnotmicInteger with value of 10. In the second line we pass a lambda expression that takes no argument, as per Supplier interface takes no arguments, and increment the number. This expression is stored into the Supplier variable called supplier. In order to execute this expression we need to call the get() method of the Supplier Interface. This number then gets printed. 

## Implementing Consumer and BiConsumer
You use ```Consumer``` when you want to do something with a parameter but not return anything. ```BiConsumer``` does the same thing except that it takes two parameters. Ommitting the default methods, the interfaces are defined as follow:

```java
@FunctionalInterface
public interface Consumer<T>{
  void accept(T t);
}

@FunctionalInterface
public interface Consumer<T, U>{
  void accept(T t, U u);
}
```

Now see how to make use of it:

```java
Consumer<String> c1 = System.out::println;
Consumer<String> c2 = x -> System.out.println(x);

c1.accept("Annie");
c2.accept("Annie");

```
This example prints Annie twice. 

Another example is to use a forEach() method that takes a Consumer as parameter, so it is perfect to show as example.

```java
List<Integer> list = Arrays.asList(2,5,3,7,1);
//list.forEach((x) -> System.out.println(x));

Consumer<Integer> consumer = new Consumer<Integer>() {
    @Override
    public void accept(Integer i) {
      System.out.println(i.intValue());
    }
};
list.forEach(consumer);

```
In the example above, we can either pass a lambda expression into the forEach() method as per the line that is commented out. Or we can also pass a consumer. Passing a consumer is the long way of doing it but it kind of demostrate the use of the Consumer Interface. 

Now, ```BiConsumer``` is called with two parameters. They don't have to be the same type. For example, we can put a key and a value in a map using this interface.

```java
Map<String, Integer> map = new HashMap<>();
BiConsumer<String, Integer> b1 = map::out;
BiConsumer<String, Integer> b2 = (k,v) -> map.put(k,v);

b1.accept("chicken", 7);
b2.accept("pollo", 1);

System.out.println(map);
```
The output is {chicken = 7, pollo = 1} which shows that both BiConsumer implementations did get called. The code to instantiate b1 is a bit shorter that b2. This might be why the exam is so fond of method references. 

As another example, we use the same type for both generic parameters:
```java
Map<String,String> map = new HashMap<>();
BiConsumer<String,String> b1 = map::put;
BiConsumer<String,String> b2 = (k,v) -> map.put(k,v) ;

b1.accept("chicken","Cluck");



```
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
