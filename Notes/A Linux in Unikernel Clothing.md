## A Linux in Unikernel Clothing

### Purpose

- 用Unikernel的技巧改造Linux，使其变成类Unikernel：在对应场景下拥有Unikernel的性能。

### Design

- **Specialization**：
	- 只包含必要的代码模块：unikernel只包含特定应用程序所需的代码模块，这可以使unikernel的图像大小更小，内存占用更低，攻击面更小。
	- 平衡专业化和兼容性：在POSIX-like的unikernels里，系统需要平衡专业化程度与兼容性和遗留代码的重用之间，通常会提供至少粗粒度的Specialization。

- **System Call Overhead Elimination**：
	- unikernel通常在同一CPU特权域和同一地址空间中运行所有代码，以提高传统系统的性能。无需在应用程序和内核函数之间切换上下文。

### Implementation

- 手动分析Linux Kernel的Kconfig，结合Application对应menifest为应用构建镜像。
- 使用KML。

> - KML(Kernel Mode Linux)：将用户进程运行在内核模式；
> 	- 为了将用户进程作为内核模式用户进程执行，KML唯一需要做的就是将该进程的CS寄存器设置为内核代码段，而不是用户代码段，然后该进程就在内核模式下执行。
> 	- The Stack Starvation
> 		- *Problem I*：如果程序在用户模式下执行，IA-32 CPU会自动将其内存堆栈切换到内核堆栈。然后，它将执行上下文（EIP、CS、EFLAGS、ESP和SS寄存器）保存到内核堆栈中。另一方面，如果程序在内核模式下执行，IA-32 CPU不会切换其内存堆栈，而是将上下文（EIP、CS和EFLAGS寄存器）保存到正在运行程序的内存堆栈中。
> 		- *Problem II*： 如果KML的内核模式用户进程访问其由CPU的页表未映射的内存堆栈会发生什么？首先，会发生页面故障，并且CPU会尝试中断该进程并跳转到IDT中指定的页面故障处理程序。但是，CPU无法完成此工作，因为没有堆栈保存执行上下文。由于该进程在内核模式下执行，CPU永远无法将内存堆栈切换到内核堆栈。为了标识这种致命的情况，CPU会尝试生成一个特殊的异常，即双重故障。同样，CPU无法生成双重故障，因为没有堆栈来保存正在运行进程的执行上下文。最终，CPU放弃并重置自身。
> 		- *Solution*：KML利用了IA-32 CPU的任务管理功能。IA-32任务管理功能提供支持内核进程管理的功能。使用此功能，内核可以使用一条指令在进程之间切换。由IA-32 CPU管理的任务可以设置为IDT。如果发生中断并且已分配任务来处理中断，则CPU首先将中断程序的执行上下文保存到程序的任务数据结构中，而不是保存到内存堆栈中。
> 		

![](C:\Users\k\Desktop\reference fold\Notes\imgs\A Linux in Unikernel Clothing\arch.png)


### Beyond Unikernel

- 思考的结论：
	- Unikernel应用程序（以及它们的开发者）通常限制了应用对多个进程、处理器、安全环和用户的需求。这些限制通常被宣传为一种特性，但在某些情况下需要放松这些限制（如需要支持多进程和多处理器）。

> 多处理器（SMP）支持实验：
> - 实验结论：实验结果表明，支持Linux的对称多处理（SMP）并不会对unikernel般的性能产生负面影响。尽管支持SMP会带来一定的开销，但在大多数情况下，使用SMP会更为优秀，而且开销也是可以接受的。
> - 实现目的：调查Linux的对称多处理（SMP）支持的效果。
> 	- 设计：设计三个实验，分别为sem_posix、futex和make-j，以展示支持SMP的最坏情况。其实验目的相同，只是测试方式不同。
> 		- 实验1（sem_posix），实验2（futex）：生成最多512个工作进程，快速练习futex和POSIX信号量等待和发布操作。每个工作进程启动4个共享futex或信号量的进程。
> 		- 实验3（make-j）：使用最多512个并发进程构建Linux内核。
> 	- 流程：分别在SMP支持和不支持的内核下运行上述实验，比较它们的结果。
> 	- 实验结果：实验结果表明，支持SMP会带来一定的开销，但在大多数情况下，使用SMP会更为优秀。
> 		- sem_posix实验产生多达3％的开销；
> 		- futex实验产生多达8％的开销；
> 		- make实验产生多达3％的开销，超过不支持SMP支持的内核。