# Reviewing OCA Collections
The *Java Collection Framework* includes classes that implement List, Map, Queue, and Set. On the OCA, we saw ArrayList.  We also saw arrays in the OCA as in ```int[]```. An array is not part of the Collection Framework. Since sorting and searching are similar these are covered in the exam. OCP is cumulative and that means that you are expected to know how to work with arrays from the OCA. In the following we are going to revise arrays, ArrayLists, wrapper classes, autoboxing, the diamond operator, searching and sorting.

## Array and ArrayList
An arrayList is an object that contains other objects. An ArrayList cannot contain primitives. An array is a built-in data structure that contains other objects or primitives. See examples of both below:

```java
List<String> list = new ArrayList<>();      //empty list
list.add("Fluffy");                         //[Fluffy]
list.add("Webby");                          //[fluffy,Webby]

String[] array = new String[list.size()];   // empty array
array[0] = list.get(1);                     // [Webby]
array[1] = list.get(0);                     // [Webby, Fluffy]
```

Now, review the link created when converting between an array and ArrayList:

```java
String[] array = {"gerbil","mouse"};            // [gerbil,mouse]
List<String> list = Arrays.asList(array)        // returns a fixed size list
list.set(1,"test")                              // [gerbil,test]
array[0] = "new";                               // [new,test]
String[] array2 = (String[]) list.toArray();    // [new, test]
list.remove(1);                                 // Throws UnsupportedOperationException
```

Line 2 converts an array to a List. Since we used asList() to make the convertion the created list is not resizable. Line 5 converts the List back to an array. Finally, the last line shows that list is not resizable because it is backed by the underlying array. 

## Searching and Sorting
```java
int[] numbers = {6,9,1,8};  
Arrays.sort(numbers);                                         //[1,6,8,9]
System.out.println(Arrays.binarySearch(numbers,6));  //1
System.out.println(Arrays.binarySearch(numbers,3));  //-2
```

Line 2 sorts the array because Binary Search assumes the input is sorted. Line 3 prints the index at which match is found. Line 4 prints one less than the negated  of where the requested value would need to be inserted. The number 3 would need to inserted at index 1 (after the number 1 but before the number 6) so index 1. Negating that index give us -1 and substracting 1 gives us -2. 

## Wrapper Classes and Autoboxing 
As shown in the table below each primitive has a correspoding wrapper class. *Autoboxing* automatically converts a primitive  to the corresponding wrapper classes when needed. On the other hand, *Unboxing* converts a wrapper class back to a primitive.

![Wrapper classes table](/img/wrapperClasses.png)

Let's see some code:

```java 
List<Integer> numnbers = new ArrayList<Integer>();
numbers.add(1);
numbers.add(3);
numbers.add(5);
numbers.remove(1);
numbers.remove(new Integer(5));
System.out.println(numbers)
```

The answer is it leaves just ```[1]```. on lines 2 to 4 we add three integer objects to numbers. The one on line 2 relies on autoboxing. Line 5 is tricky, the remove() method is overloaded. One signature takes an int as the index of the element to remove. The other takes as Object that should be removed. On line 5 Java sees a matching signature for int, so it doesn't need autobox the call to the method. Now numbers contain ```[1,5]```. Line 6 calls the other remove() method, but this time it removes the matching object, which leaves us with just ```[1]```. 

Java also converts the wrapper classes to primitives via unboxing:

```java
int num = numbers.get(0);
```

## The Diamond Operator
It is called that because <> looks like a diamond. See an example usage:

```java
import java.util.*;
class Doggies{
  List<String> names;
  Doggies(){
    names = new ArrayList<>();      
  }
  
  public void copyNames(){
    ArrayList<String> copyNames;
    copyNames = new ArrayList<>();
  }
}
```

# Working with Generics
Using generics is type-safe. So that the call doesn't put in the list that we didn't expect. The following does just that:

```java
static void printNames(List list){
  for(int i = 0; i < list.size(); i++){
    String name = (String) list.get(i);   //class cast exception here
    System.out.println(name);
  }
}

public static void main(String[] args){
  List names = new ArrayList();
  names.add(new StringBuilder("Webby"));
  printNames(names);
}

```
The code above throws ClassCastException. Line 10 adds a StringBuilder to list. This is legal because a non generic list can contain anything. However, line 3 is coded to expect a specific class. It casts to String, reflecting this assumption. Since the assumtion is not always correct, it throws a ClassCastException because StringBuilder cannot be cast to java.lang.String.

