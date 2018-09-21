# Lab5实验报告
赵岳
14231027
### Thinking 5.1 
##### 查阅资料，了解Linux/Unix 的/proc 文件系统是什么？有什么作用？Windows 操作系统又是如何实现这些功能的？proc 文件系统这样的设计有什么好处？
其实/proc并不是一个真正的文件系统，只存在于内存中，访问时，通过访问根目录下的/proc目录即可访问到内核数据。
### Exercise 5.1 
##### 参考ide_read 函数和read_sector 函数的实现方式，实现fs/ide.c中的ide_write 函数，以及fs/ide_asm.S 中的write_sector 函数，实现对磁盘的写操作。

    void
	ide_write(u_int diskno, u_int secno, void *src, u_int nsecs)
	{
	int offset_begin = secno * 0x200;
	int offset_end = offset_begin + nsecs * 0x200;
	int offset = 0;
	writef("diskno: %d\n", diskno);
	while (offset_begin + offset < offset_end) {
		user_bcopy(src + offset, (void *)0x93004000, 0x200);
		if (write_sector(diskno, offset_begin + offset)) {
			// copy data from disk buffer(512 bytes, a sector) to destination array.
			offset += 0x200;
		} else {
			// error occurred, then panic.
			user_panic("disk O error");
		}
	}
	}
这个函数根据ide_read照猫画虎就行了。

    LEAF(write_sector)
	sw  a0, 0x93000010 	// select the IDE id.
	sw	a1, 0x93000000  // offset.
	li	t0, 1
	sb	t0, 0x93000020  // start write.
	lw  v0, 0x93000030	// get status as return value.
	nop
	jr	ra
	nop
	END(write_sector)
需要注意第四行t0的值应该是1表示写。
### Exercise 5.2 
##### 文件系统需要负责维护磁盘块的申请和释放，在回收一个磁盘块时，需要更改bitmap 中的标志位。如果要将一个磁盘块设置为free，只需要将bitmap中对应的bit 的值设置为0x0 即可。请完成fs/fs.c 中的free_block 函数，实现这一功能。同时思考为什么参数blockno 的值不能为0 ？

    void
	free_block(u_int blockno)
	{
	// Step 1: Check if the parameter `blockno` is valid (`blockno` can't be zero).
	if(blockno==0)
		user_panic("zero block in free_block");
	// Step 2: Update the flag bit in bitmap.
	bitmap[blockno / 32]=1 << (blockno % 32);	//according to guide book,here should be 0x0. but.......

	}
只能说感谢助教的注释助我轻松完成代码。只是有一点比较疑惑不知道为何指导书中说把bit值设置为0，之前不是规定了空闲是1吗？疑惑。
根据布局图，blockno=0存储的是Boot Sector and Partition table，这部分当然不能被free，因此出于安全性需要对blockno进行一步check，确保不为0。
### Thinking 5.2 
##### 请思考，在满足磁盘块缓存的设计的前提下，我们实验使用的内核支持的最大磁盘大小是多少？
根据fs.h中的宏定义及其注释：


    /* Maximum disk size we can handle (3GB) */
	#define DISKMAX		0xc0000000

得知最大支持磁盘大小为3GB。

### Exercise 5.3 
##### fs/fs.c 中的diskaddr 函数用来计算指定磁盘块对应的虚存地址。完成diskaddr 函数，根据一个块的序号(block number)，计算这一磁盘块对应的512bytes 虚存的起始地址。（提示：fs/fs.h 中的宏DISKMAP 和DISKMAX 定义了磁盘映射虚存的地址空间）

    u_int
	diskaddr(u_int blockno)
	{
	u_int result;
	result=DISKMAP+blockno*BY2BLK;
	if( super && blockno>=super->s_nblocks)
		user_panic("diskaddr error of blockno %08x\n", blockno);
	else
		return result;
	}
