# Introducing Threads
We begin this chapter reviewing common terminology associated with threads. A *thread* is the smallest unit of execution that can be scheduled by the operative system.

A *process* is a group of associated threads that execute in the same, shared environment. It follows then, that a *single-threaded process* is one that contains exactly one thread. whereas a *multi-threaded process* is one that contains one or more threads.

By *shared environment* we mean that the threads in the same process share the same memory space and can communicate directly with one another. In this chapeter we will talk a lot about tasks and their relationship with threads. A task is a single unit of work performed by a thread. A task will commonly be implemented as a lambda expression in this chapter. A thread can complete multiple independent task but only one task at a time. 

## Distinguishing Thread Types
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
