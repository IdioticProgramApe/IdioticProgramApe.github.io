---
title: The Concurrency API
author: ipa
date: 2023-04-19
categories: [C-Cpp Language]
tags: [coding, modern cpp, cpp]
img_path: /moderncpp/
---

**The concurrency elements of the Standard Library:** tasks, futures/shared futures, threads, mutexes, condition variables, atomic objects, and more  

## Item 35: Prefer task-based programming to thread-based

### Thread

There are 3 meaning of "thread" in concurrent c++ software:

- **Hardware threads** are the threads that actually perform computation. Contemporary machine architectures offer one or more hardware threads per CPU core
- **Software threads** (also known as **OS threads** or **system threads**) are the threads that the operating system manages across all processes and schedules for execution on hardware threads, software threads are a **limited resource**  
- **std::threads** are objects in a C++ process that act as handles to underlying software threads. Some std::thread objects represent "null" handles, i.e., correspond to no software thread, because they:
  - are in a **default-constructed state** (hence have no function to execute);
  - have been **moved from** (the moved-to std::thread then acts as the handle to the underlying software thread);
  - have been **joined** (the function they were to run has finished);
  - have been **detached** (the connection between them and their underlying software thread has been severed)
- always need to concern about:
  - create too many std::thread, beyond the OS can provide
  - the oversubscription issue, the overhead when switch the context.
- term: **Thread-local storage (TLS)**

### Snippets

- thread-based approach and task-based approach:

  ```c++
  int doAsyncWork();
  
  std::thread t(doAsyncWork);  // thread based
  auto fut = std::async(doAsyncWork);  // task based
  ```

### Key points

- there are some situations where using threads directly may be appropriate:
  - you need **access to the API** of the underlying threading implementation. The C++ concurrency API is typically implemented using a lower-level platform-specific API, usually **pthreads** or **Windows' Threads**
  - you need to and are able to **optimize** thread usage for your application
  - You need to **implement threading technology beyond the C++ concurrency API**, e.g., thread pools on platforms where your C++ implementations don't offer them
- The std::thread API offers no direct way to get return values from asynchronously run functions, and if those functions throw, the program is terminated
- Thread-based programming calls for manual management of **thread exhaustion**, **oversubscription**, **load balancing**, and **adaptation to new platforms**
- Task-based programming via **std::async with the default launch policy** handles most of these issues for you

## Item 36: Specify std::launch::async if asynchronicity is essential

### Launch policy

- The `std::launch::async` launch policy means that f must be run asynchronously, i.e., on a different thread

- The `std::launch::deferred` launch policy means that f may run **only when get or wait is called on the future** returned by std::async. That is, f's execution is deferred until such a call is made. When get or wait is invoked, f will execute **synchronously**, i.e., **the caller will block until f finishes running**. If neither get nor wait is called, f will never run

- By default, if the launch policy is not specified, the following fut1 and fut2 have the same behavior, it permits f to be run either asynchronously or synchronously:

  ```c++
  auto fut1 = std::async(f);
  auto fut2 = std::async(std::launch::async | std::launch::deferred, f);
  ```

  Thus here are some implications for the default launch policy if the statement run on **thread t**:

  - **It's not possible to predict whether f will run concurrently with t**, because f might be scheduled to run deferred
  - **It's not possible to predict whether f runs on a thread different from the thread invoking get or wait on fut**. If that thread is t, the implication is that it's not possible to predict whether f runs on a thread different from t
  - **It may not be possible to predict whether f runs at all**, because it may not be possible to guarantee that get or wait will be called on fut along every path through the program

- Potential issues on **wait_for** and **wait_until**, check **future status** whether it is deferred first:

  ```c++
  using namespace std::literals; // for C++14 duration
  void f()
  {
      std::this_thread::sleep_for(1s);
  }
  
  auto fut = std::async(f);
  if (fut.wait_for(0s) == std::future_status::deferred)
  {
      ...
  } 
  else 
  {
      while (fut.wait_for(100ms) != std::future_status::ready) 
      {
          ...
      }
  }
  ```

- when to use default launch policy:

  - The task need not run concurrently with the thread calling **get** or **wait**.
  - It doesn't matter which thread's **thread_local** variables are read or written
  - Either there's a guarantee that **get** or **wait** will be called on the future returned by std::async or it's acceptable that the task may **never execute**
  - Code using **wait_for** or **wait_until** takes the possibility of deferred status into account
  - **otherwise**: use `std::launch::async`