这道exercise非常简单，就是线性叠加地根据块数计算虚拟地址。既然起点知道，每块大小知道，那么就迎刃而解了。
### Exercise 5.4 
##### 实现map_block 函数，检查指定的磁盘块是否已经映射到内存，如果没有，分配一页内存来保存磁盘上的数据。对应地，完成unmap_block 函数，用于接触磁盘块和物理内存之间的映射关系，回收内存。（提示：注意磁盘虚拟内存地址空间和磁盘块之间的对应关系）。

    int
	map_block(u_int blockno)
	{
	int r;
	// Step 1: Decide whether this block is already mapped to a page of physical memory.
	if(block_is_mapped(blockno))
		return 0;
    // Step 2: Alloc a page of memory for this block via syscall.
    	else{
    		r=syscall_mem_alloc(syscall_getenvid(), diskaddr(blockno) , PTE_V|PTE_R);
    		return r;
    	}
	}

	void
	unmap_block(u_int blockno)
	{
	int r;

	// Step 1: check if this block is mapped.
	if(block_is_mapped(blockno)){
	// Step 2: if this block is used(not free) and dirty, it needs to be synced to disk,
	// can't be unmap directly.
		user_assert(block_is_free(blockno) || !block_is_dirty(blockno));
			;	//do nothing (be synced to disk,)
	// Step 3: use `syscall_mem_unmap` to unmap corresponding virtual memory.
		if(r=syscall_mem_unmap(syscall_getenvid(), diskaddr(blockno))<0)
			user_panic("In unmap_block error!");
	}
	// Step 4: validate result of this unmap operation.
	user_assert(!block_is_mapped(blockno));
	}
感谢注释，根据和注释填写即可。注意map函数中如果对一个已经map过的block继续map就会导致返回0。
### Thinking 5.3 
##### 一个Block 最多存储1024 个指向其他磁盘块的指针，试计算，我们的文件系统支持的单个文件的最大大小为多大？
1024*4KB=4MB，即最大可以指向的其他磁盘块大小和为4MB，也就是支持的最大单个文件。
### Thinking 5.4 
##### 阅读serve函数的代码，我们注意到函数中包含了一个死循环for (;;) {...}，为什么这段代码不会导致整个内核进入panic 状态？
我们的操作系统是微内核的，文件系统实际上还是一个进程，那么就会有进程调度。以我们的测试程序为例，

    #include "lib.h"

	void umain()
	{
	while (1) {
		writef("IDLE!");
	}
	}
这个代码也有死循环，但运行时也不会引起内核进入panic，那么文件系统中的死循环自然也不会产生panic咯。

### Exercise 5.5
##### 文件user/fsipc.c 中定义了请求文件系统时用到的IPC 操作，user/file.c文件中定义了用户程序读写、创建、删除和修改文件的接口。完成user/fsipc.c中的fsipc_remove 函数、user/file.c 中的remove 函数， 以及fs/serv.c 中的serve_remove 函数，实现删除指定路径的文件的功能。

    int
	remove(const char *path)
	{
	int r;
	u_int i;

	if ((r = fsipc_remove(path)) < 0) {
		writef("cannont remove file %s\n", path);
		return r;
	}
	return 0;
	}	


    int
	fsipc_remove(const char *path)
	{
	struct Fsreq_open *req;

	req = (struct Fsreq_remove *)fsipcbuf;
	// Step 1: decide if the path is valid.
	if (strlen(path) >= MAXPATHLEN) {
		return -E_BAD_PATH;
	}
	strcpy((char *)req->req_path, path);
	// Step 2: Send request to fs server with IPC.
	return fsipc(FSREQ_REMOVE, req, 0 , 0);
	}
	

    void
	serve_close(u_int envid, struct Fsreq_close *rq)
	{
	struct Open *pOpen;

	int r;

	if ((r = open_lookup(envid, rq->req_fileid, &pOpen)) < 0) {
		ipc_send(envid, r, 0, 0);
		return;
	}
	
	file_close(pOpen->o_file);
	ipc_send(envid, 0, 0, 0);
	}

根据主观上的经验，remove应该是一个比较简单的文件操作，从它的参数数量上也可以看出来。对比open操作，只有open的一个参数。那么我们就可以根据open函数照猫画虎来进行remove函数的填写。



### 总结
纵观本次lab5，我们的小操作系统终于有了一些像样了，竟让我有一丝成就感。然而，咋实验过程中，许多函数都可以照猫画虎地填写，有些地方不知为何那样填写，只是对比着蒙对了正确答案。还有的地方在不得其解的时候发现助教已经给出了详尽的注释帮助我们，再次感谢！总之，在交上报告之后还要再重新思考一遍lab5的点点滴滴。最后，把实验的最终结果附在下方：
![Alt text](./lab5.JPG)
