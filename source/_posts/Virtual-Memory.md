title: 虚拟内存
tags:
  - Operating System
  - Virtual Memory
categories: Operating System
date: 2017-11-26 21:09:12
---

　　[虚拟内存](https://zh.wikipedia.org/wiki/%E8%99%9A%E6%8B%9F%E5%86%85%E5%AD%98)是[操作系统](https://zh.wikipedia.org/wiki/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F)对[主存](https://zh.wikipedia.org/wiki/%E9%9A%8F%E6%9C%BA%E5%AD%98%E5%8F%96%E5%AD%98%E5%82%A8%E5%99%A8)、磁盘I/O设备的抽象表示。抽象的目的主要有以下几点：
* 将主存看作硬盘的高速缓存，并根据需要在硬盘和主存之间传送数据
* 为每个进程提供了一个大的、一致的、私有的地址空间，简化程序对内存的管理
* 保护每个进程的地址空间不被其他进程破坏  

　　下图是单个Linux进程看到的虚拟地址空间。  

<center><img src="http://static.lzb.me/blog/%E8%99%9A%E6%8B%9F%E5%86%85%E5%AD%98/%E8%BF%9B%E7%A8%8B%E7%9A%84%E8%99%9A%E6%8B%9F%E5%9C%B0%E5%9D%80%E7%A9%BA%E9%97%B4.svg" style="text-align:center;width:80%"/></center>  

　　事实上，这个时候，是有很多个进程同时运行在操作系统上。主存可能是这样子的：  

<center><img src="http://static.lzb.me/blog/%E8%99%9A%E6%8B%9F%E5%86%85%E5%AD%98/%E5%86%85%E5%AD%98%E4%B8%AD%E8%A3%85%E5%85%A5%E5%A4%9A%E4%B8%AA%E7%A8%8B%E5%BA%8F.svg" style="text-align:center;width:80%"/></center>  
　　上图，一开始加载了1、2、3三个程序，程序2运行结束产生了两个20k空闲，这时候启动了程序4，但是程序4需要的内存是25k，只能把程序3挪一下，合并两个20k，产生40k的空闲，然后将程序4加载到内存。[分时系统](https://zh.wikipedia.org/wiki/%E5%88%86%E6%99%82%E7%B3%BB%E7%B5%B1)，如果直接使用内存条的物理地址，将会非常麻烦。  

<center><img src="http://static.lzb.me/blog/%E8%99%9A%E6%8B%9F%E5%86%85%E5%AD%98/%E7%89%A9%E7%90%86%E5%9C%B0%E5%9D%80%E8%BF%90%E8%A1%8C%E5%A4%9A%E4%B8%AA%E7%A8%8B%E5%BA%8F.svg" style="text-align:center;width:80%"/></center>

　　假设进程1占据了0-1000号地址，进程2占据了1001-2100号地址，两个进程都有mov eax,ds:[100] ,把寄存器中的值写入地址是100的内存中。如果不做任何处理，两个进程会把值写入同一个地址，值会被覆盖。或者在装载程序的时候，把程序2的指令修改成mov eax,ds:[1100]，这个也是非常困难的。  

　　所以操作系统提供了抽象，程序运行的时候，操作的都是逻辑地址，逻辑地址和物理地址的映射，由[内存管理单元(MMU)](https://zh.wikipedia.org/wiki/%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86%E5%8D%95%E5%85%83)维护。进程让cpu执行的指令里面，操作的都是逻辑地址。

<center><img src="http://static.lzb.me/blog/%E8%99%9A%E6%8B%9F%E5%86%85%E5%AD%98/%E5%9F%BA%E5%9D%80%E5%AF%84%E5%AD%98%E5%99%A8.svg" style="text-align:center;width:80%"/></center>

　　内存地址的问题解决后，又出现了的问题，程序太大怎么办？操作系统同时运行多个程序，不可能将运行的程序全部加载到内存中。为了避免互相伤害。操作系统会对程序进行分块装入。  

　　这样做的理论依据是局部性原理，即程序具有访问局部区域里的数据和代码的趋势。通俗来说就是，执行了一条指令，马上又会执行这条指令，访问了某个地址上的数据，很快又会访问这个数据和附近的数据。

<center><img src="http://static.lzb.me/blog/%E8%99%9A%E6%8B%9F%E5%86%85%E5%AD%98/%E9%A1%B5%E8%A1%A8.svg" style="text-align:center;width:80%"/></center>  

　　当程序装载的时候，会给进程分配一个页表（每页4K）讲逻辑地址和物理地址映射起来。每页会有一个有效位，如果设置了有效位，那么地址字段就表示DRAM中有相应的物理页起始位置，如果没有设置，但是有值，说明这个地址指向磁盘上的起始位置，如果，没有设置，也没有值，表示这个虚拟页面还未被分配。

　　比如，当要执行 MOV (0x560) EAX，这个指令，把地址是0x560中的值取出来放到EAX寄存器中。CPU接收到0x560这个地址，先进行拆分，6位页号，12位偏移量，获取页号0x00和偏移量0x560，然后根据页号获取物理地址页的基地址，再加上偏移量，得到物理地址，然后读取数据。中间这个计算过程由MMU完成。  

<center><img src="http://static.lzb.me/blog/%E8%99%9A%E6%8B%9F%E5%86%85%E5%AD%98/%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86%E5%8D%95%E5%85%83MMU.svg" style="text-align:center;width:100%"/></center>  

　　当程序开始执行，执行到某一页，发现没有对应的物理地址的时候，就发生了缺页。比如CPU执行了mov [0x560]  eax，把0x560中的值读出来写到EAX寄存器中，但是0x560 的值还不在内存当中。
* 设置缺页中断 (page fault)
* 缺页中断处理程序读取磁盘（这个过程cpu可能会去干其它事情）
* 选定一个牺牲页（先选一个空闲的物理页，如果满了，执行页面置换算法，选一个不太用的牺牲掉）
* 修改页表
* 重新开始执行 mov [0x560] eax

　　每个进程都要有一个页表，[进程控制块PCB](https://zh.wikipedia.org/wiki/%E8%A1%8C%E7%A8%8B%E6%8E%A7%E5%88%B6%E8%A1%A8)有指向页表的指针，一个进程不能访问另一个进程的地址。页表的访问需要非常快，所以会有一个硬件缓存， 转换缓冲区（TLB）。还有，页表本身可能非常大。比如32位的操作系统，支持4G(2^32)大小的逻辑地址空间。如果每一页4K(2^12)，那么可以有1百万个条目(2^32/2^12)。假设每个条目4B，那么每个进程需要4MB的物理地址空间来存储页表本身。所以又有几种技术，来减小页表本身的大小，多级页表，哈希页表，反向页表等。  

　　内存总是有限的，不可能一直从硬盘读取数据写入到内存中，这种时候，需要进行页面置换。在发生缺页的时候，会进行页面置换。做web开发的时候，redis缓存什么的，也会需要置换。介绍一种算法LRU，最近最少使用。

<center><img src="http://static.lzb.me/blog/%E8%99%9A%E6%8B%9F%E5%86%85%E5%AD%98/LRU%E7%AE%97%E6%B3%95.svg" style="text-align:center;width:100%"/></center>  

　　因为LRU算法硬件很难实现，所以硬件一般用的是效果和LRU差不多的clock算法。  
* 每个页加一个引用位, 默认值为0，无论读还是写，都置为1
* 把所有的页组成一个循环队列
* 选择淘汰页的时候，扫描引用位， 如果是1则改为0（相当于再给该页面一次存活的机会）， 并扫描下一个；如果该引用位是0， 则淘汰该页， 换入新的页面  

<center><img src="http://static.lzb.me/blog/%E8%99%9A%E6%8B%9F%E5%86%85%E5%AD%98/Clock%E7%AE%97%E6%B3%95.svg" style="text-align:center;width:100%"/></center>  

　　用分页管理内存，还是有一些问题，用户视角的内存和实际物理内存分离。用户无法将内存看做一个线性字节数组。对用户来说，内存最好是这样的。这个用户可以理解为JVM。  

<center><img src="http://static.lzb.me/blog/%E8%99%9A%E6%8B%9F%E5%86%85%E5%AD%98/Java%E8%99%9A%E6%8B%9F%E6%9C%BA%E8%BF%90%E8%A1%8C%E6%97%B6%E6%95%B0%E6%8D%AE%E5%8C%BA.svg" style="text-align:center;width:80%"/></center>

　　分段就是这种支持用户视角的内存管理方案。
<center><img src="http://static.lzb.me/blog/%E8%99%9A%E6%8B%9F%E5%86%85%E5%AD%98/%E5%88%86%E6%AE%B5.svg
" style="text-align:center;width:80%"/></center>

　　最后，就是段页结合，让分段面向用户，地址指明了段名和段内偏移。让分页面向硬件，对用户透明。先根据段找到页，再找到物理内存地址。
<center><img src="http://static.lzb.me/blog/%E8%99%9A%E6%8B%9F%E5%86%85%E5%AD%98/%E6%AE%B5%E9%A1%B5%E7%BB%93%E5%90%88.svg
" style="text-align:center;width:100%"/></center>
