# Understanding Files and Directories
WHat can java do outside the scope of managing objects anb attributes in memory? How can they save data so that information is not lost every time the program is terminated? The use files of course! You can design a program that writes the current state of an application to a file every time the application is closed and then reloads the data when the application executed the next time. IN this manner the information is preserved between program executions.

We begin this Chapter by describing what a file is and what a directory is within a file system. We then present the ```java.io.File``` class and demonstrate how to use it to read and write file information.

## Conceptualizing the File System
A *file* is record within a file system that stores user and system data. Files are organized using directories. A *directory* is a record within a file system that contains files as well as other directories. For simplicity, we often refer to a directory reference as a file record throughout this chapter , since it is stored in the file system with a unique name and with attributes similar to a file.Finally a *root directory* is the topmost directory in the file system, from which all files and directories inherit. In Windows, it is denoted with a drive name such as c:\ while on linux it is denoted with a single forward slash /. The *file system* is in charge of reading and writing data within a computer. Different operative systems use different file systems to manage their data. For example, Windows-based systems use a different file system than Unix-based ones. Java includes numerous methods, which automatically connect to the local file system for you, allowing you to perform the same operations across multiple file systems.

A *path* is a string representation of a file or directpry within a file system. Each file system defines own path separator character that is used between directory entries. In most file systems, the value of the left of the separator is the parent of the value of the right of the separator. For example, ```/user/home/zoo.txt``` means that zoo.txt is inside the home directory, with the home directory inside the user directory. You will see that paths can be absolute or relative later in the Chapter. In the figure below we show how a durectory and a file system is organized in hierarchical manner. In this example, directories are rectangles and files as ovals. Directories can be empty, as shown with tyhe c:\zoo and c:\app\employees directories.

![Directory and File Hierarchy](img/directoryHierarchy.png)

## Introducing the File Class
The first class the we will discuss is one of the most commonly used in the java.io API, the java.io.File class. The File class is used to read information about existing files and directories, list the contents of a directory, and create/delete files and directories.
An instance of a File class represents the pathname of a particular file or directory on the file system. The File class cannot read and write data within a file, altough it can be passed as reference to many stream classes to read or write data. 

:yin_yang: one common mistake new Java developers make is forgetting that File class can be used to represent directoris as well as files.

## Creating a File Object
A File object often initilizes with String containing either an absolute or relative path to the file or directory within the file system. Different operative systems vary in their format of path names. For example, Unix based systems use the forward slash / for paths, whereas Windows-based systems use the backslash \ character. That said, many programming languages and file systems support both types of slashes when writing path statements. For convenience, Java offers two options to retrieve the local separator character: a system property and a static variable defined in the File class. Both of the following examples will output the separator character:

```
System.out.println(System.getProperty("file.separator"));

System.out.println(java.io.File.separator);
```

The following code creates a File object and determines if the path it referenes exists within the file system:

```java
import java.io.File;

public class FileSample{
  public static void main(String[] args){
    File file = new File("/home/smith/data/zoo.txt");
    System.out.println(file.exists());
  }
}

```

This example uses the absolute path to a file and outputs true or false, depending on whther the file exists or not.The most common File construcot we will use takes a single String as an argument representing a relative of absolute path. The are other constructors such as the one that joins an exising File path with a relative child path, as shown in the following example:

```
File parent = new File("/home/smith");
File child = new File(aprent, "data/zoo.txt");
```

In this example we create a path that is equivalent to our previous example, using a combination of a child and a parent path. If the parent object happens to be null, then it would be skipped and the method would revert to our Sstring constructor.

## Working with a File Object
The File class contains numerous useful methods for interacting with files and directories within the file system. We represent the most commonly used one in the table below. Although this table may seem loke alot to learn, many of them are self explanatory:

**Method Name**    | **Description**
-------------------|---------------
exists()           | Returns true if the file or directory exists
getName()          | Returns the name of the file or directory denoted by this path
getAbsolutePath()  | Returns the absolute pathname string of this path
isDirectory()      | Returns true if the file denoted by this path is directory.
isFile()           | Returns true if the file denoted by this path is a file
length()           | Returns the number of bytes in the file. For performance reasons, the file system may allocate more bytes on disk than the file actually uses
lastModified()     | Returns the number of milliseconds since the epoch when the filke was last modified
delete()           | Deletes the file or directoryu. If this pathname denotes a directory, then the directory must be empty in order to be deleted
reanmeTo(File)     | Renames the file denoted by this path
mkdir()            | Creates the directory named by this path
mkdirs()           | Creates the directory named by this path
getParent()        | Returns the abstract pathname of this path including any nonexisitent parent directories.
lastFiles()        | Returns a File[] array denoting the files in the directory

