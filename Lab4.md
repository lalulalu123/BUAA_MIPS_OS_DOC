# lab4实验报告
赵岳
14231027
#### Exercise 4.1 
##### 填写 msyscall，使得系统调用机制可以正常工作。 

	   move v0,a0
	   sw a0,0(sp)
	   sw a1,4(sp)
	   sw a2,8(sp)
	   sw a3,12(sp)
	   syscall
	   jr ra

mysyscall主要完成的功能是，设置syscall的各种参数，包括系统调用号（传入v0），和其他参数。由于寄存器的个数限制，只是将a0~a3四个参数使用寄存器进行传递，其余的参数使用栈传递。因此，结合注释，不难写出如上代码。值得注意的是，始终需要记得填写.s汇编函数时，要使用jr ra返回。
####Exercise 4.2 
#####实现 lib/syscall\_all.c 中的 void sys\_ipc\_recv(int sysno,u\_int dstva) 函数和 int sys\_ipc\_can\_send(int sysno,u\_int envid, u\_int value, u\_int srcva, u\_int perm) 函数。 

    void sys_ipc_recv(int sysno,u_int dstva)
	{
    	if ((unsigned int)dstva >= UTOP || dstva != ROUNDDOWN(dstva, BY2PG)){
		return;
	}
	curenv->env_ipc_dstva = dstva;
	curenv->env_ipc_recving = 1;
	curenv->env_status = ENV_NOT_RUNNABLE;
	sys_yield();	//not sched yield！！
	return 0;
	}
这个函数的主要目的如下：设置当前进程为可接收的状态，然后使当前运行的进程不可运行，暂停当前进程调度，调度其它进程进入cpu。其中参数 dstva是收到的页面的虚拟地址。暂停当前进程调度后，需要使用进程调度函数重新进行进程调度。值得注意的是，这里我认为应该使用在syscallall.c中给出的sys\_yield()函数，而不是我们在lab3中实现的sched\_yield函数。实际上，这两个函数的唯一区别就是 ` bcopy((int)KERNEL_SP-sizeof(struct Trapframe),TIMESTACK-sizeo f(struct Trapframe),sizeof(struct Trapframe)); `这一句话。这句代码的目的是将当前进程的运行环境保存到TIMESTACKsizeof(struct Trapframe)。在理论课中我们知道进程的切换很关键的步骤就是保存线程，既然我们需要将cpu交给其他进程，那这个暂时被搁置的进程就不能不管。因此这一步是很关键的

    int sys_ipc_can_send(int sysno,u_int envid, u_int value, u_int srcva, u_int perm)
	{
	struct Env *target;
	struct Page *page;
	Pte *pte;
	int r, ret = 0;
	if ((r = envid2env(envid, &target, 0)) < 0)
		return -E_BAD_ENV;
	if (!target->env_ipc_recving)
		return -E_IPC_NOT_RECV;
	if (srcva) {
		if (((unsigned int)srcva >= UTOP)||(srcva != ROUNDDOWN(srcva, BY2PG))||((page = page_lookup(curenv->env_pgdir, srcva, &pte)) == NULL))
			return -E_INVAL;
		//illegal address or not rounded or no such a page  return -E_INVAL
		if (page_insert(target->env_pgdir, page, target->env_ipc_dstva, perm) < 0)
			return -E_NO_MEM;
		ret = 1;			//success = 1
	}
	target->env_ipc_recving = 0;
	target->env_ipc_value = value;
	target->env_ipc_from = curenv->env_id;
	target->env_ipc_perm = ret? perm : 0 ;
	target->env_status = ENV_RUNNABLE;			//runnable again
	return 0;
	}