Generics fixes this by allowing parametrized types. You specify that you want an ArrayList of String objects. Now the compiler has enough information to prevent from causing a runtime exception. So:

```java
List<String> names = new ArrayList<String>;
names.add(new StringBuilder("Webby"));            //DOES NOT COMPILE
```

## Generic Classes
You can introduce generics to your own classes. The syntax for introducing a generic is to declare a formal type parameter in angle brackets. For example:

```java
public class Crate<T>{
  private T contents;
  
  public T emptyCrate(){
    return contents;
  }
  
  public void packCrate(T contents){
    this.content = contents;
  }
}

```

The generic type is available anywhere within the Crate class. When you instantiate the class you tell the compiler what T should be for that particular instance.

:yin_yang: **Naming convetions for Generics** It can be named anything you want but by convention is to use a single uppercased letter to make it obvious that they arent real class names. The most common uses are:
- E for element.
- K for map key.
- V for map value.
- N for number.
- T for generic data type.
- S,U,V and so forth for mutiple generic types.

For example, suppose an Elephant class, and we are moving it to a much larger zoo.

```java
Elephant elephant = new Elephant();
Crate<Elephant> crateForElephant = new Crate<>();
crateForElephant.packCrate(elephant);
Elephant newHome = crateForElephant.emptyCrate();
```

The Crate class can deal with the Elephant object without knowing anything about it. We could have had type in Elephant instead of T when coding Crate. But, what if we wanted to create a Crate for another animal ?

```java
Crate<Zebra> crateForZebra = new Crate<>();
```

Now, we couldn't have simply hard-coded Elephant in Crate because a Zebra is not an Elephant. However, we could have created an Animal superclass or an interface and use that in Crate.

Generics became useful when the classes used as the type param have absolutely nothing to do with each other. For example, we need to ship our 120-pound robot to another city.

```java
Robot joeRobot = new Robot();
Crate<Robot> robotCrate = new Crate();
robotCrate.packCrate(joeRobot);

//ship to St. Louis
Robot atDestination = robotCrate.emptyCrate();

```
As you can see the Crate class works with any type of classes. Before generics, we would have needed Crate to use the Object class for its instance variable, which would have put the burden on the caller of needing to cast the object it receives on emptying the crate. In addition, the Crate class does not need to know the object that goes into it, and those object don't need to know about Crate neither. We aren't needed to implement an interface Crateable or anything like that. 

Geberic classes aren't limited to having a single type parameter. See code below how it uses two generic parameters:

```java
public class SizeLimitedCrate<T,U>{
  private T contents;
  private U sizeLimits;
  
  public SizeLimitedCrate(T contents, U sizeLimit){
    this.contents = contents;
    this.sizeLimits = sizeLimit;
  }
}
```
T represents the type that we are putting in the crate. U represents the unit that we are using to measure the max size for the crate. To use this generic class we can do the following:

```java
Elephant elephant = new Elephant();
Integer maxWeight = 15_000;
SizeLimitedCrate<Elephant, Integer> c1 = new SizeLimitedCrate<>(elephant,maxWeight);
```

:yin_yang: **Type Erasure** Specifying a generic type allows the compiler to enforce proper use of the generic type. For example, specifying the generic type of Crate as Robot is like replacing the T in the Crate class with Robot. Behind the scenes the compiler replaces all references of T in Crate to Object. The process of removing the generic syntax from our code is called *Type Erasure*. Type erasure allows your code to be compatible with older versions of Java that do not contain generics. And then the compiler adds the relevant casts for your code, like below:

```java
Robot r = crate.emptyCrate();

and comiler turns it into

Robot r = (Robot) crate.emptyCrate();
```

## Generic Interfaces
Just like a class, an interface can declare a formal type parameter. For example the interface below uses a generic type as the argument to its ship() method:

```java
public interface Shippable<T>{
  void ship(T t);
}
```
There are three ways a class  can approach implementing this interface. The first is to specify the generic type in the class. The following concrete class says it deals only with robots. 

```java
class ShippableRobotCrate implements Shippable<Robot>{
  public void ship(Robot r){ }
}

```

The next way is to create a generic class. The following concrete class allows the caller to specify the type of the generic:

```java
class ShippableAbstractCrate<U> implements Shippable<U>{
  public void ship(U t)
}
```

