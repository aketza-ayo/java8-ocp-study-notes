NIO.2 is an acronym that stands for the second version of the Non-blocking Input/Output API, and it is sometimes referred to as "New I/O".

# Introducing NIO.2
Java introduced a replacement for java.io streams in Java 1.4 called Non-blocking I/O or NIO for short. The NIO API introduced the concept of buffers and channels in place of java.io streams. The basic idea is that you load the data from a file channel into a temporary buffer that, unlike byte streams, can be read forward and backward without blocking on the underlying resource. Unfortunately, the NIO API was never particularly popular, so much so that nothing for the original version NIO will be on the OCP exam. This book is geared towards teaching you NIO.2. Java 7 introduced the NIO.2 API. While the NIO API was intended to a replacement for java.io streams, the NIO.2 API is actually a replacement for the java.io.File class and related interactions. The goal of the NIO.2 API implementation is to provide a more intuitive, more feature rich API for working with files. As you shall see in this chapter, it also provides a number of notable performance improvements over the exisiting java.io.File class.

## Introducing Path
The java.nio.file.Path interface, is the primary entry point for working with the NIO.2 API. A Path object represents a hierarchical path on the storage system to a file or a directory. In this manner, Path is a direct replacement for the legacy java.io.File class, and conceptually contains many of the same properties. Unlike the File class, the Path interface contains support for symbolic links. A *symbolic link* is a special file within an operating system that servers as a reference or pointer to another file or directory. The NIO.2 API includes full support for creating, detecting, and navigating symbolic links within the file system.

### Creating Instances with Factory and Helper Classes
For example you can create an instance of a Path interface using a static method available in the Path's factory class. Note the 's' at the end of Path class to distunguish it from the Path interface. The reason why Path is an interface and not a class is that creating a file or a drectory is considered a file system dependant task in NIO.2. When you obtain a Path object from the default file system in NIO.2, the JVM gives you back an object that unlike java.io.File transparently handless system specific details for the current platform. If you didn't use the factory pattern to create an instance, you would have to know what the underlying file system was and use this in every create method.

NIO.2 also includes helper classes such as java.nio.file.Files, whose primary purpose is to operate on instances of Path objects. Helper or utility classes are similar factory classes in that they are often composed primarily of static methods that operate on a particular class. They differ in that helper classes are focused on manipulating or creating new objects from existing instances, whereas factory classes are focused primarily on object creation. You should become confortable with this paradigm, if you are not already, as most of your interactions with NIO.2 API will require accessing at least two classes: an interface and a factory helper class. As a guideline for this section, we present the NIO.2 class and interface relationship in the figure below.

![NIO2 class and interface Relationships](img/NIO2classInterfaceRelationship.png)

## Creating Paths
Since path is an interface you need a factory class to create instances of one. The NIO.2 API provides a number of classes and methods that you can use to create  Path objects, which we will review in this section.

### Using the Path Class
The simplest and most straightforward way to obtain a Path object is using the java.nio.files.Paths factory class, or Paths for short. To obtain a reference to a file or directory, you would call the static method Paths.getPath(String) method, as shown in the following example:

```
Path path1 = Paths.get("pandas/cuddly.png");

Path path2 = Paths.get("c:\\zooinfo\\November\\employees.txt");

Path path3 = Paths.get("/home/zoodirector");

```
The first example creates a Path reference to a relative file in current working directory. The second example creates a Path reference to an absolute file in a Windows-based system. The third example creates a Path reference  to an absolute directory in a Linux or Mac based system.

You can also create a Path usoing the Paths class using a vararg of type String such as Paths.get(Stirng, String...). This allows you create a Path from a list of String valuesin which the operating system-dependant path.separator is automatically inserted between elements. As you may remember from Chapter 8, System.getProperty("path.separator") can be used to get the operating system dependent file separator from the JVM. That said, most JVM implementations support both forward and backward slashes regadless of the file system, allowing the same code to run on multiple operating systems without having to rewrite the slashes.

```
Path path1 = Paths.get("pandas", "cuddly.png");

Path path2 = Paths.get("c:","zooinfo","November","employees.txt");

Path path3 = Paths.get("/","home","zoodirector");

```

These examples are rewritten of your previous set of Path examples, using the param list of String values instead of a single String values. The advantage of using this overloaded method is that it is more robust when manually constructing path values, as it inserts the proper separator for you.

Note that be wary of Path vs Paths on the exam. Java is fond of using one name for the class and the plural form of the name for the for the factory or the helper class. When you see questions with Path or Paths on the exam, be sure that the class reference and usage are correct. For example, the following usage is incorrect and will not compile:

```
Paths path1 = Paths.get("/alligator/swim.txt");     //DOES NOT COMPILE

Path path2 = Path.get("/crocodile/food.csv");       //DOES NOT COMPILE

```
The key point to remeber is that the singular for of Path represents the instance with which you want to work, whereas the plural form of Paths is the factory classes containing methods for creating Path instances.

Another way to construct a Path using the Paths class is with URI value. A *uniform resource indetifier (URI)* is a string of characters that indetify a resource. It begins with a schema that indicates the resource type, followed by a path value. Examples of schema values includes file://, http://, https://, and ftp://. The java.net.URI class is used to create and manage UNI values.

```
Path path1 = Paths.get(new URI("file://pandas/cuddly.png"));    //THROWS EXCEPTION AT RUNTIME

Path path2 = Paths.get(new URI("file:///c:/zoo-info/November/employees.txt"));

Path path3 = Paths.get(new URI("file:///home/zoodirectory"));

```

These examples show how the Paths.get(URI) method can be used to obtain a reference to a URI-based resource. Notice that these are actually rewrites of our earlier examples, as we can use URI values for both local and network paths. The first example actually throws an exception at runtime, as URIs must reference absolute paths at runtime. The URI class. The UNI class does have an isAbsolute() method, although this is related to whether or not UNI has a schema, not the file location.

We now present two additional methods that uese other types of non-local file system schemas. For the exam, you do not need to know the syntax of these schemas, but you should be aware that they exist:

```
Path path4 = Paths.get(new URI("http://www.wiley.com"));

Path path5 = Paths.get(new URI("ftp://username:password@ftp.the-ftp-server.com"));

```

Note that the constructor new URI(String) does not throw a checked URISyntaxException, which would have to be caught in any application where the previous code snippets are used. Finally the Path interface also contains a reciprocal method toUri() for converting a Path instance back to a URI instance, as shown in the following sample of code:

```
Path path4 = Paths.get(new URI("http://www.wiley.com"));
URI uri4 = path4.toUri(); 
```

### Accesing the Underlying FileSystem Object
The Paths.get() method used throughout the previous example is actually shorthand for the class java.nio.file.FileSystem method getPath(). The FileSystem class has a protected constructor, so we use the plural FileSystem factory class to obtain an instance of FileSystem, as shown in the following example code:

```
Path path1 = FileSystems.geDefault().getPath("pandas/cuddly.png");

Path path2 = FileSystems.getDefault().getPath("c:","zooinfo", "November","employees.txt");

Path path3 = FileSystems.getDefault().getPath("/home/zoodirector");

```

Again, we are able to rewrite our previous set of examples, with this code behaving in the exact same manner as before. While most of the time we want access to a Path object that is within the local file system, the FileSystems factory class does give us the ability to connect to a remote file system, as shown in the following sample code:

```
FileSystem fileSystem = FileSystems.getFileSystem(new URI("http://www.selikoff.net"));

Path path = fileSystem.getPath("duck.txt");
```

This code is useful when we need to construct Path objects frequently for a remote file system. The power of the NIO.2 API here is that it lets us rely on the default file system for files and directories as before, while giving us the ability to build more complex applications that reference external file systems.

### Working with Legacy File Instances
When Path was added in Java 7, the legacy java.io.File class was updated with a new method, toPath(), that operates on an instance of File variable.

```
File file = new File("pandas/cuddly.png");
Path path = file.toPath();

```
For backwards compatibility, the Path interface also contains a method toFile() to return a File instance:

```
Path path = Paths.get("cuddlt.txt");
File file = path.toFile();

```

