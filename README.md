# Lab 3: Basic Process Management and IPC

#### Learning Objectives

- Understand fork() system call - How processes are created and copied
- Understand wait() system call - How parent processes manage child lifecycle
- Practice basic pipe() communication - Simple inter-process data transfer

#### Prerequisites

- Basic C programming knowledge
- Familiarity with system calls from previous labs
- Understanding of file descriptors and I/O operations

#### Introduction

Process management is a fundamental aspect of operating systems. This lab explores the three most important process
management concepts: process creation with fork(), process cleanup with wait(), and basic inter-process communication
with pipes.

You'll implement a simple producer-consumer system where producers generate sequential numbers and consumers add them
up. The focus is entirely on understanding process behavior - the arithmetic is just busy work to give the processes
something to do.

#### What You'll Implement

You need to write just 2 functions:

1. `run_basic_demo()` - Creates one producer and one consumer child process
2. `run_multiple_pairs()` - Creates multiple producer-consumer pairs

Everything else is provided for you.

#### Lab File Structure

**main.c (PROVIDED - Complete)**

This file contains the main entry point for your lab application. It presents a simple menu-driven interface for the
user, allowing them to select between running the basic producer-consumer demo, running multiple producer-consumer
pairs, or exiting the program.

The file handles user input, calls functions implemented in process_manager.c, and displays results and status messages.
This file is fully implemented and requires no modifications.

**process_manager.h (PROVIDED)**

This header file contains function prototypes for the process management operations implemented in process_manager.c. It
includes all necessary system headers for process operations, function prototypes for process creation and management,
constants used throughout the lab, and helper function declarations.

No changes are necessary for this file.

**process_manager.c (TEMPLATE - TO COMPLETE)**

This file contains the core logic for your process management operations. You will implement process creation using
fork(), pipe creation and management for inter-process communication, parent process coordination and cleanup using
wait(), and multiple process pair management.

The main functions are provided with TODO comments indicating where you must fill in the missing code. Follow the
instructions and hints to complete the required functionality using system calls.

#### Project Setup

1. Download the three files
2. Complete the 6 TODOs in process_manager.c
3. Compile: `gcc -o lab3 main.c process_manager.c`
4. Run: `./lab3`

#### The Functions You'll Implement

#### Function 1: run_basic_demo() - Understanding Basic Process Creation

This function demonstrates the fundamental process creation pattern used throughout operating systems. Think of it like
a factory manager who needs to create two workers - one to produce items and another to package them, with a conveyor
belt between them for communication.

The function teaches you how fork() creates a copy of the current process, how parent and child processes can execute
different code paths, how pipe() enables communication between related processes, and how wait() ensures proper process
cleanup.

Here's what happens step by step:

1. Parent process creates a pipe for communication
2. Parent forks a producer child that generates numbers 1-5
3. Parent forks a consumer child that receives and adds the numbers
4. Parent waits for both children to complete their work
5. All processes terminate cleanly

#### Function 2: run_multiple_pairs() - Scaling Process Management

This function demonstrates how operating systems manage multiple concurrent processes efficiently. It's like a factory
manager creating multiple production lines, each with their own producer-consumer pair and conveyor belt, all running
simultaneously.

The function teaches you how to create and coordinate multiple process pairs, how each pair operates independently with
its own communication channel, how the parent process tracks and manages multiple children, and how proper resource
cleanup prevents system resource leaks.

Here's what happens:

1. Parent creates multiple independent producer-consumer pairs
2. Each pair gets its own pipe and sequential number range (1-5, 6-10, etc.)
3. All pairs run concurrently, demonstrating process parallelism
4. Parent tracks all child processes and waits for each to complete
5. Resources are properly cleaned up to prevent zombie processes

#### Expected Output

**Option 1: Basic Producer-Consumer Demo**

```
Starting basic producer-consumer demonstration...

Parent process (PID: 15234) creating children...
Created producer child (PID: 15235)
Created consumer child (PID: 15236)

Producer (PID: 15235) starting...
Consumer (PID: 15236) starting...

Producer: Sent number 1
Producer: Sent number 2
Producer: Sent number 3
Producer: Sent number 4
Producer: Sent number 5
Producer: Finished sending 5 numbers

Consumer: Received 1, running sum: 1
Consumer: Received 2, running sum: 3
Consumer: Received 3, running sum: 6
Consumer: Received 4, running sum: 10
Consumer: Received 5, running sum: 15
Consumer: Final sum: 15

Producer child (PID: 15235) exited with status 0
Consumer child (PID: 15236) exited with status 0

SUCCESS: Basic producer-consumer completed!
```

