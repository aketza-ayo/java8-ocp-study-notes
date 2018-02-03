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
With the announcement of the Concurrency API, Java introduced the ExecutorService, which creates and manages threads for you. You first obtain an instance of an ExecutorService interface, and the you send the service task to be processed. The framework includes numerous useful features, such as thread pooling and scheduling  which would be cumbersome for you to implement in every project. Therefore, it is recommended that you use this framework anytime you need to create and execute a separate task, even if you need only a single thread.

## Introducing the Single-Thread Executor
Since ExecutorService is an interface, how do you obtain an instance fo it? The Concurrency API includes the Executors factory class that can be used to create instances of the ExecutorService object. As you may remember the factory pattern is a creational pattern in which the underlying implementation details of the object creation are hidden from us. Let's start with a simple example using the ```newSingleThreadExecutor()``` method to obtain an ExecutorService instance and the execute() method to perform asynchronous task:

```java
import java.util.concurrent.*;

public class ZooInfo{
  public static void main(String[] args){
    
    ExecutorService = null;
    
    try{
      service = Executor.newSingleThreadExecutor();
      
      System.out.println("begin");
      service.execute(() -> System.out.println("Printing zoo inventory"));
      service.execute(() -> {
        for(int i = 0; i < 3; i++){
          System.out.println("Printing record: " + i);
        }
      });
      service.execute(() -> System.out.println("Printing zoo inventory"));
      System.out.println("begin");
    }finally{
      if(service != null) service.shutdown();
    }
  }
  
}

```

As you may notice, this is just a rewrite of our earlier PrintData and RreadInventoryThread clases to use Runnable-based lambda expressions and an ExecutorService instance. In this example, we used the newSingleThreadExecutor() method, which is the simplest ExecutorService that we could create. Unlike our earlier example, in which we had three extra threads for newly created tasks, this example uses only one, which means that the threads will order their results. For example the following is a possible output:

```
begin
Pinting zoo inventory
Printing record: 0
Printing record: 1
end
Printing record: 2
Printing zoo inventory
```

With a single-therad executor, results are guaranteed to be executed in the order in which they are addded to the executor service. Notice that the end text is output while our thread executor tasks are still running. This is because main() method is still an independent thread from ExecutorService, and it can perform tasks while the other thread is running. On the other hand, as you will see later in chapter, when we increase the number of threads in the excutor service,  the gurantee disapears

## Shutting Down a Thread Executor
Once you have finished using a thread executor it is important that you call shutdown() method. A thread executor creates a non-daemon thread on the first task that is executed, so failing to call shutdown() will result in your application never terminating. The shutdown process of a thread executor involves first rejecting any new tasks submitted to the thread executor while continuing to execute any previously submitted tasks. During this time, submitted to the thread executor while it is shutting down, a RejectedExecutionException will be thrown. Onec all active tasks have been completedm isShutdown() and isTerminated() will both return true. See figure below that shows the life cycle of an ExecutorService object.

![ExecutorService life cycle](img/executorServiceCycle.png)

For the exam you should be aware that shutdown() does not actually stop any tasks that have already been submitted to the thread executor. What if you want to cancel all running and upcoming tasks? The ExecutorService provides a method called shutdownNow(), which attempts to stop all running tasks and discards any that have not been started yet. Note that shutdownNow() attempts to stop all running tasks. It is possible to create a thread that will never terminate, so any attempt to interrupt it may be ignored. Lastly, shutdownNow() returns a List<Runnable> of tasks that were submitted to the thread executor but that were never started. The ExecutionService interface does not implement AutoClosable, so you cannot use a try-with-resources statement. You can still use a finally block, as we do throughout this chapter. While not required, it is considered a good practice to do so.

## Submitting Tasks
Ypu can submit tasks to an ExecutionService instance multiples ways. The first method we presented, execute(), is inherited from the Executor interface, which the ExecutorService interface extends. The execute() method takes a Runnable lambda expression or instance and completes the task asynchronously. Because the return type of the method is void it does not tell us anything about the result of the task. It is considered "fire and forget" method, as once it is submitted, the results are not directly available to the calling thread. 
Fortunately, the writers of the Java added submit() methods to the ExecutorService interface, which like execute(), can be use to complete tasks asynchronoulsy. Unlike execute(), though, submit() returns a Future that can be used to determine if the task is complete. It can also be used to return a generic result object after the task has been completed

**Method Name**                  | **Description**
---------------------------------|---------------
void execute(Runnable command)           | Executes a Runnable task at some point in the future.  
Future<?> submit(Runnable task)          | Executes a Runnable task at some point in the future and returns a Future representing tha task.
<\T>Future<T> submit(Callable<\T> task)   | Executes a Callable task at some point in the future and returns a Future representing the pending results of the task.
<\T>List<Future<T>> invokeAll(Collection<? extends Callable<\T> tasks>) throws InterrupedException | Executes the given tasks, synchronoulsy returning the results of all tasks as a Collection or Future objects, in the same order they were in the original collection.
<\T>T invokeAny(Collection<? extends Callable<\T>> tasks) throws InterruptedException, ExecutionException | Executes the given tasks, synchronoulsy returning the result of one of finished tasks, cancelling any unfished tasks.

