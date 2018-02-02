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
## Introducing Runnable
## Creating a Thread
## Polling with Sleep

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