As you can see the Java API is quite flexible, and it allows easy conversion between legacy code using the File class and newer code using Path. Although Java suppors both methods for working with files, it is generally recommended that you rely on the Path API in your applications going forward as it is more feature rich and has built-in support for various file systems and symbolic links.

# Interacting with Paths and Files
Now that we've covered how to obtain an instance of the Path object, you might ask, what can we do with it? The NIO.2 API provides a rich plethora of methods and classes that operate on Path objects - far more that we are available in the java.io API. We will discuss the methods that you should know for the exam in this section.

**Path Object vs Actual File** One thing to keep in mind when reading this section is that a Path object is not a file but a prepresentation of a location within the file system. In this manner, most operations available in the Path and Paths classes can be accomplished  regardless of whether the underlying file that the Path object references actually exists. For example, retrieving the parent or root directory of a Path object does not require the file to exists, although the JVM may access the underlying file system to know how to process the path information.

As you shall see in this section, a handful of operations in the Path and Paths classes, such as Path.toRealPath() do require the file to exist and will throw a checked execption if the file is not available.

## Providing Optional Arguments
Throuout this section, we introduce numerous methods for interacting with files and directories in NIO.2. Many of the mehods in the NIO.2 API that interact with real files and directories take additional option flags in the form of a vararg. For the exam, you do not need to memorize which of the dozens of NIO.2 methods take which optional arguments, but you should be able to recognize what they do when you see them on the exam. The table below lists the values that you should know for the exam. Note that these descriptions apply to both files and directories. If you are not familiar with the operations to which these attributes apply, don't worry; we'll explain them later in this chapter.

![Common Optional Arguments in NIO.2](img/commonOptionalArgumentsNIO2.png)

For simplicity as well as better readability, we purposely omit the enum class names to which the values belong throughout the text, although we do include them in any practice questions. For example, the copy methods take a list of CopyOption interface values, of which StandardCopyOption is an enum that implements the intreface and includes StandardCopyOption.COPY_ATTRIBUTES as an option. As we said, for simplicity, we omit these details as the exam won't require you to understand this relationship. The ATOMIC_VALUE might be new to you. An *atomic operation* is an operation that is performed as a single indivisable unit of execution, which appears to the rest of the system as occurring instantaneously. Furthermore, an atomic move is one in which any process monitoring the file system never sees an incomplete or partially written files. If the file system does not support this feature , an AtomicMoveNotSupportedException will be thrown. For the reminder of the chapter, we leave out these enumn values in the method definitions so that you can focus on the core functionality. For the exam, you should understand their effect if you see them providede to a method.    

## Using Path Objects
The Path interface includes numerous methods for using Path objects. You have alrady seen two of them, toFile() and toUri(), used to convert Path objects to other types of resources. Many of the methods in the Path interface transform the path value in some way and return a new Path object, allowing the method to be chained. We demonstrate chaning in the following example, the details of which we will discuss in this section of the chapter.

```
Paths.get("/zoo/../home").getParent().normalize().toAbsolutePath();
```
If you start to feel overwhelmed by the number of methods available in the Path interface, just remember: the function of many of them can be inferred by their method name, such as getParent(), getNameCount(), toAbsolutePath() and so on. In this section, we organize the methods by related functionality.

### Viewing the Path with toString(), getNameCount(), and getName()
The Path interface contains three methods to retrieve basic information about the path representative. The first method, toString() returns an String representation of en entire path. In fact, it is the only method in the Path interface to return a String. Most of the other methods that we will discuss in this section return a new Path object. The second and third methods, getNameCount() and getName(int) are often used in conjunction to retrieve the number of elements in the path and and a reference to each element, respectively. For greater compatibility with other NIO.2 methods, the getName(int) method returns the component of the Path as a new Path object rather than a String. 

The following sample code uses these methods to retrieve path data:

```
Path path = Paths.get("/land/hippo/harry.happy");
System.out.println("The path name is: " + path);

for(int i=0; i < path.getNameCount(); i++){
  System.out.println("Element " + i + " is: " + path.getName(i));
}

```

As you might remember from our discussion of PrintStream/PrintWriter in Chapter 8, printing an object automatically invokes the object's toString(). The output of this code snippet is the following:

```
The Path Name is : /land/hippo/harry.happy
  Element 0 is: land
  Element 1 is: hippo
  Element 2 is: harry.happy
```

Notice that the root element / is not included in the list of names. If the path object represents the root elements itself, then the number of names in the Path objects returned by getNameCount() will be 0. What if we ran the preceding code using the relative path land/hippo/harry.happy ? the output would be as follows:

```
The path name is: land/hippo/harry.happy
  Element 0 is: land
  Element 1 is: hippo
  Element 2 is: harry.happy
```
Notice that the individual names are the same. For the exam, you should be aware that the getName(int) method is zero-indexed, with the file system root executed from the path components

### Accesing Path Components with getFileName(), getParent(), and getRoot()
The Path interface contains numerous methods for retrieving specific information subelements of a Path object, returned as Path objects themselves. The first method, getFileName() returns a Path  instance representing the filename, which is the farthest element from the root.  Like most methods in the Path interface, getFileName() returns  a Path instance representing the parent paths or null if there us no such parent. If the instance of the Path object is relative, this method will stop at the top-level element defined in the Path object. In other words, it will not traverse outside the working directory to the file system root.
The last method getRoot() returns the root element for the Path object or null if the Path object is relative. We represent a sample application that traverses absolute and relative Path objects to show how each handles the root differently:

```java
import java.nio.file.*;

public class PathFilePathTest{
  public static void printPathInformation(Path path){
    
    System.out.println("File is: " + path.getFileName());
    System.out.println("Root is: " + path.getRoot());
    
    Path currentPath = path;
    while((currentParent = currentParent.getParent()) != null){
      System.out.println(" current parent is: " + currentParent);
    } 
  }
  
  public static void main(String[] args){
    printPathInformation(Paths.get("/zoo.armadillo/shells.txt"));
    System,out.println();
    printPathInformation("armadillo/shells.txt");
  }
}

```
The while loop in the printPathInformation() method continues until getParent() returns null. This sample application produces the following output:

```
File name is: shells.txt
Root is: /
  Current parent is: /zoo/armadillo
  Current parent is: /zoo
  Current parent is: /


File name is: shells.txt
Root is: null
  Current parent is: armadillo

```

Reviewing the sample output, you can see the difference in the behaviour of getRoot() on absolute and relative paths. Also notice that traversing the second path stopped at the top of the relative directory. As you can see in the example, it does not traverse relative directories outside of the working directory.

### Checking Path Type with isAbsolute() and toAbsolutePath()
The Path interface contains two methods for assisting with relative and absolute paths. The first method, isAbsolute() returns true if the path the object references is absolute and false if the path the object references is relative. 

The second method, toAbsolutePath() converts a relative Path object to an absolute Path object by joining  it to the current working directory. If the path object is already absolute, then the method just returns an equivalent copy of it.

The following code snippets shows usage of both of these methods:

```
Path path1 = Paths.get("C:\\birds\\egert.txt");
System.out.println("Path1 is absolute?" + path1.isAbsolute());
System.out.println("Absolute Path1: " + path1.toAbsolutePath());


Path path2 = Paths.get("birds/condor.txt");
System.out.println("Path2 is absolute?" + path2.isAbsolute());
System.out.println("Absolute Path2: " + path2.toAbsolutePath());
```
The output for the code snippet is shown in the following sample code. Since the precise output is file system dependent, we will treat the first example as beign run on windows based system, whereas the second example is run in a Linux or Mac based system with the current working directory of /home·

```
Path1 is Absolute? true
Absolute Path1: C:\birds\egret.txt

Path2 is Absolute? false
Absolute Path2 /home/birds/condor.txt

```

Keep in mind that if the Path object already represents an absolute path, then the putput is new Path object with the same value. As discussed earlier in this chapter, absolute and relative path types are actually file system dependent. In fact, you might be surprised by the output of the following lines of code on various operating systems:

```
System.out.println(Paths.get("/stripes/zebra.exe").isAbsolute());

System.out.println(Paths.get("c:/goats/Food.java").isAbsolute());

```
Although the first line outputs true on a Linux or Mac based system, it outputs false on a windows based system since it is missing a drive letter prefix. IN the same manner, the second path outputs true on Windows but false on a linux or mac systems, as it is missing the root slash /.

