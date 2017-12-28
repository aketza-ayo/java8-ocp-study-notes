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

## Purpose of an Interface
# Introducing Functional Programming
## Defining a Functional Interface
## Implementing Functional Interfaces with Lambdas
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
