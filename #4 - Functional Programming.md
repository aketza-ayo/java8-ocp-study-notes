# Using Variables in Lambda
In Chapter one we covered the ide of "effectively final". This means that if you could add the final modifier to a local variable, it is "effectively final".
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
```Predicate``` is often used when filtering or matching. A ```BiPredicate``` is just like a ```Predicate``` except that it takes two parameters instead of one. 

```java
@FunctionalInterface
public interface Predicate<T>{
  boolean test(T t);
}

@FunctionalInterface
public interface BiPredicate<T, U>{
  boolean test(T t, U u);
}
```

It should be old news by now that you can use a ```Predicate``` to test a condition. See below:

```java
Predicate<String> p1 = String::isEmpty;
Predicate<String> p2 = x -> x.isEmpty();

System.out.println(p1.test(""));
System.out.println(p2.test(""));

```
This prints true twice. More interesting is a BiPredicate. This example also prints true twice:

```java
BiPredicate<String, String> b1 = String::startsWith;
BiPredicate<String, String> b2 = str,word -> str.startsWith(word);

System.out.println(b1.test("chicken","chick"));
System.out.println(b2.test("chicken","chick"));
```

The method refernce conbines two techniques that you have seen. StartsWith() is an instance method. This means that the first parameter in the lambda is used as the instance on which to call the method. The second parameter in the lambda is passed to the startsWith() method itself. This is another example of how method references save a bit of typing. The downside is that are less explicit, and you really have to understand what is going on.

## Implementing Function and BiFunction
A ```Function``` is responsible for turning one parameter into a value of a potentially different type and returning it. Similarly, a ```BiFunction``` is resposible for turning two parameters into a value and returning it. Omitting anu default or static methods, the interface are defined as the following:

```java
@FunctionalInterface
public interface Function<T, R>{
  R apply(T t);
}

@FunctionalInterface
public interface BiFunction<T, U, R>{
  R apply(T t, U u);
}

```
For example, this function converts a String to the length of the String:

```java
Function<String, Integer> f1 = String::length;
Function<String, Integer> f2 = s -> s.length();

System.out.println(f1.apply("aloha"));       //5
System.out.println(f2.apply("panda"));       //5

```

This function turns a String into an Integer. Well, it turns String into an int, which is autoboxed into an Integer. The following example combines two string objects and produces another one.

```java
BiFunction<String, String, String> b1 = String::concat;
BiFunction<String, String, String> b2 = (str,toAdd) -> str.contact(toAdd);

System.out.println(b1.apply("baby ","panda"));
System.out.println(b2.apply("baby ", "panda"));
```

The first two types in the BiFunction are the input types. The third is the return type. For the method reference, the first parameter is the instance that contact() is called on and the second is the param passed to contact();

:yin_yang: **Creating Your Own Functional Intrefaces** Java provides a built-in interface for functions with one or two params. What of you need more? Suppose you want to create a functional interface for the wheel spped of each wheel in a tricycle. The it would look like:
```java
@FunctionalInterface
public interface TriFunction<T,U,V,R>{
    R apply(T t, U u, V v);
}
```
Now suppose that you want to create for the same but for your quad. Then:

```java
@FunctionalInterface
public interface QuadFunction<T,U,V,W,R>{
    R apply(T t, U u, V v, W w);
}
```

Java built in interface are meant to facilitate the most common functional interfaces. But you can add any functional interface you'd like and Java matches them when you use lambdas or method references.

## Implementing UnaryOperator and BinaryOperator
Unary and binary operator are special case of a function. They require all parameters to be of the same type. A ```UnaryOperator``` transforms its value into one of the same type. For example incrementing by one is a UnaryOperator. In fact extends ```Function```. A ```BinaryOperator``` merges two values into one of the same type. Adding two numbers is a binary operation. Similarly, ```BinaryOperator``` extends ```BiFunction```. 

```java
@FunctionalInterface
public interface UnaryOperator<T> extends Function<T,T>{
  T apply(T t);
}

@FunctionalInterface
public interface BinaryOperator<T,T> extends BiFunction<T,T,T>{
  T apply(T t1,T t2);
}

```

Below you can find an example of usage:

```java
UnaryOperator<String> u1 = Stirng::toUpperCase;
UnaryOperator<String> u2 = s -> s.toUpperCase();

System.out.println(u1.apply("chirp"));
System.out.println(u2.apply("chirp"));
```

This prints CHIRP twice. We don't have to specify the return type in the generics because ```UnaryOperator``` requires it to be the same type as the parameter. And now for the binary example:

```java
BinaryOperator<String> b1 = String::concat;
BinaryOperator<String> b2 = (string, toAdd) -> string.concat(toAdd); 

System.out.println(b1.apply("baby ", "panda"));   //baby panda
System.out.println(b2.apply("baby ", "panda"));   //baby panda 
```


## Checking Functional Interfaces
Now would be a good time to memorize table above on functional interfaces. Let's see a few examples to put it into practice. What functional interface would you use in these situations?

- Returns String without taking any parameter?
- Returns a Boolean and takes a String.
- Returns an Integer an takes two Integers.

Ready? OK, the first one is a ```Supplier``` because it generates an Object and takes zero params. The second is a ```Function``` because it takes one parameter and returns another type. Its a little tricky because you might thing it returns a boolean and therefore is a predicate but it returns a Boolean Object. Finally the third one is either a ```BiFunction``` or a ```BinaryOperator``` but ```BinaryOperator``` is a bettern answer because it is more specific.

Now, let's do another exercise but with code:

