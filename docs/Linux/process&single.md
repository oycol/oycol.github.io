## linux 抢占式多任务处理

cpu衡量单位是时间，cpu百分比不是工作频率，而是进程占用时间的百分比

子进程创建的三种方式：

- fork 进程复制
- exec 加载另外一个程序，替代当前进程
- clone  用于线程实现，和父进程共享资源



### 进程状态

就绪态

运行态

阻塞态

转换流程：

- 新 -> 就绪态：  等待队列非满时，内核将进程移入到等待队列

- 就绪态 -> 运行态：调度类选中等待队列中的进程，进程进入运行态

- 运行态 -> 睡眠态： 正在运行的程序需要等待io事件或者信号无法进行下一步，进入睡眠态

- 睡眠态 -> 就绪态： 等待事件完成，进程进入等待队列，等待下次被执行

    睡眠态分为可中断睡眠和不可中断睡眠。可中断睡眠是允许接收外界信号和内核信号而被唤醒的睡眠，绝大多数睡眠都是可中断睡眠，能ps或top捕捉到的睡眠也几乎总是可中断睡眠；不可中断睡眠只能由内核发起信号来唤醒，外界无法通过信号来唤醒，主要表现在和硬件交互的时候

- 运行态 -> 就绪态： 进程由于时间片被用完或者抢占式调度方式中，高优先级的程序强制抢占正在运行的低优先级的程序

- 运行态 -> 终止态：进程已经完成或者发生特殊事件，进程变为终止状态，返回退出状态码。 

    