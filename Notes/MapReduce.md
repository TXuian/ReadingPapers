## MapReduce

### Problem
- [Purpose] Process and generate large data sets.

### Solution
- [Program Model] `map(k, v)` and `reduce(k, v)` : Program Model that fits in real world needs
	- `map(k, v)`:
		- written by user,
		- process (distributed stored) large data sets and generate **intermediate key/value pairs**.
	- `reduce(k, v)`:
		- written by user, 
		- accept *intermediate key/value pairs*, and reduce them to generate wanted key/value pairs.

> One example: 
> ```
> map   (k1, v1)         --> list(k2, v2)
> reduce(k2, list(v2))   --> list(v2)
> ```
> 

#### [Main Contribution] Implementation in GFS

- [Implementation] Execution Model:
	- **Job**: one *MapReduce* procedure;
	- **Task**: *job* is split into *tasks* and tasks are assigned to workers.  
![](C:\Users\k\Desktop\reference fold\Notes\imgs\MapReduce\execution_overview.png)

- Details of Implementation: 
	- *Master Data Structure*: 
		- *Task* status: idle, in-progress, or completed;
		- *Intermediate k/v locations*: locations and size of intermediate k/v that should be forwarding to reduce workers.
	- *Fault Tolerance*: 
		- Worker Failure: master pings every workers periodically, if worker has failed:
			- All completed *map task* is reassigned,
			- All in-progress *map task* and *reduce task* is reassigned.
		- Master Failure: abort MapReduce and inform User (Failed Job).
	- *Intermediate Files*:
		- *Map tasks* writes its output to private files, and inform the private file name to *master*
		- *Reduce tasks* write its output to temporary file and then atomically rename the temporary file to final output file. 
	- *Locality*: allocate *Map task* to the *host that contains the input file* as possible.

#### Extensions and Performance
[Ignored] ...

