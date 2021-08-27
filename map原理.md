## 结构

![jiegou](https://img.draveness.me/2020-10-18-16030322432679/hmap-and-buckets.png)

1.count:表中元素总数量

2.B：桶数量，由于hashmap的桶数量一定是2^n，所以B代表的是n，而不是实际的桶数量

3.hash0：用于初始化hash函数的随机种子，作用是增加hash函数的随机性，减少hash碰撞

4.oldbuckets：指向老桶的指针(即扩容前的老桶)

每个map里保存了多个bmap，bmap就是hash桶，bmap的大小为8,桶内元素如果超过8个并且发生hash碰撞，则会使用extra指向的溢出桶保存溢出的元素

至于每个bmap和溢出桶之间的对应关系，由bmap中的overflow指针维护，该指针指向当前bmap对应的溢出桶

## 
