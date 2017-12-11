
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