### Creating a New Path with subpath()
The method subpath(int,int) returns a relative subpath of a Path object, referenced by an inclusive start index and an exclusive end index. It is useful for constructing a new relative path from a particular parent path element to another parent path element, as shown in the following example.

```
Path path = Paths.get("/mammal/carnivore/raccoon.image");
System.out..println("Path is " + path);

System.out.println("Subpath from 0 to 3 is: " + path.subpath(0,3));
System.out.println("Subpath from 1 to 3 is: " + path.subpath(1,3));
System.out.println("Subpath from 1 to 2 is: " + path.subpath(1,2));

```

You might notice that the subpath() and getName(int) methods are similar in that they both return a Path object that represents a component of an exisiting Path. The difference is that the subpath() method may include multiple path components, whereas the getName(int) method only includes one. The output of this code snippet is the following:

```
Path is : /mammal/carnivore/raccoon.image
Subpath from 0 to 3 is: mammal/carnivore/raccoon.image
Subpath from 1 to 3 is: carnivore/raccoon.image
Subpath from 1 to 2 is: carnivore
```

This code demonstrates that the subpath(int, int) method does not inlcude the root of the file. Notice that the 0-indexed element is mammal in this example and not the root directory: therefore, the maximum index that can be used is 3. The following two examples both throw java.lang.IllegalArgumentException at runtime:

```
System.out.println("Subpath from 0 to 4 is:" + path.subpath(0,4));  //THROWS EXCEPTION AT RUNTIME
System.out.println("Subpath from 1 to 1 is:" + path.subpath(1,1));  //THROWS EXCEPTION AT RUNTIME

```

The first example throws an exception at runtime, since the maximum index value allowed is 3. The second example throws an exception since the start and end indexes are the same, elading to an empty value.

## Using Path Symbols
Many file systems support paths that contain relative path information in the form of path symbols. For example, you might want a path that refers to the parent directory, regardless of what a current directory is. In this scenario the double period value .. can be used to reference the parent directory. In addition, the single period value . can be used to reference the current directory within a path.

For example, the path value ../bear.txt refers to a file named bear.txt in the parent of the current directory. Likewise the path value ./penguin.txt refers to a file named penguin.txt in the curent directory. These symbols can also be combined for greater effect. For example, ../../lion.data refers to a file lion.data that is two directories up from the current working directory. 

### Delivering a Path with relativize()
The Path interface privides a method relativize(Path) for constructing the relative Path from one Path object to another. Consider the following relative and absolute path examples using the relativize() method.

```
Path path1 = Paths.get("fish.txt");
Path path2 = Paths.get("birds.txt");
System.out.println(path1.relativize(path2));
System.out.println(path2.relativize(path1));
```

The code snippet produces the following output when executed:

```
..\birds.txt
..\fish.txt
```
If both path values are relative, then the relativize() mehod computes the paths as if they are in the same current working directory. Notice that ../is included at the start of the first set of examples. Since our path value points to a file, we need to move to parent directory that contains the file.

Alternatively, if both path values are absolute, then the method computes the realtive path from one absolute location to another, regarless of the current working directory. The following example demonstrates this property:

```
Path path3 = Paths.get("E:\\habitat");
Path path4 = Paths.get("E:\\sanctuary\raven");
System.out.println(path3.relativize(path4));
System.out.println(path4.relativize(path3));
```

This code snippet produces the following output when executed:

```
..\sanctuary\raven
..\..\habitat
```

In this set of examples, the two path values are absolute, and the relativize() method constructs the relative path between the two absolute path values within the file system. Note that the file system is not accessed to perform this comparison. For example, the root path element E: may not exist in the file system, yet the code would execute without issue since Java is referencing the path elements and not the actual file values.

The relativize() method requires that both paths be absolute or both relative, and it will throw an IllegalArgumentException if a relative path value is mixed with an absolute path value. For exaample, the following would thorw an exception at runtime.

```
Path path1 = Paths.get("/primate/chimpanzee");
Path path2 = Paths.get("bananas.txt");
path1.relativize(path2);      // THROWS AN EXPECTION AT RUNTIME

```

On windows based systems, it also requires that if absolute paths are used, then both paths must have the same root directory or drive letter. For example, the following would also throw an IllegalArgumentException at runtime in a Windows based system since they use different roots:

```
Path path3 = Paths.get("c:\\primate\\chimpanzee.txt");
Path path4 = Paths.get("d:\\storage\\bananas.txt");
path3.relativize(path4);    //THROWS EXCEPTION AT RUNTIME
```

### Joining Path Objects with resolve()
The Path interface includes a resolve(Path) method for creating a new Path by joining an existing path to the current path. To put it another way, the object on which the resolve() method is invoked becomes the basis of the new Path object, with the input argument being appended onto the Path. Let's see what happens if we apply resolve() to an absolute path and a relative path:

```
final Path path1 = Paths.get("/cats/../panther");
final Path path2 = Paths.get("food");
System.out.println(path1.resolve(path2));
```

The code snippet generates the following outputs:

``` /cats/../pather/food```

For the exam, yopu should be aware that, like the relativize() method, the resolve() method does not clean up path symbols, such as the parent durectory .. symbol. For that, you will need to use the normalize() method, which we will cover next. In this example, the input argument to the resolve() method  was a relative path, but what if had been an absolute path?

```
final Path path1 = Paths.get("/turkey/food");
final Path path2 = Paths.get("/tiger/cage");
System.out.println(path1.resolve(path2));
```

Since the input parameter path2 is an absolute path, the output would be the following:

``` /tiger/cage ```

For the exam, you should be cognizant of mixing absolute and relative paths with the resolve() method. If an absolute path is provided as input to the method, such as path1.reosolve(path2), then path1 would be ignored and a copy of path2 would be returned.

### Cleaning Up a Path with normalize()
As you saw with the relativize() method, file system can construct relative paths using .. and . values. There are times, however, when relative paths are combined such that there are redundancies in the path value. Luckily, Java provides us with the normalize(Path) method to eliminate the redundancies in the path.
For example, lets take the output of one of our previous examples that resulted in the path value ../user/home and try to reconsititute the original absolute path using the resolve() method:

```
Path path3 = Paths.get("E:\\data");
Path path4 = Paths.get("E:\\user\\home");

Path relativePath = path3.relativize(path4);
System.out.println(path3.resolve(relativePath));
```

The result of this sample code would be the following output:

``` E:\data\..\user\home ```

You can see that this path value contains a redundancy. Worse yet, it does not match our original value, E:\user\home. We can resolve this redundancy by applying the normalize() method as shown here:

```
System.out.println(path3.resolve(relativePath).normalize())
```

The modified last line of code nicely produces our original path value:

```
E:\user\home
```

Like relativize(), the normalize() method does not check that the file actually exists. As you shall see with our final Path  method, toRealPath(), Java provides a way to verify the file does exactly exist.

### Checking for File Existence with toRealPath()
The toRealPath(Path) method takes a Path object that may or may not point to an existing file within the file system, and it returns a reference to a real path within the file syste. It is similar to the toAbsolutePath() method in that it can convert a relative path to an absolute path except that it also verifies that the file referenced by the path actually exists, and thus it throws a checked IOException at runtime if the file cannot be located. It is also the only Path method to support the NOFOLLOW_LINKS option.
The toRealPath() method performs additional steps, such as removing redundant path elements. In other words, it implicitly calls normalize() on the resulting absolute path. 

Let's say that we have a file system in which we have a symbolic link from food.source to food.txt, as described in the following relationship:

```
/zebra/food.source -> /horse/food.txt
```

Assuming the our current working durectory is /horse/schedule, then consider the following code:

```java
try{
  System.out.println(Paths.get("/zebra/food.source").toRealPath());
  
  System.out.println(Paths.get(".././food.txt").toRealPath());
}catch(IOException e){
  //handle file I/O exception ...
}

```

Notice that we have to catch IOException, since unlike the toAbsolutePath() method, the toRealPath() method interacts with the file system to check if the path is valid. Given the symbolic link and current working directory as described, then the output would be the following:

