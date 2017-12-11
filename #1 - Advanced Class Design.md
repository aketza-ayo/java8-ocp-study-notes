
# Reviewing OCA Concepts
- The OCP questions are a lot tougher than the OCA questions. Really need to know the fundamentals covered in the next 5 sections. After that the book moves onto the new topics such as instanceOf, implement equals/hashCode/toString, Enums and nested classes.

## Access Modifiers
These are ```private```, no modifier(the default or aka package-private), ```protected``` and ```public```:
- **private**: Only accessible within the class.
- **no-modifier**: Is the **default** and used if any other access modifier keyword is missing. Only code in the same package can access code with default access.
- **protected**: It allows subclasses and code in the same package to access members. If in a different package, the member is only accessible through inheritance otherwise it errors.
- **public**: Any code can access public members.

![access modifiers diagram](img/accessModifiers.png)

:yin_yang: Remember that there is a ```default``` keyword introduced in Java 8 for interfaces. The keyword is not an access modifier.


## Overloading and Overriding 
- *Overloading* and *overriding* happen only when the method name is the same (watch out for case sensitiveness in that case it is just a different method entirely). 
- *Overriding* occurs only when method signature is the same. *Method signature* is the method name and the parameter list. 
- *Overloading* occurs when the method parameters vary by type and/or number.
- When mutiple overloaded methods are present, Java looks for the closest match first. It tries to find the following:

  - Exact match by type.
  - Matching a superclass type.
  - Converting to a large primitive type.
  - Converting to an autoboxed type.
  - Varargs
- For overriding, the overridden method has a few rules
  - The access modifiers must be the same or more accessible.
  - The return type must be the same or a more restricted type AKA *covariant return type*.
  - If any check exceptions are thrown, only the same exception or subclasses of those are allowed to be thrown.
  - The methods must not be static. If they are the method is hidden and not overriden

## Abstract Classes
For example, check the snippet below. What are three ways that you can fill in the blank to make this code compile? Try to think of ways that use the clean() method rather than just putting a comment there:

```java
abstract class Cat{
  -------------------
}

class Lios extends Cat{
  void clean();
}
```
One of them is tricky and that is that you can leave it blank. An abstract class is not required to have any methods in it. A second answer is ``` abstract void clean(); ``` this one is the actual abstract method. It has the abstract keyword and a semicolon instead of a method body. A third answer is the default implementation ``` void clean(){} ``` notice that this time there is no abstract keyword in the method and the body is implemented even though it is empty. Furthermore, there are plenty of other ways. For example, ``` void clean() throw RuntimeException {} ```

Things to remember about abstract classes are:
- An *abstract class* may contain any number of methods including zero. 
- The methods can be abstract or concrete.
- Abstract methods may not appear in a class that is not abstract.
- The first concrete subclass of an abstract class is required to implement all abstract methods that are not implemented by a superclass.


## Static and Final
- ```final``` prevents a variable from changing or a method from beign overriden.
- ```static``` makes a variable shared at the class level and uses the class name to refer to a method.
- ```static``` and ```final``` are allowed to be added on the class level too.
- You will see ```static``` classes in the section about nested classes.
- Using ```final``` on a class means that it cannot be subclassed.
- As with methods, a class cannot be both ```abstract``` and ```final```.
- In the Java core classes, String is ```final``` which means that it cannot be sub-typed 

## Imports

Oracle no longer list imports and packages in the objectives for OCP 8 exam. They include visibility modifiers, which means you still need to understand packages and imports.
Remeber that you can add static in the imports section if referencing a static method such as sort() from the Collections framework library.

For example, in the following code which imports do we need to include in order to compile the code below?
```java
public class ListHelper{
  public List <String> copyAndSortList(List <String> original){
    List<String> list = new ArrayList<String>(original);
    sort(list);
    return list;
  }
}
```
By looking at the code, we don't need to worry about the String class because that is included by default as part of the ```java.lang.*;``` import. Although we need to import libraries for List, ArrayList and sort() method. There are several ways to achieve this. For adding the sort() method either of these two will do:

```import static java.utils.Collections.sort; ```

```import static java.utils.Collections.*; ```

For the other part, that is List and ArrayList classes. There are two ways, either using the wildcard or importing each class individually.

```import java.util.*;```

The other way is to list them individually:

```import java.util.List;```

```import java.util.ArrayList;```