The final way, is to not use generics at all. This is the old way of writing code. It generates a compiler warning about Shippable beign a *raw type*, but it does compile. Here the ship() method has an Object parameter since the generic type is not defined:

```java
class ShippableCrate implements Shippable{
  public void ship(Object t){ }
}

```

:yin_yang: **What you can't do with Generic Types** These aren't in the exam.
- *Call the constructor* new T() is not allowed because at runtime it would be an Object.
- *Create an array of that static type* this one is the most annoying, but it makes sense because you'd be creating an array of Objects.
- *Call instanceof*. This is not allowed because at runtime List<Integer> and List<String> look the same to Java thanks to type erasure.
- *Use a primitive type as a generic type parameter*. This isn't a big deal because you can use the wrapper class instead. If you want a type int, just use Integer
- *Create a static variables as generic type parameter*. This is not allowed because the type is linked to the instance of the class.
  
## Generic Methods
Up until now you have seen generic types declared in classes and interfaces. It is also possible to declare them at method level. This is often useful for static methods since they aren't part of the instance . However, they are also allowed on non-static methods as well.

```java
public static <T> Crate<T> ship(T t){
    System.out.println("Preparing " + t);
    return new Crate<T>();
}

```
The method parameter is the generic type T. The return type is a Crate<T>. Before the return type, we declare the formal type parameter of <T>. Unless a method is obtaining the generic formal type parameter from the class/interface, it is specified immediately before the return type of the method. Observe code below:
  
```java
public static <T> void sink(T t){ }
public static <T> T indentity(T t){ return t;}
public static T noGood(T t){ return t; }          //DOES NOT COMPILE        
```

Line 1 shows the formal param type immediately before the return type of void. Line 2 shows the return the type beign the formal param type. It looks weird, but it is correct. Line 3 ommits the formal param type, and therefore it does not compile.

## Interacting with Legacy Code
*Legacy code* is older code. It is usually code that is in a different code that you would normally write today. In this section we are referring to code that was written in version 1.4 and therefore does not use generics. Collections written without generics are also known as *raw collection*. Remember using collections gives us compile time safety. 

```java
class Dragon{}

class Unicorn{}

public class LegacyDragons{
  public static void main(String[] args){
    List unicorns = new ArrayList();
    unicorns.add(new Unicorn());
    printDragons(unicorns);
  }
  
  private static void printDragons(List<Dragon> dragons){
    for(Dragon dragon : dragons){       //ClassCastException
      System.out.println(dragon);
    }
  }
}
```

Now, look at the problem with autoboxing:
```java
public class LegacyAutoboxing{
  public static void main(Stirng[] args){
    List numbers = new ArrayList();
    numbers.add(5);
    int result = numbers.get(0);        //DOES NOT COMPILE
  }
}
```

The good news is that unboxing fails with a compiler error rather than a runtime error. On line 3 we create a raw list. Onm line 4 we try to add an int to the list. This works because Java automatically autoboxes to an Integer. On line 5, we have a problem. Since we are not using generics Java does not know the list contains Integers. It just knows about Objcts and an object cannot be unboxed in a primitive int. To review, the lesson is to be careful when you see code that doesn't use generics.

## Bounds
You might have noticed that generics don't seem useful since they are treated as Object and therefore don't have many methods available. Bounded wildcards solve this by restricting what types can be used in that wildcard position.

A *bounded parameter type* is a generic type that specifies a bound for the generic. Be warned that this is the hardest section in the chapter.

A *wildcard generic type* is an unknown generic type represented with a question mark (?). You can use generic wildcards in three ways. See table below 


## Unbounded Wildcards

## Upper-Bounded Wildcards

## Lower-Bounded Wildcards

## Putting It All Together

# Using Lists, Sets, Maps, and Queues

## Common Collections Methods

### add()

### remove()

### isEmpty() and size()

### clear()

### contains()

## Using the List Interface

## Comparing List Implementations

## Working with List Methods
## Using the Set Interface

## Comparing Set Implementations

## Working with Set Methods

## Using the Queue Interface

## Comparing Queue Implementations

## Working with Queue Methods

## Map

### Comparing Map Implementations

### Working with Map Methods

## Comparing Collection Types

# Comparator vs. Comparable

## Comparable

## Comparator

# Searching and Sorting 

# Additions in Java 8

## Using Method References

## Removing Conditionally 

## Updating All Elements

## Looping through a Collection

# Using New Java 8 Map APIs

## merge

## computeIfPresent and computeIfAbsent


