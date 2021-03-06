Java Threads - a quick run though

Interfaces, classes and methods of interest

- Runnable interface
  - Used to create tasks for Threads
  - Implement this interface to provide the task
  - Runnable types do not return values

- Callable interface
  - Just like the Runnable interface, its used to define tasks
  - Callable is used if you want tasks to return values from its execution

- Thread class
  - Instances of this type are the threads that run tasks
  - Can be constructed with or without a runnable type
  - When provided with a runnable at construction, will run that task when started
  - Can extend this class to provide its own task
  - Can be set to daemon threads. these threads are best served as background processes
  - Normal threads are also known as user threads (as well as the main thread).
  - Daemon threads will abruptly die when all user threads die

- Executor interface
  - Interface that provides a single execute methods that other executors Implement

- ExecutorService interface
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

- Executors class
  - Utility class that has static methods to build different types of ExecutorServices
  - newFixedThreadPool()
  - newWorkStealingPool()
  - newFixedThreadPool()
  - newSingleThreadExecutor()
  - newCachedThreadPool()
  - newSingleThreadScheduledExecutor()
  - newScheduledThreadPool()

- Future interface
  - These are containers that are returned from executors for the results of threads
  - Has a get method that will return the value from the thread - IT BLOCKS THE CURRENT THREAD IF THERE IS NO VALUE YET

- CompletionService interface
  - Generic interface that will wrap an executor to enable a client to retrieve values from futures efficiently
  - Methods of interest
    - take() - pops the next future that contains a result - if theres none then it will wait/block
    - poll() - pops the next future that has a value - null if none
    - submit() - send a task to the event service

- ExecutorCompletionService class
  - Implementation of the CompletionService interface
  - To create an instance of this you provide it with the ExecutorService pool

- Stoping threads
  - There are 3 ways in which you can stop a thread
    - Program the task to check a variable and exit early when this variable holds true (have the variable togglable by providing a public method that modifies it)
    - throw an exception
    - Interrupt the thread by using the thread classes interrupt() method from either inside the task or outside. Doing it this way requires you to use the method isInterrupted() and program to exit.
      - NOTE! there is another method called interrupted() which is the same but it resets the value back to false after it returns true
    - Interrupts can happen when a task is running or blocked (in sleep), if thread is currently blocked, you usually have an opportunity within a try catch block to exit early, as its using in a catch interrupt block
  - Threads running within an executor can also be terminated. You do so by using the Future returned from the executorService and calling the cancel method on it
  - You can also use the shutdownNow() method on the executorService, this uses interrupts underneath and will interrupt any running thread and stop any queued up threads from starting

- Exceptions
  - Threads that leak exceptions terminate
  - Exceptions leaked from a thread cannot be caught in a in a try catch block from the break that creates it
  - You can catch a leaked thread in a number of ways
    - You can create a class that implements the UncaughtExceptionHandler interface and register an instance of it as the default Thread exception handler by using the static method Thread.setDefaultUncaughtExceptionHandler()
    - You can register different handlers for different threads, you do this by using the method setDefaultUncaughtExceptionHandler() BUT this time using it on the current thread instance rather than using the static method
    - You can use a combination of both default and thread specific handlers. This would use the thread specific one when possible but then the default when not set
  - Using the default exception handler will work for threads in both an executor pool and normal threads
  - To use different handlers for different threads in an executor, you have to use a ThreadFactory and give that to the executor during initialization. This thread factory creates the thread and returns it but before returning, will set the handler for that specific thread. Using this method will limit the same handler to all the threads in the same executor though (unless you write specific code that does something smarter)

- Waiting for threads
  - You can wait for threads in 3 ways
    - use wait and notify methods
    - use the Threads join method
      - calling this on a thread will cause the calling thread to wait for the completion of the thread object your calling on
    - for threads in pools, you can use the class CountDownLatch
      - you create an instance of it with the amount of calls required for the latch to unleash
      - Methods of interest
        - countDown() decrement the count
        - getCount()
        - await()
      - You then provide this instance to threads, each thread needs to call the countDown() method to decrement the counter
      - You block the current thread by calling the await() method, this will block until the count is zerp

