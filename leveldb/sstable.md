[leveldb sstable](https://leveldb-handbook.readthedocs.io/zh/latest/sstable.html) 这篇讲leveldb中sstable的文章确实不错，但刚开始看仍然会很懵逼。  
![](https://leveldb-handbook.readthedocs.io/zh/latest/_images/sstable_logic.jpeg)  
比如说这张图，当对sstable有一定理解后，回头看，这张图确实画得很好，leveldb中的sstable就是这么回事儿。但是一开始就看到这张图，我的第一反应就是——好复杂，不就是存个key value吗，为什么有这么多block，分这么多层？  
不妨去想想，如果我去设计sstable，会怎么设计？最开始设计sstable的时候，不可能一开始就会分这么多层吧，一定是为了解决某个问题，然后引入了某个block。  

[SSTable and Log Structured Storage: LevelDB](https://www.igvita.com/2012/02/06/sstable-and-log-structured-storage-leveldb/)
> A "Sorted String Table" then is exactly what it sounds like, it is a file which contains a set of arbitrary, sorted key-value pairs inside.  

sstable的本质就是字面意思，"Sorted String Table"，存储按序排列key value字符串，就像下面这张图一样。  
![](/leveldb/images/sstable.png)  

现在再来看看leveldb中sstable的设计。  
首先来看看我们核心要存储的key value放在什么地方？  
data block  

我们先忽略data block的具体设计，只需要知道data block是存储key value就行了。  

一个data block不要存储太多的数据，就会把key value存储在多个data block中。那现在sstable就变成  
![](/leveldb/images/data_block.png)  