这个函数的目的是： 尝试向目标进程(env) 'envid'发送'value' ，如果目标进程没有使用sys\_ipc\_recv请求IPC，那么发送会失败，并且返回-E\_PC\_NOT\_RECV。 否则，发送成功，目标进程的ipc相关的域被按照如下方式设置：env\_ipc\_recving 被设置为 0 以阻塞之后发送的值，env\_ipc\_from 被设置为发送者的 envid，env\_ipc\_value 被设置为'value' 。目标进程被重新设置为可运行的。成功时返回0，错误时返回值小于0。  
那么，根据这个详细的步骤，我们不难写出如上代码。当然，目前还存在几个难以回答的问题：
1.sysno参数的作用是什么？该函数中好像并没有使用。
2.难道srcva的作用只是为了判断一下目标地址的正确性吗？
3.在本函数中的ret是什么作用，为了标记进程通信的成功吗？在我的函数中，如果ret成功置1才能对perm进行赋值，否则为0。
最后，仔细理解一下该函数中调用的env2envid函数：

    int envid2env(u_int envid, struct Env **penv, int checkperm)
	{
	struct Env *e;

    /* Hint:
	    *  If envid is zero, return the current environment.*/
	if (envid == 0) {
		*penv = curenv;
		return 0;
	}

	e = &envs[ENVX(envid)];

	if (e->env_status == ENV_FREE || e->env_id != envid) {
		*penv = 0;
		return -E_BAD_ENV;
	}

    /* Hint:
     *  Check that the calling environment has legitimate permissions
     *  to manipulate the specified environment.
     *  If checkperm is set, the specified environment
     *  must be either the current environment.
     *  or an immediate child of the current environment. */

    if (checkperm && e != curenv && e->env_parent_id != curenv->env_id){
        *penv = 0;
        return -E_BAD_ENV;
    }
	*penv = e;
	return 0;
	}
这个函数是作用是：传入一个 envid，然后函数会把 envid 所对应的 env 结构体的指针存在传入的 penv参数中。如果出错会返回 -E\_BAD\_ENV。
那么至此，我们就相对清楚了该函数的作用。我们在ipc_send函数中，我们将&target作为调用该函数的第二个参数，envid作为第一个参数。这里，由于没有看懂checkperm的机制，因此我们直接跳过了checkperm这一阶段，将0传入。最后，我们将该函数的错误返回值作为调用该函数时的错误返回值即可。
至此，进程通信部分已经完成。
####Thinking 4.1 
#####思考下面的问题，并对这两个问题谈谈你的理解：
#####• 子进程完全按照 fork() 之后父进程的代码执行，说明了什么？
#####• 但是子进程却没有执行 fork() 之前父进程的代码，又说明了什么？
子进程完全按照父进程fork之后的代码执行，说明子进程的代码段和父进程的代码段是一致的。他们是同一个程序，只不过是两个不同的进程。然而子进程是“半路杀出”，从一半开始执行，说明子进程虽然和父进程代码一样，可是开始的位置却不同。具体一点，子进程执行的代码的起始位置应该是父进程的epc。这一点在我们的代码中也有体现：`child->env_tf.pc = child->env_tf.cp0_epc;`。
####Thinking 4.2 
#####关于 fork 函数的两个返回值，下面说法正确的是： 
#####A、fork 在父进程中被调用两次，产生两个返回值 
#####B、fork 在两个进程中分别被调用一次，产生两个不同的返回值 
#####C、fork 只在父进程中被调用了一次，在两个进程中各产生一个返回值 
#####D、fork 只在子进程中被调用了一次，在两个进程中各产生一个返回值
C.
####Exercise 4.3 
#####填写./lib/syscall_all.c 中的函数 sys\_env\_alloc，可以不填返回值。 

    int sys_env_alloc(void)
	{
	struct Env *child;
	if (env_alloc(&child, curenv->env_id) < 0)
		return -E_NO_FREE_ENV;
	bcopy(KERNEL_SP - sizeof(struct Trapframe), &child->env_tf,sizeof(struct Trapframe));	
	child->env_status = ENV_NOT_RUNNABLE;
	child->env_tf.pc = child->env_tf.cp0_epc;
	child->env_tf.regs[2] = 0;

	return child->env_id;
	}