```java
6:  _____<List> ex1 = x -> "".equals(x.get(0));
7:  _____<Long> ex2 = (Long l) -> System.out.println(l);
8:  _____<String, String> ex3 = (s1, s2) -> false;
```

Ready? OK, line 6 is a ```Predicate``` because it takes one parameter and returns a boolean value. Line 7 is ```Consumer``` because it takes one param and does not return anything. Line 8 is a ```BiFunction``` because it takes two parans and returns a boolean value.

Now, let's do some tricky examples to identify the error:

```java
6: Function<List<String>> ex1 = x -> x.get(0);  //DOES NOT COMPILE
7: UnaryOperator<Long> ex2 = (Long l) -> 3.14;  //DOES NOT COMPILE
8: Predicate ex4 = String::isEmpty;             //DOES NOT COMPILE

```

Line 6 is wrong because a function has to specify two generics - the input and the return value type. The return value type is missing from line 6 causing the code not to compile. Line 7 is a ```UnaryOperator``` which returns the same type as the parameter it is passed in. Line 8 is missing the generic of the parameter for ```Predicate```. This makes the parameter that was passed an Object rather than a String. The lambda expects a String because it calls a method that only exists in the String object rather than an Object. Therefore, it does not compile.


# Returning an Optional
How do we express the idea that a variable's value is not set yet if we don't want to use ```null``` ? In java 8, we use the ```Optional``` type. An Optional is created using a factory. Yoyu can either request an empty Optional or pass a value for the Optional to wrap. Think of an Optional as a box that might have something in it or might instead be empty.

Let's see how it looks in the code:

```java
public static Optional<Double> average(int... scores){
  if(scores.length == 0) return Optional.empty();
  
  int sum = 0;
  for(int score : scores) sum += score;
  return Optional.of((double) sum / scores.length);
  
}
```

As you can see the code returns an empty optional when we can't calculate the average. The next lines add the scores. The last line creates an Optional to wrap the average. See below to see what's the value in the Optional box:

```java
System.out.println(average(90,100));      //Optional[95.0]
System.out.println(average());            //Optional.empty
```
You can see that one optional contains a value and the other is empty. Normally, we want to check if a value is there and/or get it out the box. Here is one way to do that:

```java
Optional<Double> opt = average(90,100);
if(opt.isPresent()){
  System.out.println(opt.get());      //95.0
}
```

The secod line checks whether it contains a value and the next line prints it out. What if we didn't do the check and the Optional was empty? 

```java
Optional<Double> opt = average();
System.out.println(opt.get());      //bad

```

We'd get an exception since there is no value inside the Optional:

``` java.util.NoSuchElementExcetion: No value present ```

When creating an Optinal it is common to want to use empty when the value is null. You can do this with an if or ternary operation. 

```java
Optional o = (value == null) ? Optional.empty() : Optional.of(value);
```

If value is null, o is assigned the empty Optional. Otherwise, we wrap the value. Since this is such a common pattern, Java provides a factory method to do the same thing. The code below and above achieve the same thing.

```java
Optional o = Optional.ofNullable(value);
```

The following table covers what I need to know for the exam.

![optional instance methods](img/OptinalInstanceMethods.png)

You've already seen get() and isPresent(). The other methods allow you to write code that uses an Optional in one line without having to the ternay operator. This makes the code easier to read. Instead of using an if statement we can specify a ```Consumer```  to be run when there is a value inside the Optional. When there isn't the method simply skips running the ```Consumer```: 

```java
Optional<Double> opt = average(90,100);
opt.ifPresent(System.out::print);
```

Using ifPresent() better expresses our intent. We want something done if a value is present. The other methods allow you to specify what to do if a value isn't present. There are three choices:

```java
30: Optional<Double> opt = average();
31: System.out.println(opt.orElse(Double.NaN));
32: System.out.println(opt.orElseGet(() -> Math.random()));
33: System.out.println(opt.orElseThrow(() -> new IllegalStateException()));

```
and this prints something like this:

```
NaN
0.445566787878712
Exception in  thread "main" java.lang.IllegalStateException
  at optional.Average.lambda$3(Average.java:56)
  at optional.Average$$lambda .......
  at java.util.optional.orElseThrow(Optional.java:290)
```
Line 31 shows that you can return a specific value or variable. In our case we print "Not a Number". Line 32 shows using a Supplier to generate a value at runtime. Line 33 shows using a different Supplier to create a exception that should be thrown. 

Notice that the two methods that takes Supplier have different names. Do you see why this code does not compile? 

```
System.out.println(opt.orElseGet(() -> new IllegalStateException()));   //DOES NOT COMPILE
```

opt is an Optional<Double>. This means the Supplier must return a Double. Since this supplier returns a exception , the type does not match. The last example with Optional is really easy. What do you think this does?
  
  ```java
  Optional<Double> opt = average(90,100);
  System.out.println(opt.orElse(Double.NaN));
  System.out.println(opt.orElseGet(() -> Math.random()));
  System.out.println(opt.orElseThrow(() -> new IllegalStateException()));
 ```
 
 In this case it prints out 95 three time. Since the value does exist, there is no need to use the or else logic. 
 
 :yin_yang:  **Is Optional the same as null?** before Java 8 programmers would return null instead of Optional. There were a few  shortcomings with these approach. One is that there was a clear way to express that null might be a special case. By contrast, returning an Optional is a clear statement in the API that there might be a value in there. Another advantage of Optional is that you can use functional programming style with isPresent() and the other methods rather than needing an if statement. Finally, you can chain Optional calls.
 
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
