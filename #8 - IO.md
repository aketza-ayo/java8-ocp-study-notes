# Understanding Files and Directories
WHat can java do outside the scope of managing objects anb attributes in memory? How can they save data so that information is not lost every time the program is terminated? The use files of course! You can design a program that writes the current state of an application to a file every time the application is closed and then reloads the data when the application executed the next time. IN this manner the information is preserved between program executions.

We begin this Chapter by describing what a file is and what a directory is within a file system. We then present the ```java.io.File``` class and demonstrate how to use it to read and write file information.

## Conceptualizing the File System
A *file* is record within a file system that stores user and system data. Files are organized using directories. A *directory* is a record within a file system that contains files as well as other directories. For simplicity, we often refer to a directory reference as a file record throughout this chapter , since it is stored in the file system with a unique name and with attributes similar to a file.Finally a *root directory* is the topmost directory in the file system, from which all files and directories inherit. In Windows, it is denoted with a drive name such as c:\ while on linux it is denoted with a single forward slash /. The *file system* is in charge of reading and writing data within a computer. Different operative systems use different file systems to manage their data. For example, Windows-based systems use a different file system than Unix-based ones. Java includes numerous methods, which automatically connect to the local file system for you, allowing you to perform the same operations across multiple file systems.

A *path* is a string representation of a file or directpry within a file system. Each file system defines own path separator character that is used between directory entries. In most file systems, the value of the left of the separator is the parent of the value of the right of the separator. For example, ```/user/home/zoo.txt``` means that zoo.txt is inside the home directory, with the home directory inside the user directory. You will see that paths can be absolute or relative later in the Chapter. In the figure below we show how a durectory and a file system is organized in hierarchical manner. In this example, directories are rectangles and files as ovals. Directories can be empty, as shown with tyhe c:\zoo and c:\app\employees directories.

![Directory and File Hierarchy](img/directoryHierarchy.png)

## Introducing the File Class
## Creating a File Object
## Working with a File Object

# Introducing Streams
## Stream Fundamentals
## Stream Nomenclature
## Byte Streams vs. Character Streams
## Input and Output
## Low-Level vs. High-Level Streams
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

