ostep

---

### direct execution

当执行系统调用时 ，将进程当前上下文保存到kernal stack中，当从系统调用中返回前，将上下文恢复出来，继续在user mode下执行。

How can the operating system regain control of the CPU so that it can switch between processes?

A timer device can be programmed to raise an interrupt every
so many milliseconds; when the interrupt is raised, the currently running
process is halted, and a pre-configured interrupt handler in the OS runs.
At this point, the OS has regained control of the CPU, and thus can do
what it pleases: stop the current process, and start a different one.
As we discussed before with system calls, the OS must inform the
hardware of which code to run when the timer interrupt occurs; thus,
at boot time, the OS does exactly that. Second, also during the boot
sequence, the OS must start the timer, which is of course a privileged
operation. Once the timer has begun, the OS can thus feel safe in that
control will eventually be returned to it, and thus the OS is free to run
user programs. The timer can also be turned off (also a privileged operation), something we will discuss later when we understand concurrency
in more detail.



