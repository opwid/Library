# Process Priorities
Most processes are normal processes that have no specific time constraints but can still be classified as more important or less important by assigning priorities to them.  

In this scheme, known as preemptive multitasking, each process is allocated a certain time period during which it may execute. Once this period has expired, the kernel withdraws control from the process and lets a different process run — regardless of the last task performed by the previous process. Its runtime environment — essentially, the contents of all CPU registers and the page tables — is, of course, saved so that results are not lost and the process environment is fully reinstated when its turn comes around again. The length of the time slice varies depending on the importance of the process (and therefore on the priority assigned to it).

# Process Life Cycle

The system saves all processes in a process table — regardless of whether they are running, sleeping, or waiting. However, sleeping processes are specially ‘‘marked‘‘ so that the scheduler knows they are not ready to run. There are also a number of queues that group sleeping processes so that they can be woken at a suitable time — when, for example, an external event that the process has been waiting for takes place.  

__Running__ — The process is executing at the moment.  
__Waiting__ — The process is able to run but is not allowed to because the CPU is allocated to another process. The scheduler can select the process, if it wants to, at the next task switch.  
__Sleeping__ — The process is sleeping and cannot run because it is waiting for an external event. The scheduler cannot select the process at the next task switch.  

A special process state not listed above is the "__zombie__"state. As the name suggests, such processes are defunct but are somehow still alive. In reality, they are dead because their resources (RAM, connections to peripherals, etc.) have already been released so that they cannot and never will run again. However, they are still alive because there are still entries for them in the process table.

# Preemptive Mutlitasking
The structure of Linux process management requires two further process state options — user mode and kernel mode.  

If a process wants to access system data or functions (the latter manage the resources shared between all processes, e.g., filesystem space), it must switch to kernel mode. Obviously, this is possible only under control — otherwise all established protection mechanisms would be superfluous — and via clearly defined paths. Chapter 1 mentioned briefly that "system calls" are one way to switch between modes. A second way of switching from user mode to kernel mode is by means of interrupts — switching is then triggered automatically. Unlike system calls, which are invoked intentionally by user applications, interrupts occur more or less arbitrarily.  

One option known as kernel preemption was added to the kernel during the development of kernel 2.5. This option supports switches to another process, if this is urgently required, even during the execution of system calls in kernel mode (but not during interrupts). Although the kernel attempts to execute system calls as quickly as possible, the time needed may be too long for some applications that are reliant on constant data streams. Kernel preemption can reduce such wait times and thus ensure "smoother" program execution. However, this is at the expense of increased kernel complexity because many data structures then need to be protected against concurrent access even on single-processor systems. This technique is discussed in Section 2.8.3.

# Process Representation
All algorithms of the Linux kernel concerned with processes and programs are built around a data structure named task_struct and defined in include/sched.h.

http://lxr.linux.no/#linux+v4.10.1/include/linux/sched.h


Linux provides the resource limit (rlimit) mechanism to impose certain system resource usage limits on processes. The mechanism makes use of the rlim array in task_struct , whose elements are of the struct rlimit type.  

* rlim_cur is the current resource limit for the process. It is also referred to as the soft limit.
* rlim_max is the maximum allowed value for the limit. It is therefore also referred to as the hard limit.

The setrlimit system call is used to increase or decrease the current limit. However, the value specified in rlim_max may not be exceeded. getrlimits is used to check the current limit.  

_cat /proc/self/limits_

