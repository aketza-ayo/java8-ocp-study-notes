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
### Using the Path Class
### Accesing the Underlying FileSystem Object
### Working with Legacy File Instances

# Interactiung with Paths and Files
## Providing Optional Arguments
## Using Path Objects
## Viewving the Path with toString(), getNameCount(), and getName()
## Accesing Path Components with getFileName(), getParent(), and getRoot()
## Checking Path Type with isAbsolute() and toAbsolutePath()
## Creating a New Path with subpath()
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
