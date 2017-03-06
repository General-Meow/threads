Java Threads - a quick run though

Interfaces, classes and methods of interest

Runnable interface
  - Used to create tasks for Threads
  - Implement this interface to provide the task
  - Runnable types do not return values
Callable interface
  - Just like the Runnable interface, its used to define tasks
  - Callable is used if you want tasks to return values from its execution
Thread class
  - Instances of this type are the threads that run tasks
  - Can be constructed with or without a runnable type
  - When provided with a runnable at construction, will run that task when started
  - Can extend this class to provide its own task
  - Can be set to daemon threads. these threads are best served as background processes
  - Normal threads are also known as user threads (as well as the main thread).
  - Daemon threads will abruptly die when all user threads die
Executor interface
  - Interface that provides a single execute methods that other executors Implement
ExecutorService interface
  - Extends the Executor interface and provides additional important methods
    - shutdown()
    - isShutdown()
    - shutdownNow()
    - submit()
    - etc
  - When using an executor service pool you must remember to shut it down!!!
  - To provide an executor pool with a task, you use the submit method
  - Submit method is overloaded that can take Runnable and Callable types
  - Submit method returns a Future that is a container for the returned value of the thread
  - If you wish to iterate through all future values in an efficient way with minimal blocking use the CompletionService class
Executors class
  - Utility class that has static methods to build different types of ExecutorServices
  - newFixedThreadPool()
  - newWorkStealingPool()
  - newFixedThreadPool()
  - newSingleThreadExecutor()
  - newCachedThreadPool()
  - newSingleThreadScheduledExecutor()
  - newScheduledThreadPool()
Future interface
  - These are containers that are returned from executors for the results of threads
  - Has a get method that will return the value from the thread - IT BLOCKS THE CURRENT THREAD IF THERE IS NO VALUE YET
CompletionService interface
  - Generic interface that will wrap an executor to enable a client to retrieve values from futures efficiently
  - Methods of interest
    - take() - pops the next future that contains a result - if theres none then it will wait/block
    - poll() - pops the next future that has a value - null if none
    - submit() - send a task to the event service
ExecutorCompletionService class
  - Implementation of the CompletionService interface
  - To create an instance of this class, you instantiate it and provide it with the ExecutorService pool
Terminating threads
  - You can terminate normal threads in 3 ways
    - program the task (runnable/callable) such that it checks a variable and you code it to break early
    - throw an exception
    - either within the runnable or outside the thread, call the Threads interrupt method




    PLEASE IGNORE THIS CHANGE

    sdffdfwfwfew
