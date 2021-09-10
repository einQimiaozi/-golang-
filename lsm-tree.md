lsm tree是一种类似树的数据结构，用于磁盘顺序写以提高数据库的写入性能，在传统索引(如b+树)情况下，不同的索引即便在内存结构中数据相邻，但是其磁盘中所属的block可能不在同一个区域，会造成随机写，降低io效率

结构：

![lsm](https://s7.51cto.com/images/blog/202008/13/556f314e54d0f06a120fe3a9c181a732.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

lsm采用多级存储(类似树)

MemTable：是LSM Tree在内存中还未刷盘的写入数据，这里面的数据可以修改。是一个有序的树状结构，如RocksDB中的跳表。

SSTable（Sorted String Table）：是一个键是有序的，存储字符串形式键值对的磁盘文件。当SSTable太大时，可以将其划分为多个block；我们需要通过 下图中的Index 记录下来每个block的起始位置，那么就可以构建每个SSTable的稀疏索引。这样在读SSTable前，通过Index结构就知道要读取的数据块磁盘位置了。
图中的每一层就是一个sstable

查询：
 - 先从memtable中查询，如果没有命中，则进入磁盘查询
 - 磁盘查询时从lsm顶部向下查询，通过sstable的范围用类似归并的方法查询
 - 查询可以是单条查询也可以是范围查询 



