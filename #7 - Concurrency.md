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
Oftentimes in Java, we need to schedule a tasks to happen at some point in the future time. We might even need to schedule the task to happen repeatedly, at some set interval. For example, imagine you want to check the food supply of zoo for the animals once an hour and fill it as needed. The ```ScheduleExecutorService``` which is a subinterface of ExecutorService, can be used for just such task. Like ```ExecutorService``` we obtain an instance of ```ScheduleExecutorService``` using a factory method in the Executors class, see below:
```
ScheduleExecutorService service = Executors.newSingleThreadScheduledExecutor();
```
Note that we could implicitly cast an instance of ScheduleExecutorService to ExecutorSerice, although doing so will remove the scheduled method that we want to use.

![Scheduled Executor Service Methods](img/scheduledExecutorServiceMethods.png)

In practice, these methods are the most convenient in the Concurrency API, as they perform complex tasks with a single line of code. The first two methods take a Callable or a Runnable, perform the task after some delay, and return a ScheduledFuture<V> instance. ScheduledFuture<V> is identical to Future<V> class, except that it includes a getDelay() method that returns the delay set when the process was created. The following uses the schedule() method with Callable and Runnable task.
  
```java
ScheduleExecutorService service = Executor.newSingleThreadScheduledExecutor();

Runnable task1 = () -> System.out.println("Hello Zoo");
Callable<String> task2 = () -> "Monkey";

Future<?> result1 = service.schedule(task1, 10, TimeUnit.SECOND);
Future<?> result2 = service.schedule(task2, 8, TimeUnit.MINUTES);
```

The first task is scheduled 10 seconds in the future, whereas the second task is scheduled 8 minutes in the fuuture. Note that these task are scheduled in the future, the actual execution may be delayed. For example, there may be no threads available to perform the task, at which point they will just wait in the queue. Also, if the ScheduledExecutorService is shutdown by the time the scheduled task execution is reached they will be discarded.

The last two method in the table above might be a little confusing if you have not seen them before. They both seem to be similar. The difference is related to the timing of the process and when the next task starts. The ```scheduleAtFixedRate()``` method creates a new task and submits it to the executor every period, regardless of whether or not the previous task finished. The following example executes a Runnable task every minute, following an initial five minute delay.

```
service.scheduleAtFixedRate(command, 5, 1, TimeUnit.MINUTE);
```

One risk of using this method is that a task could consistently take longer to run than the period between tasks. Despite the fact that the task is still running, The ScheduledExecutorService would summit a new task to be started every minute. is a single thread executor was used, over time this would result in endless set tasks beign scheduled, which would run back to back assuming that no other tasks were submitted to the ScheduledExecutorService. On the other hand, the scheduledAtFixedDelay() method creates a new task after the previous task has finished. For example, if the first task runs at 12:00 and takes five minutes to finish, with a period of 2 minutes, then the second task will start at 12:07.

```
service.scheduleAtFixedDelay(command, 0, 2, TimeUnit.MINUTE);
```
Notice that neither of the methods, scheduleAtFixedRate() and scheduleAtFixedDelay() take a callable object as a input parameter. Since these tasks are scheduled to run infinetly, as long as the ScheduledExecutorService is still alive, the would generate an endless series of Future objects.

Each of the ScheduledExecutorService methods is important and has real-world applications. For example, you can use the schedule() command to check on the state of processing a task and send out notifications if it is not finished or even call schedule() again to delay processing.

The scheduleAtFixedRate() is useful for tasks that need to be run at specific intervals, such as checking the health of the animals once a day. Even if it takes two hours to examine an animal on Monday, this does not mean that Tuesday examination will start any later.

Finally, scheduleAtFixedDelay() is useful for process that you want to happen repeately but whose specific time is unimportant. For example, imagine that we have a zoo cafeteria worker who peridically restock the salad bar throughout the day. The process can take 20 min or more, since it is requires the worker to haul a large number of items from the back room. Once the worker has filled the salad bar with fresh food, he doesnt need to check at some specific time, just after enough time has passed for it to become low on stock again.

