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
The syntax of lambda expression is tricky because many ooptions are optional. For example the following two lines are equivalent:
```a -> a.canHop()``` and ```(Animal a) -> {return a.canHop();}```.
Let's see. Left side of the arrow operator -> indicates the input parameter for the lambda expression. It can be comsumed by the functional interface whose abstract method has the same number of parameters and compatible data types. The right side is referred to as body of the lambda expression. It can be consumed by the functional interface whose abstract method returns a compatible data type.

The differences between these two is that the second example uses parentheses (), while the first one does not. The () parentheses can be omitted in a lambda expression if there is exactly one input parameter and the type is not explicitly stated in the expression. this means that expressions that have zero or more than one input parameter will still require parentheses. For example, the following are all valid lambda expressions, assuming that are valid functional interfaces:

```java
() -> new Duck()

d -> {return d.quack();}

(Duck d) -> d.quack()

(Animal a, Duck d) -> d.quack()

```

## Spotting Invalid Lambdas
Can you figure out why each of the following lambda expressions is invalid and will not compile when used as an argument to a method?

```java
Duck d -> d.quack()                     // DOES NOT COMPILE

a,d -> d.quack()                        // DOES NOT COMPILE 

Animal a, Duck d -> d.quack()           // DOES NOT COMPILE

```

They all require parentheses ()! Parentheses can only be omitted if there is exactly one parameter and the data type is not specified. Next the {} allows you to write multiple lines of code in the body of the lambda. What's tricky here is that when you add {} you must terminate each statement in the body with a semicolon ; In the examples earlier we were able to omitt the braces {}, the semicolon ; and return keyword becaus there was a single-line lambda body.
When using {} you must also include the return keyword if that functional interface returns a value. Alternatively, the return keyword is optional when the return type of the method is void. See some examples:

```java
() -> true                                      // 0 parameters
  
a -> {return a.startsWith("test");}             // 1 parameter

(String a) -> a.startsWith()                    // 1 parameter

(int x) -> {}                                   // 1 parameter

(int y) -> {return;}                            // 1 parameter

```
The first example takes no arguments and always returns true. The second and third examples both takes single String value, using different syntax to accomplish the same thing. 

Now let's look at some lambda that take more than one argument:

```java
(a,b) -> a.startsWith("test")                             // 2 parameters

(String a, String b) -> a.startsWith("test")              // 2 parameters

```
These examples both take two params and ignore one of them, but there is no rule that says it must use all of the input parameters.

More examples:

```java
a,b -> a.startsWith("test")                 //DOES NOT COMPILE

c -> return 10;                             //DOES NOT COMPILE

a -> {return a.startsWith("test") }         //DOES NOT COMPILE

```
 
 The first lambda needs parentheses () around the paremeter list. Remember that the parentheses are optional only when there is one parameter and it doesn't have a type declared. The second uses return without using braces {}. The last line is missing the semicolon after the return statement. The following rewritten lambda expressions are each valid:

```java
(a,b) -> a.startsWith("test")                 

c -> {return 10;}                            

a -> {return a.startsWith("test"); }        

```

As mentioned, the data types for the input parameters of a lambda expression are optional. When one parameter has a data type listed, though, all parameters must provide a data type. The following lamda expressions are each invalid for this reason:

```java
(int y, z) -> { int x = 1; return y + 10; }        //DOES NOT COMPILE

(String s, z) -> { return s.length() + z; }        //DOES NOT COMPILE

(a, Animal b, c) -> a.getName()                    //DOES NOT COMPILE

```

If we add or remove all the data types then this lambda expressions do compile. 

There is one more issue you might see with lambdas. We've been defining an argument list in our lambda expressions. Since Java doesn't allow us to re-declare a local variable, the following is an issue:

```java
(a,b) -> {int a = 0; return 5;}                     //DOES NOT COMPILE
```
We tries to redeclare a, which is not allowed. By contrast the following line is permitted because it uses a different vaiable name:

```java
(a,b) -> {int c = 0; return 5;}

```

## Applying the Predicate Interface
In the earlier example we created a simple functional interface to test Animal trait:

```java
public interface CheckTrait{
  public boolean test(Animal animal);
}
```

You can imagine that we'd have to create lots of functional interfaces like this to use lambdas. Luckily, Java sees that as common problem and provides such an interface for us. It's in the package ```java.util.function``` and the gist of it is as follows:

```java
public interface Predicate<T>{
  public boolean test(T t);
}
```

it is very similar to CheckTrait interface apart from the the type T instead of Animal. This is a generic type. The result of using ```Predicate``` is that we longer need our own functional interface. The following is rewrite of our program:

