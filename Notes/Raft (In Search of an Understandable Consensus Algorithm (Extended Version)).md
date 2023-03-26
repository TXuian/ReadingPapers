## Raft (In Search of an Understandable Consensus Algorithm (Extended Version))

### Problem
- [Failure Tolerant] *Consensus Algorithm* that allows *large-scale systems* to survive after *machine failures* .
	- [Replicated State Machine] Technic that allows a group of machines to maintain same state while running.

- Need an algorithm to achieve *Replicated State Machine*.

### Solution
#### [Main Contribution] Raft
- [Goals] Design principles
	- It should provide a complete and practical foundation for system building. (Raft as a library)
	- It must be safe under all conditions and available under typical operating conditions.
	- It must be Efficient. (Log replication)
	- Understandable:
		- Decomposition problems: [Leader elections|Log replication|Membership changing].
		- Simplify states ad possible. (Keep the definition of state simple)

> - **Split Brain** means there are more than one *Primaries* providing services at one time.
>

- **[Leader Election]** Idea to solve **Split Brain**
	- 