## Increasing Concurrency with Pools
All of our examples up until now have been with single-thread executors, which, while interesting , weren't particularly useful. After all, the name of this chapter is "Concurrency",and you can't do a lot of that with a single-thread executors!
We now present three additional factory methods in the Executor class that act on a pool of threads, rather than on a single thread. A *thread pool* is a group of pre-instantiated reusable threads that are available to perform a set of arbitrary tasks. The table below includes our two previous single-thread executor methods, along with the new one that you should know for the exam.

**Method Name**                           |  **Return type**  | **Description**
------------------------------------------|-------------------|-------------------------------------------------------------
newSingleThreadExecutor()                 | ExecutorService   | Creates a single threaded executor that uses a single worker thread operating off an unbounded queue. Results are processed sequentially in order in which they are submitted.
newSingleThreadScheduledExecutor()        | ScheduledExecutorService | Creates single-threaded executor that can schedule commands to run after a given delay or to execute periodically.
newCachedThreadPool()                     | ExecutorService  | Creates a thread pool that creates new threads as needed, but will reuses previously constructed threads when they are available.
newFixedThreadPool(int nThreads)          | ExecutorService  | Creates a thread pool that reuses a fixed number of threads operating off a shared unbounded queue.
newScheduledThreadPool(int nThreads)      | ScheduledExecutorService | Creates a thread pool that can schedule commands to run after a given delay or to execute periodically.



As shown in table above these methods return the exact same instance types, ExecutorService and ScheduledExecuterService that we used earlier in this chapter. In other words, all of our previous examples are compatible with these new pooled-thread executors! There are also overloaded versions of each of the methods in the table above that create threads using a ThreadFactory input parameter. For the exam, you are only required to know the methods to create thread executors in the table above.

The difference betwen a single-thread and a pooled-thread executor is what happends when a task is already running. While a single-thread executor will wait for an available thread to become available before running the next task, a pooled-thread executor can execute the next task concurrently. If the pool runs out of available threads, the task will be queued by the thread executor and wait to be completed.

The newCachedThreadPool() method will create a thread pool of unbounded size, allocating a new thread anytime one is required or all existing threads are busy. this is commonly used for pools that require executing many short-lived asynchronous tasks. For long-lived processes, usage of this executor is strongly dioscouraged, as it could grow to encompass a large number of threads over the application life cycle.

The newFixedThreadPool() takes a number of threads and allocates them all upon creation. As long as our number of tasks is less than our number of threads, all tasks will be executed concurrently. If at any point the number of tasks exceed the number of threads in the pool, they will wait in similar manner as you saw with a single-thread executor. In fact, calling newFixedThreadPool() with a value of 1 is equivalent to calling newSingleThreadExecutor().

The newScheduledThreadPool() is indentical to the newFixedThreadPool() method, except that it returns an instance of ScheduledExecutorService and is therefore compatible with scheduling tasks. This executor has subtle differences in the way that the scheduleAtFixedRate() performs. For example, recall our previous example in which tasks took five minutes to complete:

```
ScheduledExecutorService service = Executors.newScheduledThreadPool(10);
service.scheduleAtFixedRate(command, 3, 1, TimeUnit.MINUTE);
```

Whereas with a single-thread executor and five-minutes task execution time, an endless set of tasks would be scheduled over time. With a pooled executor, this can be avoided. if the pool size is sufficiently large, 10 for example, then as each thread finishes, it is returned to the pool and results in new threads available for the next tasks as they come up.

# Synchronizing Data Access
Recall that thread safety is the property of an object that guarantees safe execution by multiple threads at the same time. Now that we have multiple threds capable of accessing the same objects in memory, we have to make sure to organize our access to this data such that we don't end up with invalid or unexpected results. Since threads run in a shared environment and memory space, how do we prevent tow threads from interfering with each other?