这个函数我认为有如下几点需要注意：
1.首先在申请新的进程控制块时判断一下能否申请新的进程，不能的话返回-E\_NO\_FREE\_ENV，尽管这种情况出现的较少。
2.当前运行进程的现场保存在了`(int)KERNEL_SP-sizeof(struct Trapframe) `（这是有证据可循的，比如sys_yield函数）,因此我们需要将其拷贝到新创建进程的tf中。
3.将新申请的进程状态标记为env_status = ENV_NOT_RUNNABLE，这也是符合逻辑的，我们只是创建了一个新的进程控制块，其数据、栈、代码等其他还未设置好。
4.在将父进程的运行环境拷贝到子进程后，还需要调整一些东西，首先，使得子进程的调用返回0。由于返回值v0是2号寄存器，因此我们还需要一句`child->env_tf.regs[2] = 0`。
5.根据实验前的父子进程fork小实验，我们知道子进程是从父进程fork它的地方继续执行，而不是代码的开头。因此我们要在创建子进程时的pc置为父进程的epc，因此我们需要`child->env_tf.pc = child->env_tf.cp0_epc;`
6.父子进程在fork函数中`envid = sys_env_alloc()`后便“分道扬镳”了，那么我们设置了子进程的返回值为0后，我们还需要管一下父进程。为了区别父子进程，将父进程的返回值设置为子进程的env_id。
####Exercise 4.4 
#####补充./lib/syscall\_all.c 中的函数 sys\_env\_alloc 的返回值。
如上，不再赘述。
####Exercise 4.5 
#####补充./user/fork.c 中的函数 fork 中关于 sys\_env\_alloc 的部分和“子进程”执行的部分。 

    int fork(void)
	{
	u_int envid;
	int pn;
	extern struct Env *envs;
	extern struct Env *env;

	set_pgfault_handler(pgfault);
	if((envid = syscall_env_alloc()) < 0)
		user_panic("syscall_env_alloc failed!");
	if(envid == 0){
		env = &envs[ENVX(syscall_getenvid())];
                	return 0;
	}
	for(pn = 0; pn < ( UTOP / BY2PG) - 1 ; pn ++){
		if(((*vpd)[pn/PTE2PT]) != 0 && ((*vpt)[pn]) != 0)
			duppage(envid, pn);
	}
	if(syscall_mem_alloc(envid, UXSTACKTOP - BY2PG, PTE_V|PTE_R) < 0)
                	user_panic("In fork! syscall_mem_alloc error!");

    if(syscall_set_pgfault_handler(envid, __asm_pgfault_handler, UXSTACKTOP) < 0)
                	user_panic("In fork! syscall_set_pgfault_handler error!");

	if(syscall_set_env_status(envid, ENV_RUNNABLE) < 0)
	               	user_panic("In fork! syscall_set_env_status error!");

    return envid;	
	}
