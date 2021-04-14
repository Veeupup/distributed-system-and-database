为什么要有buffer pool？  
减少磁盘IO，提升读写效率。对于读，如果磁盘页面已经加载到buffer pool，就不用从磁盘读了；对于写，对某一页面多次写入，只需将最终结果写入磁盘，从而减少写入磁盘的次数。  
数据库中的buffer pool更接近于操作系统中cache的概念，操作系统中buffer和cache是有区别的，具体参见[Cache 和 Buffer 都是缓存，主要区别是什么？ - pansz的回答 - 知乎](https://www.zhihu.com/question/26190832/answer/830615125)  

内存的大小有限，不可能将所有磁盘文件都加载到内存中，当buffer pool满了之后，再想从磁盘读页面的时候就需要将已经缓存到buffer pool的页面踢出。那选择将哪些页面踢出就很讲究了，核心目标就是尽可能的cache命中，选择踢出哪些页面的策略就叫做页面置换算法。  

有哪些页面置换算法？  
LRU：踢出最久没有使用过的页面
CLOCK：踢出较久没有使用过的页面

LRU怎么实现？  
双向链表，每次使用一个页面就将这个页面插入到链表的头部，每次都淘汰链表尾部的页面  可以加一个hash table，用于以O(1)的时间复杂度查找cache是否命中  

LRU有什么问题？  
最近使用的页面真的之后更可能使用到吗？ 
Sequential flooding：比如一次scan需要读很多页面，这些页面基本都只用这一次，但是会把很多“热数据页”挤出buffer pool。 

LRU改进  
> 在LRU列表的中间位置打了一个old标识，可以简单的理解为将LRU列表分为两个部分，这个标记到LRU列表头部的页面称之为young的页面，这个标志到LRU列表尾部的页面称之为old页面。再进行抽象的话，我们简单地理解为缓存池被分成两个池子，一个叫young池子，一个叫old池子。当一个页面从磁盘上加载缓存池中的时候，会将它排放在这个old标识之后的第一个位置，也就是说放在了old池子中。这个机制的作用就是，在做大表的一次性全表扫描的时候，大量新进来的页面，是存放在old池子中的，当old池子的大小不够缓存新进来的页面的时候，也只是在old池子中内部进行循环冲洗，这样就不会冲洗young池子中的热点页面，从而保护了热点页面。这就是LRU列表的简单机制。 
[InnoDB Buffer Pool详解 - 爱折腾的邦邦的文章 - 知乎](https://zhuanlan.zhihu.com/p/65811829) 

Dirty  
往一个页面写，首先是写到这个页面对应的缓存中，如果写入的数据只存在于缓存中，还没有flush到磁盘中，那这个页面就是dirty的。dirty的页面也是可以被踢出的，但踢出之前得先flush。  

Pin & Unpin  
正在被使用的页面是不能被踢出的。一个页面可能会被多个用户使用，会为每个页面保存一个pin_count，记录当前有多少用户在使用这个页面。每次使用页面的时候，都会Pin住这个页面，pin_count加1，不使用的时候就Unpin，对应的pin_count减1，只有当一个页面的pin_count为0时，才可以踢出该页面。

还有哪些优化点？  
Multiple Buffer Pools: reduce latch contention and improve locality.   
Pre-Fetching: Sequential Scans, Index Scans  
Scan Sharing: If a query wants to scan a table and another query is already doing this, then the DBMS will attach the second query's cursor to the existing cursor.  
Buffer Pool Bypass: The sequential scan operator will not store fetched pages in the buffer pool to avoid overhead   
[CMU 15445 2020 buffer pools lec](https://15445.courses.cs.cmu.edu/fall2020/slides/05-bufferpool.pdf)