Imagine our zoo has a program to count sheep, preferably one that won't put the zoo workers to sleep! Each zoo worker runs out to a field, adds a new sheep to the flock, count the total number of sheep, and runs back to us to report the result. We present the following code to represent this conceptually:

```java
import java.util.concurrent.*;

public class SheepManager{
  
  private int sheepCount = 0;
  private void incrementAndReport(){
    System.out.println((++sheepCount)+ " ");
  }
  
  public static void main(String[] args){
    ExecutorService service = null;
    try{
      service = Executors.newFixedThreadPool(20);
      SheepManager manager = new SheepManager();
      
      for(int = 0; i < 10; i++){
        service.submit(() -> manager.incrementAndreport());
      }
    }finally{
      if(service != null) service.shutdown();
    }
  }

}

```

A problem occurs when two threads both execute the right side of the expression, reading "old" value before either thread writes the new value of the variable. The two assignments become redundant; they both assign the same new value, with one thread overwriting the results of the other. Both threads read and write the same values, causing one of the two ++sheepCount operations to be lost. Therefore, the increment operator ++ is not thread safe. As you will see later in this chapter, the unexpected result of two tasks executing at the same time is referred to as *race condition*.

Returning to the sheep example, we choose a large thread size of 20 so that all tasks can be run concurrently. Let's say that each lambda expression submitted to the thread executor corresponds to a zoo worker. Each time a zoo worker increments the sheep counter, they run back to report the results. What would you expect the putput of this program to be? Although the putput will vary, the following are some samples created by this program:

```
1 2 2 3 4 5 6 7 8 9

2 4 5 6 7 8 1 9 10 3

2 1 3 4 5 6 7 8 9 10
```

In this example, multiple workers are sharing the sheepCount variable. In the first sample, two zoo workers both call ++sheepCount at the same time, resulting in one of the increment operations actually beign lost, with the last total beign 9 instead of 10. In the other examples, results from earlier threads are output before ones that started later, such as 3 beign output after 4 in the second example. We know we had 10 workers but results are incomplete and out of order.
The idea here is that some zoo workers may run faster on their way to the field but more slowly on their way back and report late.

## Protecting Data with Atomic Classes
Withe the release of the ConcutrrentyAPI, Java added a new ```java.util.concurrent.atomic``` package to help coordinate access to primitive values and object references. As with most classes in the Concurrency API, these classes are added solely for convenience. In our first SheepManager smaple output, the same value, 2, was printed twice, with the highest counter beign 9 instead of 10. As we demonstrated in the previous section, the increment operator ++ is not thread safe. Furthermore, the reason that it is not thread safe is that the operation is not atomic, carrying out two tasks, read and write, that can be interrupted by other threads.
*Atomic* is the property of an operation to be carried out as single unit of execution without any interference by another thread, A thread-safe atomic version of the increment operator would be one that performed the read and write of the variableas as single operation, not allowing any other threads to access the variable during the operation. Any thread trying to access the sheepCount variable while an atomic operation is in process will have to wait until the atomic operation on the variable is complete. Of course, this exclusivity applies only to threads trying to access the sheepCount variable, with the rest of memory space not affected by this operation. Since accessing primitives and references in Java in shared environments, the concurrency API includes numerous useful classes that are conceptually the same as our primitive classes but that support atomic operations. The table below list atomic classes with which you should be familiar with for the exam:

**Class Name**       |  **Description**
---------------------|-----------------------------------------------------------------------------
AtomicBoolean        | A boolean value that may be updated automatically
AtomicInteger        | An int value that may be updated automatically
AtomicIntegerArray   | An int array in which elements may be updated automatically
AtomicLong           | A long value that may be updated automatically
AtomicLongArray      | A long array in which elements may be updated automatically
AtomicReference      | A generic object reference that may be updated automatically
AtomicReferenceArray | An array of generic object references in which elements may be updated automatically

