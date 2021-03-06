# HashMap和ConcurrentHashMap原理

## 简单阐述

- 定义一个数组，其长度为2幂，设默认为16，空间利用率75%左右

- 空间利用率达到预设值，进行数组扩容，以2倍速进行，将其容量始终保持为2的幂次方

- 插入时，利用Hash函数计算Key的索引，将其存放数组对应的位置

- 取值时，利用Hash函数计算Key的索引，直接取数组对应位置的值

- 存在不通的Key经Hash计算后其索引相同，则需要冲突解决处理

## 数组空间管理2幂次方

存在很多场景，使用位操作。若是以Hash & (length - 1)确定其index，则2的幂次方长度更为重要。
核心是2幂次方减少冲突的概率，有两个方面：
(1)(length - 1) 所有位全为1，其&操作具有唯一性
(2)(length - 1) 扩容后只有左边高位存在差别，其余位置全为1，保证原数组中元素扩容索引不改变。
若非2的幂次方，如下例子：
length = 6, 则index = 5
index & 1 = 1
index & 3 = 1

扩容为length=10, index = 9
index & 1 = 1
index & 3 = 3

## 计算Hash值

哈希(Hash)函数是一个映象，将关键字的集合映射到某个地址集合上，它的设置很灵活，只要这个地址集合的大小不超出允许范围即可；
1、直接定址法，以数据元素关键字k本身或它的线性函数作为它的哈希地址
2、平方取中法，先取关键字的平方，然后根据可使用空间的大小，选取平方数是中间几位为哈希地址。
3、除留余数法，模p取不大于表长且最接近表长m素数时效果最好

所有的字符串哈希算法都是基于对字符编码的迭代运算，保证每个字母参与到Hash值的计算并且其位置关系在Hash值中有体现，若字符串太长可取前缀。
如下是经典的BKDRHash算法：

```C++
// BKDR Hash Function
unsigned int BKDRHash(char *str)
{
    unsigned int seed = 131; // 31 131 1313 13131 131313 etc..
    unsigned int hash = 0;
    while (*str)
    {
        hash = hash * seed + (*str++);
    }
    return (hash & 0x7FFFFFFF);
}
```

## Hash值冲突解决

1、再散列法
其基本思想是：当关键字key的哈希地址p=H（key）出现冲突时，以p为基础产生另一个哈希地址p1，如果p1仍然冲突，再以p为基础产生另一个哈希地址p2，...，直到找出一个不冲突的哈希地址pi ，将相应元素存入其中
2、再哈希法
    构造多个不同的哈希函数，若hash1产生的Value冲突，则使用Hash2
3、链地址法
基本思想是将所有哈希地址为i的元素构成一个称为同义词链的单链表，并将单链表的头指针存在哈希表的第i个单元中，因而查找、插入和删除主要在同义词链中进行。链地址法适用于经常进行插入和删除的情况。

## ConcurrentHashMap

1、细分锁粒度 Java
HashMap使用全局锁保证数据的一致性，在多线程环境下，对锁的挣用将非常激烈进而影响性能。
任何环境下，数据的一致性都必须满足，所以并发MAP通过将锁的粒度减小，提高的并发量实现并发MAP。
主流方式就是分段加锁，提高并发度。段与段之间不存在竞争关系，段内仍然通过锁来保证数据一致性。

2、空间换时间 golang
使用了空间换时间策略，通过冗余的两个数据结构(read、dirty),实现加锁对性能的影响。
通过引入两个map将读写分离到不同的map，其中read map提供并发读和已存元素原子写，而dirty map则负责读写。
sync.Map并不适合同时存在大量读写的场景,大量的写会导致read map读取不到数据从而加锁进行进一步读取,同时dirty map不断升级为read map

3、写时拷贝Map