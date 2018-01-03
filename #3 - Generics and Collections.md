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

![Type of bounds](/img/typesOfBounds.png)


## Unbounded Wildcards
An unbounded wildcard represents any data type you use. You use ? when you want to specify that any type is OK with you. For example:
```java
public static void printList(List<Object> list){
  for(Object o: list) System.out.println(o);
}

public static void main(String[] args){
  List<String> keywords = new ArrayList<>();
  kaywords.add("java");
  printList(keywords);                // DOES NOT COMPILE
}
```
Why if String is a subtype of Object? However, List<String> cannot be assigned to List<Object>. I know it doesn't sound logical. But imagine if we could write code like this:
  
```java
List<Integer> numbers = new ArrayList<>();
number.add(new Integer(42));
List<Object> objects = numbers;           
objects.add("forty two");
System.out.println(numbers.get(1));
```

On line 1, the compiler promises that only Integer objects will appear in numbers. If line 3 were to have compiled, line 4 would break that promise by putting a String in there since numbers and objects are references to the same object. Good thing that the compiler prevents this.

:yin_yang: **Storing wrong Objects - Arrays vs ArrayLists**
We can't write ```List<Object> l = new ArrayList<String>();``` so you might think that Java is protecting us from doing  
```java
Integer[] numbers = { new Integer(42)};
Object[] objects = numbers;
objects[0] = "forty two";               //throws ArrayStoreException
```
Although the code above comiles, it throws an expection at runtime. With arrays Java knows the type allowed in the array. Just because we've assigned an ```Integer[]``` to an ```Object[]``` doesn't change the fact that Java knows it is really an ```Integer[]```. Due to type erasure there is no such protection for an ArrayList. At runtime the ArrayList does not know what is allowed in it. Therefore Java uses the compiler to prevent this situation from happening. So whay Java doesn't allow this knowledge to ArrayList? the reason is backwards compalibility.

Going back to the example, we cannot assign List<String> to List<Object>. Fine. What we need is a List of "Whatever" that's what List<?> is. The following does what we expect:

```java
public static void printList(List<?> list){
  for(Object x: list) System.out.println(x);
}

public static void main(String[] args){
  List<String> keywords = new ArrayList<>();
  keywords.add("java");
  printList(keywords);
}

```

printList() takes any type of list as a parameter. keywords is of type List<String>. We have a match! List<String> is a list of anything. "Anything" just happens to be a String here
  
## Upper-Bounded Wildcards
Let's try to write a method that adds up the total of a list of numbers. We've established that a generc type cannot use a subclass.

ArrayList<Number> list = new ArrayList<Integer>();        //DOES NOT COMPILE
  
Instead, we need to use a wild card:

List<? extends Number> list = new ArrayList<Integer>();
  
The upper-bounded wildcard says that any class that extends Numnber of Number itself can be used as the formal parameter type:

```java
public static long total(List<? extends Number> list){
  long count = 0;
  for(Number number : list){
    count += number.longValue();
  }
  return count;
}

```
Remember how we kept saying that type erasure makes Java think that a generic type is an Object ? That is still happening here. Java converst the code to something like this:

```java
public static long total(List list){
  long count = 0;
  for(Object obj : list){
    Number number = (Number) obj;
    count += number.longValue();
  }
  return count;
}

```

Something interesting happens when we work with upper bounds and unbounded wildcards. The list becomes immutable.

```java
static class Sparrow extends Bird{ }
static class Bird{ }

public static void main(String[] args){
  List<? extends Bird> birds = new ArrayList<Bird>();
  birds.add(new Sparrow());                 // DOES NOT COMPILE
  bords.add(new Bird());                    // DOES NOT COMPILE
}
```
The problem comes from the fact that Java does not know what type List<? extends Bird> really is. It could be List<Bird> or List<Sparrow>. Line 6, doesn't compile because we can't add a Sparrow to Lis<Bird> and line 7 doesn't compile because we can't add a Bird to List<Sparrow>. From Java point of view both scenarios are equally possible so netither is allowed.  
  
Now let's try an example with an interface. We have an interface and two classes that implement it:

```java
interface Flyer{ void fly();}

class HangGlider implements Flyer { public void fly(){ }}

class Goose implements Flyer { public void fly(){ }}

```
We also have two methods that use it. 

```java
private void anyFlier(List<Flyer> flyer){ }

private void groupOfFlyers(List<? extends Flyer> flyer){}
```
Notice that we used keyword extends rather than implements. **Upper bounds are like anonymous classes in that they use extends regardless of whether we are working with a class or an interface.** 

