存什么？  
key value  

Key Value编码
![](http://www.grakra.com/2017/06/17/Leveldb-RTFSC/leveldb_images/leveldb_memtable_key.png)
图来源[http://www.grakra.com/2017/06/17/Leveldb-RTFSC/]  
注意上图中地key_length指的是internal key的length。如果key_length表示user_key的length也应该没有问题，毕竟type+sequence的length是固定的。  
另外type相比sequence编码在更低的地址，这一点在做internal_key的比较时是很重要的，我看到网上有资料把type和sequencede位置画反了。。。  

Memtable Entry在MemTable::Add函数中生成   
```cpp
void MemTable::Add(SequenceNumber s, ValueType type, const Slice& key, const Slice& value)
```
Add的参数分别对应MemTable Entry的sequence，type，user_key和value。type表示该操作是插入还是删除，sequence为系统维护，user_key和value是用户想要存储的key value。  

用什么数据结构存储MemTable Entry？  
SkipList  
[原论文](https://15721.courses.cs.cmu.edu/spring2018/papers/08-oltpindexes1/pugh-skiplists-cacm1990.pdf) (还没看= =)  
[跳表──没听过但很犀利的数据结构](https://lotabout.me/2018/skip-list/)

SkipList in LevelDB  
不允许有重复的Key (SkipList::Insert)  
每次插入以1/4的概率决定是否要要添加层数 (SkipList::RandomHeight)  
插入时按InternalKey进行排序 (Memtable::KeyComparator)  
提供插入和查找接口，没有删除接口，也就是说不支持删除  

InternalKeyComparator  
1. 比较user_key，user_key大的整个internal_key就大  
2. 比较sequence num，sequence num小的internal_key大   

不可能存在user_key和sequence_num同时相同的情况  
ref: InternalKeyComparator::Compare  
（比较sequence num的时候，是将type+sequence共8个字节Decode为uint64再做比较的，type比sequence内存地址更低，little endian模式Deocode为uint64时，type并不会影响sequence之间的大小比较。）  

user_key的大小比较用户是可以自定义的，系统默认提供了一个ByteWiseComparator。  

用户如何自定义比较函数？  
https://github.com/google/leveldb/blob/master/doc/index.md#comparators



