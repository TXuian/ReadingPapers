## On Micro-Kernel Construction

### Problem: 
- Concepts of micro-kernel;
- Facts that affect performance; 

### Solution:
1. Micro-Kernel **Functional** Concepts
	- *Address Space*
		- *Grant*; *Map*; *Flush*
		- Introducing ownership of users (procs) to memories.
	> - in my view, kernel *grant*/*map* ownership to user, so that user can use this memory freely. (In this way allow Memory management happened outside kernel)
	> - kernel, or owner user can *flush* so to retrieve privilege *mapped*
	
	- *IPC*: implementation varies.
	- *Interrupts*: use *IPC* to handle *interrupts*
	- *Unique Identifiers*
  
2. Things outside kernel  
	- *Pager*: *pager* is integrated into *memory manager*.   
	- *Memory manager*:  other procs ask *manager* for memory, whose ownership is *granted*/*mapped* by kernel. 
	- *Resource Allocation*
	- *Device driver*: there is nothing special about a driver, when comparing to other procs.
	- *Second Level Cache and TLB*
	- *Remote communication*: based on *IPC*
	- *Unix Server*

3. **Performance (Experiments)**
	
	- **Switch Overhead**
		- Kernel-User: test costs (CPU cycles) of kernel-user switch:
			- less switch the better.
		- Address space: cost due to TLB flushing.
		- Thread Switch and IPC: test ping-pong (time and cycles): 
		- In micro-kernel, IPC is actually implementation of RPC. 
	
	- Memory Effect: test MCPI (Memory Cycle overhead Per Instruction) for different apps:
		- differences essentially caused by cache misses.
	- micro-kernel requires for more cache working sets.
	
4. Potability (scanned and ignored)
5. Other kernels (scanned and ignored)