**Option 2: Multiple Producer-Consumer Pairs**

```
Running multiple producer-consumer pairs...

Parent creating 2 producer-consumer pairs...

=== Pair 1 ===
Producer (PID: 15240) starting...
Consumer (PID: 15241) starting...
Producer: Sent number 1
Consumer: Received 1, running sum: 1
[... continues with numbers 2-5 ...]
Consumer: Final sum: 15

=== Pair 2 ===
Producer (PID: 15242) starting...
Consumer (PID: 15243) starting...
Producer: Sent number 6
Consumer: Received 6, running sum: 6
[... continues with numbers 7-10 ...]
Consumer: Final sum: 40

All pairs completed successfully!
Child (PID: 15240) exited with status 0
Child (PID: 15241) exited with status 0
Child (PID: 15242) exited with status 0
Child (PID: 15243) exited with status 0

SUCCESS: Multiple pairs completed!
```

#### Key System Calls You'll Use

#### fork() - Process Creation

The fork() system call creates a new process by copying the current process. It's one of the most important system calls
in Unix-like operating systems. When you call fork(), the operating system creates an identical copy of your process,
including all variables, open files, and program state.

The magic happens in the return value. In the parent process, fork() returns the process ID (PID) of the newly created
child. In the child process, fork() returns 0. If something goes wrong, fork() returns -1.

Here's the typical pattern:

```c
pid_t pid = fork();
if (pid == 0) {
    // Child process code
    child_function();
    exit(0);
} else if (pid > 0) {
    // Parent process code
    printf("Created child (PID: %d)\n", pid);
} else {
    perror("fork failed");
}
```

#### pipe() - Inter-Process Communication

The pipe() system call creates a communication channel between processes. Think of it as creating a one-way tube where
one process can write data and another can read it. The pipe() call creates two file descriptors: pipe_fd[0] for reading
and pipe_fd[1] for writing.

The key rule with pipes is that you must close the unused ends. If the producer doesn't close the read end, or the
consumer doesn't close the write end, your processes might hang waiting for data that will never come.

Here's how to use pipes:

```c
int pipe_fd[2];
if (pipe(pipe_fd) == -1) {
    perror("pipe failed");
    return -1;
}

// Producer (writer)
close(pipe_fd[0]); // Close read end
write(pipe_fd[1], &data, sizeof(data));
close(pipe_fd[1]);

// Consumer (reader)
close(pipe_fd[1]); // Close write end
read(pipe_fd[0], &data, sizeof(data));
close(pipe_fd[0]);
```

#### wait() and waitpid() - Process Cleanup

When a child process finishes, it doesn't immediately disappear. Instead, it becomes a "zombie" process that hangs
around until its parent calls wait() to collect its exit status. If the parent never calls wait(), these zombie
processes accumulate and can eventually exhaust system resources.

The wait() system call makes the parent process pause until one of its children exits. The waitpid() variant lets you
wait for a specific child process. Both functions return the PID of the child that exited and provide its exit status.

Here's the pattern:

```c
int status;
pid_t child_pid = waitpid(specific_pid, &status, 0);
printf("Child %d exited with status %d\n", child_pid, status);
```

#### Implementation Guide

#### TODO 1: Create Pipe for Communication

You need to create a pipe that will allow the producer and consumer processes to communicate. Remember to check for errors when creating the pipe, as system calls can fail.

**Hint:** Use the pipe() system call to create an array of two file descriptors. Don't forget to handle the case where pipe creation fails.

#### TODO 2: Fork Producer Process

Create a child process that will act as the producer. The child process should close the appropriate pipe end and call the producer function, while the parent should store the child's PID and continue.

**Hint:** Use fork() and check its return value. Child processes (return value 0) should become producers, parent processes (positive return value) should continue, and errors (negative return value) should be handled appropriately.

#### TODO 3: Fork Consumer Process

Create another child process that will act as the consumer. Similar to the producer, the child should close the appropriate pipe end and call the consumer function.

**Hint:** This follows the same pattern as the producer fork, but the child calls the consumer function instead. Make sure to close the correct pipe end for reading vs writing.

#### TODO 4: Parent Cleanup and Wait

The parent process needs to clean up its resources and wait for both child processes to complete. This prevents zombie processes and ensures proper resource management.

