## FlexSC: Flexible System Call Scheduling with Exception-Less System Calls

### Problem:
- Evaluate the performance impact of synchronous system calls (and reason).
- Enabling flexibility of scheduling system calls.

### Solution
1. Sign: Instructions Per Cycle drops after a system call. (system call cause lack of locality)
	- **Mode Switch Cost**: 
		- implement a new system call to measure time of *solely mode switch*
		- *processer state pollution* (flushing of L1, L2, TLB...): use HPC handler to invoke system call so that we record the *system call footprint*.
			- system calls significantly pollute the processor structures.
	- **Impact on User Application**: measure time ratio of executing system call every scale of instructions.

> - **hardware performance counter**: special registers built in processor that store the counts of hardware activities like cache miss or branch misprediction.
> - **perf**: performance analyzing tool in Linux. (To be learned)
> 

> - **Mode Switch**:
> 	1. set `sstatus`, `sepc` , `scause/stval`
> 	2. jump to handler at `stvec`
> 	3. save context: registers, and change stack (User's to Kernel's)
> 	4. go to dispatcher and then handler

2. [Main Contribution] **Exception-Less System Calls**
	- **Syscall Pages** as syscall interface.
		- user find a *free* entry in pages, and *submit* syscall request.
		- check until *done*.
	- **Syscall Thread** to execute syscall request.
		- syscall thread can be blocked, but should never sleep. (By create new syscall thread if needed)
		- in a multi-core scenery, a *syscall thread* locks the *syscall page* until all the submitted entries are issued. (To avoid cache line contention)
		- **syscall thread scheduler**

> \*This paper spawn one *syscall thread* for each entry in *syscall page*. (should be different to micro-kernel implementation)
> \*This paper needs user to wait explicitly for the syscall to be *done*.

3. [Implementation] FlexSC-Threads
	- Goal: transforming legacy synchronous syscalls into exception-less syscalls
	- Solution: 
		- redirect *libc*.
		- execute the exception-less syscall by blocking.

> \*FlexSC-Threads map each *syscall page* a *syscall thread* (should be same as micro-kernel implementation)
