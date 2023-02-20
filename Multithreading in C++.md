## Multithreading in C++

A process is a program that is running on the computer. In modern computers, many processes run at the same time. A program can be broken down into sub-processes for the sub-processes to run at the same time. These sub-processes are called threads. Threads must run as parts of one program.
Some programs require more than one input simultaneously. Such a program needs threads. If threads run in parallel, then the overall speed of the program is increased. Threads also share data among themselves. This data sharing leads to conflicts on which result is valid and when the result is valid. This conflict is a data race and can be resolved.

Since threads have similarities to processes, a program of threads is compiled by the g++ compiler as follows:
```cpp
g++ -std=c++17 main.cpp -lpthread -o main
```
Where <font color="red">main.cpp</font> is the source code file, and the <font color="red">main</font> is the executable file and compiled under standard <span style="color:green;font-size:12px">(C++17)</span>.

A program that uses threads, is begun as follows:
```cpp
#include <iostream>
#include <thread>
using namespace std;
```
Note the use of <font color="red">#include \<thread></font>.

This article explains Multi-thread and Data Race Basics in C++. The reader should have basic knowledge of C++, it’s Object-Oriented Programming, and its lambda function; to appreciate the rest of this article.

#### Table of Contents
- [Thread Library](#1)
- [Thread Object Members](#2)
- [Thread Returning a Value](#3)
- [Communication Between Threads](#4)
- [The thread_local Specifier](#5)
- [Sequences, Synchronous, Asynchronous, Parallel, Concurrent, Order](#6)
- [Blocking a Thread](#7)
- [Locking](#8)
- [Mutex Library](#9)
- [Timeout in C++](#10)
- [Lockable Requirements](#11)
- [Mutex Types](#12)
- [Data Race](#13)
- [Locks](#14)
- [Call Once](#15)
- [Condition Variable Library](#16)
- [Future Library](#17)
- [Conclusion](#18)

<div id='1'/>

#### Thread Library
The flow of control of a program can be single or multiple. When it is single, it is a thread of execution or simply, thread. A simple program is one thread. This thread has the main() function as its top-level function. This thread can be called the main thread. In simple terms, a thread is a top-level function, with possible calls to other functions.
Any function defined in the global scope is a top-level function. A program has the main() function and can have other top-level functions. Each of these top-level functions can be made into a thread by encapsulating it into a thread object. A thread object is a code that turns a function into a thread and manages the thread. A thread object is instantiated from the thread class.
So, to create a thread, a top-level function should already exist. This function is the effective thread. Then a thread object is instantiated. The ID of the thread object without the encapsulated function is different from the ID of the thread object with the encapsulated function. The ID is also an instantiated object, though its string value can be obtained.
If a second thread is needed beyond the main thread, a top-level function should be defined. If a third thread is needed, another top-level function should be defined for that, and so on.

##### Creating a Thread
The main thread is already there, and it does not have to be recreated. To create another thread, its top-level function should already exist. If the top-level function does not already exist, it should be defined. A thread object is then instantiated, with or without the function. The function is the effective thread (or the effective thread of execution). The following code creates a thread object with a thrdFn (with a function):
```cpp
#include <iostream>
#include <thread>
using namespace std;

void thrdFn() {
    cout << "seen" << '\n';
}  

int main(int argc, char const *argv[])
{
    thread thr(&thrdFn);

    return 0;
}
```
The name of the thread is <font color="red">thr</font>, instantiated from the thread class, thread. Remember: to compile and run a thread, use a command similar to the one given above.

The constructor function of the thread class takes a reference to the function as an argument.

This program now has two threads: the main thread and the thr object thread. The output of this program should be: <font color="red">seen</font> (printing from the thread function). This program as it is has no syntax error; it is well-typed. This program, as it is, compiles successfully. However, if this program is run, the thread (function, thrdFn) may not display any output; an error message might be displayed. This is because the thread, <font color="red">thrdFn()</font> and the <font color="red">main()</font> thread, have not been made to work together. In C++, all threads should be made to work together, using the <font color="red">join()</font> method of the thread – see below.

<div id='2'/>

#### Thread Object Members
The most important members functions of the thread class are:
| Members                       | Description                       |
| -----------                 | -----------                       |
|join|Waits for the thread to finish its execution <span style="color:green;font-size:12px">(public member function)</span>|
|detach|Permits the thread to execute independently from the thread handle <span style="color:green;font-size:12px">(public member function)</span>|
|get_id|Returns the id of the thread <span style="color:green;font-size:12px">(public member function)</span>|
|joinable|Checks whether the thread is joinable, i.e. potentially running in parallel context <span style="color:green;font-size:12px">(public member function)</span>|

##### void join()
If the above program did not produce any output, the two threads were not forced to work together. In the following program, an output is produced because the two threads have been forced to work together:
```cpp
#include <iostream>
#include <thread>
using namespace std;

void thrdFn() {
    cout << "seen" << '\n';
}  

int main(int argc, char const *argv[])
{
    thread thr(&thrdFn);

    thr.join();

    return 0;
}
```
Now, there is an output: 
```cpp
seen
```
without any run-time error message. As soon as a thread object is created, with the encapsulation of the function, the thread starts running; i.e., the function starts executing. The join() statement of the new thread object in the main() thread tells the main thread (main() function) to wait until the new thread (function) has completed its execution (running). The main thread will halt and will not execute its statements below the join() statement until the second thread has finished running. The result of the second thread is correct after the second thread has completed its execution.

If a thread is not joined, it continues to run independently and may even end after the main() thread has ended. In that case, the thread is not really of any use.

The following program illustrates the coding of a thread whose function receives arguments:
```cpp
#include <iostream>
#include <thread>
using namespace std;

void thrdFn(char str1[], char str2[]) {
    cout << str1 << str2 << '\n';
}  

int main(int argc, char const *argv[])
{
    char st1[] = "I have ";
    char st2[] = "seen it.";

    thread thr(&thrdFn, st1, st2);
    thr.join();

    return 0;
}
```
The output is:
```cpp
I have seen it.
```
##### Returning from a Thread
The effective thread is a function that runs concurrently with the main() function. The return value of the thread (encapsulated function) is not done ordinarily. "How to return value from a thread in C++" is explained below.

Note: It is not only the main() function that can call another thread. A second thread can also call the third thread.
##### void detach()
After a thread has been joined, it can be detached. Detaching means separating the thread from the thread (main) it was attached to. When a thread is detached from its calling thread, the calling thread no longer waits for it to complete its execution. The thread continues to run on its own and may even end after the calling thread (main) has ended. In that case, the thread is not really of any use. A calling thread should join a called thread for both of them to be of use. Note that joining halts the calling thread from executing until the called thread has completed its own execution. The following program shows how to detach a thread:
```cpp
#include <iostream>
#include <thread>
using namespace std;

void thrdFn(char str1[], char str2[]) {
    cout << str1 << str2 << '\n';
}  

int main(int argc, char const *argv[])
{
    char st1[] = "I have ";
    char st2[] = "seen it.";

    thread thr(&thrdFn, st1, st2);
    thr.join();
    thr.detach();

    return 0;
}
```
Note the statement, <font color="red">thr.detach();</font>. This program, as it is, will compile very well. However, when running the program, an error message may be issued. When the thread is detached, it is on its own and may complete its execution after the calling thread has completed its execution.
##### id get_id()
id is a class in the thread class. The member function, get_id(), returns an object, which is the ID object of the executing thread. The text for the ID can still be gotten from the id object – see later. The following code shows how to obtain the id object of the executing thread:
```cpp
#include <iostream>
#include <thread>

void PrintID(const std::string& str) {
    std::cout << str << " has the ID: " << std::this_thread::get_id() << '\n';
}

void thrdFn(const std::string& str) {
    PrintID(str);
    std::thread t2(PrintID,"t2");
    t2.join();
}

int main(int argc, char const *argv[])
{
    std::thread t1(thrdFn,"t1");
    t1.join();

    return 0;
}
```
The output is:
```cpp
t1 has the ID: 15404
t2 has the ID: 29128
```
<div id='3'/>

#### Thread Returning a Value
The effective thread is a function. A function can return a value. So a thread should be able to return a value. However, as a rule, the thread in C++ does not return a value. This can be worked around using the C++ class, <font color="red">\<future></font> in the standard library, and the C++ <font color="red">std::async()</font> function in the Future library. A top-level function for the thread is still used but without the direct thread object. The following code illustrates this:
```cpp
#include <iostream>
#include <thread>
#include <future>

char* thrdFn(char* str) {
    return str;
}  

int main(int argc, char const *argv[]) {
    char st[] = "I have seen it.";
    
    std::future<char*> output;
    
    output = std::async(thrdFn, st);
    
    char* ret = output.get();   //waits for thrdFn() to provide result
    
    std::cout << ret <<'\n';

    return 0;
}
```
The output is:
```cpp
I have seen it.
```
Note the inclusion of the future library for the future class. The program begins with the instantiation of the future class for the object, output, of specialization . The async() function is a C++ function in the std namespace in the future library. The first argument to the function is the name of the function that would have been a thread function. The rest of the arguments for the async() function are arguments for the supposed thread function.
The calling function (main thread) waits for the executing function in the above code until it provides the result. It does this with the statement:
```cpp
char* ret = output.get();
```
This statement uses the get() member function of the future object. The expression <font color="red">output.get()</font> halts the execution of the calling function (main() thread) until the supposed thread function completes its execution. If this statement is absent, the main() function may return before async() finishes the execution of the supposed thread function. The get() member function of the future returns the returned value of the supposed thread function. In this way, a thread has indirectly returned a value. There is no join() statement in the program.
<div id='4'/>

#### Communication Between Threads
The simplest way for threads to communicate is to be accessing the same global variables, which are the different arguments to their different thread functions. The following program illustrates this. The main thread of the main() function is assumed to be thread-0. It is thread-1, and there is thread-2. Thread-0 calls thread-1 and joins it. Thread-1 calls thread-2 and joins it.
```cpp
#include <iostream>
#include <thread>
#include <string>

std::string global1 = "I have ";
std::string global2 = "seen it.";

void thrdFn2(std::string str2) {
        std::string loc = global1 + str2;
        std::cout << loc << std::endl;
    }

void thrdFn1(std::string str1) {
        global1 = "Yes, " + str1;

        std::thread thr2(&thrdFn2, global2);  
        thr2.join();
    }  

int main(int argc, char const *argv[])
{
    std::thread thr1(&thrdFn1, global1);  
    thr1.join();

    return 0;
}
```
The output is:
```cpp
Yes, I have seen it.
```
Note that the string class has been used this time, instead of the array-of-characters, for convenience. Note that thrdFn2() has been defined before thrdFn1() in the overall code; otherwise thrdFn2() would not be seen in thrdFn1(). Thread-1 modified global1 before Thread-2 used it. That is communication.

More communication can be got with the use of condition_variable or Future – see below.
<div id='5'/>

#### The thread_local Specifier
A global variable must not necessarily be passed to a thread as an argument of the thread. Any thread body can see a global variable. However, it is possible to make a global variable have different instances in different threads. In this way, each thread can modify the original value of the global variable to its own different value. This is done with the use of the thread_local specifier as in the following program:
```cpp
#include <iostream>
#include <thread>

thread_local int inte = 0;      //keyword introduced since C++11

void thrdFn2() {
    inte = inte + 2;
    std::cout << inte << " of 2nd thread\n";
}

void thrdFn1() {
    std::thread thr2(&thrdFn2);
    inte = inte + 1;
    std::cout << inte << " of 1st thread\n";

    thr2.join();
}  

int main(int argc, char const *argv[])
{
    std::thread thr1(&thrdFn1);  
    std::cout << inte << " of 0th thread\n";
    thr1.join();

    return 0;
}
```
The output is:
```cpp
0, of 0th thread
1, of 1st thread
2, of 2nd thread
```
<div id='6'/>

#### Sequences, Synchronous, Asynchronous, Parallel, Concurrent, Order
##### Atomic Operations
Atomic operations are like unit operations. Three important atomic operations are store(), load() and the read-modify-write operation. The store() operation can store an integer value, for example, into the microprocessor accumulator (a kind of memory location in the microprocessor). The load() operation can read an integer value, for example, from the accumulator, into the program.
##### Sequences
An atomic operation consists of one or more actions. These actions are sequences. A bigger operation can be made up of more than one atomic operation (more sequences). The verb `sequence ` can mean whether an operation is placed before another operation.
##### Synchronous
Operations operating one after the other, consistently in one thread, are said to operate synchronously. Suppose two or more threads are operating concurrently without interfering with one another, and no thread has an asynchronous callback function scheme. In that case, the threads are said to be operating synchronously.
If one operation operates on an object and ends as expected, then another operation operates on that same object; the two operations will be said to have operated synchronously, as neither interfered with the other on the use of the object.
##### Asynchronous
Assume that there are three operations, called operation1, operation2, and operation3, in one thread. Assume that the expected order of working is: operation1, operation2, and operation3. If working takes place as expected, that is a synchronous operation. However, if, for some special reason, the operation goes as operation1, operation3, and operation2, then it would now be asynchronous. 
Asynchronous behavior is when the order is not the normal flow.

Also, if two threads are operating, and along the way, one has to wait for the other to complete before it continues to its own completion, then that is asynchronous behavior.

##### Parallel
Assume that there are two threads. Assume that if they are to run one after the other, they will take two minutes, one minute per thread. With parallel execution, the two threads will run simultaneously, and the total execution time would be one minute. This needs a dual-core microprocessor. With three threads, a three-core microprocessor would be needed, and so on.
If asynchronous code segments operate in parallel with synchronous code segments, there would be an increase in speed for the whole program. Note: the asynchronous segments can still be coded as different threads.
##### Concurrent
With concurrent execution, the above two threads will still run separately. However, this time they will take two minutes (for the same processor speed, everything equal). There is a single-core microprocessor here. There will be interleaved between the threads. A segment of the first thread will run, then a segment of the second thread runs, then a segment of the first thread runs, then a segment of the second, and so on.

In practice, in many situations, parallel execution does some interleaving for the threads to communicate.

##### Order
For the actions of an atomic operation to be successful, there must be an order for the actions to achieve synchronous operation. For a set of operations to work successfully, there must be an order for the operations for synchronous execution.
<div id='7'/>

#### Blocking a Thread
By employing the join() function, the calling thread waits for the called thread to complete its execution before it continues its own execution. That wait is blocking.
<div id='8'/>

#### Locking
A code segment (critical section) of a thread of execution can be locked just before it starts and unlocked after it ends. When that segment is locked, only that segment can use the computer resources it needs; no other running thread can use those resources. An example of such a resource is the memory location of a global variable. Different threads can access a global variable. Locking allows only one thread, a segment of it, that has been locked to access the variable when that segment is running.
<div id='9'/>

#### Mutex Library

Defined as <font color="red">\<mutex></font> library owns mutual exclusion algorithms prevent multiple threads from simultaneously accessing shared resources. This prevents data races and provides support for synchronization between threads.

| Types                       | Description                       |
| -----------                 | -----------                       |
|mutex <span style="color:green;font-size:12px">(C++11)</span>                |Provides basic mutual exclusion facility <span style="color:green;font-size:12px">(class)</span>|
|shared_mutex <span style="color:green;font-size:12px">(C++17)</span>         |Provides shared mutual exclusion facility <span style="color:green;font-size:12px">(class)</span>|
|timed_mutex <span style="color:green;font-size:12px">(C++11)</span>          |Provides mutual exclusion facility which implements locking with a timeout <span style="color:green;font-size:12px">(class)</span>|
|shared_timed_mutex <span style="color:green;font-size:12px">(C++14)</span>   |Provides shared mutual exclusion facility and implements locking with a timeout <span style="color:green;font-size:12px">(class)</span>|
|recursive_mutex <span style="color:green;font-size:12px">(C++11)</span>      |Provides mutual exclusion facility which can be locked recursively by the same thread <span style="color:green;font-size:12px">(class)</span>|
|recursive_timed_mutex <span style="color:green;font-size:12px">(C++11)|Provides mutual exclusion facility which can be locked recursively by the same thread and implements locking with a timeout <span style="color:green;font-size:12px">(class)</span>|

Generic mutex management.

|Locks                                                         | Description                     |
| --------                                                         | -----------                     |
|lock_guard <span style="color:green;font-size:12px">(C++11)</span>|Implements a strictly scope-based mutex ownership wrapper <span style="color:green;font-size:12px">(class template)</span>|
|unique_lock <span style="color:green;font-size:12px">(C++11)</span>|Implements movable mutex ownership wrapper <span style="color:green;font-size:12px">(class template)</span>|
|shared_lock <span style="color:green;font-size:12px">(C++11)</span>|Implements movable shared mutex ownership wrapper <span style="color:green;font-size:12px">(class template)</span>|
|scoped_lock <span style="color:green;font-size:12px">(C++17)</span>|Deadlock-avoiding RAII wrapper for multiple mutexes <span style="color:green;font-size:12px">(class template)</span> |

Tag constants used to specify locking strategy.

|Tag arguments                                                     | Description                     |
| --------                                                         | -----------                     |
|defer_lock <span style="color:green;font-size:12px">(C++11)</span>|Do not acquire ownership of the mutex <span style="color:green;font-size:12px">defer_lock_t</span>|
|try_to_lock <span style="color:green;font-size:12px">(C++11)</span>|Try to acquire ownership of the mutex without blocking <span style="color:green;font-size:12px">try_to_lock_t</span>|
|adopt_lock <span style="color:green;font-size:12px">(C++11)</span>|Assume the calling thread already has ownership of the mutex <span style="color:green;font-size:12px">adopt_lock_t</span>|

Generic locking algorithms for multiple mutexes.

|Functions   | Description                     |
| --------   |  --------                       |
|try_lock <span style="color:green;font-size:12px">(C++11)</span>|Attempts to obtain ownership of mutexes via repeated calls to try_lock <span style="color:green;font-size:12px">(function template)</span>|
|lock <span style="color:green;font-size:12px">(C++11)</span>|Locks specified mutexes, blocks if any are unavailable <span style="color:green;font-size:12px">(function template)</span>|

Call once.
|Functions   | Description                     |
| --------   |  --------                       |
|once_flag <span style="color:green;font-size:12px">(C++11)</span>|Helper object to ensure that call_once invokes the function only once <span style="color:green;font-size:12px">(class)</span>|
|call_once <span style="color:green;font-size:12px">(C++11)</span>|Invokes a function only once even if called from multiple threads <span style="color:green;font-size:12px">(function template)</span>|

<div id='10'/>

#### Timeout in C++
An action can be made to occur after a duration or at a particular point in time. To achieve this, `Chrono` has to be included, with the directive, <font color="red">#include \<chrono></font>.

##### duration
duration is the class-name for duration, in the namespace chrono, which is in namespace std. Duration objects can be created as follows:
```cpp
chrono::hours hrs(2);
chrono::minutes mins(2);
chrono::seconds secs(2);
chrono::milliseconds msecs(2);
chrono::microseconds micsecs(2);
```
Here, there are 2 hours with the name, hrs; 2 minutes with the name, mins; 2 seconds with the name, secs; 2 milliseconds with the name, msecs; and 2 microseconds with the name, micsecs.

1 millisecond = 1/1000 seconds. 1 microsecond = 1/1000000 seconds.
##### time_point
The default time_point in C++ is the time point after the UNIX epoch. The UNIX epoch is 1st January 1970. The following code creates a time_point object, which is 100 hours after the UNIX-epoch.
```cpp
chrono::hours hrs(100);
chrono::time_point tp(hrs);
```
Here, tp is an instantiated object.
<div id='11'/>

#### Lockable Requirements

the most popular are:

| Types                       | Description                       |
| -----------                 | -----------                       |
|std::mutex <span style="color:green;font-size:12px">(C++11)</span>                |Provides basic mutual exclusion facility <span style="color:green;font-size:12px">(class)</span>|
|std::timed_mutex <span style="color:green;font-size:12px">(C++11)</span>          |Provides mutual exclusion facility which implements locking with a timeout <span style="color:green;font-size:12px">(class)</span>|

##### BasicLockable Requirements (std::mutex)
Represented as <font color="red">std::mutex</font> defined as class in header <font color="red">\<mutex></font> (since C++11)
To see this topic, we will start from an instantiated object of the mutex class.
```cpp
std::mutex m;
```
The class mutex has the following member functions:
| Members                       | Description                       |
| -----------                 | -----------                       |
|lock     |Locks the mutex, blocks if the mutex is not available <span style="color:green;font-size:12px">(public member function)</span>         |
|try_lock |Tries to lock the mutex, returns if the mutex is not available <span style="color:green;font-size:12px">(public member function)</span>|
|unlock   |Unlocks the mutex <span style="color:green;font-size:12px">(public member function)</span>                                       |

Do not confuse public member function of <font color="red">mutex class</font> with functions of the <font color="red">\<mutex></font> library. 

Member functions of mutex class:
```cpp
std::mutex m;

m.lock();
m.unlock();
m.try_lock();
```
Functions of the \<mutex> library are used for multiple mutexes:

```cpp
std::mutex m1,m2,m3 ...;

std::lock(m1, m2, m3 ...);
std::try_lock(m1, m2, m3 ...)
```
##### m.lock()
Locks the mutex. If another thread has already locked the mutex, a call to lock will block execution until the lock is acquired.
If lock is called by a thread that already owns the mutex, the behavior is undefined: for example, the program may deadlock. An implementation that can detect the invalid usage is encouraged to throw a std::system_error with error condition resource_deadlock_would_occur instead of deadlocking.

##### m.unlock()
Unlocks the mutex (done from tha above lock() explanation), releasing ownership over it.The mutex must be locked by the current thread of execution, otherwise, the behavior is undefined. An example:
```cpp
#include <iostream>     //std::cout std::endl
#include <thread>       //std::thread
#include <mutex>        //std::mutex  lock()  unlock()

int globalVar = 5;
std::mutex m;

void thrdFn() {
    //some statements
    m.lock();
        globalVar += 2;
        std::cout << globalVar << '\n';
    m.unlock();
}

int main(int argc, char const *argv[])
{
    std::thread thr(&thrdFn);
    thr.join();

    return 0;
}
```
The output is:
```cpp
7
```
There are two threads here: the main_thread or main() body and the child_thread thr running thrdFn() function. Note that the mutex library has been included. The expression to instantiate the mutex is <font color="red">std::mutex m;</font>. Because of the use of <font color="red">lock()</font> and <font color="red">unlock()</font>, the code segment,
```cpp
globl = globl + 2;
cout << globl << endl;
```
Which must not necessarily be indented, is the only code that has access to the memory location (resource), identified by globl, and the computer screen (resource) represented by cout, at the time of execution.
##### m.try_lock()
This is the same as m.lock() but does not block the current execution agent. Returns immediately. On successful lock acquisition returns true, otherwise returns false.
This function is allowed to fail spuriously and return false even if the mutex is not currently locked by any other thread.
If try_lock is called by a thread that already owns the mutex, the behavior is undefined.
Example:
```cpp
#include <iostream>     //std::cout std::endl;
#include <thread>       //std::thread
#include <mutex>        //std::mutex

std::mutex m;

void thrdFn(const std::string& str) {
    if (m.try_lock()){
        std::cout << str << " locked the mutex" << "\n";
        m.unlock();
    }
    else{
        std::cout << str << " failed to lock the mutex" << "\n";
    }
}  

int main(int argc, char const *argv[])
{
    std::thread t1(thrdFn,"t1");
    std::thread t2(thrdFn,"t2");

    t1.join();
    t2.join();

    return 0;
}
```
The output is:
```cpp
t1 locked the mutex
t2 failed to lock the mutex
```
##### TimedLockable Requirements (std::timed_mutex)

Represented as <font color="red">std::timed_mutex</font> defined as class in header <font color="red">\<mutex></font> (since C++11)
The class timed_mutex has the following member functions:
| Members                       | Description                       |
| -----------                 | -----------                       |
|lock     |Locks the mutex, blocks if the mutex is not available <span style="color:green;font-size:12px">(public member function)</span>         |
|try_lock |Tries to lock the mutex, returns if the mutex is not available <span style="color:green;font-size:12px">(public member function)</span>|
|try_lock_for|Tries to lock the mutex, returns if the mutex has been unavailable for the specified relative time duration <span style="color:green;font-size:12px">(public member function)</span>         |
|try_lock_until|Tries to lock the mutex, returns if the mutex has been unavailable for the specified absolute time point has been reached <span style="color:green;font-size:12px">(public member function)</span>         |
|unlock   |Unlocks timed mutex <span style="color:green;font-size:12px">(public member function)</span>    

Member functions in timed_mutex class:
```cpp
std::timed_mutex m;

m.lock();
m.unlock();
m.try_lock();
m.try_lock_for(rel_time);
m.try_lock_until(abs_time);
```

##### m.try_lock_for(rel_time)
Attempts to lock the `std::timed_mutex` object, blocking for rel_time at most or by timed_mutex object unlock() member: 
If the timed_mutex isn't currently locked by any thread, the calling thread locks it.
If the timed_mutex is currently locked by another thread, execution of the calling thread is blocked until unlocked or once rel_time has elapsed, whichever happens first.
If the timed_mutex is currently locked by the same thread calling this function, it produces a deadlock (with undefined behavior). Returns true if the function succeeds in locking the timed_mutex for the thread.
false otherwise. 
Taking as parameter rel_time:The maximum time span during which the thread will block, that represents a specific relative time. Example:
```cpp
#include <iostream>       // std::cout
#include <chrono>         // std::chrono::milliseconds
#include <thread>         // std::thread
#include <mutex>          // std::timed_mutex

std::timed_mutex m;

void thrdFn(int i){
    // waiting to get a lock: each thread prints "-" every 200ms:
    while (!m.try_lock_for(std::chrono::milliseconds(200))) {
        std::cout << "-";
    }
    // got a lock! - wait for 1s, then this thread prints its own index
    std::this_thread::sleep_for(std::chrono::milliseconds(1000));
    std::cout << i << "\n";
    m.unlock();
}

int main(int argc, char const *argv[])
{
    std::thread threads[10];
    // spawn 10 threads:
    for (int i=0; i<10; ++i)
        threads[i] = std::thread(thrdFn,i);

    for (auto& th : threads) th.join();
    return 0;
}
```
The output is: 
```cpp
------------------------------------0
----------------------------------------4
-----------------------------------5
------------------------------3
-------------------------6
----------------8
---------------2
----------7
-----1
9
```
mutex is a library with a class, mutex. This library has another class, called timed_mutex. The mutex object, m here, is of timed_mutex type. Note that the thread, mutex, and Chrono libraries have been included in the program.
##### m.try_lock_until(abs_time)
Attempts to lock the `std::timed_mutex` object, blocking until abs_time at most or by timed_mutex object unlock() member: 
If the timed_mutex isn't currently locked by any thread, the calling thread locks it. 
If the timed_mutex is currently locked by another thread, execution of the calling thread is blocked until unlocked or until abs_time, whichever happens first.
If the timed_mutex is currently locked by the same thread calling this function, it produces a deadlock (with undefined behavior). Returns true if the function succeeds in locking the timed_mutex for the thread.
false otherwise. 
Taking as parameter abs_time: A point in time at which the thread will stop blocking, that represents a specific absolute time. Example:
```cpp
#include <iostream>       // std::cout
#include <chrono>         // std::chrono::milliseconds
#include <thread>         // std::thread
#include <mutex>          // std::timed_mutex

std::timed_mutex m;

void thrdFn(int i){
    // waiting to get a lock: each thread prints "-" every 200ms:
    auto now = std::chrono::steady_clock::now();
    while (!m.try_lock_until(now + std::chrono::milliseconds(200))) {
        std::cout << "-";
    }
    // got a lock! - wait for 1s, then this thread prints its own index
    std::this_thread::sleep_for(std::chrono::milliseconds(1000));
    std::cout << i << "\n";
    m.unlock();
}

int main(int argc, char const *argv[])
{
    std::thread threads[10];
    // spawn 10 threads:
    for (int i=0; i<10; ++i)
        threads[i] = std::thread(thrdFn,i);

    for (auto& th : threads) th.join();
    return 0;
}
```
The output is: 
```cpp
------------------------------------0
----------------------------------------4
-----------------------------------5
------------------------------3
-------------------------6
----------------8
---------------2
----------7
-----1
9
```
If the time-point is in the past, the locking should take place now.
Note that the argument for m.try_lock_for() is duration and the argument for m.try_lock_until() is time point. Both of these arguments are instantiated classes (objects).
<div id='12'/>

#### Mutex Types

Defined in header <font color="red">\<mutex></font>, it has different type of classes as we saw above: 

| Types                       | Description                       |
| -----------                 | -----------                       |
|mutex <span style="color:green;font-size:12px">(C++11)</span>                |provides basic mutual exclusion facility <span style="color:green;font-size:12px">(class)</span>|
|shared_mutex <span style="color:green;font-size:12px">(C++17)</span>         | provides shared mutual exclusion facility <span style="color:green;font-size:12px">(class)</span>|
|timed_mutex <span style="color:green;font-size:12px">(C++11)</span>          |provides mutual exclusion facility which implements locking with a timeout <span style="color:green;font-size:12px">(class)</span>|
|shared_timed_mutex <span style="color:green;font-size:12px">(C++14)</span>   | provides shared mutual exclusion facility and implements locking with a timeout <span style="color:green;font-size:12px">(class)</span>|
|recursive_mutex <span style="color:green;font-size:12px">(C++11)</span>      |provides mutual exclusion facility which can be locked recursively by the same thread <span style="color:green;font-size:12px">(class)</span>|
|recursive_timed_mutex <span style="color:green;font-size:12px">(C++11)|provides mutual exclusion facility which can be locked recursively by the same thread and implements locking with a timeout <span style="color:green;font-size:12px">(class)</span>|

The `recursive_mutex` and `recursive_timed_mutex` will not be covered in this article.

Note: a thread owns a mutex from the time the call to lock is made until unlock.

##### mutex
|Functions| Description                                                                           |
| --------| -----------                                                                           |
|try_lock |Tries to lock the mutex, returns if the mutex is not available <span style="color:green;font-size:12px">(public member function)</span>|
|lock     |Locks the mutex, blocks if the mutex is not available <span style="color:green;font-size:12px">(public member function)</span>         |
|unlock   |Unlock multiple mutexes <span style="color:green;font-size:12px">(public member function)</span>                                       |
##### shared_mutex
With shared mutex, more than one thread can share access to the computer resources. So, by the time the threads with shared mutexes have completed their execution, while they were at lock-down, they were all manipulating the same set of resources (all accessing the value of a global variable, for example).

|Functions       | Description                                                                                                |
| --------       | -----------                                                                                                |
|try_lock_shared |Tries to lock the mutex for shared ownership, returns if the mutex is not available <span style="color:green;font-size:12px">(public member function)</span>|
|lock_shared     |Locks the mutex for shared ownership, blocks if the mutex is not available <span style="color:green;font-size:12px">(public member function)</span>         |
|unlock_shared   |Unlock the mutex (shared ownership) <span style="color:green;font-size:12px">(public member function)</span>                                                |

##### timed_mutex

|Functions       | Description                                                                                                |
| --------       | -----------                                                                                                |
|lock|Locks the mutex, blocks if the mutex is not available <span style="color:green;font-size:12px">(public member function)</span>
|try_lock|Tries to lock the mutex, returns if the mutex is not available <span style="color:green;font-size:12px">(public member function)</span>| 
|try_lock_for|Tries to lock the mutex, returns if the mutex has been unavailable for the specified timeout duration <span style="color:green;font-size:12px">(public member function)</span>|
|try_lock_until|Tries to lock the mutex, returns if the mutex has been unavailable until specified time point has been reached <span style="color:green;font-size:12px">(public member function)</span>|
|unlock   |Unlock multiple mutexes <span style="color:green;font-size:12px">(public member function)</span>                                       |

##### shared_timed_mutex
With shared_timed_mutex, more than one thread can share access to the computer resources, depending on time (duration or time_point). So, by the time the threads with shared timed mutexes have completed their execution, while they were at lock-down, they were all manipulating the resources (all accessing the value of a global variable, for example).

|Functions       | Description                                                                                                |
| --------       | -----------                                                                                                |
|lock_shared|Locks the mutex for shared ownership, blocks if the mutex is not available <span style="color:green;font-size:12px">(public member function)</span>|
|try_lock_shared|Tries to lock the mutex for shared ownership, returns if the mutex is not available <span style="color:green;font-size:12px">(public member function)</span>|
|try_lock_shared_for|tries to lock the mutex for shared ownership, returns if the mutex has been unavailable for the specified timeout duration <span style="color:green;font-size:12px">(public member function)</span>|
|try_lock_shared_until|tries to lock the mutex for shared ownership, returns if the mutex has been unavailable until specified time point has been reached <span style="color:green;font-size:12px">(public member function)</span>|
|unlock_shared|Unlocks the mutex (shared ownership) <span style="color:green;font-size:12px">(public member function)</span>|

<div id='13'/>

#### Data Race
Data Race is a situation where more than one thread access the same memory location simultaneously, and at least one writes. This is clearly a conflict.
A data race is minimized (solved) by blocking or locking, as illustrated above. It can also be handled using, Call Once – see below. These three features are in the mutex library. These are the fundamental ways of a handling data race. There are other more advanced ways, which bring in more convenience – see below.
<div id='14'/>

#### Locks
Locks are objects that manage a mutex by associating its access to their own lifetime. Defined in header <font color="red">\<mutex></font>.
|Functions                                                         | Description                     |
| --------                                                         | -----------                     |
|lock_guard <span style="color:green;font-size:12px">(C++11)</span>|Implements a strictly scope-based mutex ownership wrapper <span style="color:green;font-size:12px">(class template)</span>|
|unique_lock <span style="color:green;font-size:12px">(C++11)</span>|Implements movable mutex ownership wrapper <span style="color:green;font-size:12px">(class template)</span>|
|shared_lock <span style="color:green;font-size:12px">(C++11)</span>|Implements movable shared mutex ownership wrapper <span style="color:green;font-size:12px">(class template)</span>|
|scoped_lock <span style="color:green;font-size:12px">(C++17)</span>|Deadlock-avoiding RAII wrapper for multiple mutexes <span style="color:green;font-size:12px">(class template)</span> |


Tag constants used to specify locking strategy.

|Tag arguments                                                     | Description                     |
| --------                                                         | -----------                     |
|defer_lock <span style="color:green;font-size:12px">(C++11)</span>|Do not acquire ownership of the mutex <span style="color:green;font-size:12px">defer_lock_t</span>|
|try_to_lock <span style="color:green;font-size:12px">(C++11)</span>|Try to acquire ownership of the mutex without blocking <span style="color:green;font-size:12px">try_to_lock_t</span>|
|adopt_lock <span style="color:green;font-size:12px">(C++11)</span>|Assume the calling thread already has ownership of the mutex <span style="color:green;font-size:12px">adopt_lock_t</span>|


scoped_lock is not covered in this article.

##### lock_guard
Represented as <font color="red">std::lock_guard</font> defined as class in header <font color="red">\<mutex></font> (since C++11)
The class lock_guard is a mutex wrapper that provides a convenient RAII-style mechanism for owning a mutex for the duration of a scoped block.
When a lock_guard object is created, it attempts to take ownership of the mutex it is given. When control leaves the scope in which the lock_guard object was created, the lock_guard is destructed and the mutex is released. The lock_guard class is non-copyable. The following code shows how a lock_guard is used:
```cpp
#include <iostream>
#include <thread>
#include <mutex>

int globalVar = 5;
std::mutex m;

void thrdFn() {
    //some statements
    std::lock_guard<std::mutex> lck(m);
    globalVar += 2;
    std::cout << globalVar << '\n';
    //statements
}

int main(int argc, char const *argv[])
{
    std::thread thr(&thrdFn);
    thr.join();

    return 0;
}
```
The output is:
```cpp 
7
```
The type (class) is lock_guard in the mutex library. In constructing its lock object, it takes the template argument, mutex. In the code, the name of the lock_guard instantiated object is lck. It needs an actual mutex object for its construction (m). Notice that there is no statement to unlock the lock in the program. This lock died (unlocked) as it went out of the scope of the thrdFn() function.
##### unique_lock
Represented as <font color="red">std::unique_lock</font> defined as class in header <font color="red">\<mutex></font> (since C++11)
Only its current thread can be active when any lock is on, in the interval, while the lock is on. The main difference between unique_lock and lock_guard is that ownership of the mutex by a unique_lock, can be transferred to another unique_lock. unique_lock has more member functions than lock_guard.

|Functions       | Description                                                                                                |
| --------       | -----------                                                                                                |
|lock |Locks (i.e., takes ownership of) the associated mutex <span style="color:green;font-size:12px">(public member function)</span>
|try_lock |Tries to lock (i.e., takes ownership of) the associated mutex without blocking <span style="color:green;font-size:12px">(public member function)</span>
|try_lock_for |Attempts to lock (i.e., takes ownership of) the associated TimedLockable mutex, returns if the mutex has been unavailable for the specified time duration <span style="color:green;font-size:12px">(public member function)</span>
|try_lock_until| Tries to lock (i.e., takes ownership of) the associated TimedLockable mutex, returns if the mutex has been unavailable until specified time point has been reached <span style="color:green;font-size:12px">(public member function)</span>
|unlock| Unlocks (i.e., releases ownership of) the associated mutex <span style="color:green;font-size:12px">(public member function)</span>

Ownership of a mutex can be transferred from unique_lock1 to unique_lock2 by first releasing it off unique_lock1, and then allowing unique_lock2 to be constructed with it. unique_lock has an unlock() function for this releasing. In the following program, ownership is transferred in this way:
```cpp
#include <iostream>
#include <thread>
#include <mutex>

std::mutex m;

int globalVar = 5;

void thrdFn2() {
    std::unique_lock<std::mutex> lck2(m);
    globalVar += 2;
    std::cout << globalVar << '\n';
}

void thrdFn1() {
    std::unique_lock<std::mutex> lck1(m);
    globalVar += 2;
    std::cout << globalVar << '\n';

    lck1.unlock();
    std::thread thr2(&thrdFn2);
    thr2.join();  
}  

int main(int argc, char const *argv[])
{
    std::thread thr1(&thrdFn1);  
    thr1.join();

    return 0;
}
```
The output is:
```cpp
7
9
```
The mutex of unique_lock, lck1 was transferred to unique_lock, lck2. The unlock() member function for unique_lock does not destroy the mutex.

##### shared_lock
Represented as <font color="red">std::shared_mutex</font> defined as class in header <font color="red">\<mutex></font> (since C++17)
The shared_mutex class is a synchronization primitive that can be used to protect shared data from being simultaneously accessed by multiple threads. In contrast to other mutex types which facilitate exclusive access, a shared_mutex has two levels of access:
- shared - several threads can share ownership of the same mutex.
- exclusive - only one thread can own the mutex.

If one thread has acquired the exclusive lock (through lock, try_lock), no other threads can acquire the lock (including the shared).
If one thread has acquired the shared lock (through lock_shared, try_lock_shared), no other thread can acquire the exclusive lock, but can acquire the shared lock.
Only when the exclusive lock has not been acquired by any thread, the shared lock can be acquired by multiple threads.
Within one thread, only one lock (shared or exclusive) can be acquired at the same time.
Shared mutexes are especially useful when shared data can be safely read by any number of threads simultaneously, but a thread may only write the same data when no other thread is reading or writing at the same time.

|Exclusive locking| Description                                                                           |
| --------| -----------                                                                           |
|try_lock |Tries to lock the mutex, returns if the mutex is not available <span style="color:green;font-size:12px">(public member function)</span>|
|lock     |Locks the mutex, blocks if the mutex is not available <span style="color:green;font-size:12px">(public member function)</span>         |
|unlock   |Unlock multiple mutexes <span style="color:green;font-size:12px">(public member function)</span>                                       |

|Shared locking       | Description                                                                                                |
| --------       | -----------                                                                                                |
|try_lock_shared |Tries to lock the mutex for shared ownership, returns if the mutex is not available <span style="color:green;font-size:12px">(public member function)</span>|
|lock_shared     |Locks the mutex for shared ownership, blocks if the mutex is not available <span style="color:green;font-size:12px">(public member function)</span>         |
|unlock_shared   |Unlock the mutex (shared ownership) <span style="color:green;font-size:12px">(public member function)</span>                                                |

##### Deadlock
Deadlock is a situation where a set of processes are blocked because each process is holding a resource and waiting for another resource acquired by some other process. 
Consider an example when two trains are coming toward each other on the same track and there is only one track, none of the trains can move once they are in front of each other. A similar situation occurs in operating systems when there are two or more processes that hold some resources and wait for resources held by other(s). For example, in the below diagram, Thread 1 is holding Resource 1 and waiting for resource 2 which is acquired by Thread 2, and Thread 2 is waiting for resource 1. 

![image](/deadlock.png)

Let's see an example:
```cpp
#include <iostream>     //std::cout std::endl;
#include <thread>       //std::thread
#include <mutex>        //std::lock_guard

std::mutex m1, m2;      //assigned for resource 1 and 2 respectively

void Resource1(std::string str) {
    std::cout << "Resource 1 owned by " << str << std::endl;
}

void Resource2(std::string str) {
    std::cout << "Resource 2 owned by " << str << std::endl;
}

void Thread1(std::string str) {
    std::lock_guard<std::mutex> l1(m1);
    Resource1(str);

    std::lock_guard<std::mutex> l2(m2);
    Resource2(str); 
}

void Thread2(std::string str) {
    std::lock_guard<std::mutex> l2(m2);
    Resource2(str);

    std::lock_guard<std::mutex> l1(m1);
    Resource1(str);
}

int main(int argc, char const* argv[])
{
    std::cout << "running process" << "\n";

    std::thread t1(Thread1, "Thread 1");
    std::thread t2(Thread2, "Thread 2");

    t1.join();
    t2.join();

    std::cout << "finished process" << "\n";
    return 0;
}
```
The output is:
```cpp
running process
Resource 1 owned by Thread 1
Resource 2 owned by Thread 2


```
As we can see, the main thread keeps waiting for the Join() of both threads t1 and t2 and therefore the Deadlock does not allow our program to finish executing the message line "finished process" .
##### Avoiding Deadlock
One possible solution for the above Deadlock case would be the following:
```cpp
#include <iostream>     //std::cout std::endl;
#include <thread>       //std::thread
#include <mutex>        //std::mutex std::lock std::lock_guard std::adopt_lock

std::mutex m1, m2;      //assigned for resource 1 and 2 respectively

void Resource1(std::string str) {
    std::cout << "Resource 1 owned by " << str << std::endl;
}

void Resource2(std::string str) {
    std::cout << "Resource 2 owned by " << str << std::endl;
}

void Thread1(std::string str) {
    std::lock(m1, m2);
    std::lock_guard<std::mutex> l1(m1, std::adopt_lock);
    Resource1(str);

    std::lock_guard<std::mutex> l2(m2, std::adopt_lock);
    Resource2(str);
}

void Thread2(std::string str) {
    std::lock(m1, m2);
    std::lock_guard<std::mutex> l2(m2, std::adopt_lock);
    Resource2(str);

    std::lock_guard<std::mutex> l1(m1, std::adopt_lock);
    Resource1(str);
}

int main(int argc, char const* argv[])
{
    std::cout << "running process" << "\n";

    std::thread t1(Thread1, "Thread 1");
    std::thread t2(Thread2, "Thread 2");

    t1.join();
    t2.join();

    std::cout << "finished process" << "\n";
    return 0;
}
```
The output is:
```cpp
running process
Resource 1 owned by Thread 1
Resource 2 owned by Thread 1
Resource 2 owned by Thread 2
Resource 1 owned by Thread 2
finished process
```
Some strategies to avoid deadlock:
- Prefer locking single mutex by using separate scopes: { }.
- Avoid locking a mutex and then calling a user provided function.
- Use std::lock() to lock more than one mutex.
- Lock the mutex in the same order.

<div id='15'/>

#### Call Once
Represented as <font color="red">std::call_once</font> in header <font color="red">\<mutex></font>, ensures execution of a function exactly once by competing threads. It throws std::system_error in case it cannot complete its task. Used in conjunction with <font color="red">std::once_flag</font>.
Imagine that there is a function that has to increment a global variable of 10 by 5. If this function is called once, the result would be 15 – fine. If it is called twice, the result would be 20 – not fine. If it is called three times, the result would be 25 – still not fine. The following program illustrates the use of the `call_once` feature:
```cpp
#include <iostream>
#include <thread>
#include <mutex>

int globalVar = 10;

std::once_flag flag1;

void thrdFn(int no) {
    call_once(
                flag1,                       //flag object ensures the call_once runs just once    
                [no](){ globalVar += no;}    //Lambda function or callable object
            );
}

int main(int argc, char const *argv[])
{
    std::thread thr1(&thrdFn, 5);  
    std::thread thr2(&thrdFn, 5);  
    std::thread thr3(&thrdFn, 5);  
    
    thr1.join();
    thr2.join();
    thr3.join();

    std::cout << globalVar << std::endl;

    return 0;
}
```
The output is: 
```cpp
15 
```
Confirming that the function, thrdFn(), was called once. That is, the first thread was executed, and the following two threads in main()were not executed. `void call_once()` is a predefined function in the mutex library. It is called the function of interest (thrdFn), which would be the function of the different threads. Its first argument is a flag – see later. In this program, its second argument is a void lambda function. In effect, the lambda function has been called once, not really the thrdFn() function. It is the lambda function in this program that really increments the global variable.
<div id='16'/>

#### Condition Variable Library

Condition Variable is a class and kind of Event defined <span style="color:green;font-size:12px">(C++11)</span> in header <font color="red">\<condition_variable></font> and used for communication between two or more threads. It allows some number of threads to wait (possibly with a timeout) for notification from another thread that they may proceed to use that shared resource that was being used by another thread. A condition variable is always associated with a mutex.

|Notification |Description |
| --------    | -----------|
|notify_one| Notifies one waiting thread <span style="color:green;font-size:12px">(public member function)</span>|
|notify_all| Notifies all waiting thread <span style="color:green;font-size:12px">(public member function)</span>|

|Waiting |Description |
| --------    | -----------|
|wait| Blocks the current thread until the condition variable is awakened <span style="color:green;font-size:12px">(public member function)</span>|
|wait_for| Blocks the current thread until the condition variable is awakened or after the specified timeout duration <span style="color:green;font-size:12px">(public member function)</span>|
|wait_until| Blocks the current thread until the condition variable is awakened or until specified time point has been reached <span style="color:green;font-size:12px">(public member function)</span>|

Basically <font color="red">condition_variable</font> works using the notify and waiting strategy. While the waiting thread must have <font color="red">std::unique_lock\<std::mutex></font>, the notifying thread can have <font color="red">std::lock_guard\<std::mutex></font>.
The wait() function statement should be coded just after the locking statement in the waiting thread. All locks in this thread synchronization scheme use the same mutex.
The following program illustrates the use of the condition variable, with two threads:
```cpp
#include <iostream>
#include <thread>
#include <condition_variable>

std::mutex m;
std::condition_variable cv;

bool dataReady = false;

void waitingForWork(){
    std::cout << "Waiting" << '\n';
    std::unique_lock<std::mutex> lck1(m);
    cv.wait(lck1, []{ return dataReady; });  
    std::cout << "Running" << '\n';
}

void setDataReady(){
    {
        std::lock_guard<std::mutex> lck2(m);
        dataReady = true;
        std::cout << "Data prepared" << '\n';
    }
    cv.notify_one();                        
}

int main(int argc, char const *argv[]){
    std::thread thr1(waitingForWork);
    std::thread thr2(setDataReady);

    thr1.join();
    thr2.join();

    return 0;
}
```
The output is:
```cpp
Waiting
Data prepared
Running
```
This program consists of two child threads (thr1 and thr2). When thr1 runs, it calls function waitingForWork() and displays a message <font color="red">"Waiting"</font> and the second step is initializing lck1 from unit_lock. How it's said before condition_variable works with unique_lock together. The funcion wait() is waiting that dataReady global variable will be true.
At the moment thr2 runs, it calls function setDataReady(), where in other followed scope it is initializing lck2 from lock_guard. 
As thr1 depends on thr2 finishing first. It is for this reason that thr1 is related to the wait() function. Then in this secondary scope after creating thr2, setting dataReady in true and displaying <font color="red">"Data prepared"</font>. Once this scope is finished, lck2 is destroyed, the mutex object m is automatically unlocked and the notification is executed, which is what the wait() function was waiting for to continue. Then lck1 with dataReady being true will set lck1 locking to  mutex object m, and now thr1 will finish its execution displaying <font color="red">"Running"</font>. 

<div id='17'/>

#### Future Library

Represented in header <font color="red">\<Future></font> provides facilities to obtain values that are returned by separated threads. These values are communicated in a shared state, in which the asynchronous task may write its return value or store an exception, and which may be examined, waited for, and otherwise manipulated by other threads that hold instances of std::future or std::shared_future that reference that shared state.

|Content      |Description |
| --------    | -----------|
|future <span style="color:green;font-size:12px">(C++11)</span>|Waits for a value that is set asynchronously <span style="color:green;font-size:12px">(class template)</span>|
|promise <span style="color:green;font-size:12px">(C++11)</span>|Stores a value for asynchronous retrieval <span style="color:green;font-size:12px">(class template)</span>|
|shared_future <span style="color:green;font-size:12px">(C++11)</span>|Waits for a value (possibly referenced by other futures) that is set asynchronously <span style="color:green;font-size:12px">(class template)</span>|
|async <span style="color:green;font-size:12px">(C++11)</span>|Runs a function asynchronously (potentially in a new thread) and returns a std::future that will hold the result <span style="color:green;font-size:12px">(function template)</span>|
|launch <span style="color:green;font-size:12px">(C++11)</span>|Specifies the launch policy for std::async <span style="color:green;font-size:12px">(enum)</span>|
|packaged_task <span style="color:green;font-size:12px">(C++11)</span>|Packages a function to store its return value for asynchronous retrieval <span style="color:green;font-size:12px">(class template)</span>|

##### promise

Promise is a class in the future library. A promise object can store a value of type T to be retrieved by a future object (possibly in another thread), offering a synchronization point.
In this section we will discuss the uses of std::promise together with std::future and std::thread.
Basically both "std::promise" as "std::future" objects must work engaged.
The two main uses of this engagement of these two objects are:
- Set a value from parent thread to child thread.
- Return a value from child thread to parent thread.

Promise and future once engaged, the main rule is: while promise is used to set a value and future to get it.

An example about the first one use:
```cpp
#include <iostream> // std::cout  std::endl
#include <thread>   // std::thread
#include <future>   // std::future  std::promise

void fn(std::future<std::string>& fut){
    std::string ret = fut.get() + 
    " and printing through child thread function.";
    std::cout << ret << std::endl;;
}

int main(int argc, char const *argv[]){
    //creating promise object
    std::promise<std::string> pro;

    //engagement future object with promise object
    std::future<std::string> fut = pro.get_future();

    //creating and launching thread
    std::thread t1(fn,std::ref(fut));

    //sets the promise object and then gets in the future object.
    pro.set_value("Sets value from main thread");

    t1.join();

    return 0;  
}
```
The output is:
```cpp
Sets value from main thread and printing through child thread function.
```

An example about the second one use:
```cpp
#include <iostream> // std::cout  std::endl
#include <thread>   // std::thread
#include <future>   // std::future  std::promise

void fn(std::promise<std::string> pro, int inpt) {
    pro.set_value("Returns value from thread function.");
}

int main(int argc, char const* argv[]) {
    //creating promise object
    std::promise<std::string> pro;                  
    
    //engagement future object with promise object
    std::future<std::string> fut = pro.get_future();
    
    //creating and launching thread
    std::thread t1(fn, std::move(pro), 6);

    std::string ret = fut.get();
    
    t1.join();

    std::cout << ret << std::endl;

    return 0;
}
```
The output is:
```cpp
Returns value from thread function.
```
Another same example:
```cpp
#include <iostream> // std::cout  std::endl
#include <thread>   // std::thread
#include <future>   // std::promise  std::future  std::move

void setDataReady(std::promise<int>&& ret, int inpt){
    int result = inpt + 4;
    ret.set_value(result);                    
}

int main(int argc, char const *argv[]){
    std::promise<int> adding;
    std::future<int> fut = adding.get_future();
    //launch the thread
    std::thread thr(setDataReady, std::move(adding), 6);
    
    //main() thread waits the result
    int res = fut.get();
    
    // printing returns from future
    std::cout << res << std::endl;  

    thr.join();
    return 0;  
}
```
The output is:
```cpp
10
```
 There are two threads here: the main() function and thr. Note the inclusion of `<future>`. The function parameters for setDataReady() of thr, are `promise<int>&& ret` and `int inpt`. The first statement in this function body adds 4 to 6, which is the inpt argument sent from main(), to obtain the value for 10. A promise object is created in main() and sent to this thread as `ret`.
One of the member functions of promise is set_value(). Another one is set_exception(). set_value() puts the result into the shared state. If the thread thr could not obtain the result, the programmer would have used the set_exception() of the promise object to set an error message into the shared state. After the result or exception is set, the promise object sends out a notification message.

The future object must: wait for the promise's notification, ask the promise if the value (result) is available, and pick up the value (or exception) from the promise.

In the main function (thread), the first statement creates a promise object called adding. A promise object has a future object. The second statement returns this future object in the name of `fut`. Note here that there is a connection between the promise object and its future object.

The third statement creates a thread. Once a thread is created, it starts executing concurrently. Note how the promise object has been sent as an argument (also note how it was declared a parameter in the function definition for the thread).

The fourth statement gets the result from the future object. Remember that the future object must pick up the result from the promise object. However, if the future object has not yet received a notification that the result is ready, the main() function will have to wait at that point until the result is ready. After the result is ready, it would be assigned to the variable, `res`.

Another example using multiple objects:
```cpp
#include <iostream> // std::cout
#include <thread>   // std::thread
#include <future>   // std::promise  std::future  std::move

void setDataReady(std::promise<int>&& ret, int inpt) {
    int result = inpt + 4;
    ret.set_value(result);
}

int main(int argc, char const* argv[]) {

    std::promise<int> pro[5];                   // create promise
    std::future<int> fut[5];                    // create future
    std::thread thr[5];                         // create child threads

    // engagement future with promise
    for (int i = 0; i < 5; i++) {
        fut[i] = pro[i].get_future();          
    }

    //launching threads
    for (int i = 0; i < 5; i++) {
        thr[i] = std::thread(setDataReady, std::move(pro[i]), i);   
    }

    // waiting and printing the return from future
    for (int i = 0; i < 5; i++) {
        std::cout << "thr[" << i << "] = " << fut[i].get() << std::endl;   
    }
    
    // waiting child threads are finished
    for (int i = 0; i < 5; i++) {
        thr[i].join();                           
    }

    return 0;
}
```
The output is:
```cpp
thr[0] = 4
thr[1] = 5
thr[2] = 6
thr[3] = 7
thr[4] = 8
```
##### async()
The future library has the function async(). This function returns a future object. The main argument to this function is an ordinary function that returns a value. The return value is sent to the shared state of the future object. The calling thread gets the return value from the future object. Using async() here is, that the function runs concurrently to the calling function. The async function can be used two different ways:
- std::future object and async()
- std::future and std::promise objects together with async()

The following program illustrates the first one case:
```cpp
#include <iostream> // std::cout  std::endl
#include <future>   // std::future  std::async()

int fn(int input){
    int result = input + 4;
    return result;
}

int main(int argc, char const *argv[]){

    std::future<int> output = std::async(fn, 6);
    
    //main() thread waits here the result
    int res = output.get();
    
    std::cout << res << std::endl;  

    return 0;  
}
```
The output is:
```cpp
10
```
The following program illustrates the second one case:
```cpp
#include <iostream> // std::cout  std::endl
#include <future>   // std::future std::promise std::async()

int fn(std::future<int>& input) {
    int result = input.get() + 4;
    return result;
}

int main(int argc, char const* argv[]) {

    std::promise<int> pro;
    std::future<int> fut = pro.get_future();
    
    std::future<int> output = std::async(fn, std::ref(fut));
    pro.set_value(6);
    
    //main() thread waits here the result
    int res = output.get();

    std::cout << res << std::endl;

    return 0;
}
```
The output is:
```cpp
10
```

##### shared_future

The class template std::shared_future provides a mechanism to access the result of asynchronous operations, similar to std::future, except that multiple threads are allowed to wait for the same shared state. Unlike std::future, which is only moveable (so only one instance can refer to any particular asynchronous result), std::shared_future is copyable and multiple shared future objects may refer to the same shared state.
Access to the same shared state from multiple threads is safe if each thread does it through its own copy of a shared_future object.
 The following program illustrates the use of shared_future:
```cpp
#include <iostream>     //std::cout std::endl;
#include <thread>       //std::thread
#include <future>       //std::promise  std::shared_future

std::promise<int> pro;
std::shared_future<int> fut = pro.get_future();

void thrdFn2() {
    int rs = fut.get();         // rs=10 <- fut
    //thread, thr2 waits here
    int result = rs + 4;        // result = 14
    std::cout << result << std::endl;
}

void thrdFn1(int input) {

    int result = input + 4;     // result = 6 + 4
    pro.set_value(result);      // 10 -> pro -> fut

    std::thread thr2(thrdFn2);
    thr2.join();

    int res = fut.get();        // res=10 <- fut
    //thread, thr1 waits here
    std::cout << res << std::endl;
}

int main(int argc, char const* argv[]) {
    std::thread thr1(&thrdFn1, 6);
    thr1.join();

    return 0;
}
```
The output is:
```cpp
14
10
```
Two different threads have shared the same future object. Note how the shared future object was created. The result value, 10, has been gotten twice from two different threads. The value can be gotten more than once from many threads but cannot be set more than once in more than one thread. Note where the statement, `thr2.join();` has been placed in thr1

let's see another example:
```cpp
#include <iostream> // std::cout  std::endl
#include <chrono>   // std::chrono
#include <future>   // std::shared_future  std::async()

int work(std::shared_future<float> fu)
{
    std::this_thread::sleep_for(std::chrono::milliseconds(1000));
    double Result1 = fu.get();
    std::cout << "Child::Result1 : " << Result1 << std::endl;


    double Result2 = fu.get();
    std::cout << "Child::Result2 : " << Result2 << std::endl;

    return 10;
}

int main(int argc, char const* argv[]) {
    // template typename <float> is the type of parameter
    std::promise<float> p;
    std::shared_future<float> fu = p.get_future();
    
    // template typename <int> must match with function return
    std::shared_future<int> fr = std::async(std::launch::async, work, fu);
    p.set_value(20.5);

    // fr has "10" is the return value of the work() function. 
    int result1 = fr.get();
    std::cout << "Parent::Result1 : " << result1 << std::endl;


    int result2 = fr.get();
    std::cout << "Parent::Result2 : " << result2 << std::endl;
    return 0;
}
```
The output is:
```cpp
Child::Result1 : 20.5
Child::Result2 : 20.5
Parent::Result1 : 10
Parent::Result2 : 10
```
std::shared_future can be used, as it copies value and thereby allows the user to invoke get() multiple times as required whereas std::future allows it only once.
##### packaged_task

The class template std::packaged_task wraps any Callable target (function, lambda expression, bind expression, or another function object) so that it can be invoked asynchronously. Its return value or exception thrown is stored in a shared state which can be accessed through std::future objects.
Let's see an example:
```cpp
#include <iostream>
#include <cmath>
#include <thread>
#include <future>
#include <functional>

// unique function to avoid disambiguating the std::pow overload set
int f(int x, int y) { return std::pow(x, y); }

void task_lambda()
{
    std::packaged_task<int(int, int)> task([](int a, int b) {
        return std::pow(a, b);
        });
    std::future<int> result = task.get_future();

    task(2, 9);

    std::cout << "task_lambda:\t" << result.get() << '\n';
}

void task_bind()
{
    std::packaged_task<int()> task(std::bind(f, 2, 11));
    std::future<int> result = task.get_future();

    task();

    std::cout << "task_bind:\t" << result.get() << '\n';
}

void task_thread()
{
    std::packaged_task<int(int, int)> task(f);
    std::future<int> result = task.get_future();

    std::thread task_td(std::move(task), 2, 10);
    task_td.join();

    std::cout << "task_thread:\t" << result.get() << '\n';
}

int main()
{
    task_lambda();
    task_bind();
    task_thread();
}
```
The output is:
```cpp
task_lambda:    512
task_bind:      2048
task_thread:    1024
```

<div id='18'/>

#### Conclusion
A thread (thread of execution) is a single flow of control in a program. More than one thread can be in a program, to run concurrently or in parallel. In C++, a thread object has to be instantiated from the thread class to have a thread.

Data Race is a situation where more than one thread is trying to access the same memory location simultaneously, and at least one is writing. This is clearly a conflict. The fundamental way to resolve the data race for threads is to block the calling thread while waiting for the resources. When it could get the resources, it locks them so that it alone and no other thread would use the resources while it needs them. It must release the lock after using the resources so that some other thread can lock onto the resources.

Mutexes, locks, condition_variable and future, are used to resolve data race for threads. Mutexes need more coding than locks and so more prone to programming errors. locks need more coding than condition_variable and so more prone to programming errors. condition_variable needs more coding than future, and so more prone to programming errors.
If you have read this article and understood, you would read the rest of the information concerning the thread, in the C++ specification, and understand.