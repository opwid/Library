
The virtual address space of the processor is general divided into two parts by the Linux kernel. Lower part is for user processes, and upper part is reserved for kernel. 
* UMA (uniform memory access) machines organize available memory in a contigiuous fashion. Each processor (in a symmetric  multiprocessor system) is able to access each memory area equally quickly.
* NUMA machines (non-uniform memory access) are always multiprocessor machines. Local RAM is available to each CPU of the system to support particularly fast access. The processors are linked via a bus to support access to the local RAM of other CPUs — this is naturally slower than accessing local RAM.
