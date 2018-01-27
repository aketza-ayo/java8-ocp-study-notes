# Reviewing Exceptions
A program can fail for just about any reason. Here are a few posibilities commonly covered in OCP:
- Your program tries to read a file that doesn't exist.
- Your program tries to access a database, but the network connection to the database is unavailable.
- You made a coding mistake and wrote a wrong SQL statement in your JDBC code.
- You used a wrong format specifier when using DateTimeFormatter.

As you can see some are coding mistakes while others are completely beyond your control. If your network connection goes wrong there isn't much that you can do to deal with this situation.

## Exceptions Terminology
An *exception* is Java's way of saying "I give up. I don't know what to do right now. You deal with it".

## Categories of Exceptions
Below you can find the hierarchy of the exception classes. Rember that a *runtime exception* or unchecked exceptionmight be caught, but it is not requiered to be caught. A *checked exception* is any class that extends Exception but it is not a runtime exception. A checked exception must follow a declare a delcare rule where they either catch or throw an exception. An error is fatal and should never be caught. While it is legal to cathc an error it is not reccommended.

![Exception Hierarchy](img/exceptionHierarchy.png)

In red are the unchecked exception and in blue checked exceptions. 


## Exceptions on the OCP
- **ArithmeticException:** Thrown by the JVM when code attempts to divide by zero.
- **ArrayIndexOutOfBounds:** Thrown by the JVM when code uses an illegal index to access an array
- **ClassCastException:** Thrown by the JVM when an attempt is made to cast an object to a subclass of which it is not an instance.
- **IllegalArgumentException:** Thrown by the program to indicate that a method has been passed an illegal or inappropriate argument.
- **NullPointerException:** Thrown by the JVM when there is a null reference where an object is required.
- **NumberFormatException:** Thrown by the program when an attempt ismade to convert a string to numeric type, but the string an appropriate format..

You also learnt that *java.io.IOException* is an example of a checked exception. On the OCP you need to know more exceptions. The objectives cover a number of APIs that throw a mix of checked and unchecked exceptions. Just remember that IO, parsing, and SQL exceptions are checked. Anything else is a runtime exception unless the exam states otherwise.

See table below for further information:

![OCP checked and unchecked exceptions](img/ocpExceptions.png)

## Try Statement
The try statement consists of a mandatory try clause. It can include one or more catch clause to handle the exception that are thrown. It can also include a finally clause, which runs regardless of whether the exception is thrown. This is true for both try and try-with-resources statements.

The syntax looks like below:

```
try{
  //protected code
}catch(exception indentifier){
  //exception handler
}finally{
  //finally block
}
```

A few notes on the try catch block. A finallt block can only appear as part of a try statement. The finally blok always executes whether or not an exception occurs in the try blocks. 

There is also a rule that says a try statement is required to have either or both the catch and finally clauses. This is true for try statements, but it is not true for try-with-resources statements. 

There are two other rules that you need remeber:
- Java checked the catch block in the order they appear. it is illegal to declare a subclass exception in a catch block that is lower down in the list than a super class exception because it will be unreachable code.
- Java will not allow you to declare a catch block for a checked exception type that cannot potentially be thrown by the try clause body. This is again to avoid unreacheable body.

## Throw vs Throws
The exam might try to trick you whether you are paying attention to the difference between throw and throws. Remember that throw means an exception is actually beign thrown and throws indicates that method merely has the potential to throw that exception. See code sample below:

```java
public String getDataFromDatabase() throws SQLException{
  throw new UnsupportedOperationException();
}
```

# Creating Custom Exceptions
Java creates a lot out of the box exceptions classes. Sometimes you want to create your own set of exception classes and Java allows you to do that.

When creating your own exceptions you need to decide whether they are checked or unchecked exceptions. If you want to create checked exceptions you extend Exception and for unchecked exceptions you extend RuntimeExceptions.

Creating your own exceptions is really easy:

```java
class CannotSwimException extends Exception{}
class DangerInWater extends RuntimeException{}
class SharkInWater extends DangerInWater{}
class Dolphin{
  public void swim() throws CannotSwimException{
    //logic here
  }
}

```

The following line shows the most common constructors defined in the Exception class:
```java
public class CannotSwimException extends Exception{
  public CannotSwimgException(){
    super();
  } 
  
  public CannotSwimgException(Exception e){
    super(e);
  } 
  
  public CannotSwimgException(String message){
    super(message);
  } 
}

```

