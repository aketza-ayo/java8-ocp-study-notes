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
### Checking Path Type with isAbsolute() and toAbsolutePath()
### Creating a New Path with subpath()

## Using Path Symbols
### Delivering a Path with relativize()
### Joining Path Objects with resolve()
### Cleaning Up a Path with normalize()
### Checking for File Existence with toRealPath()
## Interacting with Files
### Testing a Path with exists()
### Testing uniqueness with isSameFile()
### Making Directories with createDirectory() and createDirectories()
### Duplicating File Contents with copy()
### Copying Files with java.io and NIO.2
### Changing a File Location with move()
### Removing a File with delete() and deleteIfExists()
### Reading and Writing a File Data with newBufferedReader() and newBufferedWriter()
### Reading Files with readAllLines()

# Understanding File Attributes
## Discovering Basic File Attributes
### Reading Common Attributes with isDirtectory(), isRegularFile(), and isSymbolicLink()
### Checking File Visibility with isHidden()
### Testing File Accessibility withisReadable() and isExecutable()
### Reading File Length with size()
### Managing File Modifications with getLastModifiedTime() and setLastModifiedTime()
### Managing Ownership with getOwner() and setOwner()
### Improving Access with Views
### Understanding Views
### Reading Attributes
#### BasicFileAttributes
### Modiying Attributes
#### BasicFileAttributeView

# Presenting the New Stream Methods
## Conceptualizing Directory Walking
## Selecting a Search Strategy
## Walking a Directory
## Avoiding Circular Paths
## Searching a directory
## Listing Directory Contents
## Printing File Contents

# Comparing Legacy File and NIO.2 Methods
