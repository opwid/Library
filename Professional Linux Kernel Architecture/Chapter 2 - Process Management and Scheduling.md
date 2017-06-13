# Process Priorities
Most processes are normal processes that have no specific time constraints but can still be classified as more important or less important by assigning priorities to them.  

In this scheme, known as preemptive multitasking, each process is allocated a certain time period during which it may execute. Once this period has expired, the kernel withdraws control from the process and lets a different process run — regardless of the last task performed by the previous process. Its runtime environment — essentially, the contents of all CPU registers and the page tables — is, of course, saved so that results are not lost and the process environment is fully reinstated when its turn comes around again. The length of the time slice varies depending on the importance of the process (and therefore on the priority assigned to it).

# Process Life Cycle

The system saves all processes in a process table — regardless of whether they are running, sleeping, or waiting. However, sleeping processes are specially ‘‘marked‘‘ so that the scheduler knows they are not ready to run. There are also a number of queues that group sleeping processes so that they can be woken at a suitable time — when, for example, an external event that the process has been waiting for takes place.  

__Running__ — The process is executing at the moment.  
__Waiting__ — The process is able to run but is not allowed to because the CPU is allocated to another process. The scheduler can select the process, if it wants to, at the next task switch.  
__Sleeping__ — The process is sleeping and cannot run because it is waiting for an external event. The scheduler cannot select the process at the next task switch.  

A special process state not listed above is the "__zombie__"state. As the name suggests, such processes are defunct but are somehow still alive. In reality, they are dead because their resources (RAM, connections to peripherals, etc.) have already been released so that they cannot and never will run again. However, they are still alive because there are still entries for them in the process table.
