## A fork() in the road

### Purpose
- Argue that fork() is a terrible abstraction in modern days.

### Standpoints
- [History] History of fork():
	1. Project Genie time-sharing system: the child shares the parent's address space or be given a new but different (no facility to copy address space).
	2. Unix fork(): easy to realize.

- [Advantages] Advantages of fork():
	- Simple: fork needs no arguments while assigning address space needs a lot of parameters.
	- Orthogonal: space between fork() and exec() could be useful (e.g. kernel state could be reused in child before it's calling of exec()).
	- Concurrency: One configuration in parent with different running children.

- [Disadvantages] 