## Exokernel: An Operating  System Architecture for Application-Level Resource Management

### Problem
- [Inconvenient Design] Traditional OS Architecture: fixed abstractions presented to applications.
	- Three main disadvantages: 
		- It denies applications of domain-specific optimizations.
		- It discourages changes of implementations of existing abstractions.
		- It restricts to build new abstractions.

- [Goal] Bring a new design of OS Architecture to allow *flexible abstraction design and implementation*.

### Solution
- [Principle] Divide low-level (Protection) and high-level (Management):
	- **Exokernel** implements *low-level primitives* in kernel level, which multiplexes and exports physical resources.
	- **Library OS** uses low-level exokernel interfaces to build *high-level abstractions* in user-level, which can be linked by applications.
- [Theory Foundation] The lower the level of primitives, the more efficiently it can be implemented and the more latitude it grants to implementors of higher level abstractions. 

#### [Main Contribution] Exokernel Design
- [Purpose] (1) Provide Library OS with physical resources; (2) Protect Library OS from each other.
	- [Idea] *secure  bindings*, *visible revocation*, *abort protocol*.
	
- [Principle] Lazy Management of resources: Provide only protection (use Ownership).
	- (1) Expose Allocation: Exokernel allows *Library OS* to request specific physical resources, but should not allocate implicitly.
	- (2) Expose Names: Exokernel should allow *Library OS* to see physical names.
	- (3) Expose Revocation: Allows *Library OS* to relinquish specific physical resource instance using physical name.
	- (Ex) Exokernel is responsible to handle resource competing between *Library OS*. 

> - Physical Name: name that refer to a physical resources, like *page number*, so to access a physical resource directly.
> 

- [Main Contribution] **Secure Binding**
	- [Idea] Bind the primitives of resources to libOS (bind time), and allows the primitives to run multiple times (run time).
	- [Implementation] The *"primitives"* is implemented based on three technics:
		- *Hardware Mechanisms*: Hardware will perform the checking.
		- *Software Caching*: Caches the ownership and capabilities of resources in kernel. 
		- *Downloading Application Code*: Download code to kernel so that kernel performs application code to handle kernel events. 

- **Visible Revocation**
	- [Idea] (1): Resources are referred by *Physical Name*; (2): libOS knows when *exokernel* withdraw the ownership of one resource granted to it (by perform *revocation request* from exokernel to libOS).

- **Abort Protocol**
	- [Idea] Exokernel should capable of complete *revocation request* by force. (In the mean while record the fored loss of a resource so that libOS can know)
		- May happen when a *revocation request* is not responded. 

#### [Main Contribution] Implementation(Prototype)

- [Exokernel] Aegis: exports *processor*, *physical memory*, *TLB*, *exceptions* and *interrupts*.
	- Processor into time slices (as a linear vector): element in vector is time slice, which can treat as a resource. 
	- 

- [Library OS] ExOS: implements *processes*, *virtual memory*, *user-level exceptions*, *interprocess abstractions* and several *network protocols*.