**Hint:** Close both pipe ends in the parent (since it doesn't use them), then use waitpid() to wait for each child process individually. Print the exit status of each child.

#### TODO 5: Multiple Pairs Loop Structure

For the multiple pairs function, you'll need to create several producer-consumer pairs in a loop. Each pair needs its own pipe and should use different number ranges.

**Hint:** Use a loop to create multiple pairs, where each iteration creates a new pipe and forks two processes. Keep track of all child PIDs so you can wait for them later.

#### TODO 6: Multiple Pairs Cleanup

Wait for all child processes from all pairs to complete. This ensures that no zombie processes are left behind.

**Hint:** You'll need to wait for each child process that was created. Consider using an array or list to track all the child PIDs from both producers and consumers.

#### Why These Concepts Matter

Process creation through fork() is how every program you run gets started. When you type a command in your terminal, the
shell forks a new process to run that command. Web servers fork to handle multiple clients simultaneously. Database
systems fork to process queries in parallel.

Inter-process communication through pipes is equally fundamental. When you type `ls | grep pattern` in your terminal,
the shell creates a pipe between the ls and grep processes. System services use pipes to communicate with each other.
Data processing pipelines move information between different stages using pipes.

Process management through wait() keeps the system running smoothly. Without proper cleanup, systems would be filled
with zombie processes consuming memory and process table entries. Every well-behaved parent process must wait for its
children.

#### Common Mistakes to Avoid

Forgetting to close pipe ends is probably the most common mistake. If you don't close the unused ends of a pipe,
processes will hang waiting for data that never comes. The rule is simple: close the read end in the writer process, and
close the write end in the reader process.

Not calling wait() creates zombie processes. These are processes that have finished running but haven't been cleaned up
by their parent. They consume system resources and can eventually cause problems.

Getting the fork() logic wrong can create infinite loops of process creation that can crash your system. Always check
the return value of fork() and handle all three cases: child (0), parent (positive PID), and error (-1).

Confusing pipe directions is another common issue. Remember that pipe_fd[0] is for reading and pipe_fd[1] is for
writing. Don't try to read from the write end or write to the read end.

#### Analysis Questions

After completing the lab, think about these questions:

1. What happens to memory when fork() creates a new process? Does the child get a complete copy of the parent's memory?
    - The child receives a copy-on-write duplicate of the parent's  virtual address space: pages are shared read-only until either side writes, which triggers a page-fault and private copy. Open file descriptors are duplicated to the same open file descriptions, so offsets are shared.
2. Why must unused pipe ends be closed? What would happen if you forgot to close them?
    - A reader sees EOF only after all writers ends of that pipe are closed. If any writer fd open, read() may block indefinitely. Writing when no readers exist causes sigpipe, and write() returns -1 with epipe. Close  pipefd[0] in the writer, pipefd[1] in the reader, and close both in the parents
3. What are zombie processes, and how does wait() prevent them? What would happen to your system if zombie processes
   accumulated?
    - A zombie (state Z) has exited, but its termination status hasn’t been collected; most resources are freed, but the process table entry persists. The parent must call wait()/waitpid() to reap it; otherwise, zombies can accumulate and exhaust PID/process-table capacity.
4. How does the kernel track parent-child relationships? What happens if a parent process dies before its children?
   - The kernel records each process’s PPID to maintain lineage. If a parent exits first, remaining children are reparented (typically to PID 1 or a subreaper like systemd), which later reaps them when they terminate.

#### What You're Really Learning

The techniques you're practicing in this lab are the building blocks of modern computing. Web servers use these patterns
to create worker processes that handle client requests. Database systems fork processes to handle concurrent queries.
Operating system shells use fork and pipes to run commands and create pipelines.

Even though you're just adding numbers, you're learning the fundamental process management patterns that power
everything from simple command-line tools to complex distributed systems. Every time you open a web browser, start a
program, or run a command, these same system calls are working behind the scenes.

The number-adding is just busy work to give your processes something to do. The real learning is understanding how
fork() creates processes, how pipes let them communicate, and how wait() keeps everything running smoothly.

#### Compilation and Execution

```bash
gcc -o lab3 main.c process_manager.c
./lab3
```

#### Submission Requirements

After completing your work:

```
git add .
git commit -m "completed lab 3 - basic process management and IPC"
git push origin main
```

Include:

- Complete the 6 TODOs in process_manager.c
- Screenshots showing program output
- Test that both menu options work correctly
- Verify no zombie processes remain (all children properly waited for)
- Ensure predictable output with sequential numbers
