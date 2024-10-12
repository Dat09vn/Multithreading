**1. What is mutex?**

Mutex stand for: (Mutual exclusion object) is a synchronization primitive used in concurrent programming to protect shared resources from being accessed by multiple threads at the same time. Its primary function is to prevent race conditions by ensuring that only one thread can access a critical section of code or shared data at any given time.

A Mutex has two states: lock, unlock (try_lock)

Key Concepts of a Mutex:
- Mutual Exclusion: Only one thread can "lock" the mutex at a time, ensuring exclusive access to the shared resource.
- Critical Section: The portion of code that accesses shared resources and needs to be executed by only one thread at a time is known as a critical section.
- Lock and Unlock:
  - A thread that wants to enter the critical section must lock the mutex.
  - Once inside the critical section, other threads trying to lock the mutex will be blocked (wait) until the first thread unlocks the mutex after completing its work.
- Blocking: Threads that attempt to lock a mutex while it is already locked must wait (or "block") until the mutex becomes available again (i.e., the first thread unlocks it).

**2. Deadlock:**
- If a thread calls lock() on a mutex that it has already locked, this can cause a deadlock. The thread will block indefinitely, waiting for itself to unlock the mutex.
- Deadlocks can also occur when multiple mutexes are involved, and threads lock them in different orders. For example, if Thread 1 locks mutex A and then tries to lock mutex B, while Thread 2 locks mutex B and then tries to lock mutex A, both threads can get stuck waiting for each other.

std::lock_guard: Instead of manually calling lock() and unlock(), you can use std::lock_guard to automatically lock the mutex when it’s created and unlock it when it goes out of scope. This helps prevent issues like forgetting to unlock the mutex.

Example demomstrate Deadlock:
```c++
#include <iostream>
#include <thread>
#include <mutex>

std::mutex mtxA;  // Mutex A
std::mutex mtxB;  // Mutex B

// Task for Thread 1
void thread1() {
    std::lock_guard<std::mutex> lockA(mtxA);  // Thread 1 locks Mutex A
    std::cout << "Thread 1 locked Mutex A\n";

    // Simulate some work
    std::this_thread::sleep_for(std::chrono::milliseconds(100));

    std::cout << "Thread 1 trying to lock Mutex B\n";
    std::lock_guard<std::mutex> lockB(mtxB);  // Thread 1 attempts to lock Mutex B (Deadlock risk)
    std::cout << "Thread 1 locked Mutex B\n";
}

// Task for Thread 2
void thread2() {
    std::lock_guard<std::mutex> lockB(mtxB);  // Thread 2 locks Mutex B
    std::cout << "Thread 2 locked Mutex B\n";

    // Simulate some work
    std::this_thread::sleep_for(std::chrono::milliseconds(100));

    std::cout << "Thread 2 trying to lock Mutex A\n";
    std::lock_guard<std::mutex> lockA(mtxA);  // Thread 2 attempts to lock Mutex A (Deadlock risk)
    std::cout << "Thread 2 locked Mutex A\n";
}

int main() {
    std::thread t1(thread1);
    std::thread t2(thread2);

    t1.join();
    t2.join();

    return 0;
}
```
