# 荣誉准则
1.在完成本次实验的过程（含此前学习的过程）中，我曾分别与 以下各位 就（与本次实验相关的）以下方面做过交流，还在代码中对应的位置以注释形式记录了具体的交流对象及内容：

        询问chatgpt关于risc-v伪指令的知识，免去了我搜索的负担

2.此外，我也参考了 以下资料 ，还在代码中对应的位置以注释形式记录了具体的参考来源及内容：

        [gas directives](https://ftp.gnu.org/old-gnu/Manuals/gas-2.9.1/html_chapter/as_7.html)
		[riscv 特权级](https://www.cnblogs.com/harrypotterjackson/p/17548837.html#_label1)

3. 我独立完成了本次实验除以上方面之外的所有工作，包括代码与文档。 我清楚地知道，从以上方面获得的信息在一定程度上降低了实验难度，可能会影响起评分。

4. 我从未使用过他人的代码，不管是原封不动地复制，还是经过了某些等价转换。 我未曾也不会向他人（含此后各届同学）复制或公开我的实验代码，我有义务妥善保管好它们。 我提交至本实验的评测系统的代码，均无意于破坏或妨碍任何计算机系统的正常运转。 我清楚地知道，以上情况均为本课程纪律所禁止，若违反，对应的实验成绩将按“-100”分计。

# 总结

修改了syscall函数，使得在每次调用syscall时候都会记录次数，并且修改了task模块中的内容，给TaskManager增加了公开接口获取当前运行的程序id，以及获取当前运行程序的开始运行时间，为了记录开始运行时间，修改了TaskControlBlock，增加了两个field，一个用来判断是否是第一次运行，一个用来记录第一次被调度运行时候的时刻。然后在process.rs中完善了sys_task_info，调用以上修改内容来获取任务信息。

# 问答题

1. 

`[kernel] Loading app_0
[kernel] PageFault in application, kernel killed it.
[kernel] Loading app_1
[kernel] IllegalInstruction in application, kernel killed it.
[kernel] Loading app_2
[kernel] IllegalInstruction in application, kernel killed it.
[kernel] Loading app_3
`

这是相应的回应，依次是bad_address, bad_instructions, bad_registers

这是因为，执行这些指令的时候，硬件触发同步异常，进入trap_handler之后，根据scause.cause执行了StoreFault | StorePageFault 和 IllegalInstruction 的match分支, 并且执行了run_next_app，因此没有继续执行。

rustsbi: RustSBI version 0.3.0-alpha.4, adapting to RISC-V SBI v1.0.0

2. 
   2.1    当执行`__restore`的时候是从trap_handler返回时，其返回值是cx，因此a0应该是指向`__alltraps`保存在内核栈上的用户程序状态,其实和当前的sp指向的应该一样。当`__restore`是从调度中返回时，也就是在`__switch`中跳转过来，这时候a0是`current_task_cx_ptr`
   `__restore`可以使用在trap之后的恢复，从内核态返回到应用程序状态并恢复原来寄存器，另一个是使用在任务调度中，这时候sp是`next_task_cx_ptr.sp`, 是kstack_ptr，这个里面被push了一个TrapContext，里面存储了用户程序的开始地址和用户栈，这时候会切换到这个用户程序去执行
   2.2 sstatus 保存着这个用户态时候的中断信息，权限级别等等，sepc保存了返回的地址, sscratch这里面暂时存放的用户栈地址，随后会用它来赋值sp
   2.3 x4(tp) 没啥用不需要恢复， x2(sp) 保存在了sscratch中
   2.4 sscratch -> kernel stack, sp -> user stack
   2.5 sret 因此 上面已经设置了sepc，以及sstatus，因此sret会根据这些值设置pc和特权级
   2.6 sp -> kernel stack sscratch -> user stack
   2.7 ecall
