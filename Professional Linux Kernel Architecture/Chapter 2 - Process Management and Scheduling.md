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

## Process Types

fork generates an identical copy of the current process; this copy is known as a child process. All resources of the original process are copied in a suitable way so that after the system call there are two independent instances of the original process. These instances are not linked in any way but have, for example, the same set of open files, the same working directory, the same data in memory (each with its own copy of the data), and so on.  

exec replaces a running process with another application loaded from an executable binary file. In other words, a new program is loaded. Because exec does not create a new process, an old program must first be duplicated using fork, and then exec must be called to generate an additional application on the system.  

Linux also provides the clone system call in addition to the two calls above that are available in all Unix flavors and date back to very early days. In principle, clone works in the same way as fork , but the new process is not independent of its parent process and can share some resources with it. It is possible to specify which resources are to be shared and which are to be copied — for example, data in memory, open files, or the installed signal handlers of the parent process. clone is used to implement threads. However, the system call alone is not enough to do this. Libraries are also needed in userspace to complete implementation. Examples of such libraries are Linuxthreads and Next Generation Posix Threads.

## Namespaces
Namespaces provide a lightweight form of virtualization by allowing us to view the global properties of a running system under different aspects.  

Notice that support for namespaces in a simple form has been available in Linux for quite a long time in the form of the chroot system call. This method allows for restricting processes to a certain part of the filesystem and is thus a simple namespace mechanism. True namespaces do, however, allow for controlling much more than just the view on the filesystem.  

New namespaces can be established in two ways:  
* When a new process is created with the fork or clone system call, specific options control if namespaces will be shared with the parent process, or if new namespaces are created.
* The unshare system call dissociates parts of a process from the parent, and this also includes namespaces. See the manual page unshare(2) for more information.  

Once a process has been disconnected from the parent namespace using any of the two mechanisms above, changing a — from its point of view — global property will not propagate into the parent namespace, and neither will a change on the parent side propagate into the child, at least for simple quantities.


