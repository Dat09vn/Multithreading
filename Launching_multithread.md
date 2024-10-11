**1. what is multithreading**

Multithreading allows a program to perform multiple tasks at the same time by dividing a process into smaller threads that can be executed independently. Each thread operates independently but shares the same resources, such as memory, of the main program.

**2. Key Concepts:**
- Thread: A single sequence of execution within a program. Multiple threads can exist within the same process, sharing its resources.
- Concurrency: The ability of a system to handle multiple tasks simultaneously, though they may not actually run at the exact same time (depending on the hardware capabilities).
- Parallelism: A form of computing where multiple tasks are literally executed at the same time on different cores or processors.
- Synchronization: Mechanisms to manage access to shared resources between threads, preventing issues like race conditions (when two or more threads modify a shared resource simultaneously).
- Context switching: The process of storing the state of a thread and restoring the state of another, allowing multiple threads to run on a single CPU core.

**3. Challenges:**
- Race conditions: Multiple threads may access and modify shared data at the same time, leading to inconsistent results.
- Deadlocks: Threads may become stuck waiting for each other to release resources, leading to a halt in program execution.
- Complex debugging: Issues like race conditions or deadlocks are often hard to trace and debug.
  
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
1. In C++20, the **std::jthread (joining thread)** was introduced as an improvement over std::thread that simplifies thread management and ensures better resource safety

The main features of std::jthread compared to std::thread include:
- Automatic Joining: When a std::jthread object goes out of scope, it automatically joins (waits for the thread to finish), ensuring that no threads are left unjoined, avoiding potential program termination due to uncaught exceptions.
- Cancellation: std::jthread supports cooperative cancellation via a cancellation token (std::stop_token), allowing a thread to request cancellation in a safe and controlled manner.
```c++
#include <iostream>
#include <thread>
#include <chrono>

void threadFunction() {
    std::this_thread::sleep_for(std::chrono::seconds(1)); // Simulate work
    std::cout << "Thread finished execution." << std::endl;
}

int main() {
    std::jthread jt(threadFunction); // Automatically joins when out of scope

    std::cout << "Main thread is finishing..." << std::endl;
    // No need to call jt.join(); it is automatically joined.
    return 0;
}
```
**2. Race Condition and Data Race**

Race conditions and data races are issues that occur when multiple threads or processes attempt to access shared resources concurrently.
- Race condition happens when the behavior of a software system depends on the sequence or timing of uncontrollable events like thread execution. If threads don't execute in the order the program expects, it can cause unpredictable results.
- Data race is a specific type of race condition. It occurs when two or more threads access the same memory location concurrently, and at least one of the accesses is a write. This can lead to inconsistent or corrupted data.

Example race condition with banking system:
```c++
#include <iostream>
#include <thread>

int accountBalance = 100; // Shared resource

void withdraw(int amount) {
    if (accountBalance >= amount) {   // Step 1: Check if balance is sufficient
        std::this_thread::sleep_for(std::chrono::milliseconds(10)); // Simulate some delay
        accountBalance -= amount;     // Step 2: Deduct the amount
        std::cout << "Withdrawal of " << amount << " successful. Remaining balance: " << accountBalance << std::endl;
    } else {
        std::cout << "Insufficient funds for withdrawal of " << amount << ". Remaining balance: " << accountBalance << std::endl;
    }
}

int main() {
    std::thread t1(withdraw, 80);  // Thread 1 tries to withdraw $80
    std::thread t2(withdraw, 80);  // Thread 2 also tries to withdraw $80

    t1.join();
    t2.join();

    return 0;
}
```
Example data race:
```c++
#include <iostream>
#include <thread>

int sharedVariable = 0;

void increment() {
    for (int i = 0; i < 100000; ++i) {
        ++sharedVariable; // Data race: Unsynchronized write to sharedVariable
    }
}

int main() {
    std::thread t1(increment);
    std::thread t2(increment);

    t1.join();
    t2.join();

    std::cout << "Final value of sharedVariable: " << sharedVariable << std::endl;
    return 0;
}
```
Note: Concurrent Access: Multiple threads are incrementing the sharedVariable concurrently. This operation is not atomic; it consists of multiple steps:
- Reading the current value of sharedVariable.
- Incrementing the value.
- Writing the updated value back to sharedVariable.
  
**To avoid a data race**, you must synchronize access to shared resources (in this case, sharedVariable) using techniques such as:
- Mutexes: Ensure that only one thread can access the shared data at a time.
- Atomic Operations: Use atomic operations to guarantee that a variable is modified atomically (without interruption).