## Lower-Bounded Wildcards
Let's try to write code that adds a stirng "quack" to two lists:

```java
List<String> strings = new ArrayList<String>();
strings.add("tweet");
List<Object> objects = new ArrayList<Object>(strings);
addSound(strings);
addSound(objects);
```

The problem is that we want to pass a List<String> and a List<Object> to the same method.First, make sure that you understand why the first 3 options don't work.
  
![Why we need lower bound](/img/whyLowerBound.png)

To solve this problem, we need to use lower bounds:

public static void addSound(List<? super String> list){   // lower bound
  list.add("quack");
}

witha lower bound, we are telling Java that the list will be a list of String objects or a list of some objects that are  superclass of String. Either way, it is safe to add a String to that list.

### Understanding Generic Supertypes
when you have subclasses and superclasses, lower bounds can get tricky:

```java
List<? super IOException > exceptions = new ArrayList<Exception>();
exceptions.add(new Exception());      //DOES NOT COMPILE
exceptions.add(new IOException());
exceptions.add(new FileNotFoundException());

```
Line 1 reference a List that can be List<IOException> or List<Exception> or List<Object>. Line 2 does not compile because we could have a list<IOException> and and Exception object wouldn't fit in there.

Line 3 and 4 are fine. IOException and FileNotFoundException can be added to any of those types. 
  
## Putting It All Together
Let's see some examples to put what we learnt into practice.

```java
class A{}

class B extends A{}

class C extends B{}
```

Can you figure out why the following do or don't compile?

```java
6:  List<?> list1 = new ArrayList<A>();

7:  List<? extends A> list2 = new ArrayList<A>();

8:  List<? super A> list3 = new ArrayList<A>();

9:  List<? extends B> list4 = new ArrayList<A>();         //DOES NOT COMPILE

10: List<? super A> list5 = new ArrayList<A>();

11: List<?> list6 = new ArrayList<? extends A>();         //DOES NOT COMPILE

```
Line 6 creates creates an ArrayList that can hold instances of class A. It uses an unbounded wildcard. Any generic can be referenced from an unbounded wildcard, making this OK. 

Line 7 is OK it uses a upper-bounded wildcard. You could have List<A>, List<B>, List<C>.
  
Line 8 is also OK. This time is a lower-bounded wildcard. The lowest to can reference is A.

Line 9 has upper bounded wildcard that allows List<B> and List<C>. Since you have List<A> it does not compile.
  
Line 10, has a lower-bounded wildcard, which allows a reference to List<A>, List<B> or ArrayList<Object>

Finally, line 11 allows a reference to any generic type since it is unbounded wildcard. The problem is that you need to know what that wildcard will be when instantiating the ArrayList. 

Now, same questions for the following method:

```java
<T> T method1(List<? extends T> list){
  return list.get(0);
}

```

method1() is a normal use of generics. For example you could call it with List<String> param and have it return a String. Or you could call it with a List<Number> param and have it return a Number. 
  
Now, what is wring with this code?

```java
<T> <? extends T> method2(List<? extends T> list){  //DOES NOT COMPILE
  return list.get(0)
}

```

method2() does not compile because the return type isn't actually a type. Since you are the one writting the method you should know what method is suppose to return. You don't get to specify this as a wild card

Now this is extra tricky:

```java
<B extends A> B method3(List<B> list){
  return new B();             // DOES NOT COMPILE  
}

```

method3() does not compile <B extends A> says that you want to use B as type param for this method and that it has to extend A. The trick is that B is also the name of a class. Within the scope of the method, B can represent classes A, B or C because all extend A class. Since B no longer refers to B class in the method, you cannot instantiate it.
  
Generics bounds are very tricky. Come back tomorrow when you have a fresh mind and re-read.
  
# Using Lists, Sets, Maps, and Queues
A *collection* is a set of objects contained is an single object. The *java collections framework* is a set of subclasses in java.util for storing collections. There are four main interfaces in the Java Collections Framework

- **List**: A list is an ordered collection of elements that allows duplicate entries. Elements in a list can be accessed by an int index.
- **Set**: A set is a collection that does not allow duplicate entries.
- **Queue**: A queue is a collection that orders its elements in a specific order for processing. A typical queue process in FIFO order. First-in First-out but other orderings are possible
- **Map**: A map is a collection that maps keys to values, with no duplicate keys allowed. The elements in a map are key/value pairs.


