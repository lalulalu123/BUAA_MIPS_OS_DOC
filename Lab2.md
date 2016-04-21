#<center>Lab2实验报告</center>
赵岳
142321027
##Thinking 3.1
####我们注意到我们把宏函数的函数体写成了 do { // ... } while(0) 的形式，而不是仅仅写成形如 {// ... } 的语句块，这样的写法好处是什么？
答：简单来说，就是为了避免一些歧义和语法错误，do保证了大括号内的语句始终会被执行，while（0）保证了大括号的语句只被执行一遍，并且`do{}while()`是一个一定语法正确的独立整体，不会与其他部分发生语法冲突。下面，我举一个使用大括号产生语法错误的例子:  
```cpp
#define f(x) {x++;}
	...
	...
	for(i=0;i<n;i++)
		f(x);
	return;
```
考虑上面的例子，直观上来看没有什么问题，可是利用代换就会发现，实际上代码是这样的：  
```cpp
#define f(x) {x++;}
	...
	...
	for(i=0;i<n;i++)
		{x++;};
	return;
```
产生了语法问题。而采用do while的形式则没有这种问题。

##Exercise3.1 
####我们需要在mips\_detect\_memory()函数中这几个全局变量，以确定内核可用的物理内存的大小和范围。根据代码注释中的提示，完成mips\_detect\_memory() 函数。  
本部分完成对四个全局变量的赋值即可。运行gexmul仿真器，可以看到我们做实验的机器的内存为64MB，且无外部存储设备：  
<center>![](https://raw.githubusercontent.com/tigerzhaoyue/BUAA_MIPS_OS_DOC/master/Picture/Lab2/mem64.JPG)  </center>
所以将相应初值赋好即可。值得注意的是：  
1. 物理地址从0开始递增至最大值。  
2. 页面大小每页4KB，需将总的内存大小除以4KB算出有多少页。
##Exercise3.2
####完成 page\_init 函数，使用 include/queue.h 中定义的宏函数将未分配 的物理页加入到空闲链表 page\_free\_list 中去。思考如何区分已分配的内存块和未 分配的内存块，并注意内核可用的物理内存上限。 
由于精妙而复杂的宏定义，这段思路很清晰的代码写了很久。以对page\_free\_list变量的使用为例，先由变量找到了定义，然后在定义里又出现了LIST_HEAD宏的使用，用宏定义了一个由Page类型变量组成的新的数据结构，称之为Page\_list(页的链表)，并定义了这种类型的变量Page\_free\_list来记录所有空闲的页整个构建过程的流程图如下：  
<center>![](https://raw.githubusercontent.com/tigerzhaoyue/BUAA_MIPS_OS_DOC/master/Picture/Lab2/page_free_list.png)  </center>
经过仔细推敲，结合注释，给出了最终代码如下：  
```cpp
    void
	page_init(void)
	{
    /* Step 1: Initialize page_free_list. */
    /* Hint: Use macro `LIST_INIT` defined in include/queue.h. */
    LIST_INIT(&page_free_list);

    /* Step 2: Align `freemem` up to multiple of BY2PG. */
    freemem = ROUND(freemem,BY2PG);

    /* Step 3: Mark all memory blow `freemem` as used(set `pp_ref`
	    * filed to 1) */
    int j = 0;
    while(j < PPN(PADDR(freemem))){                              // j < (freemem-0x80000000)/4KB?
        pages[j].pp_ref = 1;
        j++;
    }

    /* Step 4: Mark the other memory as free. */
    while(j<npage){
        pages[j].pp_ref = 0;
        LIST_INSERT_HEAD(&page_free_list,&page[j],pp_link);
        j++;
	    }
	}
```
其中值得注意的是，在mmu.h中定义了许多计算的宏定义，例如PADDR、PPN等。每个计算都有用途，用这些宏定义可以大大增强我们的代码可读性，也能对该步操作的目的一目了然。（显然地， PPN(PADDR(freemem))比 (freemem-0x80000000)/4KB要清楚得多）。
##Exercise3.3
#### 完成 mm/pmap.c 中的 page\_alloc 和 page\_free 函数，基于空闲内存链表 page\_free\_list ，以页为单位进行物理内存的管理。 
#####page\_alloc
代码如下：  
```cpp
    int
	page_alloc(struct Page **pp)
	{
    struct Page *ppage_temp;

    /* Step 1: Get a page from free memory. If fails, return the error code.*/
    if(LIST_EMPTY(&page_free_list)){
        return -E_NO_MEM;                             //#define E_NO_MEM  4      in error.h
    }
    ppage_temp = LIST_FIRST(&page_free_list);
    LIST_REMOVE(ppage_temp,pp_link);

    /* Step 2: Initialize this page.
     * Hint: use `bzero`. */
    bzero(ppage_temp,sizeof(struct Page));
    *pp=ppage_temp;
    return 0;
	}
```
具体实现过程是：先探测是否还有可以使用的空闲物理页面，即用LIST\_EMPTY宏定义来探测page\_free\_list是否是一个空链表。如果空，那么我们根据error.h中的值，将返回-4(具体为什么加负号，猜测一般错误值都会小于0，并且根据alloc函数中举一反三，推测这里内存不足时也应返回-4)。如果不空，那么我们将该链表的头部第一个元素赋给在本函数中定义的ppage\_temp变量，然后将这个节点从空闲链表中移除，并将这个节点全部置0，为我们对该页面的使用做好准备。如果申请成功，那么返回0（由于指导书中并没有提及分配成功的返回值，猜测该返回值并不重要，暂且认为其缺省值为0。）
#####page\_free
代码如下：
```cpp
    void page_free(struct Page *pp)
	{
    /* Step 1: If there's still virtual address refers to this page, do nothing. */
    if(pp->pp_ref>0){
        return;
    }

    /* Step 2: If the `pp_ref` reaches to 0, mark this page as free and return. */
    else if(pp->pp_ref==0){
        LIST_INSERT_HEAD(&page_free_list,pp,pp_link);
    }

    /* If the value of `pp_ref` less than 0, some error must occurred before,
     * so PANIC !!! */
    else
	    panic("cgh:pp->pp_ref is less than zero\n");
}
```
具体实现过程是：由于pp\_ref记录的是当前物理页面是否还在使用，那么如果大于零，就不能对其进行 free操作，直接return，什么都不做。如果pp\_ref==0，说明该页面已经不再需要存在于物理内存中，可以调用LIST_INSERT将其插入到page\_free\_list链表当中了。如果pp\_ref的值小于零，那么输出一个panic警告。
##Thinking 3.2
####了解了二级页表页目录自映射的原理之后，我们知道，Win2k 内核的 虚存管理也是采用了二级页表的形式，其页表所占的 4M 空间对应的虚存起始地址 为 0xC0000000，那么，它的页目录的起始地址是多少呢？ 
答：0xc0000000+((0xc0000000>>12)<<2) = 0xc0000000+0x00300000 = 0xc0300000.  
所以页目录起始地址为0xc0300000。
##Exercise3.4
####完成 mm/pmap.c 中的 boot_pgdir_walk 和 pgdir_walk 函数，实现 虚拟地址到物理地址的转换以及创建页表的功能。
```cpp
static Pte *boot_pgdir_walk(Pde *pgdir, u_long va, int create)
{

    Pde *pgdir_entryp;
    Pte *pgtable, *pgtable_entry;

    /* Step 1: Get the corresponding page directory entry and page table. */
    /* Hint: Use KADDR and PTE_ADDR to get the page table from page directory
     * entry value. */
    pgdir_entryp=pgdir+PDX(va);
    if(*pgdir_entryp&PTE_V)
        pgtable=(Pte *)KADDR(PTE_ADDR(*pgdir_entryp));

    /* Step 2: If the corresponding page table is not exist and parameter `create`
     * is set, create one. And set the correct permission bits for this new page
     * table. */
     else if(create){
        /*temp=(Pte *)alloc(BY2PG,BY2PG,1);
        if(temp==NULL){
            panic("boot_pgdir_walk failed. Run out of memory!\n");
            return NULL;
        }*/
        *pgdir_entryp=PADDR((Pde)alloc(BY2PG,BY2PG,1))|PTE_V;
        //printf("%x\n%x",*pgdir_entryp,PTE_ADDR(*pgdir_entryp));
        pgtable=(Pte *)KADDR(PTE_ADDR(*pgdir_entryp));
     }
     else{
        panic("boot_pgdir_walk failed! No valid memory and create is 0!\n");
        return NULL;
     }
       
    /* Step 3: Get the page table entry for `va`, and return it. */
     pgtable_entry=pgtable+PTX(va);
    return pgtable_entry;

}
```
```cpp
int
pgdir_walk(Pde *pgdir, u_long va, int create, Pte **ppte)
{
    Pde *pgdir_entryp;
    Pte *pgtable;
    struct Page *ppage;

    /* Step 1: Get the corresponding page directory entry and page table. */
    pgdir_entryp=pgdir+PDX(va);
    if(*pgdir_entryp&PTE_V){
        pgtable=(Pte *)KADDR(PTE_ADDR(*pgdir_entryp));
    }

    /* Step 2: If the corresponding page table is not exist(valid) and parameter `create`
     * is set, create one. And set the correct permission bits for this new page
     * table.
     * When creating new page table, maybe out of memory. */
     else if(create){
        if(page_alloc(&ppage)==-E_NO_MEM){
            *ppte=NULL;
            return -E_NO_MEM;
        }
        ppage->pp_ref++;
        *pgdir_entryp=page2pa(ppage)|PTE_V|PTE_R;
        pgtable=(Pte *)KADDR(PTE_ADDR(*pgdir_entryp));
     }
     else{
        //panic("pgdir_walk failed! Page is not valid and create is 0\n");
        *ppte=NULL;
        return -E_INVAL;
     }

    /* Step 3: Set the page table entry to `*ppte` as return value. */
     *ppte=pgtable+PTX(va);
    return 0;
}
```
##Exercise3.5
####实现 mm/pmap.c 中的 boot_map_segment 函数，实现将制定的物理 内存与虚拟内存建立起映射的功能。 
```cpp
void boot_map_segment(Pde *pgdir, u_long va, u_long size, u_long pa, int perm)
{
    int i, va_temp;
    Pte *pgtable_entry;

    /* Step 1: Check if `size` is a multiple of BY2PG. */
    if(size%BY2PG){
        panic("size must be a multiple of BY2PG!\n");
        return;
    }

    /* Step 2: Map virtual address space to physical address. */
    /* Hint: Use `boot_pgdir_walk` to get the page table entry of virtual address `va`. */
    for(i=0;i<size/BY2PG;i++){
        pgtable_entry=boot_pgdir_walk(pgdir,va,1);
        if(!pgtable_entry)
            panic("segment fail!(run out of memory!)\n");
        *pgtable_entry=pa|perm|PTE_V;
        va=va+BY2PG;
        pa=pa+BY2PG;
    }
}
```
##Thinking 3.3
####思考一下 tlb_out 汇编函数，结合代码阐述一下跳转到 NOFOUND 的流程？ 
首先，我们需要明确几个宏定义：
\#define CP0_INDEX \$0  
\#define CP0_ENTRYHI \$10
\#define CP0_ENTRYLO0 \$2
这三个宏定义定义了CP0寄存器中的3个寄存器号。
最终通过bltz指令实现跳转，只要K0寄存器中的值小于0，则会跳转到NOFOUND。而K0实际上是CP0\_INDEX的值的拷贝，也就是说，只要CP0\_INDEX的值小于0，也就是CP0的0号寄存器的值小于0，就会跳转到NOFOUND。如果找到了（即没有跳转），那么会将CP0\_ENTRYHI和 CP0\_ENTRYLO0的值置零，如果跳转了，则就直接将K1的值又重新返还给CP0\_ENTRYHI。
#总结
###实验结果
<center>![](https://raw.githubusercontent.com/tigerzhaoyue/BUAA_MIPS_OS_DOC/master/Picture/Lab2/结果.JPG)  </center>
###小建议
1.实验指导书中的正确结果展示有两句顺序有误。
2.希望指导书中不光给出我们需要的函数填空的相关说明，适当地对例如panic和assert以及page_check函数做相关说明可以让我们更加明白整个程序的运行流程，对我们调试代码也是大有裨益的。