The following is a sample program that given a file path outputs information about the file or directory, such as whther it exists, what files are contained within it and so forth:

```java
import hava.io.File;

public class ReadFileInformation{
  public static void main(String[] args){
    File file = new File("C:\\data\\zoo.txt");
    System.out.println("File exists: " + file.exists());
    
    if(file.exists()){
      System.out.println("Absilute Path: " + file.getAbsolutePath());
      System.out.println("Is Directory: " + file.isDirectory());
      System.out.println("Parent Path: " + file.getParent());
      
      if(file.isFile()){
        System.out.println("File Size: " + file.length());
        System.out.println("File Last Modified: " + file.lastModified());
      }else{
        for(File subfile : file.listFiles()){
          System.out.println("\t" + subfile.getName());
        }
      }
    }
  }
}

```
If the path provided did not point to a file, it would output the following

```
File Exists: false
```

If the path provided pointed to a valid faile, it would putput something similar ro the following:

```java
File Exists: true
Absolute Path: C:\data\zoo.txt
Parent Path: C:\data
Is Directory: false
File Size: 12382
File lastModified: 104215000000
```

Finally, if the path provided pointed to a valid directory, such as C:\data, it would output something similar to the following:

```java
File Exists: true
Absolute Path: C:\data
Parent Path: C:\

Is Directory
  employee.txt
  zoo.txt
  zoo-backup.txt

```


# Introducing Streams
In this section, we present the concept of streams in Java and show how they are used for input/output processing. I/O refers to the natura of how data is accessed, either by reading the data fro the resource (input), or writing the data to a resource (output). This section will focus on the common ways that data using files accessed by streams. Notice that the I/O streams that we discuss in this chapter are data streams and completely unrelated to the new Stream API that you saw in Chapter 4 "Functional Programming".

## Stream Fundamentals
The contents of a file may be accessed or written via a stream, which is a list of data elements presented sequentially. Streams should be conceptually thought of as long, nerly never ending stream of water with data presented one "wave" at a time.For example, it may be helful to visualize a stream as being so large that all of the data contained in it could not possibly fit into memory. A 1 terabyte file could not be stored entirely in memory by most computer systems. The file can still be read and written by a program with very little memory, since the streams allows the application to focus on only a small portion of the overall stream at any given time. A real world scenario example is that writing a file one byte at a time is time consuming and slow in practice because the round-trip between the Java application and the file system is relatively expensive. By using a ```BufferedOutputStream``` the Java application can write a large chunk of bytes at a time, reducing the round-trips and drastically improving performance.

In fact you have been using streams since your first "Hello World" program! Java provides three built-in streams, System.in, System.err and System.out the last of which we have been using to output data to the screen throughtout this book. 

## Stream Nomenclature
The ```java.io``` API provides numerous classes for creating, accessing, and manipulating streams - so many that it tends to overwhelm most new Java developers/programmer. Stay calm! we will review the major differences between each stream class and show you how to distinguish between them. Even if you do come across a particular stream that you don't recognize, often the name of the stream gives you enough info to understand exactly what it does.

The goal in this section is to familiarize you with common terminology and naming convention used with streams. Don't worry if you don't recognize the particular stream class names used in this section or their function; we will covering in detail in the next part of the chapter.

## Byte Streams vs. Character Streams
The java.io API defines two sets of classes for reading and writing streams; those with Stream in their names and those with Reader/Writer in their name. For example, the java.io API defines both ```FileInputStream``` class as well as a ```FileReader``` class, both of which defines a Stream that reads a file. The difference between the two classes is based on how the stream is read or written. 

Differences between Stream adn Reader/Writers:

1- The stream classes are used for inputting and outputting **all types of binary or byte data**.
2- The reader and writer classes are used for inputting and outputting **only character and String data**.

It is important to remember that even though readers/writers do not contain the word Stream in their class name, they are still in fact streams! The use of Reader/Writer in the name is just to distinguish them from byte streams. Throughout the chapter, we will often refer to Reader/Writer classes as streams, since conceptually they are streams.

**Why use Character Streams?** Since they byte stream classes can be used to input and output all types of binary data, including strings, it naturally follows that you can write all of your code to use the byte stream classes, never really needing the character stream classes. There are advantages, though, to using reader/writer classes, as they are specifically focused on managing character and string data. For example, you can use a Writer class to output a String value to a file without having to worry about the underlying byte encoding of the file.  For this reason the character stream classes are sometimes referred to as convenience classes for working with thext data.