In practice using the submit() method is quite similar to using the execute() method, except that the submit() method return a Future object that can be used to determine whether or not the task has completed execution. Don't worry if you haven't seen Future or Callable before we will discuss them in detail shortly. We use and prefer the submit() method in the majority of the cases and it this chapter we will use submit() as a result. For the exam you need to be familiar with both execute() and submit(), but in your own code we recommend submit() over execute() whenver possible.

### Submitting Task Collections
The last two methods in the table above that you should know for the exam are invokeAll() and invokeAny(). Both of these methods take a Collection object containing a list of tasks to execute. Both of these methods also execute synchronously. By synchronously, we mean that unlike other methods used to submit tasks to a thread executor, these methods will wait until the results are available before returning control to the enclosing program.

The invokeAll() method executes all tasks in a provided collection and returns a List of ordered Future objects, with one Future object corresponding to each submitted task, in the order they were in the original Collection. Even though Future.isDone() returns true for each element in the returned List, a task could have completed normally or thrown an exception. The invokeAny() method executes a collection of tasks and returns the result of one of the tasks that succesfully completes execution, cancelling all unfinished tasks. While the first task to finish is often returned, this behaviour is not guaranteed, as any completed task can be returned by this method. Finally, the invokeAll() method will wait indefinitely until all tasks are complete, while the invokeAny() method will wait indefinitely until at least one task completes. The ExecutorService interface also includes overloaded versions of invokeAll() and invokeAny() that takes a timeout value and TimeUnit parameter. We will see how to use these types of parameters in the next section when discussing the Future class.

## Waiting for Results
How do we know when a task submitted to an ExecutorService is complete? AS mentioned in the last section, the submit() method returns a java.util.concurrent.Future<V> object, or Future<V> for short, that can be used to determine this result:
  
```
Future<?> future = service.submit(() -> System.out.println("Hello Zoo"));
```
The Future class includes methods that are useful in determining the state of a task, as shown in the table below:

**Method Name**                    | **Description**
-----------------------------------|-------------------------------------------------------------------------------------
boolean isDone()                   | Returns true if the task was completed, threw an exception, or was cancelled.
boolean isCancelled()              | Returns true if the task was cancelled before it completed normally
boolean cancel()                   |  Attempts to cancel execution of the task
V get()                            |  Retrieves the result of a task, waiting endlessly if it is not yet available.
V get(long timeout, TimeUnit unit) | Retrieves the result of a task, waiting the specified amount of time. if the result is not ready by the time the timeout is reached, a checked TimeoutException will be thrown.

The following is an updated version of our earlier polling example CheckResults class,which uses a Future instance to poll for the results:

```java
import java.util.concurrent.*;

public class CheckResults{

  private static int counter = 0;
  public static void main(String[] args) throws InterruptedException, ExecutionException{
      ExecutorService service= null;
      try{
        service = Executor.newSingleThreadExecutor();
        Future<?> result = service.submit(() -> {
          for(int i = 0; i < 500; i++) CheckResults.counter++;
        });
        result.get(10, TimeUnit.SECONDS);
        System.out.println("Reached!");
      }catch(TimeoutException e){
        System.out.println("Not reached in time")
      }finally{
        if(service != null) service.shutdown();
      }
  }
}
```

This example is similar to our ealier polling implementation, but it does not use the Thread class directky. In part, this is the essence of the Concurrency API: to do complex things with threads without using Thread class directly. It also waits at most 10 seconds, throwing a TimeoutException if the task is not done. What is the return value of this task? As Future<V> is a generic class, the type V is determined by the return type of the Runnable method. Since the return type of Runnable.run() is void, the get() method always returns null. In the next section, you will see that there is  another task class compatible with ExecutorService that supports other return types. As you saw in the previous example, the get() method can take an optional value and enum type java.util.concurrent.TimeUnit. We represent the full list of TimeUnit values in the table below in increasing order of duration. Note that numerous methods in the Concurrency API use the TimeUnit enum. The TimeUnit values are:
- TimeUnit.NANOSECONDS
- TimeUnit.MICROSECONDS
- TimeUnit.MILLISECONDS
- TimeUnit.SECONDS
- TimeUnit.MINUTES
- TimeUnit.HOURS
- TimeUnit.DAYS
  
