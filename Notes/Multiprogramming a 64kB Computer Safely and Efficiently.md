## Multiprogramming a 64kB Computer Safely and Efficiently

### Problem
- **[Resources]** Low power microcontrollers have only subsets of modern hardware features:
	- MPU, rather than MMU,
	- less RAM and Flash,
	- limited CPU clock.
- **[Requirements]** Newly emerged software platforms require:
	- Dependency: stable enough that can avoid breakdown of the system.
	- Concurrency: in order to retrieve energy efficiency, OS needs to maximize low-power sleep state.
	- Efficiency: maximize the usage of given resources.
	- Fault Isolation: breakdown of one process will not affect other processes.
- Satisfy requirements on limited resources.

### Solution
- [Main Contribution] Drivers into **Capsules** (drivers belong to kernel)
	- [Idea]\[Implementation\] *Capsule Isolation* relies on Rust Type and Module system.
		- (efficient) Rust type system has no runtime overhead,
		- (safe) Rust type and module system provides memory safety and access safety,
	- [Idea]\[Concurrency\] *Event-Driven Scheduler for Capsules*:
		- Capsule works because *events* (interrupts caused by hardware or syscalls),
> - Hardware function that may cause bugs (like DMA) also needs to wrap into capsules. 
> 

- **Process** 
	- [Fault Isolation]\[Loadable\] Memory region is a continuous fixed size region allocated by kernel.

- [Main Contribution] **Grant**: Shared memory between kernel and processes.
	- [Purpose] Protect kernel state from processes.
	- [Idea] *Grant* is memory space in process but can only accessed by kernel (or capsules).
	- [Implementation] Using (1) Liveness and Closure (restrict the liveness inside closure)  in Rust, (2) MPU (MPU rules the memory region where a process can access). 
		- Capsule is responsible to run closures for each process when an event comes.

> - Dependency: Grant provide safety and avoid kernel heap exhaustion.
> - Concurrency: (1) Cooperative scheduler for capsules and (2) preemptive scheduler for processes.
> - Efficiency: Grant cost no runtime overhead and saves memory.
> - Fault Isolation: (1) Independent memory region of processes and (2) Isolated capsules supported by Rust.

### Evaluation
#### *Capsule Isolation Overhead*
- [Problem] overhead of capsule isolation compare to ideal monolithic implementation.
	- [Design] micro-benchmarks:
		- (1)  extra space: dependency references in each capsules (minor),
		- (2) extra time: major in no-sleep mode, minor in deep-sleep mode.
	- [Design] macro-benchmarks: log memory footprint while running (1) a "blink" app and (2) an 802.15.4 radio,
		- Tradeoff: not a significant memory overhead.
- [Problem] overhead of capsule isolation compare to process isolation
	- [Design] Memory: compare the needed regions to retrieve isolation,
		- Capsule: dependency references in each capsule-a few bytes,
		- Process: separate memory regions, stacks-a lot.
	- [Design] Time: Compare the CPU cycles,
		- Capsule: function calls, about 25 cycles,
		- Process: context switch, about 340 cycles.

#### *Overhead of Grant*
- [Problem] Memory overhead of *Grant* 
	- [Design] Analyze the extra space,
		- The wrapper needed for rust to protect memory: several bytes.
	- [Design] Comparing the memory occupation to the alternative-the pre-allocation, 
		- Alternative strategy wastes a lot.
- [Problem] Time overhead for capsules to iteratively run closures for each process, since all processes are needed to check if a relative grant has subscribed.
	- [Design] An extra reference to next expected grant in current grant to eliminate this overhead.
> - Algorithm about how this is achieved is no mentioned. 