In the figure below, notice that Map does not implement the Collection interface. It is considered part of the Java Collections Framework even though it doen't implement the Collection interface.


![Java Collections Framework](/img/javaCollectionsFramework.png)



## Common Collections Methods
The Collection interface contains useful methods for working with list, sets and queues. We will also cover maps. In the following sections. 

### add()
The add() method inserts a new element into the Collection and returns whether it was successful. The method signature is

```java boolean add(E element)```

Example of usage:

```java

List<String> list = new ArrayList<>();
System.out.println(list.add("Sparrow"));    //true
System.out.println(list.add("Sparrow"));    //true

Set<String> set = new HashSet<>();          //true
System.out.println(set.add("Sparrow"));    //true
System.out.println(set.add("Sparrow"));    //false

```

A list allows duplicates, making the return value true each time. A Set does not allow duplicates. In the last line, we tried to add a duplicate and it returned false.

### remove()
The remove() method removes a single matching value in the Collection and returns whether it was successful. The method signature is:

```java
boolean remove(Object object)
```

This time, the boolean return value tells us whether a match was removed. The following shows how to use this method

```java
List<String> birds = new ArrayList<>();     // 
birds.add("hawk");                          //[hawk]
birds.add("hawk");                          //[hawk, hawk]

System.out.println(birds.remove("panda"))   //prints false
System.out.println(birds.remove("hawk"))    //prints true
System.out.println(birds)                    //[hawk]
```

The first remove tries to remove an element that is not in birds. It returns false because no such element is found. The next remove removes hawk and returns true because there is a match. Notice that it only removes a single match.

Since calling remove() with an int uses an index, an index that doesn't exist will throw an exception. For example birds.remove(5); throws an IndexOutOfBoundsException. Remenber there are overloaded remove methods. One takes the element to remove. The other takes the index of the element to remove.

### isEmpty() and size()
The isEmpty() and size() methods look at how many elements are in the Collection. The method signature is:

```java
boolean isEmpty()

int size()
```

The following shows how to use these methods:

```java
System.out.println(birds.isEmpty());      //true
System.out.println(birds.size());         //0
birds.add("hawk");                        //[hawk]
birds.add("hawk");                        //[hawk,hawk]
System.out.println(birds.isEmpty());      //false
System.out.println(birds.size());         //2
```
At the beggining, birds has a size of 0 and is empty. After we add elements, the size becomes positive and it is no longer empty.

### clear()
The clear() method provides an easy way to discard all elements of the Collection. The method singature is:

```java
void clear();
```

The following shows how to use the method:

```java
List<String> birds = new ArrayList<>();
birds.add("hawk");                            //[hawk]
birds.add("hawk");                            //[hawk,hawk]
System.out.println(birds.isEmpty());          //false
System.out.println(birds.size());             //2
birds.clear();                                //[]
System.out.println(birds.isEmpty());          //true
System.out.println(birds.size());             //0
```

After calling clear(), birds is back to beign an empty ArrayList of size 0.

### contains()
The contains() method checks if a certain value is in the Collection. The method signature is:

```java
boolean contains(Object object)
```

The following shows how to use this method:

```java
List<String> birds = new ArrayList<>();
birds.add("hawk");                               //[hawk]
System.out.println(birds.contains("hawk"));      //true
System.out.println(birds.contains("panda"));     //false
```
This method calls equals() on each element of the ArrayList to see if there are any matches.

## Using the List Interface
You use a list when you want an ordered collection that can contain duplicate entries. Items can be retrieved and inserted at specific position based  on an int index much like an array. List is like the go to data type. The main point about lists is that they are odered and allow duplicates.

### Comparing List Implementations
An **ArrayList** is like a resizeable array. When elements are added, the ArrayList automatically grows. The main benefit is that it can look up any elements at constant time. Adding or removing an element is slower than accessing an element.

:yin_yang: **Big O Notation** This is an order of magnitude and it measures the performance of an algorithm. The following are the most common big O notation values:
- *O(1) - constant time*: It doens't matter how large the collection is, the answer will always take the same time to return. 
- *O(log n) - logarithmic time*: A logarithm is a mathematical function that grows much more slowly that the data size. Binary Search runs in logarithmic time.
-*O(n) - Linear time*: The performance will grow linearly with respect to the size of the collection. 
- *O(n^2) - n square time*: Code that has nested loops where each loop goes through the data takes n square time.