```java
import java.util.function.Predicate;

public class FindMatchingAnimals{
  private static void print(Animal animal, Predicate<Animal> trait){
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

# Implementing Polymorphism
*Polymorphism* is the ability of a single interface to support multiple underlying forms. In Java this allows multiple types  pof objects to be passed to a single method or class. It also allows one object to take on many different forms. **A Java object may be accessed using a reference with the same type as the object, a reference that is a superclass of the object or a reference that defines an interface that the object implements, either directly or through a superclass. Furthermore, a cast is not required if the object is beign reassigned to a supertype or interface of the object.**

If you use a variable to refer to an object, then only the mehods or variables that are part of the variable's reference type can be called without explicit cast.

## Distinguishing between an Object and a Reference
In Java, all objects are accessed by reference, so as a developer ypou never have direct access to the memory of the object itself. Conceptually, you should think of an object as the entity that exists in memory.

All objects can be reassigned to ```java.lang.Object```

```java
Lemur lemur = new Lemur();

Object lemurAsObject = lemur;

```
Eventhough the Lemur object has been assigned a reference with a different type, the object itself has not changed and still exists a Lemur object in memory. What has changed is our ability to access methods within the Lemur class with the lemurAsObject reference. Unless we do an explicit cast back to Lemur (you see more about this in the next section), we no longer have access to the Lemur properties of the object.

We can summarize this principle with the following two rules:
1- The type of the object determines which properties exist within the object in memory
2- The type of the reference to the object determines which methods and variables are accessible to the Java program.

## Casting Object References
As mentioned earlier, once changed the reference type we lost access to more specific methods defined in the subclass that still exists within the object. But we can reclaim those references by casting the object back to the specific subclass it came from:
```java
Primate primate = lemur;

Lemur lemur2 = primate;       // DOES NOT COMPILE

Lemur lemur3 = (Lemur) primate;
System.out.println(lemur3.age);
```

In the example of the book, primate is a class on its own with a single method hasHair(). There is also an interface HasTail with the abstract method isTailStripped() and then the Lemur class that extends the Primate class and implements the HasTail interface. In addition, the lemur class also has a int member variable age = 10;

So, in this example above when we try to convert the primate reference back to lemur reference, lemur2, without explicit cast. The result will not compile. In the second example, though, we explicitly cast the object toa subclass of the object Primate, and we gain access to all the methods availoable to the Lemur class.

Here are some basic rules to keep in mind when casting variables:
1 - Casting an object from a subclass to a superclass doesn't require explicit cast.
2 - Casting an object from a superclass to a subclass requires an explicit cast.
3 - The compiler will not allow cast to unrelated types.
4 - Even when the code compiles without issue, an exception may be thrown at runtime if the object beign cast is not actually an instance of that class.

One example for point 4 will be. Casting is not without limitations. Even though two classes share related hierarchy, that doesn't mean an instance of one can automatically be cast to another. The example is:

```java
public class Rodent{}

public class Capybara extends Rodent{
  public static void main(String[] args){
    Rodent rodent = new Rodent();
    Capybara capybara = (Capybara) rodent;   // Throws ClassCastException at Runtime
  }
}
```
The code above creates an instance of Rodent and then tries to cast it to a subclass of Rodent, Capybara. Although this code compiles without issue, it will throw a ClassCastException at runtime since the object beign referenced is not an instance Capybara class. You can use ```instanceof``` operator to avoid throwing ClassCastException at runtime:

```java
if(rodent instanceof Capybara){
  Capybara capybara = (Capybara) rodent;
}
```
# Understanding Design Principles
A *design principle* is an established idea or best practice that facilitates the software design process.

## Encapsulating Data
One fundamental principle of object-oriented design is the concept of encapsulating data. In software development, *encapsulation* is the idea of combining fields and methods in a class such that methods operate on the data, as opposed to the users of the class accessing the fields directly. In Java, is common to have private variables and public methods to access or modify the variable commonly referred as setters and getters.

The underlying idea of encapsulation is that no actor other than the class itself should have direct access to its data. The class is said to encapsulate the data it contains and prevent anyone from directly accessing it.

## Creating JavaBeans
Encapsulation is so prevalent in Java that there is a standard for creating classes that store data, called JavaBeans. A *JavaBean* is a design principle for encapsulating data in an object in Java. It boils down to variables are marked private. A mutation method begin with set, non-boolean properties begin with get to retrieve the data. Getter for boolean property may begin with is or get. So the nethod name nust have a prefix of set/get/is followed by the first letter of the property in uppercase and followed by the rest of the property name. 

## Applying the Is-a Relationship
In object-oriented design, we describe the property of an object being an instance of a data type as having an *is-a relationship*. The is a relationship is also known as the inheritance test. For example, imagine that we have a class Cat that extends a class Pet, as shown below. So a Cat is-a Pet, because Cat extends Pet. 

![is a relationship](img/isA1.png)

## Applying the Has-a Relationship
## Composing Objects
# Working with Design Patterns
## Applying the Singleton Pattern
## Creating Inmutable Objects
## Using a Builder Pattern
## Creating Objects with the Factory Pattern