### Snippets

- true async task-based function:

  ```c++
  // version c++11
  template<typename F, typename... Ts> inline
  std::future<typename std::result_of<F(Ts...)>::type> reallyAsync(F&& f, Ts&&... params)
  {
      return std::async(
          std::launch::async,
          std::forward<F>(f), 
          std::forward<Ts>(params)...
      );
  }
  ```

  ```c++
  // version c++14
  template<typename F, typename... Ts> inline
  auto reallyAsync(F&& f, Ts&&... params)
  {
      return std::async(
          std::launch::async,
          std::forward<F>(f),
          std::forward<Ts>(params)...
      );
  }
  ```

### Key points

- The **default launch policy** for std::async permits both **asynchronous** and **synchronous** task execution
- This flexibility leads to **uncertainty when accessing thread_locals**, implies that the task may never execute, and affects program logic for timeout-based wait calls
- Specify `std::launch::async` if asynchronous task execution is essential

## Item 37: Make std::threads unjoinable on all paths

### Joinability

- **Joinable thread**: corresponds to an underlying asynchronous thread of execution that **is or could be running**, A std::thread corresponding to an underlying thread that's **blocked** or **waiting to be scheduled** is joinable
- **Unjoinable thread**: 
  - **Default-constructed std::threads**. Such std::threads have no function to execute, hence don't correspond to an underlying thread of execution
  - **std::thread objects that have been moved from**. The result of a move is that the underlying thread of execution a std::thread used to correspond to (if any) now corresponds to a different std::thread
  - **std::threads that have been joined**. After a join, the std::thread object no longer corresponds to the underlying thread of execution that has finished running
  - **std::threads that have been detached**. A detach severs the connection between a std::thread object and the underlying thread of execution it corresponds to
- if the **destructor** for a **joinable thread** is invoked, execution of the program is terminated, there are 2 other arguably worse options:
  - **An implicit join**: make dtor wait for the execution to complete, could lead to performance anomalies that would be difficult to track down
  - **An implicit detach**: let dtor sever the connection between the thread and its execution, the execution will continue, it could modify the data and could be very hard to debug

- one solution is to use **RAII (Resource Acquisition Is Initialization)** objects to handle the std::threads.
- invoking **join** or **detach** on an unjoinable thread yields **undefined behavior**  

### Snippets

- a thread wrapper using RAII, about _interruptible thread_, see **Anthony Williams' C++ Concurrency in Action (Manning Publications, 2012),section 9.2**:

  ```c++
  class ThreadRAII 
  {
  public:
      enum class DtorAction { join, detach }; 
      
      // std::thread objects aren't copyable
      ThreadRAII(std::thread&& t, DtorAction a) 
          : action(a), t(std::move(t))
      {}
      
      ~ThreadRAII()
      {
          release();
      }
      
      // support the moving, otherwise, 
      // the move ctor/operator will not be generated by the compiler
      ThreadRAII(ThreadRAII&&) = default;
      ThreadRAII& operator=(ThreadRAII&&)
      {
          release();
          action = rhs.action;
          t = std::move(rhs.t);
          return *this;
      }
      
      std::thread& get() { return t; }
  
  private:
      DtorAction action;
      std::thread t;
      
      void release()
      {
          if(t.joinable())
          {
              if(action == DtorAction::join)
                  t.join();
              else
                  t.detach();
          }
      }
  };
  ```

- a use case for the ThreadRAII defined above:

  ```c++
  constexpr auto tenMillion = 10'000'000;    // digit separator c++14
  
  bool doWork(std::function<bool(int)> filter, int maxVal = tenMillion)
  {
      std::vector<int> goodVals;
      
      ThreadRAII t(std::thread([&filter, maxVal, &goodVals]
      {
          for (auto i = 0; i <= maxVal; ++i)
          {
              if (filter(i))
                  goodVals.push_back(i); 
          }
      }), ThreadRAII::DtorAction::join);
      
      auto nh = t.get().native_handle();
      // use native handle to set thread's priority
      
      if (conditionsAreSatisfied()) 
      {
          t.get().join();
          performComputation(goodVals);
          return true;
      }
      return false;
  }
  ```

### Key points

- Make std::threads unjoinable on all paths
- join-on-destruction can lead to difficult-to-debug **performance anomalies**
- detach-on-destruction can lead to difficult-to-debug **undefined behavior**
- Declare std::thread objects **last in lists of data members**, to prevent the potential dependency between different data members for their initialization.