```
/horse/food.txt

/horse/food.txt
```

In these examples, the absolute and relative paths both resolve to the same absolute file, as the symbolic link points to a rela file within the file system. Finally we can also use the toRealPath() method to gain access to the current working durectory such as shown here:

```
System.out.println(Paths.get(".").toRealPath());
```
## Interacting with Files
Great! We now have access to a Path object, and we can find out a ton of information about it, but what can we do with the file it references? For starters, many of the same operations available to java.nio.file.Path via a helper class called java.nio.file.Files. Unlike the methods in the Path and Paths class, most of the operations within the Files class will throw an exception if the file to which the Path refers does not exist.

**Be wary of File vs Files on the exam** as you saw earlier with Path and Paths, Java is fond of singular names for container classes and plural names for factory and helper classes. In this situation, though, the NIO.2 Files hlper class is in no way related to the File class, as Files class operates on Path instances, not File instances. Kepp in mind that File belongs to the legacy java.io API, while Files belong to the NIO.2 API.

The Files class contains numerous static methods for interacting  with files, with most taking one or two Path objects as arguments. Some of these methods are capable of throwing the checked IOExcepotion at runtime, often when the file referenced does nit exist within the file system, as you saw with Path method toRealPath().

### Testing a Path with exists()
The Files.exists(Path) method takes a Path object and returns true if, and only ifg, it references a file that exists in the file system.

Let's take a look at some sample code:

```
Files.exists(Paths.get("/ostrich/feathers.png"));

Files,exists(Paths.get("/ostrich"));

```

The first example checkes wheather a file exists, while the second example checkes whether a directory exists. You can see that this method does not throw an expection if the file does not exists, as doing so would prevent this method from ever running false at runtime.

### Testing uniqueness with isSameFile()
The Files.siSameFile(Path,Path) method is useful for determining if two Path objects relate to the same file within the same file system. It takes two Path objects as input and follows symbolic links. Despite the name, the method also determines if two Path objects refer to the same directory.

The isSameFile() method first checks if the Path objects are equal in termns of equal(), and if so, it automatically returns true without checking to see if either file exists. If the Path object equals() comparison returns false, then it locates each file to which the path refers in the file system and determines if they are the same, throwing a checked IOException if either file doest not exist. Note that the method does not compare the content of the file. For example two files may have indetical content and attributes, but if they are in different locations, then this method will return false.

Let's assume that all of the files in the following examples exists within the file system and that cobra is a symbolic link to the snake file. What would be the output of the following code snippet?

```
try{
  System.out.println(Files.isSameFile(Paths.get("/user/home/cobra"), Paths.get("/user/home/snake")));
  
  System.out.println(Files.isSameFile(Paths.get("/user/tree/../monkey"), Paths.get("/user/monkey")));
  
  System.out.println(Files.isSameFile(Paths.get("/leaves/./girafe.exe"), Paths.get("/leaves/giraffe.exe")));
  
  System.out.println(Files.isSameFile(Paths.get("/flamingo/tail.data"), Paths.get("/cardinal/tail.data")));

}catch(IOException e){
  //handle file I/O expcetion
}

```

Since cobra is a symbolic link to the snake file, the first example outputs true. In the second example, the symbol .. cancels out the tree path of the paths, resulting in the method, so the results is true as well. In the third example, the symbol . leave the path unmodified, so the result is tru as well. The final example returns false, asuming that neither file is a synbolic link to the other. Even if the files have the same name and the same contents, if they are different locations, they are considered different files within the file system.

### Making Directories with createDirectory() and createDirectories()
To create directories in the legacy java.io API, we called mkdir() on a File object. In the NIO.2 API, we can use the Files.createDirectory(Path) method to create a directory. There is also a plural form of the method called Files.createDirectories(), which like mkdirs() creates the target directory along with any nonexistent parent directories leading up to the target directory in the path. The directory-creation methods can throw the checked IOException, such as when the directory cannot be created or already exists. For example, the first method, createDirectory() will throw an exception if the parent directory in which the new directry resides does not exist.

Both of these methods also accept an optional list of FileAttribute<?> values to set on the newly created directory or directories. We will discuss file attributes in the next section. We now present a code snippet that shows how to create directories using NIO.2:

```
try{
  Files.createDirectory(Paths.get("/bison/field"));
  
  Files.createDirectory(Paths.get("/bison/field/pasture/green"));
  
}cacth(IOException e){
  //handle file I/O Exception
}

```
The first example creates a new directory, field, in the directory /bison, assuming /bison exists; or else an exception is thrown. Contrast this with the second example that creates the directory green along with any of the following parent directories if they do not already exist, such as /bison, /bison/field, or /bison/pasture.

### Duplicating File Contents with copy()
Unlike the legacy java.io.File class the NIO.2 Files class provides a set of overloaded copy() methods for copying files and directories within the file system. The primary one that you should know about for the exam is File.copy(Path, Path), which copies a file or directory from one location to another. The copy() method throws the checked IOException, such as when the file or directory does not exist or cannot be read.

Directory copies are shallow rather than deep, meaning that files and subdirectories within the directory are not copied. To copy the contents of a directory, you would need to create a function to traverse the directory and copy each file and subdirectory individually:

```
try{
  Files.copy(Paths.get("/panda"), Paths.get("/panda-save"));
  
  Files.copy(Paths.get("/panda/bamboo.txt"). Paths.get("/panda-save/bamboo.txt"));
}catch(IOException e){
  //handle file I/O exception...
}

```

The first example performs a shallow copy of the panda directory, creating a new panda-save directory but it does not copy any contents of the original directory. The second example copies the bamboo.txt file from the directory panda to the directpry panda-save.

By default, copying files and directories will traverse symbolic links, although it will not overwrite a file or directory if it already exists, nor will it copy file attributes. These behaviours can be altered by providing the additional options NOFOLLOW_LINKS, REPLACE_EXISTING, and COPY_ATTRIBUTES, respectively, as discussed earlier in the chapter.

### Copying Files with java.io and NIO.2
The NIO.2 Files API class contains two overloaded copy() methods for copying files using java.io streams, as described in Chpeter8. The first copy() method takes a source using java.io,InputStream along with a target Path object. It reads the contents from the stream and writes the output to a file represented by a Path object

The second copy() method takes a source Path object and target java.io.OutputSteam. It reads the contents of the file and writes the output to the stream. The following are examples of each copy() method:

```
try(InputStream is = new FileInputStream("source-data.txt"); 
    OutputStream out = new FileOutputStream("output-data.txt")){
    
    //copy stream data to file
    FIles.copy(is, Paths.get("c:\\mammals\\wolf.txt"));
    
    //copy file data to stream
    Files.copy(Paths.get("c:\\fish\\clown.xsl"), out);

}catch(){
  //Handle file I/O exception ...
}

```
In this example, the InputStream and OutputStream parameters could refer to any valid stream, including website connection, in-memory stream resources, and so forth.

Like the first copy() method, the copy(InputStream, Path) method also supports optional vararg options, since the data is being written to a file represented by a Path object. The second method, copy(Path, OutputStream), does not support vararg values, though, since the data is beiogn written to a stream that may not represent a file system resource.

### Changing a File Location with move()
The Files.move(Path, Path) method moves ro renames a file or directory within the file system. Like the copy() method, the move() method also throws the checked IOException in the event that the file or directory could not be found or moved.

The following is some sample code that uses the move() method:

```
try{
  Files.move(Paths.get("c:\\zoo"), Paths.get("c:\\zoo-new"));
  
  Files.move(Paths.get("c:\\user\\addresses.txt"), Paths.get("c:\\zoo-new\\addresses.txt"));
}cath(IOException e){
  //handle I/O exception ...
}

```

The first example renames the zoo directory to zoo-new directory, keeping all of the original conmtents from the source directopry. The second example moves the addresses.txt file from the directory user to the directory zoo-new and it renames it to the addresses2.txt. By default the move method will follow links, throw an exception if the file already existss, and not perform an atomic move. These behaviours can be changed by providing the optional values NOFOLLOW_LINKS, REPLACE_EXISTING, or ATOMIC_MOVE, respectively, to the method. If the file system does not support atomic moves, an AtomicMoveNotSupportedException will be thrown at runtime.