## Introducing Callable
When Concurrency API was released in Java 5 the java.util.concurrent.Callable interface was added. It is similar to Runnable except that its ```call()``` method returns a value and can throw a checked exception. As you may remember from the definition of Runnable, the ```run()``` method returns void and cannot throw any checked exceptions. Along with Runnable, Callable was also made a funcionalInterface in Java 8. The following is the definition of the Callable interface:
```java
@FunctionaInterface
poublic interface Callable<V>{
  V call() throws Exception;
}
```
The Callable interface was introduced as an alternative to the Runnable interface, since it allows more details to be retrieved easily from the task after it is completed. The ExecutorService includes an overloaded version of the ```submit()``` method that takes a Callable object and returns a generic Future<T> object.
  
:yin_yang: **Ambiguous Lambda Expressions: Callable vs. Supplier** The Callable functional Interface strongly resembles the Supplier functional interface , in that they both take no arguments and return a generic type. Once difference is that Callable can throw a checked exception. How do you tell lambda expression for these two parts? The answer is that sometimes you cannot. See the following example:
```java
public class AmbiguousLambdaExample{
  public static void useCallable(Callable<Interger> expression){}
  public static void useSupplier(Supplier<Interger> expression){}
  public static void use(Callable<Interger> expression){}
  public static void use(Supplier<Interger> expression){}
  
  public static void main(String[] args){
    useCallable(() -> {throw new IOException();});    // COMPILES
    useSupplier(() -> {throw new IOException();});    // DOES NOT COMPILE
    use(() -> throw new IOEXception(););              // DOES NOT COMPILE
  }
}

```
The first line of the main method compiles as Callable is permitted to throw checked exception, unlike the Supplier in the second line. What about last line? the use method is overloaded but the compiler does not take into account the fact that the body of the lambda expression happens to throw an exception; when the compiler does not what to do it reports an error and does not compile. When the compiler is unable to assign a functional interface to a lambda expression, it is referred to as an ambiguous lambda expression. Note that ambiguity can be resolved with an explicit cast. For example the following will compile:

```java
use((Callable<Integer>)() -> {throw IOException()};);   // COMPILES
```

Unlike Runnable in which the ```get()``` method always returns null, the ```get()``` methods on the Furute object return the matching generic type or null.

Let's take a look to some examples:

```java
import java.util.concurrent.*;

public class AddData{
  public static void main(String[] args) throws InterruptedException, ExecutionException{
    ExecutorService service = null;
    try{
      service = Executor.newSingleThreadExecutor();
      Future<Integer> result = service.submit(() -> 30 + 11);
      System.out.println(result.get());
    
    }finally{
      if(service != null) service.shutdown();
    }
  }
}
```
We can now retrieve and print the output of the Callable results, 41 in this example. The results could have also been obtained using Runnable and some shared, possibly static, object, although this solution using Callable is a lot simpler to follow. The Callable and Runnable interface are interchangeable is some situations where the lambda does not throw an exception and there is no return type.

**Checked Exceptions in Callable and Runable:** Besides having a return type the Callable interface also supports Checked Exceptions, whereas the Runnable interface does not without an embedded ```try/catch``` block. Given an instance of ExecutionService called service, which of the following lines of code will compile or not?

```java
service.submit(() -> {Thread.sleep(1000); return null; });
service.submit(() -> {Thread.sleep(1000); });
```

The first line compile, while the second line does not. Why? recall that Thread.sleep() throws a checked InterruptedException. Since the first lambda expression has a return type, the compiler treats this as Callable expression that supports checked exceptions. The second lambda expression does not have a return value; therefore the compiler treats it as Runnable expression. And Runnable does not support checked exceptions , the compiler reports an error and does not compile this code.

## Waiting for All Tasks to Finish
After submitting a set of tasks to a thread executor, it is common to wait for the resut. One solution is to call ```get()``` method on each Future object  returned by the ```submit()``` method. If we don't need the results of the tasks and are finished using our thread executor, there is a much simpler approach. First we shutdown the tread executor using ```shutdown()``` method. Next, we use the ```awaitTermination(long timeout, TimeUnit unit)``` method available for all thread executors. The method waits the specified time to complete all tasks, returning sooner if all task finish or an InterruptedException is detected. You can see an example of this in the following cde snippet:
```java
ExecutorService service = null;
try{
  service = ExecutorService.newSingleThreadExecutor();
  
  //add tasks to the thread executor
  
}finally{
  if(service != null) service.shutdown();
}

if(service != null){
  service.awaitTermination(1, TimeUnit.MINUTES);
  //Check whether all tasks are finished
  if(service.isTerminated()){
  `System.out.println("All tasks are finished");
  }else{
    System.out.println("At least one task is still running");
  }
}
```
In this example we submit a number of tasks to the thread executor and then shutdown the thread executor and wait up to one minute for the result. Notice that we call isTerminated() after awaitTermination() method finishes to confirm that all tasks are actually finished.

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