这个函数可以理解为父进程“恋恋不舍地为子进程开辟了闯荡的道路并为其做好出发的准备”。那么我们要做的就是，为子进程设置页面错误的处理函数（我们暂时不需管怎样处理，这个在最后的函数中会写），用sys\_env\_alloc函数申请一个子进程的进程控制块。然后按页遍历2G的用户空间，如果存在页映射，那就用duppage函数给子进程的页面设置好权限位。调用syscall\_mem\_alloc函数为子进程申请一个错误栈。最后，将子进程的状态利用syscall\_set\_env\_status设置为可以运行的。至此，子进程就可以正式“出发”了。
这里有一些问题需要注意，首先，我们一再强调了fork是用户态下的函数，那么我们就只能利用大量提供的syscall_x函数来进行一些参数的设置。由于这些syscall函数都是int类型的，所以再调用时进行了一步判断，如果调用错误就会进入user\_panic（由于用户态，不能使用panic）,方便调试错误。
还有一个非常关键的问题，，我们需要将 \_asm\_pgfault\_handler 设置为子进程的页中断函数入口。我们知道，\_asm\_pgfault\_handler 函数(这是一个汇编函数)将出错的地址提取了出来，并作为参数传递给 pgfault 。也就是说，如果我们将子进程的页中断函数设置为pgfault，那么页中断时，就会直接进入 pgfault 函数。这样的结果就是：从cp0中提取出错的虚地 址，恢复寄存器等工作就没代码来做了。所以，我们需要将\_asm\_pgfault\_handler 设置为子 进程的页中断处理函数。
在后来和其他同学的讨论中，发现错误栈的申请和映射工作可以放在sys\_env\_alloc函数中进行。但在这里，为了和lab3中env\_alloc函数的对应关系，在sys\_env\_alloc只进行了进程控制块的申请工作，其余fork的额外工作全部在fork函数中实现。
####Thinking 4.3 
#####如果仔细阅读上述这一段话, 你应该可以发现, 我们并不是对所有的用户空间页都使用 duppage 进行了保护。那么究竟哪些用户空间页可以保护，哪些不可以呢，请结合 include/mmu.h 里的内存布局图谈谈你的看法。 
根据mmu.h中的内存布局，我们可以看到，UTOP是用户空间的极限，duppage肯定为比UTOP小的地址空间服务。其次，我们还注意到，UTOP也叫UXSTACKTOP （`#define UXSTACKTOP (UTOP)`）它的下面一页是 exception stack，每个进程的异常栈都是我们单独处理的，因此我认为这一页也不需要duppage。综上，duppage保护的页应该是截至到(UTOP/BY2PG-1)这一页。
####Exercise 4.6 
#####结合注释，补充 user/fork.c 中的函数 duppage。 

    static void duppage(u_int envid, u_int pn)
	{
	u_int perm = ((*vpt)[pn]) & 0xfff;;
	if( (perm & PTE_R)!= 0 || (perm & PTE_COW)!= 0){
         		if(!(perm & PTE_LIBRARY)) {
                   		perm = perm | PTE_V | PTE_R | PTE_COW;
               	}
                else{
                    	perm = perm | PTE_V | PTE_R;
                	}

              	if(syscall_mem_map(syscall_getenvid(), pn * BY2PG, envid, pn * BY2PG, perm) < 0)
                        	user_panic("In duppage:mem_map_envid error!");

              	if(syscall_mem_map(syscall_getenvid(), pn * BY2PG, 0, pn * BY2PG, perm) < 0)
                        	user_panic("In duppage:mem_map_0 error!");
      }
      else{
              	if(syscall_mem_map(syscall_getenvid(), pn * BY2PG,envid, pn * BY2PG, perm) < 0)
                        	user_panic("In duppage:mem_map_envid error!");
        	}
	}
这个函数原理较为简单，就是为fork函数填写一个可以正确给出子进程某个页权限位的函数。由于duppage还是在用户态进行的，所以只能使用系统调用提供的函数。我们先利用`perm = ((*vpt)[pn]) & 0xfff;`得到当前传入页面的权限位，然后开始判断：如果父进程中的页是可写或COW的，那么再判断一下是否是共享内存的（`#define PTE_LIBRARY		0x0004	// share memmory`）如果不是，则COW，否则子进程中该页就不用COW。然后在父子进程中分别调用syscall\_mem\_map进行页面的映射,均以COW的权限。当然，还存在父进程中那些只读的页，那么只需要不变地将其映射到子进程中即可。
在这里，syscall_mem_map函数的第一个参数，也就是本进程的id的确定值得思考。这里我使用了另一个系统调用来得到本进程函数，但在调试过程中发现传0也可以。猜测可能是由于envid2env函数中，传入0则会得到当前的进程，证据如下：

    int envid2env(u_int envid, struct Env **penv, int checkperm)
	{
	。。。。。。

    /* Hint:
     *  If envid is zero, return the current environment.*/
	if (envid == 0) {
		*penv = curenv;
		return 0;
	}

	。。。。。。
	}