Note that the Files.move() method can be applied to anon-empty directories only if they are on the same underlying drive. While moving an empty directory across a drive us supported, moving a non-empty directory accross a drive will throw an NIO.2 DirectoryNotEmptyException.

### Removing a File with delete() and deleteIfExists()
The Files.delete(Path) method deletes a file or empty directory within the file system. The delete() method throws the checked exception IOException under the variety of circumstances. For example, if the path represents a non-empty directory, the operation will throw the runtime DirectoryNotEmptyException. If the target of the path is a symbol link, then the symbolic link will be deleted, not the target of the link.

The deleteIfExists(Path) method is identical to the delete(Path) method, except that it will not throw an exception if the file or directory does not exist, but instead it will return a boolean value of false. It will throw an exception if the file or directory does exist but fails, such as in the case of the directory not beign empty.

We now provide sample code that performs delete() operations:

```
try{
  Files.delete(Paths.get("/vulture/feathers.txt"));
  Files.deleteIfExists(Paths.get("/pigeon"));
}cathc(){
  // hangle file I/O Exception...
}

```

The first example deletes the features.txt file in the vulture directory, and it throws a NoSuchFileException if the file or directory does not exist. The second example deletes the pigeon directory assuming it is empty. If the pigeon directory does not exist, then the second line will not throw an exception.

### Reading and Writing a File Data with newBufferedReader() and newBufferedWriter()
The NIO.2 API includes methods for reading and writing file contents using java.io streams. In this manner, the NIO.2 API bridges information about streams, which you learned about in Chapter 8; the Path and File classes are covered in this chapter.

The first method, Files.newBufferedReader(Path, Charset), reads the file specified at the Path location using java.io.BufferedReader object. It also requires a Charset value to determine what character encoding to use to read the file. You may remember that we briefly discussed character encoding and charset in chapter 8. For this chapter, you just need to know that characters can be encoded in bytes in a variety of ways. It may also be useful to know that charset.defaultCharset() can be used to get default charset for the JVM.

We now represent an example of this example:

```
Path path = Paths.get(""/animal/gopher.txt);
try(BufferedReader reader = Files.newBufferedReader(path, charset.forName("US-ASCII"))){
  //read from the stream
  String currentLine = null;
  while((currentLine = reader.readLine()) != null){
    System.out.println(currentLine);
  }

}catch(IOException e){
  //Handle file I/O exception...
}

```

This example reads the contents of the files using a BufferedReader and outputs the contents to the user. AS you shall see in the next section, there is a much simpler way to accomplish this, which uses funtional programming streams.

The second method, Files.newBufferedWriter(Path, Charset) writes to a file specified at the Path location suing a BufferedWriter. Like the reader method, it also takes a Charset value:

```
Path path = Paths.get("/animals/gorilla.txt");
List<String> data = new ArrayList();

try(BufferedWriter writer = Files.newBufferedWriter(path, Charset.forName("UTF-16")){
  writer.write("Hello world");
}catch(IOException e){

  //handle I/O Exception...
}

```
The code snippet creates a new File with the specified contents, overwriting the file if it already exists. The newBufferedWriter() method also supports taking additional enum values in an optional vararg, such as appeding to an existing file instead of overwriting it, although you do not need to memorize this list for the exam.

Since both of these methods create resources, we can use try-with-resource syntax as described in CHapter 6, as we did when working withn stream in Chapter 8. Also, note that both of these methods use buffered streams rather than low level file streams. AS we mentioned earlier in the chapter the buffered stream classes are much moire performant in parctice, so much so that the NIO.2 API includes methods the specifically retun these stream classes, in part to encourage you always to use bufferd streams in your application.

### Reading Files with readAllLines()
The Files.readAllLines() method reads all of the lines of a text file and return the results as an ordered List of String values. The Nio.2 API includes an overloaded version that takes an optional Charset value. The following sample code reads the lines of the file and outputs them to the user:

```
Path path = Paths.get("/fish/sharks.log");
try{
  final List<String> lines = Files.readAllLines(path);
  for(String line : lines){
    System.out.println(line);
  }
  
}catch(IOException e){
  //Handle file I/O exception...
}

```
The code snippet reads all of the lines of the file and then iterates over them, As you might have expected, the method may throw an IOException if the file cannot be read.

Be aware that the entire file is read when readAllLines() is called, with the resulting String array storing all of the contents of the file in memory at one. Therefore, if the file is significantly large, you may encounter an OutOfMemoryError trying to load all of it into memory. Later on in the chapter, we will revisit this method and present a new stream based NIO.2 method that is far more performant on large file.

# Understanding File Attributes
In the previous section, we reviewed methods that could create, modify, read or delete a file or directory. The Files class also provides numerous methods methods accessing file and directory metadata, referred to as file attributes. Put simply, metadata is data that describes other data. In this context, file metadata is data about the file or directory record within the file system and not the contents of th file. 
For exmple, a file or directory may be hidden within a file system or marked with a permisison that prevents the current user form reading it. The Files class provides methods for determining this information from within Java applications.

The one thing to keep in mind while reading file metadata in Java is that some methods are operating system dependent. For example, some operating systems may have not have notion of user-level permissions, in which case users can read only files tthat hey have permissions to read.

## Discovering Basic File Attributes
We begin the discussion of file attributes by representing the basi methods, defined directly within the Files class, for reading file attributes. These methods are usable within any file system although they may have limited meaning in some file systems. In the next section, we will repsent a more generalized approach using attributes views and show that they not only improve performance but also allow us to access fle system-dependent attributes.

### Reading Common Attributes with isDirectory(), isRegularFile(), and isSymbolicLink()
The Files class includes three methods for determining if a path refers to a directory, a regular file, or a symbolic link. The method to accomplish this are named Files.isDirectory(Path), Files.isRegularFile(Path), and Files.isSymbolicLink(Path) respectively.

Java defines a regular file as one that contains contents, as opposed to symbolic link, directory, resource, or other non-regular file that may be present in some operating systems. If the symbolic link points to a real file or directory, Java will perform the check on the target of the symbolic link. In other words, it is possible for isRegularFile() to return true for a symbolic link, as long as the link resolves to a regular file.

Let's take a look at some sample code:

```
Files.isDirectory(Paths.get("/canine/coyote/fur.jpg"));

Files.isRegularFile(Paths.get("/canine/types.txt"));

Files.isSymbolicLink(Paths.get("/canine/coyote"));

```

The first example returns true if fur.jpg is a directory or a symbolic link to a directory and false otherwise. Note that directories can have extensions in many file systems, so it is possible for fur.jpg to be the name of a directory. The second example returns true if types.txt points to a regular file or alternatively a symbolic link that points toa regular file.

We illuminate these concepts in table below. For this table assume that the file system with the directory /canine/coyote and file canine/types.txt exists. Furthermore, assume that /coyotes is symbolic link within the file system that points to another path within the file system.

![isDirectory(), isRegularFile(), isSymbolicLink() examples](img/isDirectoryExamples.png)

You see that the value of isDirectory() and isRegular() in table above cannot be determined on the symbolic link /coyotes without knowledge of what the symbolic link points to.

### Checking File Visibility with isHidden()
The Files class includes the Files.isHidden(Path) method to determine whther a file or directory is hidden within the file system. In Linux or Mac based sytems, this is often denoted by file or directory entries that begin with a period character (.), while in Windows-based systems this requires the hidden attribute to be set. The isHidden() method throws the checked IOException, as thre may be an I/O error reading the underlying file information. We present illustrative usage of this method in the following sample code:

```
try{
  System.out.println(Files.isHidden(Paths.get("/walrus.txt")))
}cathc(){
  // handle file I/O exception
}

```
If the walrus.txt file is available and hidden within the file system, this method will return true.

### Testing File Accessibility with isReadable() and isExecutable()
The Files class includes two methods for reading file accessibility: Files.isReadable(Path) and Files.isExecutable(Path). This is important in file systems where the filename can be viewed within a directory, but the user may not have persmission to read the contents of the file or execute it. We now presetn usage of each method:

```
System.out.println(Files.isReadable(Paths.get("/seal/baby.png")));

System.out.println(Files.isExecutable(Paths.get("/seal/baby.png")));

```