How do we use an atomic class? Each class includes numerous methods that are equivalent to many of the primitve built-in ioerators that we use on primitves, such as the assignment operator = and the increment operators ++. We describe the common atomic methods that you should know for the exam in the table below:

**Method Name**   |  **Description**
------------------|-----------------------------------------------------------------------------
get()             | Retrieve the curent value
set()             | Set the given value, equivalent to the assignment = operator
getAndSet()       | Atomically sets the new value and returns old value
incrementAndGet() | For numberic classes, atomic pre-increment operation equivalent to ++value
getAndIncrement() | For numeric classes, atomic post-increment operation equivalent to value++
decrementAndGet() | For numeric classes, atomic pre-decrement operation equivalent to --value
getAndDecrement() | For numeric classes, atomic post-decrement operations equivalent to value--

In the following examples, we update our SheepManager class with an AtomicInteger:

```java
private AtomicInteger sheepCount = new AtomicInteger(0);

private void incrementAndReport(){
  System.,out.println(sheepCount.incrementAndGet() + " ");      
}
```

When we run the above we get the following outcome:

```
2 3 1 4 5 6 7 8 9 10

1 4 3 2 5 6 7 8 9 10

1 4 3 5 6 2 7 8 10 9

```

Unlike our previous sample output, the number 1 through 10 will always output. As you might notice, the result are still not ordered, although we will get to that soon. The key here is that using the atomic classes ansures that the data is consistent between workers and that no values are lost due to concurrent modifications.

## Improving Access with Synchronized Blocks
How do we improive the result so that each worker is able to increment and report the result in order? The most common technique is to use a monitor, also called a *lock* to synchronize access. A *monitor* is a structure that supports mutual exclusion or the property that at most one thread is executing a particular segment of code at a given time. In Java, any object ca be used as monitor, along with ```synchronized``` keyword, as shown in the following example:

```
SheepManager manager = new SheepManager();
synchronized(manager){
  // work to be completed by one thread at a time
}
```

This example is referred to as *synchronized block*. Each thread that arrives will first check if any threads are un the block. In this manner, a thread "acquires a lock" for the monitor. If the lock is available, a single thread will enter the block, acquiring the lock and preventing all other threads from entering. While the first thread is executing the block, all threads that arrive will attempyt to acquire the same lock and wait for first thread to finish. Once a thread finishes executing the block, it will release the lock, allowing one of the waiting threads to proceed.

Note that in order to synchronize access across multiple threads, each threads must have access to the same Object. For example, synchronizing on different objects would not actually organize the result.

Let's revisit our SheepManager example and see if we can improve the results so that each worker increments and outputs the counter in order. Let's say that we replaced our for() loop with the following implementation:

```java
for(int i = 0; i < 10; i++){
  synchronized(manager){
    service.submit(() -> manager.incrementAndReport());
  }
}

```

Does this solution fix the problem? No it does not. Can you spot the problem? We have synchronized the creation of the threads but not the execution of the threads. In this exampl, each thread would be created one at a time, but they may all still execute and perform their work art the same time., resulting in the same type of output that you saw earlier. Diagnosing resolving threading problems is often one of the most difficult tasks in any programming language.

We now represent a corrected version of the SheepManager class, which does not order the workers:

```java

import java.util.concurrent.*;

public class SheepManager{
  
  private int sheepCount = 0;
  private void incrementAndReport(){
    synchronized(this){
      System.out.println((++sheepCount) + " ");
    }
  }
  
  public static void main(String[] args){
    ExecutorService service = null;
    
    try{
      service = Executor.newFixedThreadPool(20);
      service.submit(() -> manager.incrementAndReport());
      
    }finally{
      if(service != null) service.shutdown();
    }
  }

}
```

When this code executes, it will consistenly output the following:
```
1 2 3 4 5 6 7 8 9 10
```
Although all threads are sitll created and executed at the same time, they each wait at the synchronized block for the workers to increment and reports the result before entering. In this manner, each zoo worker waits for the previous zoo worker to come back before running out on the field. While it's random which zoo worker will run out next, it is guanranteed that there will be at most one on the field.