A **LinkedList** is special because it implements both List and Queue. It has all of the methods of List. It also has additional methods to facilitate addding or removing from beginning and/or end of list. The main benefits are that you can access, add or remove from the beginning and end of the list in constant time. 

A **Stack** is a data structure where you add and remove elements from the top of the stack. Stack hasn't been used for ages. If you need a stack, use an **ArrayDeque** instead.

### Working with List Methods
The methods in the List interface are for working with indexes.

![List Methods](/img/listMethods.png)

The following statements demonstrate these basic methods:

```java
List<String> list = new ArrayList<>();
list.add("SD");                         //[SD]
list.add(0,"NY");                       //[NY,SD]
list.set(1,"FL");                       //[NY,FL]
list.remove("NY");                      //[FL]
list.remove(0);                         //[]
```

Let's look at one more example that queries the list:

```java
lis.add("OH");                    //[OH]  
list.add("CO");                   //[OH,CO]
list.add("NJ");                   //[OH,CO,NJ]
String state = list.get(0);       //OH
int index = list.indexOf("NJ");   //2  
```
The output would be the same if you tried these examples with LinkedList, Vector or Stack. Although the code would be less efficient, it wouldn't be noticeable until you have very large lists.

## Using the Set Interface
You use a set when you don't want to allow duplicate entries. For example, you might want to keep track of the unique animals that you want to see in the zoo. You are not concerned in the order you want to see the animals but there isn't time to see them more than once.

The main thing that all set have in common is that they do not allow duplicates.

### Comparing Set Implementations
A **HashSet** stores its elements in a has table. This means that it uses the hashCode() method to retrieve them more efficiently. The benefit is that adding elements and checking if an element is in the set both have constant time. The trade off is that you lose the order in which they are inserted.

A **TreeSet** stores its elements in a sorted tree structure. The main benefit is that the tree is always in sorted order. The tradeoff is that adding and checking if an element is present are both *O(log n)*. TreeSet implements special interface called NavigableMap.


![Examples of a HashSet and TreeSet](/img/hashSetAndTreeSet.png)

### Working with Set Methods
The Set interface doesn't add any extra methods that you need to know for the exam. You need to know how sets behave with respect to the traditional Collection methods. You akso need to know the difference between the types of sets. 

Let's start with HashSet:

```java
Set<Integer> set = new HashSet<>();
boolean b1 = set.add(66);                   //true
boolean b2 = set.add(10);                   //true
boolean b3 = set.add(66);                   //false
boolean b4 = set.add(8);                    //true
for(Integer i: set) System.out.println(i)   //66,8,10

```

The add method should be straight forward. They return true unless the Integer is already in the set. Line 4, returns false because the number is already in the set, it must preserve uniqueness. Notice that when we print the elements they are not in the same order as we inserted them programatically. Remember the equals() method is used to determine equality and the hashCOde() is used to determine which bucket to look inso that Java doesn't need to look through the whole set to find out if an object is there.

If we run the same code as before with a TreeSet the boolean values are the same the only difference is that TreeSet print the values in the natural sorted order.

:yin_yang: **The NavigableSet Interface**  TreeSet implements the NavigableSet interface. This interface provides some interesting methods. Their signature is as follows:

![The NavigableSet interface ](/img/navigableSetInterface.png)

These java methods were added in Java 6, so you are more than likely to come across them
```java
NavigableSet<Integer> set = new TreeSet<>();
for(int i = 1; i <= 20; i++) set.add(i);
System.out.println(set.lower(10));        //9
System.out.println(set.floor(10));        //10
System.out.println(set.ceiling(20));      //20          
System.out.println(set.higher(20));       //null
```

## Using the Queue Interface
You use a queue when elements are added and removed in a specific order. Queues are typically used for sorting elements prior to processing them. Unless stated otherwise, a queueu is assumed to be FIFO (First-in First-out). Implementation may change and you can envision LIFO as well (Last-in First-out ).

### Comparing Queue Implementations
A double-ended queue is different from a regular queue in that you can insert and remove elements from both the front and back of the queue. The main benefit of using LinkedList is that it implements both List and Queue interfaces. The tradeoff is that it isnet as efficient as a "pure" queue .

An ArrayDeque is a "pure" double-ended queue. It was introduce in Java 6, and stores its elements in a resizable array. The main benefit of ArrayDeque is that it is more efficient than a LinkedList 

### Working with Queue Methods
The ArrayDeque contains many methods. Luckily, they are only seven methods that you need to know in addition to the inherited Collection ones. See the table below
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