The first example returns true if the baby.png exists and its content are readable, based on the permission rules of the underlying file system. The second example returns trye if the baby.png file is marked executable within the file system. Note that the file extension does not necessary determine whether a file is executable. For example, an image file that ebnds in .png could be marked exectuable within a Linux based system. Like the isDurectory(), isRegularFile() and isSymbolicLink() methods, the isReadable() and isExecutable() methods do not throw exceptions if the file does not exist but instead return false 

### Reading File Length with size()
The File.size(Path) method is used to determine the size of the file in bytes. The size returned by this method represents the conceptual siexe of the data, and this may differ from the actual size on the persistence storage device due to file systems compression and organization. The size() method throws the checked IOException if the file does not exists or if the process is unable to read the file information.
The following is a sample call to the file information:

```
try{
  System.out.println(Files.size(Paths.get("/zoo/c/animals.txt")));
}cath(IOException e){
  //handle file I/O exception...
}

```

The example outputs the number of bytes in the file, expressed as a long value. As you may have already realized, we are repeating a lot of the methods defined in java.io.File, as discussed in Chapter 8. Since the NIO.2 API was defined as a replacement for the java.io API, it includes many of the same methods in one form or another.

Note that the FIles.size() method is defined only on file. Calling Files.size() on a directory is system dependent and undefined. If you need to determine the size of a directory and its contents, you will ned to walk the directory tree, described later in this chapter.

### Managing File Modifications with getLastModifiedTime() and setLastModifiedTime()
Most operating systems support tracking a last modified dat/time values with each file. Some applications use this to determine when the file should be read again. For example, there might be a program that performs an operation anytime the file data changes. In the majority of curcustances, it is a lot faster to check a single file medatada attribute than to reaload the entire contents of the file, expecially if the file is large.
The Files class provides the method FIles.getLastModifiedTime(Path) which returns a FileTime object to accomplish this. The FileTime class is a simepl container class that stores the data/time information about when a file was accessed, modified or created. For convenience, it has a toMillis() method that returns the epoch time.
The Files class also provides a mechanism for updating the last modified date/time of a file using the Files.setLastModifiedTime(Path, FileTime) method. The FileTime class also has a static fromMillis() method that converts from epoch time to a FileTime object.
Both of these methods have the ability to throw a checked IOException when the file is accessed or modified.

we now present examples of both methods:

```
try{
  final Path path = Paths.get("/rabbit/food.jpg");
  
  System.out.println(Files.getLastModifiedTime(path).toMillis());
  
  Files.setLastModifiedTime(path, FileTime.fromMillis(System.currentTimeMillis()));
  
  System.out.println(Files.getLastModifiedTime(path).toMillis());
}catch(IOException e){
  //handle file I/O exception
}

```
The first part of the code reads and outputs the last-modified time value of the food.jpeg file. The next line sets a last modified date/time using the current time value. Finally, we repeat our earlier line and output the newly set last-modified value.

### Managing Ownership with getOwner() and setOwner()
Many file systems support the notion of user-owned files and directories. In this manner, the Files.getOwner(Path) method returns an instance of UserPrincipal that represents the owner of the file within the file system.
as you may have already guessed, there is also a method to set the owner, called Files.setOwner(Path, UserPrincipal). Note that perating system may intervene when youtry to modify the owner of a file and block the operation. For example, a proicess running under oine user may not be allowed to take ownership of a file owned by another user. Both the getOwner() and setOwnder() methods can throw the checked exception IOException in case nay issue accessing or modifying the file.
In orer to set a file owner to an arbitraty user, the NIO.2 API provides a USerPrincipalLookupService helper class for finding a UserPrincipal record for a particular user within a file system. In order to use the helper class, you first need to obtain an instance of a FileSystem objects, either by using the FIleSystems.getDefault() method or by calling getFileSystem() on the Path object with which you are working, as shown in the following two examples:

```
UserPrincipal owner = FileSystems.getDefault().getUSerPrincipalLookupService().lookupProncipalByName("jane");
Path path = ...
UserPrincipal owner = path.getFileSystem().getUserPrincipalLookupService().lookupPrincipalByName("jana");
```
We now present examples of the getOwner() and setOwner() methods, including an example of how to use the UserPrincipalLookupService:

```
try{
  //Read owner of file
  Path path = Paths.get("/chicked/feathers.txt");
  System.out.println(Files.getOwner(path).getName());
  
  //Change owner of file
  UserPrincipal owner = path.getFileSYstem().getUserPrincipalLookupService().lookupPrincipalByName("jane");
  Files.setOwner(path, owner);
  
  //output the updated owner information
  System.out.println(Files.getOwner(path).getName());
  
}catch(IOexception e){
  //Handle I/O exception
}

```

The first set of lines reads the owner of the file and outputs the name of the user. The secodn set of lines retrieves a user named jane within the related file system and uses it to set a new owner for the file. Finally, we read owner name again that is has been updated.

### Improving Access with Views
Up until now, we have been accessing individual file attriv=butes with single method calls. While this is functionally correct, there are often costs associated with accessing the file that make it far mor efficient to retrieve all file metadata ina single call. Furthermore, some attributes are file system specific and cannot be easily generalized for all file systems.

The NIO.2 API addresses both of these concerns by allowing you to construct views for various file systems ina single method call. A view is a group of related attributes for a particular file system type. A file may support multiple views, allowing you to retrieve and update various sets of inforamtion about the file. 
If you need to read multiple attributes of a file or a directory at a time, the performance advantage of suing a view may be substantial. Although more attributes are read than in a single method call, there are fewer round trips between Java and the operating system, whereas reading the same attributes with previously described single method calls would require many such trips. In parctice, the number of trips between Java and the operating system is more important in determining performance than the number of attributes read.
Tha's not to say that the single mehod calls we just finished discussing do not have their applications. If you only need to read exactly one file attribute, then there is little or no performance difference. The also tend to be more convenient to use given their concise nature.

### Understanding Views
To request a view, you need to provide both a path to the file or a directory whose information you want to read, as well as a class object, which tells the NIO.2 API method which type of view you would like returned. The Files API includes two sets of methods of analogous classes for accessing view information. The first method, Files.readAttributes(), returns a read only view of the files attributes. The second method, Files.getFileAttributeView(), returns the underlying attribute view, and it provides a direct resource for modifying file information. Both of these methods can throw a checked IOException, such as when the view class type is unsupported. For example, trying to read windows-based attributes within a linux file system may throw an UnsupportedOperationException. 

Table below lists the common used attributes and view classes: note that the first row is required knowledge for the exam. The DOS and POSIX classes are useful for reading and modifying operating system-specific properties. They also both inherit from their respective attribute and view classes. For example, PosixFileAttributes inherits from BasicFileAttributeView, meaning that all of the operations available on the parent class are available in the respective subclasses.

![The attributes and view classes](img/attributesAndViewClasses.png)

For the exam, you should be familiar with the ```BasicFileAttributes``` and ```BasicFileAttributeView``` classes and their common methods, such as creationTime(), lastModifiedTime(), and so forth. You don't need to memorize the methods available to the DoesFile and PosixFile clases for the exam, although you should be aware that they exist in case you come across them.

### Reading Attributes
The NIO.2 API provides a File.readAttributes(Path, Class<A>) method, which returns read-only version of a file view. The second parameter uses generic such that the return type of the method will be an instance of the provided class.

#### BasicFileAttributes
All attributes classes extend from BasicFileAttributes; therefore it contains attributes common to all supported file systems. It includes many of the file attributes that you previously saw as single-line method calls in the FIles class, such as Files.isDirectory(), Files.getLastModifiedTime(), and so on. We now present a sample application that retrieves BasicFileAttributes on a file and outputs various metadata about the file:

```java
import java.io.IOException;
import java.nio.File;
import java.nio.file.attribute.BasicFileAttributes;

public class BasicFileAtributesSample{
  
  public static void main(String[] args){
    Path path = Paths.get("/turtles/sea.txt");
    BasicFileAttributes data = Files.readAttributes(path, BasicFileAttributes.class);
    
    System.out.println("Is a path a directory ?" + data.isDirectory());
    System.out.println("Is a path a regular file ?" + data.isRegularFile());
    System.out.println("Is a path a symbolic link" + data.isSymbolicLink());
    System.out.println("Path not a file, directory, nor symbolic link?" + data.isOther());
    
    System.out.println("Size (in bytes)" + data.size());
    
    System.out.println("Creation date/time " + data.creationTime());
    System.out.println("Last modified date/time " + data.lastModifiedTime());
    System.out.println("Creation date/time " + data.lastAccessTime());
    System.out.println("Unique file identifier (if available) " + data.fileKey());
  }
}

```

The majority of these attributes should be familiar to you, as they were covered in the previous section of this chapter. The only one that are new are isOther(), lastAccessTime(), creationTime() nad fileKey(). The isOther() method is used to check for paths that are not files, directories, or symbolic links, such as paths that refer to resources or devices in some file systems. The lastAccessTime() and creationTime() methods return other data/time information about the file. The fileKey() method returns a file system value that represents a unique indentifier of the file within the file system or null if it is nor supported by the file system.

### Modiying Attributes
While the Files.readAttributes() method is useful for readig file data, it does not provide a direct mechanism for modifying file attributes. The NIO.2 API provides the Files.getFileAttributeView(Path, Class<V>) method, which returns a view object that can use to update the file system-dependent attributes. We can also use the view object to read the associated file systems attributes by calling readAtributes() on the view object.

#### BasicFileAttributeView
BasicFileAttributeView is used to modify a file's set of date/time values. In general, we cannot modify the other basic attributes directly, since this would change the property of the file system object. For example, we cannot set a property to change a directory into a file, since this leaves the files in the future in an ambiguous state. Likewise, we cannot change the size of the objects without modifying its contents.

We now present a sample application that reads a file's basic attributes and increments the file's last modified date/time values by 10,000 milliseconds or 10 seconds:

```
import java.io.IOException;
import java.nio.File;
import java.nio.file.attribute.*;

public class BasicFileAttributeViewSample{
  
  public static void main(String[] args) throws IOException{
     Path path = Paths.get("/turtles/sea.txt");
     BasicFileAttributeView view = Files.getFileAttributeView(path, BasicFileAttributeView.class);
     BasciFileAttributes data = view.readAttributes();
     
     FileTime lastModifiedTime = FIleTime.fromMillis(data.lastModifiedTime().toMillis() + 10_000);
     
     view.setTime(lastModifiedTime, null, null);
     
  }

}

```
Notice that although we called FIles.getFileAttributeView(), we were still able to retrieve a BasicFileAttributes object by calling readAttributes() on the resulting view. Since there is only one update method, setTimes(FIleTime lastModifiedTime, FileTime lastAccessTime, FIleTime createTime) in the BasicFileAttributeView class, and it takes these arguments, we need to pass three values to the method.

The NIO.2 API allows us to pass null for any date/time value that we don't wish to modify. For example, the following line of code would change only the last-modified date/time, leaving the other file date/time values unaffected:

```
view.setTime(lastModifiedTime, null, null);
```

# Presenting the New Stream Methods
Prior to Java 8, the techniques used to perform complex file operations in NIO.2, such as searching for a file within a directory tree, were a tad verbose and often required  you to define an entire class to perform a simple task. When Java 8 was released, new methods that rely on streams were added to the NIO.2 specification that allow you to perform many of these complex operations witha single line of code.

## Conceptualizing Directory Walking
Before delving the new NIO.2 streams methods, let's review some basci concepts about file systems. When originally described a directory in Chapter 8, we mentioned that it was organized in hierarchical manner. For example, a directory can contain files and other directories, which can in turn contain other files and directories. Every record in a file system has exactly one parent, with the exception of the rrot directory, which sits atop everything. This is commonly visualized as a tree with a single root node and many branches and leaves.

A common task in the file system is to iterate over the descendants of a particular file path, either recording information about them or, filtering them for a specific set of files. For example, you may want to search  a folder and print a list of all of the .java files. Furthermore, file systems store file records in a hierarchical manner. Generally speaking, if you want to search for a file, you have to start with a parent directory, read its child elements, then read their children and so on.

Walking or traverse a directory is the process by which you start with a parent directory and iterate over all of its decendants until some condition is met or there are no more elements over which to iterate. Then starting paths is usually a relevant directory to the application: after all, it would be time consuming to search the entire file system if your application uses only a single directory! 

## Selecting a Search Strategy
There are two common strategies associated with walking a directory tree: a ```depth-first seach``` and a ```breadth-first search```. A ```depth-first``` search traverses the sctructure from the root to an arbritary leaf and then navigates back up towards the root, traversing fully down any paths it skipped along the way. The search depth is the distance from the root to the current node. For performance reasons, some processes have a max search depth that is used to limit how many levels deep the search goes before stopping.
Alternatively, a ```breadth-first``` search starts at the root and processes all elements of each particular depth, or distance from the root, before proceeding to the next depth level. The results are ordered by depth, with all nodes at depth 1 read before all nodes at depth 2 , and so on. 

For the exam, you don't have to understand the details of each search strategy that Java employs; you just need to be aware that the Stream API uses depth-first searching with a default maximum depth of Integer.MAX_VALUE.

**Depth-First Search vs. Breadth-First Search**
In practice, each search strategy has its own advantages and disadvantages. For example, depth-first searches tend to require less memory, since breadth-first searches require mantaining all of the nodes on a particular level in memory in order to generate the next level.

On the other hand, breadth-first searches work better when the nodes for which you are searching is likely near the roor, since depth-first searches can go many levels down a completely useless path before visiting all of the children of the root. 

If you are interested in understanding search strategies in greater detail, there are numerous algorithm books and articles on the subject, including a detailed descripotion of each search strategy on Wikipedia.

## Walking a Directory
As present in CHapter 4, Java 8 includes a new Stream API for performing complex operations in s single line of code using funtional programming and lambda expressions. The first newly added NIO.2 stream based nmethod that we will cover is one used to traverse a directory. The Files.walk(path) method returns a Stream<Path> object that traverses the directory in a depth-first, lazy manner.
By lazy we mean the set of elements is built anbd rad while the directory is being traversed. For example, until a specific subdirectory is reached, its child elemements are not loaded. This performance enhancement allows the process to be run on directories with a large number of descendants in a reasonable manner. 
  
Note: keep in  mind that when you create a Stream<Path> object using Files.walk(), the contents of the directory have not yet been traversed. 
  
The Following is an example of using a stream to walk a directory structure:

```
Path path = Paths.get("/bigcats");

try{
  Files.walk(path)
    .filter(p -> p.toString().endsWith(".java"))
    .forEach(SYstem.out::println);
}catch(IOException e){
  // Handle file I/O exception
}

```

This example interates over a directory and outputs all of the files that end with a java extension. You can see that the method also throws a somewhat expected IOEXception, as there could be a problem reading the underlying file system. Sample output for this method would be similar tp the following:

```
/bigcats/version1/backup/Lion.java
/bigcats/version1/Lion.java
/bigcats/version/Tiger.java
/bigcats/Lion.java
```

If you are familiar with a FileVisitor interface pattern, which was required for version 7of the OCP exam, you migh have noticed that we did in one line what we would normally require an entire class definition to do so. By default the method iterates p to Integer.MAX_VALUE directories depp, although there is an overloaded version of walk(Path, int) that takes a maximim directory depth integer values as the second parameter. A value of 0 indicates the current path record itself. In the previous example, you would need to specify a value of at least 1 to print any child record. In practice, you may want to set a lkimit to prevent your application from searching too deeply on a lrge directory structure and taking too much time.

**Why is Integer.MAX_VALUE the Depth Limit**
Java used an integer value for its maximum depth because most file systems do not support path values deeper that what can be stopped in an int. In other words, using Integer.MAX_VALUE is effectively like using an infinite value, since you would be hard pressed to find a situation where this limit is exceeded. 

You see that the Stream<Path> object returned by the walk() method visits every descedant path, with the filter beign applied as each path is encountered. In the next section, you will see that there is a more useful method for filtering files available in the NIO.2 API.
 
