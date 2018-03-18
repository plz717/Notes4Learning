数据结构中有数组和链表来实现对数据的存储，但这两者基本上是两个极端。
数组的特点是：寻址容易，插入和删除困难；
链表的特点是：寻址困难，插入和删除容易。

那么我们能不能综合两者的特性，寻址容易，插入删除也容易的数据结构？
                                                                                         -----哈希表（Hash table）

#Hash
将结点按其关键字的地址存储到HashMap中的过程称为散列(hashing)
> 由于不同对象的地址一般不同,因此原本多个不同地址的对象,经过hashing, 可以具有在散列表中相同的index.

实现:http://www.cnblogs.com/lizhanwu/p/4303410.html

#HashMap
HashMap是一个用于存储Key-Value键值对的集合，每一个键值对也叫做Entry。这些个键值对（Entry）分散存储在一个数组当中.
对于HashMap，我们最常使用的是两个方法：Get 和 Put。
###1.Put方法的原理
比如调用 hashMap.put("apple", 0) ，插入一个Key为“apple"的元素。这时候我们需要利用一个哈希函数来确定Entry的插入位置（index）：
index =  Hash（“apple”）
假定最后计算出的index是2，就把这个entry插入在数组中index为2的位置
###2.Hash冲突
因为HashMap的长度是有限的，当插入的Entry越来越多时，再完美的Hash函数也难免会出现index冲突的情况。----------可以利用**链表**来解决
HashMap数组的每一个元素不止是一个Entry对象，也是一个链表的头节点。**每一个Entry对象通过Next指针指向它的下一个Entry节点**。当新来的Entry映射到冲突的数组位置时，只需要插入到对应的链表头部,而不是尾部(因为hashmap的设计者认为后来插入的entry有更大的可能性被查询)

###3.Get方法的原理
首先会把输入的Key做一次Hash映射，得到对应的index：
index =  Hash（“apple”）
由于刚才所说的Hash冲突，同一个位置有可能匹配到多个Entry，这时候就需要顺着对应链表的头节点，一个一个向下来查找。

###4.HashMap的默认初始长度
默认初始长度16, 且每次自动扩展或者手动初始化时, 长度都必须是2的整数幂
通常, 我们采用取模的方法求取index, 即:
	index =  HashCode（Key） % Length
	但是这种方法十分低效
高效Hash运算-----**位运算**
	index =  HashCode（Key） &  （Length - 1）
	其中, Length是HashMap的长度
下面我们以值为“book”的Key来演示整个过程：
> 
1.计算book的hashcode，结果为十进制的3029737，二进制的101110001110101110 1001。
2.假定HashMap长度是默认的16，计算Length-1的结果为十进制的15，二进制的1111。
3.把以上两个结果做与运算，101110001110101110 1001 & 1111 = 1001，十进制是9，所以 index=9。

因此, 长度16或者其他2的幂的HashMap，Length-1的值使得所有二进制位全为1，这种情况下，index的结果等同于HashCode后几位的值。**只要输入的HashCode本身分布均匀，Hash算法的结果就是均匀的。**
而其他长度的HashMap, 会使得计算出来的index的分布十分不均匀.

###5.Hashmap Resize
Hashmap在插入元素过多的时候需要进行Resize，Resize的条件是

	HashMap.Size   >=  Capacity * LoadFactor
	LoadFactor一般为0.75
Hashmap的Resize包含**扩容**和**ReHash**两个步骤:
-   扩容: 创建一个新的Entry空数组，长度是原数组的2倍。
-   ReHash: 遍历原Entry数组，把所有的Entry重新Hash到新数组。
    -    ReHash在**并发**的情况下可能会形成**链表环**。当调用Get查找一个不存在的Key，而这个Key的Hash结果恰好等于一个带有环形链表的index，程序将会进入死循环！


###利用HashMap统计出现次数
> 问题：统计一个字符串集合中，出现次数最多的字符串 
思路：
    1.建立一个哈希映射（HashMap），其key为“字符串”，value为“字符串出现次数”
    new HashMap<String, Integer>()
    2.
    for (String str)                             //遍历字符串集合
    {
    	if (map.containsKey(str))   //如果字符串已存在，将键为该字符串的值加1
    	{
    		map.put(str, map.get(str)+1)
    	}
    	else                                    //否则添加键值对“字符串,出现次数1”
    		map.put(str, 1)  
    }
    3.遍历HashMap，统计value最大的key。 
> 注意:也可以利用triel树统计频数
###如何获得一个对象的HashCode
HashCode是由对象导出的一个整型值。散列码是没有规律的，**两个不同的对象的hashcode基本上不会相同**。对象间进行比较时，默认比较的是两个对象的hash code的值是否相同。
Java中内置HashCode函数可以轻松获得一个对象的Hashcode, 但是python中并没有内置,需要手动实现:http://outofmemory.cn/code-snippet/2325/Python-achieve-java-huozhe-net-getHashCode-function

###利用Hash进行海量数据处理
	分而治之/hash映射 + hashMap统计 +堆/快速/归并排序
-  分而治之
    -   将海量数据(ip地址/字符串)映射到N个文件中:
        >  每个对象的HashCode % N
-    对于每个小文件, 构建一个key为(ip地址/字符串), value为出现次数的HashMap, 得到次数出现最多的entry
-    将N个entry中各自出现最多的key(ip地址/字符串), 根据堆排序/快速排序/归并排序, 得到最多出现次数的key
