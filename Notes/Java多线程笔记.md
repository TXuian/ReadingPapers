## 多线程

- ==用户线程==：从开发者的角度来理解用户级线程就是说：在这种模型下，我们需要自己定义*线程的数据结构、创建、销毁、调度和维护等*，这些线程运行在操作系统的某个进程内，然后操作系统直接对进程进行调度。

- ==内核线程==：从开发者的角度来理解内核级线程就是说：我们可以直接使用操作系统中已经内置好的线程，线程的创建、销毁、调度和维护等，都是直接由操作系统的内核来实现，我们只需要使用系统调用就好了，不需要像用户级线程那样自己设计线程调度等。

![](C:\Users\k\Desktop\reference fold\Notes\imgs\Java多线程笔记\Java多线程笔记.png)