## Avoiding Circular Paths
Unlike out earlier NIO.2 method, the walk() method will not traverse symbolic links by default. Following symbolic links could result in a directory tree that includes other, seemingly unrelated directories in the search. For example, a symbolic link to the root directory in a subdirectory means that every file in the file system may be traversed.

Worse yet, symbolic links could lead to a cycle. A cycle is an infinite circular dependency in which an entry in a directory is an ancestor of the directory. For exampl, imagine that we had a directory /birds/robin that contains a symbolic link called/birds/robin/allBirds that pointen to /birds. Trying to traverse the /birds/robin directory would result in an infinite loop since each time the allBirds subdirectory was reached, we would go back to the parent path. 

If you have a situation where you need to change the default behaviour and traverse symbolic links, NIO.2 offers the FOLLOW_LINKS option as a vararg to the walk() method. It ios recoommended to specify an approparite depth limit when this option is used. Also, be aware that when this option is used, the walk() method will track the path it has visited, throwing a FileSystemLoopException if a cycle is detected.

## Searching a Directory
In the previous example, we applied a filter to the Stream<Path> object to filter the results, although the NIO.2 provides a more direct method. The Files.find(Path, int, BiPredicate) method behaves in a similar manner as the Files.walk() method, except that it requires the depth value to be explicitly set along with a BiPredicate to filter the data. Like walk(), find() also supports the FOLLOW_LINK vararg option. 
  
As you might remember from Chapter 4, a BiPredicate is an interface that takes two generic objects and returns a boolean value of the form (T, U) -> boolean. In this case, the two object types are Path and BasicFileAttributes, which you saw earlier in the chapter. In this manner, the NIO.2 automatically loads the BasicFileAttributes object for you, allowing you to write complex lambda expresions that have direct access to this object. We illustrate this with the following example:

```
Path path = Paths.get("/bigcats");
long dateFilter = 1420070400000L;

try{
  Stream<Path> stream = Files.find(path, 10, 
                      (p,a) -> p.toString().endsWith(".java") && a.lastModifiedTIme().toMillies() > dateFilter);
  stream.forEach(System.out::println);                   
}catch(IOException e){
  //Hanfle file I/O exception ...
}

```
In this example is similar to our previous FIles.walk() example in that it will search a directory for files that end with .java extension. It is more advanced, though, in that it applies a last-modified tiome filtering using BasicFileAttributes object. Finally, it sets the directory depth limit for search to 10, as opposed to relying on the default Integer.MAX_VALUE values that the Files.walk() method uses.

## Listing Directory Contents
You may remember in Chapter 8 that we presented the method listFiles() that operated on java.io.File instance and returned a list of File objects representing the contents of the directory that are direct children of the parent. Although you could use the Files.walk() method with a maximum depth limit of 1 to perform this same task, the NIO.2 API includes a new stream method, Files.list(Path), that does this for you.

Consider the following code snippet, assuming that the current workling directory is /zoo:

```
try{
  Path path = Paths.get("ducks");
  Files.list(path)
    .filter(p -> !FIles.isDirectory(p))
    .map(p -> p.toAbsolutePath())
    .forEach(System.out::println);

}catch(IOException e){
  // Handle file I/O exception... 
}

```

The code snippet iterates over a directory , outputting the full path of the files that it contains. Depending on the contents of the file system, the outputs might look somthing like the following :

```
/zoo/ducks/food.txt
/zoo/ducks/food-backup.txt
/zoo/ducks/weight.txt
```

Contrast this method with the Files.walk() method, which traverses all subdirectories. For the exam, you should be aware that Files.list() searches one level deep and is analogous to java.io.File.listFiles() except that it relies on streams.

## Printing File Contents
Earlier in the chapter, we presented Files.readAllLines() and commented that using it to read very large file could result in an OutputOfMemoryError problem. Luckily, the NIO.2 API in Java 8 now includes a Files.lines(Path) method that returns a Stream<String> object and does not suffer from this same issue. The contents of the file are read and processed lazily, which means that only a small portion of the file is stored in memory at any given time.
  
We now present Files.lines(), which is equivalent to previous Files.readAllLines() sample code:

```java
Path path = Paths.get("/fish/sharks.log");
try{
  Files.lines(path).forEach(System.out::println);
}catch(IOException e){
  //Handle file I/O exception...
}

```
The first thing you may notice is that this example is alot shorter, accomplishing in a single line what it took multiple lines earlier. It is also more performant on alrge files, since it does not require the entire file to be read and store in memory. Taking thisngs one step further, we can leverage other stream methods for a more powerful example:

```
Path path = Paths.get("/fish/sharks.log");

try{
  System.out.println(Files.lines(path)
    .filter(s -> s.startsWith("WARN "))
    .map(s -> s.substring(5))
    .collect(COllectors.toList());
    
}catch(){
  // Handle file I/O exception...
}

```

This sample code now searches for lines in the file that start with WARN, outputting everyting after it to a single list that is printend to the user. You can see that lambda expressions coupled with NIO.2 allow us to perform complex file operations concisenly. Assuming that the input file sharks.log is as follows,

```
INFO  Server starting
DEBUG  Processes available = 10
WARN  No database could be detected
DEBUG  Processes available reset to 0
WARN  Performing manual recovery 
INFO  Server automatically started
```

Then the sample output would be the following:

```
[No database could be detected, Performing manula recovery]
```

**Files.readAllLines() vs. Files.lines()**
For the exam, you should be familiar with both readAllLines() and lines() and with which one returns a list and which one returns a Stream. This is even more difficult since the forEach() method can be called on both Stream and Collection objects. For example, both of the following lines compile and run wothout issue:

```
Files.readAllLines(Paths.get("birds.txt")).forEach(System.out::println);

Files.lines(Paths.get("birds.txt")).forEach(System.out::println);
```

The first code snippet reads the entire file into memory and then performs a print operation on the resulting object. The second code snippet reads the lines lazily and prints them as they are beign read. The advantage of the second code snippet is that it does not require the entire file to be stored in memory as it is being read.

You should also be aware of when they are mixing incompatible types on the exam. For example, can you determine which of the following two lines compile?

```
Files.readAllLines(path).filter(s -> s.length() > 2).forEach(System.out::println);

Files.lines(path).filter(s -> s.length() > 2).forEach(System.out::println);

```

The first line does not compile because the filter operation cannot be applied to a Colection without first converting it to a Stream using the stream() method.

# Comparing Legacy File and NIO.2 Methods
We conclude this chapter with the table below which shows a comparison between some of the legacy java.io.File methods described in Chapter 8 and the new NIO.2 methods described in this chapter. In this table, file refers to an instance of java.io.FIle class, while path refers to an instance of a NIO.2 Path interface.

![Comparison of legacy File and NIO.2 methods](img/legacyFileAndNIO2methods.png)

Bear in mind that a number of methods and features are available in NIO.2 API that are not available in the legacy API, such as support for symbolic links, setting a file ownder, and so on. As expected, the NIO.2 API is much more developed, much more powerful API that the legacy java.io.File class described in chapter 8.

# Review Questions

Complete the questions after 40 min. It takes a lot of practice to learn the use of the library and how to interact with the NIO.2 API methods and lambda expression. 

![Test chapter nine](img/testChapterNine.png)

- The normalize() method like most of the methods in the Path interface, does not modify the Path object but instead returns a new Path objetc
- isDirectory() method by default follows links. Files.deleteIfexists() method throws an exception if the directory had any content.
- setTime() method is available only on BasicFileAttributeView 
- The subpath(int, int) method does not include the root of the file
- The resolve() method does not normalize any path symbols. And calling resolve() with an absolute path as param returns the same absolute path  
- Files.readAllLines() reads the entire file into memory and the return type is List<String> which it can be converted to a stream with a stream method.
- NOFOLLOW_LINKS option means  that if the source is a symbolic link the link itself and not the target will be copied at runtime. ATOMIC_MOVE means that any process monitoring the file system will not see an incomplete file during the move.
- Files.listFiles() method retrieves the member of the current directory without traversing any subdirectories. Files.walk() and Files.find() recursively traverse a directory tree rather than a list of contents of the current directory.
- Advantages using NIO.2
- Advantages of NIO.2 over legacy File
- The normalize() method does not convert a relative path into an absolute path
