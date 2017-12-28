# Desining an Interface
There are numerous rules associated with interfaces so you may want to refer to OCA book for a more detailed explanation about interfaces. An interface defines a list of public abstract methods that any class inplementing the interface must provide. An interface may also include constant ```public static final``` variables, default methods and ```static``` methods.

An interface may inherit another interface and in doing so it inherits all of the abstract methods. See example:
```java
public interface Walk{
  boolean isQuadruped();
  abstract double getMaxSpeed();
}

public interface Run extends Walk{
  public abstract boolean canHuntWhileRunning();
  abstract double getMaxSpeed();
}

public class Lion implements Run{
  public boolean isQuadrupped(){
    return true;
  }
  
  public boolean canHuntWhileRunning(){
    return true;
  }
  
  public double getMaxSpeed(){
    return 100;
  }
}
```

In the code above, the compiler automatically adds ```public``` and ```abstract``` keywords to all ```non-static``` and ```non-default``` methods. By contrast, the contrete class implementing the interface must provide the proper modifiers otherwise it would not compile.

Remember that an interface cannot extend a class, nor can a class extend an interface.

```java 
public interface Sleep extends Lion{}       // DOES NOT COMPILE

public class Tiger extends Walk{}           // DOES NOT COMPILE

```
In the code above, the interface Sleep cannot extend Lion, since Lion is a class. Likewise, the class Tiger cannot extend the interface Walk.

Some rules about interfaces:
- Interfaces cannot extend a class nor classes can extend an interface.
- Interfaces may not be marked final or instantiated directly.
- There are additional rules about default methods, such as Java failing to compile if a class or interface inherits two default methods with the same signature and doesn't provide its own implementation.
- Refer back to the OCA study material for full explanation of rules regarding interfaces and method overriding.

# Introducing Functional Programming
A ```functional interface``` is an interface that contains a single abstract method. Funtional interfaces are the basis for lambda expressions in funtional programming. A ```lambda expression``` is block of code that gets passed around, like an anonymous method. In this chapter we will revise the basics from the OCA exam. In Chapter 3 we will apply lambda expressions to ```Collections``` classes. In Chapter 4 we will include numerous funtional inetrface classes and show you streams. As you read through the chapters you will see support for numerous APIs in Java 8.

## Defining a Functional Interface

Let's take a look at an example of a functional interface and a class that implements it:

```java
  @FunctionalInterface
  public interface Sprint{
    public void sprint(Animla animal);
  }
  
  public class Tiger implements Sprint{
    public void sprint(Animal animal){
      System.out.println("Animal is sprinting fast! " + animal.toString());
    }
  }
```
In this example, the String class is a functional interface, because it contains exactly one abstract method, and the Tiger class is a valid class that implements te interface.

:yin_yang: Applying the @FunctionalInterface annotation while it is good pratice to do so for clarity, it is not required with functional programming. Java compiler implicitly assumes that any interface that contains only one abstract method is a functional interface. Hovewer, if a class is marked with the annotation and contains more than one abstract method, or no abstract methods at all, it is not going to compile. It is a good approach to mark the interface with the annotation because it tells other developers to treat it as a functional interface

For example see the code below:

```java
  @FunctionalInterface
  public interface Sprint{
    public void sprint(Animla animal);
  }
  
  public interface Run extends Sprint{}
  
  public interface SprintFaster extends Sprint{
    public void sprint(Animal animal);
  }
  
  public interface Skip extends Sprint{
    public default int getHopCount(Kangaroo kanagaroo){ return 10;}
    public static void skip(int speed);
  }

```
Suprisingly enough, all of the interfaces above are functional interfaces. First one, Run, defines no new methods but since extends Sprint it is also a funtional intreface. The second interface, SprintFaster, extends Sprint and defines an abstract  method, but this is an override of the parent therefore the resulting interface only has a single abstract method. The third interface, Skip, extends Sprint and defines a static method and a default method, each with implementation. Since neither of these are abstract, the resulting interface has only one abstract method and is a funtional interface. 

Now look at some invalid functional interfaces:

```java
public interface Walk{}

public interface Dance extends Sprint{
  public void dance(Animal animal);
}

public interface Crawl{
  public void crawl();
  public int getCount();
}

```

Although all of this will compile none of these is a funtional interface. In this example, applying the @funtionalInterface annotation to any of these interfaces will result in a compiler error, same as trying to make use of it in a lambda expression.

## Implementing Functional Interfaces with Lambdas
In this section we are going to see how to implement lambda expressions. A lambda expression is a block of code that gets passed around, like an anonymous method. Let's start with a simple CheckTrait functional interface, which has a single method test(). Definion of the class and the functional interface as follows:

```java
public class Animal{
  private String species;
  private boolean canHop;
  private boolean canSwim;
  
  public Animal(String speciesName, boolean hopper, boolean swimmer){
    species = speciesName;
    canHop = hopper;
    canSwim = swimmer;
  }
  
  public boolean canHop(){
    return canHop;
  }
  
  public boolean canSwim(){
    return canSwim;
  }
  
  public String toString(){
    return species;
  }
}

public interface CheckTrait{
  public boolean test(Animal animal);
}
```
Now that we have a structure let's do something with it. The code below uses a lambda expression to determine if some sample animal match the specific criteria:

```java
public class FindMatchingAnimals{
  private static void print(Animal animal, CheckTrait trait){
    if(trait.test(animal)){
      System.out.println(animal)
    }
  }
  
  public static void main(String[] args){
    print(new Animal("fish", false, true), a -> a.canHop());
    print(new Animal("kangaroo", true, false), a -> a.canHop());
  }
}
```
The lambda expression ``` a -> a.canHop() ``` means that Java should call a method with an Animal parameter that returns a boolean value that's the result a.canHop(). We know all this because we wrote the code. But how does Java know? Java relies on context when figuring out what lambda expressions mean. We are passing this lambda as the second parameter of the print() method. That method expects a CheckTrait as second parameter. Since we are passing a lambda instead, Java treats CheckTrait as a funtional interface and tries to map it the the single abstract method.

```boolean test(Animal animal);```

Since this interface's method takes an Animal, it means the lambda parameter has to be an Animal. And since that interface's method returns a boolean, we know that the lambda returns a boolean.

:yin_yang: *Deferred execution* means that code is specified now but runs later. In this case, later is when print() method calls it. Even though the execution is deferred, the compiler will still validate that the code synytax is properly formed

## Understanding Lambda Syntax

## Applying the Predicate Interface
# Implementing Polymorphism
## Distinguishing between an Object and a Reference
## Casting Object References
# Understanding Design Principles
## Encapsulating Data
## Creating JavaBeans
## Applying the Is-a Relationship
## Applying the Has-a Relationship
## Composing Objects
# Working with Design Patterns
## Applying the Singleton Pattern
## Creating Inmutable Objects
## Using a Builder Pattern
## Creating Objects with the Factory Pattern
