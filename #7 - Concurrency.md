# Introducing Threads
We begin this chapter reviewing common terminology associated with threads. A *thread* is the smallest unit of execution that can be scheduled by the operative system.

A *process* is a group of associated threads that execute in the same, shared environment. It follows then, that a *single-threaded process* is one that contains exactly one thread. whereas a *multi-threaded process* is one that contains one or more threads.

By *shared environment* we mean that the threads in the same process share the same memory space and can communicate directly with one another. In this chapeter we will talk a lot about tasks and their relationship with threads. A task is a single unit of work performed by a thread. A task will commonly be implemented as a lambda expression in this chapter. A thread can complete multiple independent task but only one task at a time. 

## Distinguishing Thread Types
All Java applications are multi-threaded. Even a simple hello world application. To help you understand this we introduce the conceots of system threads and user-defined threads.

A *system thread* is created by the JVM and runs in the background of the application. Foe example, garbage collector thread is a system thread created by the JVM and runs in the background helping to free memory that is no longer use. For the most part, the execution of system-defined threads is invisible to the application developer. When a system-defined thread encounters a problem an cannot recover, such as running out of memory, it generates a Java Error, as opposed to an Exception.
(Remember that even though it is possible to catch an error, it is considered a poor practice to do so since it is rare than an application can recover from system-level failures.)

Alternatively, a *user-defined thread* is one created by the application developer to accomplish a specific task. All of the applications that have been created up to this point have been multi threaded, but they contain only one user-defined thread, which calls the main method. For simplicity, we commonly refer to threads that contain only a single user-defined tread as single-threaded application, sisnce we are often not interested in the system threads. 

(The following is not in the exam, a *daemon thread* is one that will not prevent JVM from existing when the program finishes. A Java application terminates when the only threads that are running are daemon threads. For example if the garbage collector thread is the only thread left running, the JVM will automatically shut down. Both system and user defined threads can be marked as daemon threads.)

## Understanding Thread Concurrency
The property of executing multiple threads and processes at the same time is referred to as *concurrency*. Of course, with a single-core CPU system, only one task is actually executing at a given time. Even in multi core or multi CPU systems, there often far more threads than CPU processors available. How does the system decide what to execute when there are multiple threads available? OS use a *thread scheduler* to determine which thread should be currently executing. A trad scheduler may employ a *round-robin schedule* in which each available thread receives an equal number of CPU cycles with which to execute, with threads visited in a circular order. When a thread alloted time is complete but the thread has not finished processing, a contect switch occurs. A *contect switch* is the process os storing a thread's current state and later restoring the state of the thread to continue execution. Be aware that there is often a cost associated with a contect switch by way of lost time saving and reloading a thread's state.
Finally, a thread can interrupt or supersede another thread if it has a higher thread priority than the other thread. A *thread priority* is a numeric value associated with a thread should currently be executing . In Java, thread priorities are specified as integer values. The Thread class includes 3 static constants. Bu default, user defined threads receive a thread priority value of Thread.NORM_PRIORITY. If you have a thread that needs to be executed right away, you can increase this value to 6 or higher or use the Thread.MAX_PRORITY. If two threads have the same priority, the thread scheduler will arbitrarily choose the one to process first in most situations.

See below the Java thread priority constants

Thread.MIN_PRIORITY = value 1

Thread.NORM_PRIORITY = value 5

Thread.MAX_PRIORITY = value 10


## Introducing Runnable
Runnable is a functional interface that takes no arguments and returns no data. The following is the definition

```java
@FunctionalInterface
public interface Runnable{
  void run();
}

```

The runnable intreface is commonly used to define the work a thread will execute, separate from the main application thread. We will be relying on the Runnable interface throughout this chapter. The following lambda expressions each rely on the Runnable interface:
```
() -> System.out.println("Hello World");
() -> {int 1 = 10; i++;}
() -> {return;}
() -> {}
```

Notice that all of these lambda expressions start with a set of empty parentheses, (). Also notice that none of them return a value. For these reason the following lambdas while valid for other functional interfaces, are not compatible with Runnable:

```
() -> ""
() -> 5
() -> {return new Object();}
```

These examples are invalid Runnable expressions because they each returns a value. In this chapter we focus on creating lambda expressions that implicitly implement the Runnable interface.

## Creating a Thread
The simplest way to execute a thread is using java.lang.Thread class, or Thread for short. Defining the task, or work, that a Thread instance will execute can be done two ways in Java:
- Provide a Runnable object or lambda expression to the Thread constructor.
- Create a class that extends Thread and overrides the run() method.

The following are examples of these techniques. One way implementing Runnable:
```java

public class PrintData implements Runnable{
  public void run(){
    for(int i = =; i < 3; i++){
      System.out.println("Printing record:" + i);
    }
  }
  
  public static void main(String[] args){
    (new Thread(new PrintData())).start();
  }
}

```
The other extending Thread:

```java
public class ReadInventoryThread extends Thread{
  public void run(){
    System.out.println("Printing zoo inventory");
  }
  
  public static void main(String[] args){
    (new ReadInventoryThread()).start();
  }
}

```

Anytime you create a Thread instance, make sure that you remember to start the task with the Thread.start() method. This starts the task in a separate OS thread. For example, what is the output of the following examples?

```java
public static void main(String[] args){
  System.out.println("begin");
  (new ReadInventoryThread()).start();
  (new Thread(new PrintData())).start();
  (new ReadInventory()).start();
  System.out.println("end");
}

```

The answer is that it is unknown until runtime. For example, the following is just one possible output:

```
begin
Pinting zoo inventory
Printing record: 0
end
Printing zoo inventory
Printing record: 1
Printing record: 2
```
This example uses a total of four threads - the main() user thread and three additional threads created by the main() method. While the order of execution once the threads have been started is indeterminate, the order within a single thread is still linear. For example, the for() loop in PrintData is still ordered, as is the end appearing after the begin in the main() method. On the exam, be careful about cases where the Thread or Runnable is created but no start() method is called. While the code will compile, none will actually execute a task on a separate processing thread. Instead, the thread that made the call will be used to execute the task causing the thread to wait until each run() method is complete before moving on to the next line. 

```
new PrintData().run();
(new Thread(new PrintData()).run();
(new ReadInventoryThread()).run();
```

In general you should extend thread under very specific circustances, such as when you are creating your own priority-based thread. In most situations you should implement Runnable interface rather than extend the Thread class. The OCP 8 exam strongly encourages developers to use the Concurrency API to create and manage Thread objects for them.

## Polling with Sleep
Often you need a thread to poll for a result to finish. *Polling* is the process of intermittnely checking data at some fixed interval. For example see example below:
```java
public class CheckResults{
  private static int counter = 0;
  public static void main(String[] args){
    new Thread(() -> {
      for(int i = 0; i < 500; i++){ CheckResult.counter++; }
    }).start();
    
    while(CheckResult.counter < 100){
      System.out.println("Not reached yet");
    }
    System.out.println("Reached!");
  }
}
```

How many times will the while() loop execute and output "Not reached yet?" the answer is, we don't know! It could be zero, ten, or a million times. If our thread scheduler is poor, it could operate infinitely! Using a while() loop to check for data without any delay is considered a very bad coding practice as it ties up with the CPU resources for no reason. We can improve the example by using Thread.sleep() method to implement polling. The Thread.sleep() method requests the current thread of execution rest for a specific number of miliseconds. When used inside the body of the main() method, the thread associated with the main() method will pause, while the separate thread will cointinue to run. Compare the previous implementation with the following one that uses Thread.sleep():

```java
public class CheckResults{  
  private static int counter = 0;  
  public static void main(String[] args) throws InterrupedException{    
    new Thread(() -> {      
      for(int i = 0; i < 500; i++){ CheckResult.counter++; }    
    }).start();        
    
    while(CheckResult.counter < 100){      
      System.out.println("Not reached yet");    
      Thread.sleep(1000);
    }    
    System.out.println("Reached!");  
  }
}
```

While this might seems an small amount, we have prevented a possible infinite loop from executing and locking up our main program. Notice that we also changed the signature of the main method. That is because the sleep method throws a checked exception. Alaternatively we could have used try-catch block insisde the loop. Nother issue to be concerned about is the shared counter variable. What if one thread is reading the counter variable while another thread is writing it? The thread reading the shared variable may end up with an invalid or incorrect value. We will discuss these issues in detail in the upcoming section on synchronization.

# Creating Thread with the ExecutorService

## Introducing the Single-Thread Executor
## Shutting Down a Thread Executor
## Submitting Tasks
### Submitting Task Collections
## Waiting for Results
## Introducing Callable
## Waiting for All Tasks to Finish
## Scheduling Tasks
## Increasing Concurrency with Pools
# Synchronizing Data Access
## Protecting Data with Atomic Classes
## Improving Access with Synchronized Blocks
## Synchronizing Methods
## Understanding the Cost of Synchronization

# Using Concurrent Collections
## Introducing Concurrent Collections
## Understanding Memory Consistency Errors
## Working with Concurrent Classes
## Understanding Blocking Queues
## Understanding SkipList Collections
## Understanding CopyOnWrite Collections
## Obtain Synchronized Collections

# Working with Parallel Streams
## Creating Parallel Streams
### parallel()
### parallelStream()
## Processing Task in Parallel
## Understanding Performance Improvements
## Understanding Independent Operations
## Avoiding Stateful Operations
## Processing Parallel Reductions
### Performing Order-Based Tasks
### Combining Results with reduce()
### Combining Results with collect()
### Using the One-Argument collect() Method

# Managing Concurrent Processes
## Creating a CyclicBarrier
## Applying the Fork/Join Framework
## Working with a RecursiveTask
## Indentifying Fork/Join Issues

# Idetifying Threading Problems
## Understanding Liveness
## Deadlock
## Starvation
## Livelock
## Managing Race Condition