We could have synchronized on any object, so lonmg as it was the same object. For example, the following code snippet would have also worked:

```java

private final Object lock = new Object();
private void incrementAndReport(){
  synchronized(lock){
    System.out.println((++sheepCount) + " ");
  }
}
```

Although we didn't need to make the lock variable final, doing so ensured that is not reassigned after threads starts using it. Note that we could have used an atomic count variable along with the synchronized block in this example, although it is unnecessary. Since synchronized blocks allow only one thread to enter, we're not ganing any improvement by using an atomic variable only time that we access the variable is within synchronized block.

## Synchronizing Methods
In the previous example, we estabilished our monitor using ```synchronized(this)``` around the body of the method. Java actually provides a more convenient compiler enhancement for doing so. We can add the synchronized modifier to any instance method to syncronize automatically on the objet itself. For example, the follosing two method definitions are equivalent:

```java
private void incrementAndReport(){
  synchronized(this){
    System.out.println((++sheepCount) + " ");
  }
}

private synchronized void incrementAndReport(){  
  System.out.println((++sheepCount) + " ");
}
```

The first uses a synchronized block whereas the second method uses a synchronized method modifier. Which one you use if completely up to you. We can also add the synchronzed modifier to static methods. What object is used as the monitor when we synchronize on a static method? The class object, of course! For example the following two methods are equivalent for static synchronization inside our SheepManager class:

```java
public static void printDaysWork(){
  synchronized(SheepManager.class){
    System.out.println("Finished work");
  }
}

public static synchronized void printDaysWork(){
  System.out.println("Finished work");
}
```

As before, the first uses a synchronized block, with the second example using the synchronized modifier. You can use static synchronization if you need to order thread access across all instances, ratrher trhan a single instance.

## Understanding the Cost of Synchronization
We complete this section by noting that synchronization, while useful, may be constly in practice. While multi-threaded programming is about doing multiple things at the same time, synchronization is about taking multiple threads and making them perform in a more single threaded manner.
For example, let's say that we have a highly concurrent class with numerous methods that synchronize on the same object. Let's say that 50 concurrent thread access it. Let's also say that, on average, each thread takes a modes 100 millisecs to execute. In this example, if all of the threads try to access the lock/monitor at the same time, how long will it take for them to complete their work, assuming that 50 threads are available in the thread pool?

```
50 threads * 100 millisecs
  = 5,000 millisecs
  = 5 secs
```

Even though five seconds many not seem like a lot, it's actually pretty long in computter time. What if 50 new tasks are created before the five seconds are up? This will pile up into the workload, resulting in most threads constantly entering a waiting or "stuck" state. In this application, this may cause tasks that would normally be quick to finish ina a non-synchronized environment to take a significantly long amount of time to complete. Synchronization is about protecting data integrity at the cost of performance. In many cases, performance costs are minimal, but in extreme scenarios the application could slow down significanlty due to the inclusion of synchronization. Beign able to identify synchronization problems, including finding ways to improve performance in synchronized multi-threaded environments, is a value of skills in practice.

# Using Concurrent Collections
Besides managing threads, the Concurrency API includes interfaces and classes that help you coordinate access to collections across multiple tasks. By collections, we are of course referring to the Java Collections Framework that we introduced in chapter 3 "generics and collections". In this section, we will demonstrate many of the new concurrent classes available to you when using the Concurrency API.

## Introducing Concurrent Collections
The first questions you might be asking is "Do we really need a new concurrent collection classes?". After all, in the previous section you saw that we can use the synchronized keyword on any method or block, so couldn't we do the same for our existing collection classes? The short answer is "We could". For example, take a look a the following code to access a map using synchronized keyword:

