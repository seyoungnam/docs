# Process vs Thread

## Process
A **process** is an independent execution unit that contains its own program counter, stack, and address space.

- **Isolation**: Processes are fully isolated from each other. If one crashes, it does not directly affect others.
- **Resources**: Each process has its own memory space (heap, stack, data).

??? Question "What Stack, Heap, and Data represent"

1. **Stack (Static Allocation)**
The Stack is used for "automatic" memory management during function calls.

* What it stores: Local variables, function parameters, and return addresses.
* Behavior: It follows a LIFO (Last-In, First-Out) structure. When a function is called, a "stack frame" is added; when it returns, that frame is removed.
* Thread Context: In your documentation, this is a key point: Each thread has its own private Stack to track its own function calls and local variables.

- **Inter-Process Communication (IPC)**: Since memory is isolated, processes must use IPC mechanisms (pipes, sockets, shared memory) to communicate, which adds overhead.

## Thread
A **thread** is a subset of a process and is the smallest unit of CPU utilization.

- **Shared Memory**: All threads within the same process share the heap, code, and data sections. However, each thread maintains its own register set and stack.
- **Efficiency**: Creating and managing threads is much faster than processes because they don't require a new address space.
- **Communication**: Communication between threads is efficient as they can directly access shared memory, though this requires synchronization (mutexes, semaphores) to prevent race conditions.

## Key Differences

| Feature | Process | Thread |
| :--- | :--- | :--- |
| **Memory** | Independent address space | Shared address space within a process |
| **Overhead** | High (creation, termination, switching) | Low |
| **Isolation** | High (fault-tolerant) | Low (one thread crash can kill the process) |
| **Communication** | IPC (slow) | Direct shared memory (fast) |

## Context Switching
Context switching is the process of storing the state of a CPU so that execution can be resumed from the same point later.

- **Process Context Switch**: Expensive. Requires saving/loading entire address spaces and updating memory mapping (TLB flush).
- **Thread Context Switch**: Cheaper. Since threads share the same address space, the system only needs to swap the registers and stack pointers.
