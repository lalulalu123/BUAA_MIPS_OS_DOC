#Lab6实验报告
赵岳
14231027
此版本为待修订版！
###Thinking 6.1 
#####示例代码中，父进程操作管道的写端，子进程操作管道的读端。如果现在想让父进程作为“读者”，代码应当如何修改？
很简单，调换case0和default的代码即可。此时就读写者互换了。
###Exercise 6.1 
#####仔细观察pipe 中新出现的权限位PTE_LIBRARY，根据上述提示修改fork 系统调用，使得管道缓冲区是父子进程共享的，不设置为写时复制的模式。
Lab4中已经写好了，就不再赘述了。
###Exercise 6.2 
#####根据上述提示与代码中的注释， 填写user/pipe.c 中的piperead、pipewrite、_pipeisclosed 函数并通过testpipe 的测试。

    static int
	piperead(struct Fd *fd, void *vbuf, u_int n, u_int offset)
	{
	int i;
	struct Pipe *p;
	char *rbuf;

    /*Step 1: Get the pipe p according to fd. And vbuf is the reading buffer. */
	p = (struct Pipe*)fd2data(fd);
	rbuf = vbuf;
    /*Step 2: If pointer of reading is ahead of writing,then yield. */
    	while(p->p_rpos>=p->p_wpos){
		if(_pipeisclosed(fd,p))	// _pipeisclosed to check whether the pipe is closed.
			return 0;
		syscall_yield();		//syscall
	}
    /*Step 3: p_buf's size is BY2PIPE, and you should use it to fill rbuf. */
    	for(i=0 ; (p->p_rpos<p->p_wpos)&&(n--) ; i++ , p->p_rpos++)
    		rbuf[i] = p->p_buf[p->p_rpos%BY2PIPE];
	return i;
	}


    static int
	pipewrite(struct Fd *fd, const void *vbuf, u_int n, u_int offset)
	{
	int i;
	struct Pipe *p;
	char *wbuf;

    /*Step 1: Get the pipe p according to fd. And vbuf is the writing buffer. */
	p = (struct Pipe*)fd2data(fd);
	wbuf = vbuf;
    /*Step 2: If the difference between the pointer of writing and reading is larger than BY2PIPE, then yield. */
	while(p->p_wpos-p->p_rpos>=BY2PIPE){
		if(_pipeisclosed(fd,p))	// _pipeisclosed to check whether the pipe is closed.
			return 0;
		syscall_yield();		//syscall
	}
    /*Step 3: p_buf's size is BY2PIPE, and you should use it to fill rbuf. */
    	for(i=0 ; i<n ; i++ , p->p_wpos++){
    		p->p_buf[p->p_wpos%BY2PIPE] = wbuf[i];
    		while((i<n)&&(p->p_wpos-p->p_rpos>=BY2PIPE))
			syscall_yield();
    	}
    	
	return n;
	}



    static int
	_pipeisclosed(struct Fd *fd, struct Pipe *p)
	{
	int pfd, pfp, runs;

    /*Step 1: Get reference of fd and p, and check if they are the same. */
  	pfd = pageref(fd);
  	runs = env->env_runs;
 	pfp = pageref(p);
	while(runs!=env->env_runs){
		pfd = pageref(fd);
		runs = env->env_runs;
		pfp = pageref(p);
	}  //again and again
    /*Step 2: If they are the same, return 1; otherwise return 0. */
  	if(pfd == pfp)
		return 1;
	return 0;
	}

###Thinking 6.2 
#####上面这种不同步修改pp_ref 而导致的进程竞争问题在user/fd.c 中的dup 函数中也存在。请结合代码模仿上述情景，分析一下我们的dup 函数中为什么会出现预想之外的情况？
dup函数的作用是将旧的文件描述符和对应的数据映射到新文件描述符上，类似于我们Lab4中的duppage。dup原来的机制是先映射文件描述符本身再映射其中的内容，通俗地讲，也就是先增加了fd\_ref，再增加了pipe\_ref，类似于close的竞争，我们应该先增加pipe\_ref，否则在某一时刻会出现fd\_ref=prpe_ref的情况从而关闭端口。
###Thinking 6.3 
#####阅读上述材料并思考：为什么系统调用一定是原子操作呢？如果你觉得不是所有的系统调用都是原子操作，请给出反例。希望能结合相关代码进行分析。
Linux中并不是所有的系统调用都是原子操作，例如write系统调用就不是。两个进程同时写一个文件，就会出现各自写的数据中相互穿插叠加的情况。我们可以设置一个“专职打字员”负责写，其他进程均以IPC的形式给它发送消息，然后打字员负责将待写入内容排成队列有序地写入。这样可以实现write的原子性。
###Thinking 6.4 
####仔细阅读上面这段话，并思考下列问题
#####• 按照上述说法控制pipeclose 中fd 和pipe unmap 的顺序，是否可以解决上述场景的进程竞争问题？给出你的分析过程。
#####• 我们只分析了close 时的情形，那么对于dup 中出现的情况又该如何解决？请模仿上述材料写写你的理解。
1.不能。dup还有竞争存在，还需要解决。
2.照猫画虎，先处理fd，再处理pipe。
###Exercise 6.3 
#####修改user/pipe.c 中的pipeclose 与user/fd.c 中的dup 函数以避免上述情景中的进程竞争情况。
调换两个if语句即可，先处理fd，再处理pipe。
###Exercise 6.4 
#####根据上面的表述，修改_pipeisclosed函数，使得它满足“同步读”的要求。注意env_runs 变量是需要维护的。

    static int
	_pipeisclosed(struct Fd *fd, struct Pipe *p)
	{
	int pfd, pfp, runs;

    /*Step 1: Get reference of fd and p, and check if they are the same. */
  	pfd = pageref(fd);
  	runs = env->env_runs;
 	pfp = pageref(p);
	while(runs!=env->env_runs){
		pfd = pageref(fd);
		runs = env->env_runs;
		pfp = pageref(p);
	}  //again and again
    /*Step 2: If they are the same, return 1; otherwise return 0. */
  	if(pfd == pfp)
		return 1;
	return 0;
	}
###阶段性现象
![Alt text](./捕7获.JPG)
至此，pipe和竞争测试已经成功完成，下来完成shell部分。
###Exercise 6.5 
#####模仿现有的系统调用，增加系统调用syscall_cgetc。(提示:sys_cgetc函数不需要传入参数)

    int
	syscall_cgetc()
	{
	return msyscall(SYS_cgetc, 0, 0, 0, 0, 0);
	}


###Exercise 6.6 
#####根据以上描述，补充完成user/sh.c 中的void runcmd(char *s)。

    			case '|':
				pipe(p);
				if((r = fork())<0)
					writef("fork error %d",r);
				else if(r == 0) {
					dup(p[0],0);
					close(p[0]);
					close(p[1]);
					goto again;
				}
				else{
					dup(p[1],1);
					close(p[1]);
					close(p[0]);
					rightpipe = r;
					goto runit;
				}


###实验现象与总结			
Lab6还有许多不完美的地方，还需继续加深理解，调试！
![Alt text](./捕6获.JPG)