## Item 38: Be aware of varying thread handle destructor behavior

### Flow of information

![flow of information](flow_of_information.png){: width="542" height="68" }
_Concurrency Communication_

The shared state determines the behavior of the future's destructor

- **The destructor for the last future referring to a shared state for a non-deferred task launched via std::async blocks** until the task completes. In essence, the destructor for such a future does an **implicit join** on the thread on which the asynchronously executing task is running
- **The destructor for all other futures simply destroys the future object**. For asynchronously running tasks, this is akin to an **implicit detach** on the underlying thread. For deferred tasks for which this is the final future, it means that the deferred task will never run

### Snippets

- create a shared state using `std::packaged_task` instead of `std::aync`:

  ```c++
  int calcValue();
  
  {
      // wrap calcValue so it can run asynchronously
      std::packaged_task<int()> pt(calcValue); 
      auto fut = pt.get_future();
      // package task is non copyable, need to move it
      std::thread t(std::move(pt));
      
      // further processing on the std::thread, either join or detach
  }
  ```

### Key points

- Future destructors normally just **destroy the future's data members**, and decrements the reference count stored in the shared state
- The **final future** referring to **a shared state** for a **non-deferred task** launched via `std::async` **blocks until the task completes**

## Item 39: Consider void futures for one-shot event communication

### Inter-thread communication

#### Condition variable/flag-based design

- use **condition variable**, however there are 2 issues need paying attention to:

  - If the detecting task **notifies the condvar before** the reacting task waits, the reacting task will **hang**

  - The wait statement fails to account for **spurious wakeups**, which means the false alarms, one solution is adding a test into the wait function:

    ```c++
    cv.wait(lock, []{ return whether the event has occurred; });
    ```

- use **shared boolean flag**, however it will make the reacting task "blocked" and yet still running which occupies a hardware thread that could be used to run some other tasks.

- use the combination of the underlying approches:

  - for the detecting task:

    ```c++
    std::condition_variable cv;
    std::mutex m;
    bool flag(false); // not std::atomic
    
    ... // detecting event 
    
    {
        std::lock_guard<std::mutex> g(m);
        flag = true; // tell reacting task
    }
    
    cv.notify_one();
    ```

  - for the reacting task:

    ```c++
    ... // prepare to react
    
    {
        std::unique_lock<std::mutex> lk(m);
        
        // use lambda to avoid spurious wakeups
        cv.wait(lk, [] { return flag; }); 
        
        ... // react to event (m is locked)
    }
    ... // continue reacting (m now unlocked)
    ```

#### std::promise/future design

The `std::promise` and `std::future`/`std::shared_future` pair can be used on any situation where there is a need for transmitting information from one end to another.

- no data is to be conveyed across the communications channel: 

  - the detecting task will use a `std::promise<void>`
  - the reacting task a `std::future<void>` or `std::shared_future<void>`

- The communications channel between a std::promise and a future is a **one-shot mechanism: it can't be used repeatedly**

  ```c++
  std::promise<void> p;
  
  void detect()
  {
      auto sf = p.get_future().share(); // std::shared_future<void>
      
      std::vector<std::thread> vt;
      for (int i = 0; i < threadsToRun; ++i) 
      {
          vt.emplace_back([sf]
          {
              sf.wait(); // wait on local
              react(); 
          });
      }
      
      ... // detect hangs if this "..." code throws!
      
      p.set_value(); // unsuspend all threads
      
      ...
      
      for (auto& t : vt) { t.join(); }
  }
  ```
  
- ref: [https://scottmeyers.blogspot.com/2013/12/threadraii-thread-suspension-trouble.html](https://scottmeyers.blogspot.com/2013/12/threadraii-thread-suspension-trouble.html)

### Key points

- For simple event communication, **condvar-based designs** require a superfluous mutex, impose constraints on the relative progress of detecting and reacting tasks, and require reacting tasks to verify that the event has taken place.
- Designs employing a **flag** avoid those problems, but are based on polling, not blocking
- **A condvar and flag** can be used together, but the resulting communications mechanism is somewhat stilted
- Using **std::promises and futures** dodges these issues, but the approach uses **heap memory** for shared states, and it's limited to **one-shot communication**

## Item 40: Use std::atomic for concurrency, volatile for special memory  

### Key points

- `std::atomic` is for data accessed from multiple threads without using mutexes. It's a tool for writing **concurrent** software
- `volatile` is for memory where reads and writes should not be optimized away. It's a tool for working with **special memory**

