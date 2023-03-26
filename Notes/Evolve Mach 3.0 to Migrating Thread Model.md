## Evolve Mach 3.0 to Migrating Thread Model

- [Problem] Apply **Migrating thread model** on Mach (for better performance).

#### [Solution(Thought)]

- **Split thread in Mach into two parts:**	
	1. <u>the logical flow of control</u>: *activation* stack;
		- **activation**: *execution context* and *control structs* of task.
	2. <u>the schedulable entity</u>: priority, resource counting... (attributes related to schedule). 

- **Thread in based in kernel (not bounded to user task)**
	- First part of thread migrates through activations (using *activation stack*).
	- Second part of thread enters/leaves the kernel.
	- **Glue code** refers to code that migrate threads.