```
public class JoinExample {
	public static void main(String[] args) throws InterruptedException {
		Thread thread = new Thread(new LongRunnable(5000L));
		thread.start();
		System.out.println("Main thread is now blocked when waiting for other thread to complete");
		thread.join();
		System.out.println("Main thread now running as thread is complete");
	}
}

class LongRunnable implements Runnable {

	private long workDuration;
	public LongRunnable(long workDuration) {
		this.workDuration = workDuration;
	}

	@Override
	public void run() {
		LocalTime now =  LocalTime.now().plus(workDuration, ChronoUnit.MILLIS);
		while(now.isAfter(LocalTime.now())){
			System.out.println("Working");
			try {
				Thread.sleep(800L);
			} catch (InterruptedException e) {
			}
		}
	}
}
```

```
public class LatchExample {
	public static void main(String[] args) throws InterruptedException {
		CountDownLatch latch = new CountDownLatch(2);
		Thread t1 = new Thread(new LongRunnable2(3000L, latch));
		Thread t2 = new Thread(new LongRunnable2(3000L, latch));
		t1.start();
		t2.start();
		System.out.println("Main thread going to wait on the latch");
		latch.await();
		System.out.println("Main thread now running");
	}
}

class LongRunnable2 implements Runnable {

	private long workDuration;
	private CountDownLatch latch;
	public LongRunnable2(long workDuration, CountDownLatch latch) {
		this.workDuration = workDuration;
		this.latch = latch;
	}

	@Override
	public void run() {
		LocalTime now =  LocalTime.now().plus(workDuration, ChronoUnit.MILLIS);
		String name = Thread.currentThread().getName();
		while(now.isAfter(LocalTime.now())){
			System.out.println("[" + name +"] Working");
			try {
				Thread.sleep(800L);
			} catch (InterruptedException e) {
			}
		}
		this.latch.countDown();
	}
}
```

- Scheduling Threads
  - The schedule threads to run in the future, you have to extend the abstract class TimerTask and provide the implementation of the task there
  - Timetasks should be short tasks as theres only one thread that executes the timer tasks in the the timer object, otherwise other tasks will be delayed
  - call cancel on the timer if your application is exiting, as any running task will be a user thread and will keep the application alive
  - Scheduling tasks on a cancelled timer will throw an illegal state exception
  - To schedule as TimerTask thread, you use the java.util.Timer methods to schedule a task to run once or multiple times with intervals.
    - Methods of interest in the Timer class
      - schedule(task, date)
      - schedule(task, delay)
      - schedule(task, delay, intervals)
      - scheduleAtFixedRate(...)
      - cancel()
    - Timers can be instantiated with a the contructor that takes a name for the thread and a boolean to say if the thread runs as a daemon or not
    - Its IMPORTANT TO NOTE that once all the tasks in a timer finish, the timer thread will shutdown gracefully. BUT! there is no gaurentee when that will happen, so it is VERY IMPORTANT to understand that running the thread as a user thread MAY keep the application alive when nothing is actually running
  - Executor pool threads can also be scheduled
  - To schedule these threads, you use 2 other they of pools created from the Executors static methods
    - singleThreadScheduledExecutor
    - scheduledThreadPool
  - These return an instance of an executor pool that implements the ScheduledExecutorService interface which extends the executorService interface
    - methods of interest
      - schedule(runnable, long, timeunit)
      - schedule(callable, long, timeunit)
      - scheduleFixedDelay x2
  - To schedule threads in an executor service, use the class Executors methods:
    - newSingleThreadScheduledExecutor
    - newScheduledThreadPool
    - unconfigurableScheduledExecutorService
  - This will return a ScheduleExecutorService which you can call methods to return values
    - schedule
    - scheduleAtFixedRate
  - You do not need to extend the TimerTask type to create a task, just use a runnable/Callable
- Fixed Rate methods start the subsequent scheduled tasks relative to the start time of the first task and NOT the end time
