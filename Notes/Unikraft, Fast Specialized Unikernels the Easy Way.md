## Unikraft: Fast, Specialized Unikernels the Easy Way

### Work (Contribution)
> unikernel 存在两个主要缺点：
> - 它们需要重要的专家工作来构建和提取高性能；这项工作在很大程度上必须针对每个目标应用程序重新进行。
> - 它们通常不符合 POSIX 标准，需要移植应用程序和语言环境。


- 完全模块化了操作系统原语，使得易于自定义unikernel并仅包含相关的组件;
- 暴露了一组可组合的、面向性能的API，以使开发人员轻松获得高性能。
- Unikraft 的关键概念创新在于为核心操作系统组件定义了一小组 API，使得在不需要时易于替换出一个组件，并根据性能指示符从多个实现中选择和组合。

### Design and it's Principle
> - OS功能分析：
> 	- 在虚拟化环境中，应用程序和内核之间的保护域切换可能是多余的，因为隔离由超级管理程序保证，导致可测量的性能降低。
> 	- 在单个应用程序域中，多个地址空间可能是无用的，但从标准操作系统中删除此类支持需要进行大量的重新实现努力。
> 	- 对于RPC式服务器应用程序，线程不需要，单个运行到完成的事件循环足以实现高性能。这将消除VM中调度程序及其相关开销的需要，以及客户机和超级管理程序调度程序之间的不匹配[19]。
> 	- 对于性能导向的基于UDP的应用程序，大部分操作系统网络堆栈是无用的：应用程序可以简单地使用驱动程序API，就像DPDK样式的应用程序一样。目前没有办法轻松地从标准操作系统中仅删除网络堆栈而不删除整个网络子系统。
> 	- 应用程序直接访问NVMe存储器可以消除文件描述符、VFS层和文件系统的需要，但从现有的围绕存储器API层构建的操作系统中删除这种支持非常困难。
>	- 内存分配器对应用程序性能有很大影响，通用分配器已被证明对许多应用程序不是最优的[66]。因此，每个应用程序选择自己的分配器将是理想的；但在今天的操作系统中这是非常困难的，因为内核使用的分配器是固定的。

- 设计决策：
	- 单地址空间：针对单个应用程序场景，可能有不同的应用程序通过网络通信相互交流。
	- 完全模块化系统：所有组件，包括操作系统原语、驱动程序、平台代码和库，都应该易于根据需要添加和删除；甚至API也应该是模块化的。
	- 单保护级别：不应该有用户/核心空间分离，以避免昂贵的处理器模式切换。这并不排除分隔（例如微型库）的可能性，这可以以合理的成本实现[69]。
	- 静态链接：启用编译器功能，例如死代码消除（DCE）和链接时优化（LTO），以自动摆脱不需要的代码。
	- POSIX支持：为了支持现有或遗留应用程序和编程语言，同时仍然允许在该API下进行专业化。
	- 平台抽象：无缝生成各种不同虚拟机/虚拟机监控程序的镜像。

### Implementation

![](C:\Users\k\Desktop\reference fold\Notes\imgs\Unikraft\arch.png)


- **微库（micro-libraries）**是实现核心 Unikraft APIs 的软件组件；
	1. POSIX-compatibility layer，提供 POSIX 支持的组件。
	2. Standard socket interface，提供标准的套接字接口。
	3. Vfscore micro-library，提供磁盘访问支持的组件。
	4. Pluggable schedulers，可插拔调度程序。
	5. Boot code micro-libraries，提供引导代码的微库。
	6. Memory allocators，提供不同的内存分配器。
	7. Uknetdev API，提供较低级别、高性能的网络支持。
		- 将*网络设备驱动程序*和*网络堆栈*或底层网络应用程序解耦;
	8. Ukblock API，提供针对磁盘访问的高性能 API。
	- syscall-shim：针对musl库，将*系统调用*转化为函数调用。

> - uknetdev API:  
> ```c++
> int uk_netdev_tx_burst(struct uk_netdev *dev, 
> 						uint16_t queue_id, 
> 						struct uk_netbuf **pkt, 
> 						__u16 *cnt); // 要接收的包数
int uk_netdev_rx_burst(struct uk_netdev *dev, 
>						uint16_t queue_id, 
>						struct uk_netbuf **pkt, 
>						__u16 *cnt); // 接收到的包数
> ```
> 



- **构建系统（Build System）**负责编译、链接和生成可执行文件；


### Related Work

- LightVM - 一个小型unikernel项目，使用了定制的Xen工具栈来实现低启动时间。
- MirageOS - 一个OCaml特定的unikernel项目，专注于类型安全，但不支持主流应用程序。
- UniK - 一个unikernel项目，提供了一个包装器，支持多个其他unikernel项目。
- Containers and Kata Containers - 基于Linux的虚拟机（VM），定制化运行容器。
- Drawbridge - 一个Library OS，支持最新版本的Microsoft应用程序。
- Graphene - 一个Library OS，旨在高效执行单个和多进程应用程序。
- Rump - 引入了Anykernel概念，提供了良好的兼容性，但其依赖单体内核意味着没有太多的专业化空间。
- SEUSS - 使用Rump构建了一个作为服务（FaaS）系统。
- HermiTux - 通过syscall重写提供二进制兼容性，但不可定制。
- OSv - 一个专注于云计算的操作系统，与HermiTux一样，具有二进制兼容性支持，以及对许多编程语言和平台的支持。
- Sandstorm - 引入了一个极其（手动）定制的网络堆栈，以支持高性能Web和域名系统（DNS）服务器。
- null-Kernel - 提供了不同抽象层次的接口，允许应用程序/进程组合这些不同的接口。但并没有提供任何实现和应用程序兼容性的细节。
- Application-Specific Software Stacks - 实现了编译器扩展，以自动消除即使对于解释语言和共享库也是如此的代码，并且是Unikraft的补充。
- HALO - 一个自动后链接优化工具，分析内存分配方式以专业化内存管理例程，从而增加空间局部性。
- Demikernel - 一个新的Library OS，专门针对绕过内核的设备。
- IX - 一个新型操作系统，针对高吞吐量和低延迟。
- Arakis - 一个新的操作系统，其中应用程序直接访问虚拟化I/O设备，并透明地多路复用访问由多个进程共享的设备。
- Parakernel - 允许进程直接访问非共享设备，并透明地多路复用访问由多个进程共享的设备。
- MDev-NVMe - 实现了NVMe SSD的透传。
- User Mode Linux (UML) - 允许将内核作为用户空间进程运行。
- LibOS-nuse - 将内核网络堆栈作为共享库在用户空间中运行。
- Linux Kernel Library (LKL) - 将内核转换为可以作为虚拟机运行的库。
- Unikernel Linux（UKL）- 将用户空间应用程序静态编译到Linux内核中，使用一个shim层将glibc syscall重定向到内核函数（但为每个映像添加了45MB的glibc代码）。
- Lupine - 使用Linux内核在内核模式下加载应用程序二进制文件，从而消除系统调用开销；它还利用内核配置选项来丢弃不必要的特性，从而减小映像大小。