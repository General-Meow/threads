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
    - awaitTermination(int, Unit) //block current thread for defined time or until all tasks in executor have stopped
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
  - To create an instance of this you provide it with the ExecutorService pool
Stoping threads
  - There are 3 ways in which you can stop a thread
    - Program the task to check a variable and exit early when this variable holds true (have the variable togglable by providing a public method that modifies it)
    - throw an exception
    - Interrupt the thread by using the thread classes interrupt() method from either inside the task or outside. Doing it this way requires you to use the method isInterrupted() and program to exit.
      - NOTE! there is another method called interrupted() which is the same but it resets the value back to false after it returns true
    - Interrupts can happen when a task is running or blocked (in sleep), if thread is currently blocked, you usually have an opportunity within a try catch block to exit early, as its using in a catch interrupt block
  - Threads running within an executor can also be terminated. You do so by using the Future returned from the executorService and calling the cancel method on it
  - You can also use the shutdownNow() method on the executorService, this uses interrupts underneath and will interrupt any running thread and stop any queued up threads from starting
Exceptions
  - Threads that leak exceptions terminate
  - Exceptions leaked from a thread cannot be caught in a in a try catch block from the break that creates it
  - You can catch a leaked thread in a number of ways
    - You can create a class that implements the UncaughtExceptionHandler interface and register an instance of it as the default Thread exception handler by using the static method Thread.setDefaultUncaughtExceptionHandler()
    - You can register different handlers for different threads, you do this by using the method setDefaultUncaughtExceptionHandler() BUT this time using it on the current thread instance rather than using the static method
    - You can use a combination of both default and thread specific handlers. This would use the thread specific one when possible but then the default when not set
  - Using the default exception handler will work for threads in both an executor pool and normal threads
  - To use different handlers for different threads in an executor, you have to use a ThreadFactory and give that to the executor during initialization. This thread factory creates the thread and returns it but before returning, will set the handler for that specific thread.