The first constructor is default constructor with no param. The second constructor shows how to wrap another exception inside yours. The third one shows how to pass a custome message.

## Using Multi-Catch
When somethig goes wrong it is common to log the error and convert it a different exception type. In this example we print the stack trace rather than log to a file. Next we throw a expection:

```java
public static void main(String[] args){
  try{
    Path path = Paths.get("dolphinsBorn.txt");
    String text = new String(Files.readAllBytes(path));
    LocalDate date = LocalDate.parse(text);
    System.out.println(date);
  
  }catch(DateTimeParseException e){
    e.printStackTrace();
    throw new RuntimeException();
  }catch(IOException e){
    e.printStackTrace();
    throw new RuntimeException();
  }
}

```

This works however duplicating code is bad! To deal with it we could catch Exception instead of the specific types. The duplicate code migt be gone but it isnt a good approach because it catches other exceptions too. The catch code could catcha NullPointerException and this was never out intention. Another approach would be to use a helper method to handle the exception but we would still have a little duplication. The Java designers realize this problem and they introduced the ability to catch multiple exceptions in the same catch block. Now we have an elegant solution:

```java
public static void main(String[] args){
  try{
    Path path = Paths.get("dolphinsBorn.txt");
    String text = new String(Files.readAllBytes(path));
    LocalDate date = LocalDate.parse(text);
    System.out.println(date);
  
  }catch(DateTimeParseException | IOException e){
    e.printStackTrace();
    throw new RuntimeException();
  }
```
This is much better. There is no code duplication, the common logic is in one place, and the logic is where we would expect it to find. Notice how there is only one variable name in the catch clause. Java is saying that the variable named e can be of type Exception1 or Exception2.

The exam might try to trick you on the syntax. Remember that that exception can be listed in any order within the catch clause. However the variable name must appera only once and at the end. Do you see why these are valid or invalid?

```java
catch(Exception1 e | Exception2 e | Exception3 e)       //DOES NOT COMPILE

catch(Exception1 e1 | Exception2 e2 | Exception3 e3)    // DOES NOT COMPILE

catch(Exception | Exceotion | Exception e)

```

Java intends multi catch for exceptions that aren't related, and it prevents you from specifying redundant types in a multi-catch. Do you see what is wring here?

```java
try{
  throw new IOExceotion();
}catch( FileNotFoundException | IOException e){}    //DOES NOT COMPILE
```

FileNotFoundException is a subclass of IOException. Specifying it in the multi-catch is redundant, and the compiler gives a message like this:

```
The exception FileNotFoundException is already caught by the alternative IOException.
```

Since we can ommit that exception type without changing the behaviour of the program, java does not allow declaring it. The correct code is as follows.

```
try{
  throw new IOException();
}catch(IOException e){ }

```

**Multi-catch is effectively final** this try statement is legal. it is bad idea to reassing the variable in the catch bock, but is is allowed:

```
try{
  // do some work
}catch(RuntimeException e){
  e = new RuntimeException();
}
```

When adding multi-catch, this pattern is no longer allowed:

```
try{
  // do some work
}catch(IOException | RuntimeException e){
  e = new RuntimeException();         // DOES NOT COMPILE
}
```
Java uses the exception variable internally.

To review the multi catch see how many erros you can find:

```java
public void doesNotCompile(){    //DOES NOT COMPILE
  try{
    mightThrow();
  }catch(FileNotFoundException | IlegalStateException e){
  }catch(InputMismatchException e | MissingResourceException e){
  }catch(SQLException | ArrayIndexOutOfBoundException e){
  }catch(FileNotFoundException | IlegalStateException e){
  }catch(Exception e){
  }catch(IOException e){
  }
}

private void mightThrow() throws DateTimeParseException, IOException {}

```
- The second catch has an extra variable name.
- Last two catch blocks are reversed. The more general superclass must be caught after their sublcass. While these does nothing to do with multi-catch, you will see regular catch blocks mixed with multi catch blocks.
- The fourth catch it is already caught. You cannot list the same exception type more than once in the same try statement. 
- The third catch block you cannot SQLException because nothing in the try statement can potentially throw one. Again, just like regular catch blocks any RuntimeException might be caught. However only checked exceptions that have potential to be thrown are allowed to be caught.

# Using Try-With-Resources
## Try-With_resource Basics
## AutoCloseable
## Suppressed Exceptions
## Putting It Together

# Rethrowing Exceptions

# Working with Assertions
## The assert Statement
## Enabling Assertions
## Using Assertions