至此，duppage函数就完成了	
####Exercise 4.7 
#####结合注释，补充 user/fork.c 中的函数 pgfault。 
	static void
	pgfault(u_int va)
	{
	int r,i;
	va = ROUNDDOWN(va, BY2PG);

      	if (!((*vpt)[VPN(va)] & PTE_COW ))
                	user_panic("In pgfault:PTE_COW error!");

      	if (syscall_mem_alloc(0, PFTEMP, PTE_V|PTE_R) < 0)
                	user_panic("In pgfault:syscall_mem_alloc error!");

      	user_bcopy((void*)va, PFTEMP, BY2PG);

      	if (syscall_mem_map(0, PFTEMP, 0, va, PTE_V|PTE_R) < 0)
            	user_panic("In pgfault:syscall_mem_map error!");

      	if (syscall_mem_unmap(0, PFTEMP) < 0)
                	user_panic("In pgfault:syscall_mem_unmap error!");
	}
像前两个函数一样，这个函数还是运行在用户空间的，这对我们函数的实现造成了很大的阻碍。好在丰富的系统调用给了我们足够多的工具。
步骤大致如下：首先判断该虚拟地址对应的页是否是COW的，不是得话则panic一个error(由于用户态，还是需要使用user\_panic函数)。然后，在PFTEMP位置申请一个新的物理页，将va这里的页拷贝到这个临时物理页上，最后，把临时位置映射的的物理页映射到va上，并且解除临时位置对内存页的映射。
####Thinking 4.4 
#####请结合代码与示意图, 回答以下两个问题： 
#####1、vpt 和 vpd 宏的作用是什么, 如何使用它们？ 
#####2、它们出现的背景是什么? 如果让你在 lab2 中要实现同样的功能, 可以怎么写？   

1.首先，在mmu.h中，我们可以找到如下定义：

    extern volatile Pte* vpt[]; 
    extern volatile Pde* vpd[]; 

 查阅资料得知，volatile修饰变量 ，说明这两个变量可能在汇编等C语言之外的地方被改变。那么我们很自然地想到了汇编，因此继续查找.S文件，最终在entry.S发现了这两个变量的定义。
 

	     .globl vpt 
     vpt:     
	     .word UVPT 
	     .globl vpd 
    vpd:     
	     .word (UVPT+(UVPT>>12)*4) 

根据 mmu.h 和 lab3 得知，UVPT是用户空间中页表的位置。那么，vpd就也是页目录的位置。所以，遍历页目录也就是遍历vpd。遍历vpd时，我们将所有分配给内存的物理页映射给子进程。遍历时还需要判断二级页表是否存在，存在则映射。在映射二级页表的同时，二级页表中分配的物理页也需要映射。 二级页表则通过vpt来访问。

2.假设有这样一种需求，我们知道了一个确定的虚拟地址va，需要知道该地址对应的页目录项和页表项的虚拟地址，该如何求？实际上，这个问题的答案就是vpd和vpt。va首先可以拆分为3部分，高10位的PDX，中间10位的PTX，以及低12位的OFFSET。那么要求va对应页目录项的虚拟地址，实际上就是UVPT+(PDX<<2)，由于UVPT的低12位是为0的(按页对齐)，vpt求法同理。因此:

```
vpd = UVPT[31:12] | PDX | 00;
vpt = UVPT[31:22] | PDX | PTX | 00;
```
至于如何使用，那就显而易见了。由于我们的定义方法，vpt里存着UVPT的首地址，所以在lab2中，作如下代换即可：`(*vpt)[N] = UVPT[N]`。只不过有一点不同就是lab4我们都是工作在用户空间的，而lab2中大量工作在内核空间。