The java.io API is structured shuch that all of the stream classes have the word ```InputStream``` or ```OutputStream``` in their name, while all Reader/Writer classes have either Reader or Writer in their name. Pay close attention to the name of the java.io class on the exam, as decoding it often gives you context clues as to what the class does. For example, without needing to look it up, it should be clear that ```FileReader``` is a class that reads data from a file as characters or strings. Furthermore, ```ObjectOutputStream``` sounds like a class that writes object data to a byte stream.  

## Input and Output
Most input stream classes have corresponding Output class and vice versa. For example, the ```FileOutputStream``` class writes data that can be read by a ```FileInputStream```. If you understand the features of a particular Inout or Output stream class, you should naturally know what its complementary class does. It follows, then, that most Reader classes have corresponding Writer class. For example, the FileWriter class writes data that can be read by a FileReader.
There are exceptions to this rule. For the exam, you should know that PrinterWriter has no accompanying PrinterReader class. Likewise, the PrintStream class has no corresponding InputStream class. We will discuss these classes later in the chapter.

## Low-Level vs. High-Level Streams
Another way you can familiarize yourself with java.io API is by segmenting streams into low-level and high-level streams. A *low-level stream* connects directly with the source of the data, such as a file, an array, or a String. Low-level streams process the raw data or resources and are accessed in a direct and unfiltered manner. For example, a ```FileInputStream``` is a class that reads file data one byte at a time. 

Alternatively, a *high-level stream* is built on top of another stream using wrapping. *Wrapping* is the process by which an instance is passed to the constructor of another class and operations on the resulting instance are filtered and applied to the original instance. For example, take a look at the FileWriter and BufferedWriter objects in the following sample code. 

```java
try(Buffered bufferedReader = new BufferedReader(new FileReader("zoo-data.txt")){
  System.out.println(bufferedReader.readLine());
}
```
In this example, FileReader is the low-level stream reader, whereas BufferReader is the high-level stream that takes a FileReader as input. Many operations on the high-level stream pass through as operations to the underlying low-level stream, such as read() or close(). Other operations override or add new functionality to the low-level stream methods. The high level stream adds new methods, such as readLine(), as well as performance enhancements for reading and filtering the low-level data.

High-level streams can take other high-level streams as input. For example, although the following code might seem a litte odd at first, the style of wrapping a stream is quite common in practice:

```java
try (ObjectInoutStream objectStream = new ObjectInputStream(
                                              new BufferedInputStream(
                                                new FileInputStream("zoo-data.txt")))){
    System.out.println(objectStream.readObject());                                                
}
```

In this example, FileInputStream is the low level stream that interacts directly with the file, which is wrapped by a high level BufferedInputStream to improve performance. Finally, the entire object is wrapped by a high level ObjectInputStream, which allows us to filter the data as Java objects. 

For the exam the only low-level stream classes you need to be familiar with are the one that operates on files. The rest of the non-abstract stream classes are all high level streams.

***Real World Scenario*** **Use Buffered Streams When Working with Files**. As briefly mentioned, Buffered classes read or write data in groups, rather than a single byte or character at a time. The performance gain from using a Buffered class to access a low-level file stream cannot be overstated. Unless you are doing something very specilized in your application, you should always wrap a file stram with a Buffered class in practice.

The reason that Buffered streams tend to perform so well in practice is that file systems are geared for sequencial disk access. The more sequencial bytes you read at a time, the fewer round-trips between the Java process and the file-system, improving the access of your application. For example, accessing 16 sequential bytes is a lot faster than accessing 16 bytes spread accross the hard drive.


## Stream Base Classes

## Decoding Java I/O Class Names
### Review of java.io Class Properties

## Common Stream Operations
### Closing the Stream
### Flushing the Stream
### Making the Stream
### Skipping over Data

# Working with Streams
## The FileInputStream and FileOutputStream Classes
## The BufferedInputStream and BufferedOutputStream Classes
## The FileReader and BufferedWriter Classes
### Comparing the Two Copy Applications
## The ObjectInputStream and ObjectOutputStream Classes
## The Serializable Interface
## Serializing and Deserializing Objects
## Understanding Object Creation
## The PrintStream and PrintWriter Classes
### print()
### println()
### format() and printf()
## Sample PrintWriter Application
## Review of Stream Classes
## Other Stream Classes

# Interacting with Users
## The Old Way
## The New Way
### reader() and writer()
### format() and printf()
### flush()
### readLine()
### readPassword()

