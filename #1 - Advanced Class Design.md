
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

## Imports

# Using InstanceOf

# Understanding Virtual Method Invocation

# Annotating Overriddden Methods

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