```java
public class ZooManager{
  private Map<String, Object> fooData = new HashMap<String, Object>();
  
  public synchronized void put(String key, String value){
    fooData.put(key, value);
  }
  
  public synchronized Object get(String key){
    return fooData.get(key);
  }
}

```

So then, why use a concurrent collection class? Like using ExecutorService to manage threads for us, using concurrent collections is extremely convenient in practice. It also prevents us from introducing mistakes in own custom implementation, such as if we forgot to synchronize one of the accessor methods. In fact, the concurrent collections often include performance enhancements that avoid unnecessary synchronization. Accessing collections from accross multiple threads is so common that the writters of Java thought it would be a good idea to have alternative versions of many of the regular collection classes just for multi threaded access. The following is an alternate version of our implementation that does not use the synchronized keyword but instead uses a concurrent collection class:

```java
public class ZooManager{
  private Map<Strin,Object> foodData = new ConcurrentHashMap<String,Object>();
  
  public void put(String key, String value){
    foodData.put(key, value);
  }
  
  public Object get(String key){
    return foodData.get(key);
  }

}

```

You may notice that this code is almost the same to our previous example. In fact, even our reference type for the object, Map, remained unchanged. As you may remeber from our discussion of polymorphism in capter 2, even though the reference type changes, the underlying object is still a ConcurrentHashMap. Also, notice that since ConcurrentHashMap implements Map, it uses the same get()/put() methods. Therefore, there is no need to use the ConcurrentHashMap reference type in this example.

## Understanding Memory Consistency Errors
The purpose of the concurrent collection classes is to solve common memory consistency errors. A *memory consistency error* occurs when two threads have inconsistent views of what should be the same data. Conceptually, we want writes on one thread to be available to another thread if it accesss the concurrent collection after the write has occurred.  When two thread try to modify the same non-concurrent collection, the JMV may throw a ConcurrentModificationException at runtime. In fact, it can happen witha single thread. Take a look at the following code snippet:

```java
Map<String, Object> fooData = new HashMap<String, Object>();
foodData.put("penguin",1);
foodData.put("flamingo", 2);
for(String key: foodData.keySet()){
  foodData.remove(key);
}

```
This snippet will throw a ConcurrentModificationException at runtime, since the iterator keySet() is not properly updated after the first element is removed. Changing the first line to use a ConcurrentHashMap will prevent the code from throwing an exception at runtime:

```java
Map<String, Object> foodData = new ConcurrentHashMap<String, Object>();
foodData.put("penguin", 1);
foodData.put("flamingo", 2);
for(String key: foodData.keySet()){
  foodData.remove(key);
}
```

Although we don't usually modify a loop variable, this example highlights the fact that the ConcurrentHashMap is ordering read/write access such that all access to the class is consistent. In this code snippet, the iterator created keySet() is updated as soon as an object is removed from the Map. The concurrent classes help to avoid common issues in which multiple threads are adding and removing objects from the same collections. At any given instance, all threads are adding and removing objects from the same collections. At any given instance, all threads should have the same consistent view of the structure of the collection.

## Working with Concurrent Classes
There are numerous collections classes with which you should be familiar for the exam. Luckily, you already know how to use most of them, as the methods available are a superset to the non-concurrent collection classes that you learned about it in chapter 3.
You should use a concurrent collection class anytime that you are going to have multiple threads modify a collections object outside a synchronized block or method, even if you don't expect a concurency problem. On the other hand, if all of the threads are accessing an establish immutable or read-only collection, a concurrent collection class is not required. In the same way that we instantiated an ArrayList object but pass around a List reference, it is considered a good practice to instantiate a concurrent collection but pass it around using a non-concurrent interface whenever possible.

The table below list a common concurrent classes with which you should be familiar for the exam:

**Class Name**    |    **Java Collection Framework Interface**  | **Elements ordered?** |  **Sorted?**  |   **Blocking?**
------------------|---------------------------------------------|-----------------------|---------------|-----------------
ConcurrentHashMap | ConcurrentMap                               | No                    | No            | No
ConcurrentLinkedDeque | Deque                                   | Yes                   | No            | No
ConcurrentLinkedQueue|  Queue                                   | Yes                   | No            | No
ConcurrentSkipListMap |  ConcurrentMapSortedMapNavigableMap     | Yes                   | Yes           | Yes
ConcurrentSkipListSet |  SortedSetNavigableSet                  | Yes                   | Yes           | No
CopyOnWriteArrayList  | List                                    | Yes                   | No            | No
CopyOnWriteArraySet   | Set                                     | No                    | No            | No
LinkedBlockingDequeue | BlockingQueue / BlockingDeque           | Yes                   | No            | Yes
LinkedBlockingQueue   | BlockingQueue                           | Yes                   | No            | Yes

Based on your knowledge on Collections from Chapter 3, classes like ConcurrencyHashMap, ConcurrentLinkedQueue and ConcurrentLinkedDeque should be quite easy for you to learn. Take a look at the following sample:

```java
Map<String, Integer> map = new ConcurrentHashMap<>();
map.put("zebra", 52);
map.put("elephant", 10);
Sustem.out.println(map.get("elephant"));

Queue<Integer> queue = new ConcurrentLinkedQueue<>();
queue.offer(31);
System.out.println(queue.peek());
System.out.println(queue.poll());

Deque<Integer> deque = new ConcurrentLinkedDeque<>();
deque.offer(10);
deque.push(4);
System.out.println(deque.peek());
System.out.println(deque.pop());

```

As you may have noticed, these samples strongly resemble the collection snippets that you saw earlier in this book, with the only differnce beign the object creation call. In each of the samples, we assign an interface reference to newly created object and use it the same way as we would a non-concurrent object.

The ConcurrentHaspMap implements the ConcurrentMap interface, also found in the Concurrency API. You can use either reference type Map or ConcurrentMap, to access a ConcurrentHashMap object, depending on whther or not you want the caller to know anything about underlying implementation. For example, a method signature may require a ConcurrentMap reference to ensure that object passed to it is properly supported in multi-threaded environment.

## Understanding Blocking Queues
As you may have noticed in the previous table, it included two queue classes that implement blocking interfaces: linkedBlockingQueue and LinkedBlockingDeque. The BlockingQueue is just like a regular Queu, except that it includes methods that will wait specific amount of time to complete an operation. Since BlockingQueue inherits all of the methods from Queue, we skip the inherited methods and present the new waiting methods, see table below BlockingQueue waiting methods:

**Method Name**                           | **Description**
------------------------------------------|-----------------------------------------------------------------------------
offer(E e, long timeout, TimeUnit unit)   | Adds item to the queue waiting the specified time, returning flase if time elapses before space is available
poll(long timeout, TimeUnit unit)         | Retrieves and removes an item from the queue, waiting the specified time, returning null if the time elapses before the item is available

A LinkedBlockingQueue as the name implies mantains a linked list between elements. The following sample is using a LinkedBlockingQueue to wait for the results of some of the operations. The methods in table above can each throw a checked IterruptedException, as they can be interrupted before they finish waiting for result; therefore they must be properly caught.

```java
try{
  BlockingQueue<Integer> blockingQueue = new LinkedBlockingQueue<>();
  
  blockingQueue.offer(39);
  blockingQueue.offer(3, 4, TimeUnit.SECONDS);
  
  System.out.println(blockingQueue.poll());
  System.out.println(blockingQueue.poll(10, TimeUnit.MILLISECONDS));
}catch(InterruptedException e){
  // handle exception
}

```
As shown in this example, since LinkedBlockingQueue implements both Queue and BlockingQueue, we can use methods available to both, such as those that don't take any wait argument. The other table above also includes LinkedBlosckingDeque class that mantains a doubly linked list between elements and implements a BlockingDeque interface. The BlockingDeque interface extends Deque much in the same way that BlockingQueue extends Queue, providing numerous waiting methods. See table below for waiting methods defined in BlockingDeque:

**Method Name**                              |    **Description**
---------------------------------------------|-----------------------------------------------------------------------------
offerFirst(E e, long timeout, TimeUnit unit) | Adds an item to the front of the queue, waiting a specified time, returning false if time elapses before space is available.
offerLast(E e, long timeout, TimeUnit unit)  | Adds an item to the tail of the queue, waiting a specified time, returning false if time elapses before space is available.
pollFirst(long timeout, TimeUnit unit)       | Retrieves and removes an item from the front of the queue, waiting the specified time, returning null if the time elapses before the item is available.
pollLast(long timeout, TimeUnit unit)        | Retrieves and removes an item from the tail of the queue, waiting the specified time, returning null if the time elapses before the item is available.

The following is a smaple code of using LinkedBlockingDeque. As before, since the methods in the table above each throw a checked InterruptedException , they must be properly caught in the code that uses them.
```java
try{
  BlockingDeque<Integer> blockingDeque = new LinkedBlockingDeque<>();
  
  blockingDeque.offer(91);
  blockingDeque.offerFirst(5, 2, TimeUnit.MINUTES);
  blockingDeque.offerLast(47, 100, TimeUnit.MICROSECONDS);
  blockingDeque.offer(3, 4, TimeUnit.SECONDS);
  
  System.out.println(blockingDeque.poll());
  System,out.println(blockingDeque.poll(950, TimeUnit.MILLISECONDS));
  System,out.println(blockingDeque.pollFirst(200, TimeUnit.NANOSECONDS));
  System,out.println(blockingDeque.pollLast(1, TimeUnit.SECONDS));
  
}catch(){
  //handle interruption
}
```

This exam creates a LinkedBlockingDeque and assigns it to a BlockingDeque rederence. Since BlockingDeque extends Queue, deque and BlockingQueue, all of the previously defined queue methods are available for use.

## Understanding SkipList Collections
The SkipList classes, ConcurrentSkipListSet and ConcurrentSkipListMap, are concurrent versions of their sorted counterparts, TreeSet and TreeMap respectively. They maintein their elements or keys in the natural ordering of their elements. When you see SkipList or SkipSet on the exam, just think "sorted" concurrent collections and the rest should follow naturally.
Like other queue examples, it is recommeded that you assign these objects to interface references, such as SortedMap or NavigableSet. In this manner, using them is the same as the code that you worked with in Chapter 3.

## Understanding CopyOnWrite Collections
The concurrent collection classes table from before included two classes, CopyOnWruteArrayList and CopyOnWriteArraySet, that behave a little differently than the other concurrent examples that you have seen. These classes copy all of their element to a new underlying structure anytime an element is added, modified, or removed from the collection. By a modified element, we mean that the reference in the collection is changed. Modifying the actual contents of the collection will not cause a new structure to be allocated. 
Although the data is copied to a new underlying structure, our reference to the object does not change. This is particularly useful in multi-threaded environments that need to iterate the collection. Any iterator established prior to a modification will not see the changes, but instead it will iterate over the original elements prior to the modification. Let's see an example:

```java
List<Integer> list = new CopyOnWriteArrayList<>(Arrays.asList(4,3,52));
for(Integer item : list){
  System.out.println(item + " ");
  list.add(9);
}

System.out.println();
System.out.println("Size: " + list.size());
```

When executed as part of a program, this code snippet outputs the following:
```
4, 3, 52
Size: 6
```
Despite aadign elements to the array, while iterating over it, only those elements in the collection at the time the for() loop was created were accessed. Alternatively, if we had used a regular ArrayList object, a ConcurrentModificationException would have been thrown at runtime. With either class, though, we avoid entering an infinite loop in which elements are constantly added to the array as we iterate over them.

Note: the CopyOnWrite classes are similar to the immutable object pattern that you saw in Chapter 2, as a new underlying structure is created every time
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
