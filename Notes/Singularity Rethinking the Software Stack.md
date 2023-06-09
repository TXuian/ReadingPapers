## Singularity : Rethinking the Software Stack

### Kernel
#### ABI
> Singularity操作系统中的设计思路是以最小权限原则为基础，通过软件隔离而不是硬件保护来保障系统安全。

- SIPs提供了一个最小、安全、独立的计算环境，并通过通道（Channel）和能力（Capability）实现对更高级别系统服务的访问。
- ABI设计非常简单，其设计更倾向于显式语义，而不是最小的函数入口点。同时，版本控制机制确保向后兼容。
	- ABI调用保证了系统范围的状态隔离不变式，从而避免了进程之间的相互干扰和冲突。 
	- *状态隔离不变式*：ABI调用仅影响调用进程的状态，一个进程不能直接通过ABI函数修改另一个进程的状态，对象引用也不能通过ABI传递到内核或另一个SIP；
- 在保证安全的前提下，SIPs内部放置特权代码，句柄表提供了SIP代码命名内核中抽象对象的机制。

#### 内存控制
- 在大多数Sigularity操作系统中，内核和所有SIP都驻留在一个受软件隔离保护的单一地址空间中。
- 内核和每个SIP都有自己的栈，同时有一个共用的*交换栈*；
	- *交换栈* 用于存放channel中的数据。
> - 每个SIP同时只能只有一个指针指向交换栈；
> - 每个交换栈的对象只能被一个SIP指向和拥有；
> 

#### 线程
- 只有内核线程的概念（SIP运行的是内核线程），没有传统的用户线程概念。

### Sigularity的机制扩展

#### 辅以硬件保护

> - 操作系统通过CPU内存管理单元（MMU）硬件隔离进程，包括限制进程访问特定的物理内存页面和防止不受信任的代码执行特权指令。
> 	- 硬件隔离的主要成本包括页面表维护、软件TLB未命中、跨处理器的TLB维护、硬页异常和额外的缓存压力，同时也会增加调用内核和进程上下文切换的成本。
> 	

- Singularity系统通过**保护域**提供硬件隔离，其中每个保护域都有一个不同的虚拟地址空间，由处理器的MMU强制内存隔离。
	- 每个保护域都有一个交换堆，用于域内SIP之间的通信。
	- 硬件和软件隔离之间需要进行权衡，硬件保护的成本较高，而软件隔离的运行时成本较低。