# Using instanceof
- The ```instanceof``` operator is used to test whether the object is an instance of the specified type (class or subclass or interface). The instanceof is also known as type comparison operator because it compares the instance with type. It returns either true or false.

Let's see a few examples:
```java
class HeavyAnimal { }
class Hippo extends HeaveAnimal{ }
class Elephant extends HeavyAnimal { }

HeavyAnimal hippo = new Hippo();
booelan b1 = hippo instanceof Hippo;          //true
boolean b2 = hippo instanceof HeavyAnimal;    //true
boolean b3 = hippo instanceof Elephant;       //false
```
- All Java classes inherit from Object, which means that ``` x instanceof Object``` is usually ```true```, except for one case where is ```false```. It the literal ```null``` or a variable reference pointing to ```null``` is used to check instanceof, the result is ```false``` because ```null``` is not an Object. See below:

```java
HeavyAnimal hippo = new Hippo();
boolean b4 = hippo intanceof Object;        //true
Hippo nullHippo = null;
boolean b5 = nullHippo instanceof Object;   //false 
```

- Sometimes the comparison might not even compile if the compiler knows that there is no possible way for a Hippo variable reference to be an Elephant for example, since Hippo doesn't extend Elephant directly or indirectly. See below:

```java 
Hippo anotherHippo = new Hippo();
boolean b5 = anotherHippo instanceof Elephant;    //DOES NOT COMPILE
boolean b6 = anotherHippo instanceof Cat;         //DOES NOT COMPILE
```

- The compilation check above only applies when instanceof is called on a class. When checking whether an object is an instanceof an interface, Java waits until runtime to do the check. Subclass could implement that interface and the compiler wouldn't know it. There is no way for Hippo to be subclass of Elephant in our hierarchy by now.

- When the class type is an interface, the compiler allows the statement because there could later be a class that implements it. 

```java
public interface Mother{}
class Hippo extends HeavyAnimal
```

This code below compiles even though hippo is not a sub type or implement the Mother interface

```java 
HeavyAnimal hippo = new Hippo();
boolean b6 = hippo instanceof Mother;
```

# Understanding Virtual Method Invocation
Consider the following example:
```java 
class A{
  public String getName(){
    return "A";
  } 
}

class B extends A{
  public String getName(){
    return "B"
  }
}

class Driver{
  public static void main(String[] args){
    A foo = new B();
    System.out.println(foo.getName());
  }
}
```
We get the result ``` B ``` 

- If Java didn't have virtual method invocation, it would determine at compile time that the getName() to be called is the one that belongs to the A class. Since it doesn't, but determines this at runtime depending on the actual class that foo points to, we get the result above, ```B```.

- Notice that instance variables don't work the same way. In this example, the Animal class refers to name. It uses the one in the super class and not the subclass. That is because these type of Animals 

```java
class Animal{
  String name = "unknown";
  public void printName(){
    System.out.println(name);
  }
}

class Lion extends Animal{
  String name = "Leo";
}

class Driver{
  public static void main(String[] args){
    Animal foo = new Lion();
    foo.printName();
  }
}
```
And the output is ```unknown```. The name declared in Lion would only be used if name was referred to from Lion (or a subclass of Lion). 

- One more example to make sure that the exam is not going to trick you. What does the following print?
```java
abstract class Animal{
  public void careFor(){
    play();
  }
  
  public void play(){
    System.out.println("pet animal");
  }
}

class Lion extends Animal{
  public void play(){
    System.out.println("toss in meat");
  }
}

public class PlayWithAnimal{
  public static void main(String[] args){
    Animal animal = new Lion();
    animal.careFor();
  }
}
```

The correct answer is ```toss in meat```. 

# Annotating Overridden Methods
- When you see code that begins with @ symbol, it is an annotation. 
- Annotation is extra information about the program, and it is a type of *metadata*.
- When you see ```@Override``` in the exam, you must check carefully that the method is doing one of three things:
  - Implementing a method for an interface
  - Overriding a superclass method of a class
  - Overriding a method declared in Object, such as ```hashCode, equals``` or ```toString```

# Coding equals, hashCode, and toString

## toString 

## equals

## hashCode

# Working With Enums

## Using Enums in Switch Statements

## Adding Constructors, Fields and Methods

# Creating Nested Classes

## Member Inner Classes

## Local Inner Classes

## Anonymous Inner Classes

## Static Nested Classes
