# JavaEE面试问题总结

##### @Author LucI_PhAN

## 基础篇

### Java常见集合

**1. List 和 Set 区别**

Java中的数组是大小固定的，并且一个数组只能存放类型一样的数据。  
Java中的集合可以存储和操作数目不固定的一组数据。集合中只能存放引用类型的数据，不能存放基本数据类型。  
Collection是最基本的集合接口，声明了适用于Java集合（只包括set和list）的通用方法。Set和List都实现了Collection接口。

1.1 列表List  
List的特性是其元素以线性方式存储，集合中可以存放重复对象。  
List接口主要实现类：
- ArrayList:底层数据结构是数据。效率高，线程不安全。可以对元素进行随机的访问，查找快，增删慢。
- Vector：底层数据结构是数组。效率低但线程安全。查找快，增删慢。
- LinkedList：底层数据结构是链表。效率高，线程不安全。查找慢，增删快。

List保证插入顺序排序，集合内元素可以重复且可以根据索引直接操作元素。

1.2 集合Set  
Set是最简单的一种集合。集合中的对象不按特定的方式排序，并且没有重复对象。  
Set接口主要实现类：
- HashSet：底层数据结果是哈希表（一个元素为链表的数组），哈希表底层依赖两个方法：hashCode()和equals()。所以HashSet中存放的引用对象必须重写了Object类中的这两个方法。
- TreeSet：底层数据结构是红黑树（是一个自平衡的二叉树）。它保证元素的排序方式。（分为自然排序和比较器排序）
- LinkedHashSet：由链表保证元素有序，由哈希表保证元素唯一。

Set存储和取出顺序不一致，Set中存储的元素是惟一的，不允许重复元素，且不能根据索引获取元素。

参考资料：  
[《我们为什么要使用List和Set（List,Set详解）》](https://blog.csdn.net/qq_34149805/article/details/68943004)  
[《浅谈Java中的Set、List、Map的区别》](https://www.jianshu.com/p/7a8c2e895b5d)

**2. Set和hashCode以及equals方法的联系**

Set的实现类HashSet是一个无序不重复的集合，它根据hashCode()返回值和equals()来判断两个对象是否相同。Object中的hashCode()在默认情况下为了确保这个哈希值的唯一性，是通过将该对象的内部地址转换成一个整数来实现的。
而equals()在Object中是直接使用==（比较地址值）来判断的。因此，这不是我们通常所说的“对比是否相等”。  
因此，为了保证HashSet的元素唯一性，我们必须重写hashCode()和equals()方法来确保判断元素是否重复时是根据元素内容而不是地址值判断的。  

使用hashSet的add()方法插入元素时：
- hashSet会调用元素的hashCode()方法
- 根据hashCode()方法的返回值，确定元素要插入的位置
- 如果该位置上已经存在元素了，则调用equals()方法比较
- 如果equals()返回true，认为两个元素重复，则不存入欲插入的元素
- 如果equals()返回false，则新元素被添加到另一个位置（根据冲突检测来决定哪一个位置）

因此，使用hashSet存入一个引用类型的元素，该引用类型必须重写Object的hashCode()和equals()方法。

但是对于TreeSet，则并不需要重写这两个方法：因为TreeSet底层是红黑树，只需要为那个类实现Comparable接口病重写compareTo()方法就可以同时完成两个工作：排序和消除重复。

参考资料：  
[《深入详解SetHash的元素为什么要重写hashCode和equals方法》](https://www.2cto.com/kf/201706/648015.html)  
[《关于TreeSet 的equals 和hashcode（）问题》](https://bbs.csdn.net/topics/392043416)

**3. List 和 Map 区别**

3.1 List  
在Java文档中，List的定义是：  
一个有序的Collection，使用这个接口可以精确掌控元素的插入位置，还可以根据index获取相应位置的元素。List和Set不同，能够插入重复的数据。  
在原来Collection的基础上，List是一个可以指定索引，有序的容器。

以实现类ArrayList来举例，查看源码，ArrayList里面封装的是一个Object数组，实例化时默认数组大小为10

3.2 Map  
在Java文档中，Map的定义为：  
一个有键值对的对象。一个map不能包含重复的键值对。每一个键最多只能匹配一个值。这个接口可以取代Dictionary类（一个抽象类，不是接口）。

以实现类HashMap为例，查看源码，里面封装的是一个内部类Entry的数组，默认容量是16。

在不牵扯底层实现的前提下，
- List允许有重复元素，Map不允许重复的键但允许重复的值。
- List允许任意数量的Null值，Map只允许一个Null键但允许任意数量的Null值（此处因实现类不同而有具体的区别）
- List及其所有的实现类保持了每个元素的插入顺序，而Map对元素进行无序存储（但某些实现类对元素进行了排序-TreeMap， LinkedHashMap）

而如果深入底层，Map集合是一个关联数值，它包含两组值：一组是所有key组成的集合，key值不允许重复，而且Map不会保存key加入的顺序。因此这些key可以组成一个Set集合。  
另一组是value组成的集合，因为Map集合的value完全可以重复，所以这些value可以组成一个List集合。  

Map集合的keySet()方法可以返回一个Set集合。但Map的values()方法并未返回一个List集合。HashMap和TreeMap的values()方法直接返回的是HashMap$Value对象和TreeMap$Value对象。  
按照通俗的理解，values应该是一个List集合，因为Map的多个value允许重复。但实际上，HashMap和TreeMap的values()方法实现相当巧妙。这两个Map对象的values()方法返回的是一个不存储元素的Collection集合。当程序便利该Collection时，实际就是Map对象的value，这样是为了减低性能开销。

参考：  
[《Java 语言中 List、Set 和 Map 的区别》](https://blog.csdn.net/defonds/article/details/47837867)  
[《深入源码分析Map与List的关系》](https://blog.csdn.net/canot/article/details/51249777)  
[《List、Set、Map的源码初级分析》](https://blog.csdn.net/kklt21cn/article/details/41786577)

**4. Arraylist 与 LinkedList 区别**

通常情况下，ArrayList和LinkedList的区别如下：
- ArrayList是实现了基于动态数组的数据结构；而LinkedList是基于链表的数据结构。
- 对于随机访问get和set，ArrayList要优于LinkedList，因为LinkedList要移动指针，无法通过下标直接获取元素。
- 对于添加和删除操作，普遍认知为LinkedList比ArrayList快，因为ArrayList要移动数据。但真实情况并非一定如此。

查看源码，ArrayList想要get(int index)元素时，直接返回index位置上的元素，而LinkedList需要通过for循环查找，虽然LinkedList已经在查找上做出了优化（index<size/2，从左开始查找，否则从右），但仍然比ArrayList慢。  
ArrayList想要在指定位置插入或删除元素时，只要的耗时在System.arraycopy动作，移动index后面所有的元素。而LinkedList主要耗时在通过for循环找到index，然后直接插入或删除。这就导致了两者的速度并非是LinkedList一定快。

执行插入或删除元素时，有两个因素将决定效率：插入的数据量和插入的位置。
通过测试，当数据量较小时，两者效率差不多，没有显著的差别。当数据量较大（容量的1/10处开始），LinkedList的效率就没有ArrayList高了，特别到一半以及好变的位置插入式，LinkedList的效率明显低于ArrayList，而且数据量越大，越明显。

参考：  
[《java集合框架05——ArrayList和LinkedList的区别》](https://www.cnblogs.com/shanheyongmu/p/6439202.html)

**5. ArrayList 与 Vector 区别**

ArrayList和Vector都继承了相同的父类和实现了相同的接口；  
底层都是数组实现的；  
初始默认长度都为10。  

它俩的不同点在于：
- Vector中的public方法多数添加了synchronized关键字，以确保方法同步，而ArrayList中并没有；所以Vector线程安全，ArrayList线程不安全。
- 两者的扩容不同。ArrayList有两个属性：存储数据的数组elementData，和存储记录数目的size；而Vector有三个属性：存储数据的数组elementData，存储记录数目的elementCount，还有扩展数组大小的扩展因子capacityIncrement。导致二者扩容方式不同：虽说都采用的是线性连续存储空间，当存储空间不足时，ArrayList默认增加为原来的50%，Vector默认增加为原来的一倍。

参考：  
[《ArrayList和Vector的区别》](https://segmentfault.com/a/1190000007335150)

**6. HashMap 和 Hashtable 的区别**

HashMap和HashTable都实现了Map接口，它们的区别主要在于：线程安全性，同步，以及速度。
- HashMap几乎可以等价于HashTable，但HashMap是线程不安全的，且可以接受null作为一个键或多个值，而HashTable不论键值都不允许null。
- HashMap是线程不安全的，但HashTable是线程安全的，如果没有正确的同步机制的话，多个线程是不能共享HashMap的。而JDK1.5则提供了ConcurrentHashMap，作为HashMap的替代，而且比HashTable的扩展性更好。
- 另一个区别则在于迭代器。HashMap提供了快速失败机制的Iterator迭代器；而HashTable的enumerator迭代器不是基于快速失败机制的。所以HashMap不能再多线程下发生并发修改（迭代过程中被修改），但使用迭代器本身的remove()方法则不会抛出异常。
- HashTable是线程安全的，所以在单线程环境下它比HashMap要慢。如果不是多线程的情况下，HashMap的性能要好过HashTable。

HashMap可以通过工具类Collections实现同步：
``` java
Map m = Collections.synchronizeMap(hashMap);
```

参考：  
[《HashMap和Hashtable的区别》](http://www.importnew.com/7010.html)

**7. HashSet 和 HashMap 区别**

7.1 HashSet  
HashSet实现了Set接口，它仅仅存储对象。  
通过add()方法将元素放入Set中，使用成员对象来计算hashCode值，对于两个对象来说hashCode可能相同，这时通过equals()方法来判断对象的相等性，如果两个对象不同的话，则返回false。  
HashSet底层是通过HashMap来实现的，较直接使用HashMap，HashSet较慢。  
HasheSet内部使用HashMap，它将元素存储为键和值（把存储的值作为key）

7.2 HashMap  
HashMap实现了Map接口，存储的是键值对。  
通过put()方法将元素放入map中，使用键对象来计算hashCode值。  
HashMap比较快，因为是使用唯一的键来获取对象。  
HashMap使用后台数组（backing array）作为桶，并使用链表（LinkedList）存储键/值对。

参考：  
[《图解HashMap和HashSet的内部工作机制》](http://www.importnew.com/21841.html)  
[《HashMap和HashSet的区别》](http://www.importnew.com/6931.html)

**8. HashMap 和 ConcurrentHashMap 的区别**

HashMap本质是数组加链表。根据Key取得hash值，然后计算出数组的下标；如果多个key对应到同一个下标，就用链表串起来，新插入的在前面。  
ConcurrentHashMap是在HashMap的基础上，将数据分为多个segment（默认为16个），然后每次操作对一个segment加锁，避免多线程锁的几率，从而提高并发效率。

查看ConcurrentHashMap源码，它引入了一个分段锁的概念，具体可以理解为把一个大的Map拆分成了N个小的HashTable，根据key.hashCode()来决定把key放到哪个HashTable中。  
在ConcurrentHashMap中，就是把Map分成了N个segment。在put和get的时候，都是先根据key.hashCode()算出放在哪个segment。   

ConcurrentHashMap和HashTable主要区别就是围绕着锁的粒度以及如何锁。ConcurrentHashMap是一个分段的HashTable，根据自定的hashCode()算法生成的对象来获取对应的hashCoide的分段块进行加锁，而不是采用整体加锁，提高了效率。

[《HashMap与ConcurrentHashMap的区别》](https://www.cnblogs.com/signheart/p/21d463eebb54f3e9139da3d43ee7bfda.html)  
[《HashMap和ConcurrentHashMap的区别，HashMap的底层源码。》](https://www.cnblogs.com/remember-forget/p/6021644.html)

**9. HashMap 的工作原理及代码实现，什么时候用到红黑树**

在JDK1.6中，HashMap采用位桶+链表实现，即使用链表来处理hash冲撞，同一hash值得链表都存储在一个链表里。但是当位于一个桶中的元素较多（即hash值相等的元素较多时），通过key值依次查找的效率会变得非常低下（链表只能通过遍历来获取需要的数据）。  
针对这种情况，在JDK1.8中，HashMap采用位桶+链表+红黑树实现。当链表传唱度超过阈值（8）时，将链表转换成红黑树，这样可以大大减少查找时间。

HashMap的实现原理：
- HashMap基于hashing原理，通过put()和get()方法存储和获取对象。当将键值对传递给put()方法时，将调用相应对象的hashCode()方法来计算hashCode，然后根据hashCode来找到桶的相应位置来存储值对象。当获取对象时，通过键对象的equals()方法找到正确的键值对，然后返回值对象。  
- 当发生哈希碰撞时，HashMap使用LinkedList来解决碰撞问题——对象将被存储在LinkedList的一个节点中。即当两个不同键对象的hashCode相同时，它们会被存储在同一个桶位置的LinkedList中。
- 在JDK1.8中，如果一个位桶中的元素个数超过TREEIFY_THRESHOLD（默认为8）时，就使用红黑树替换链表，从而提高速度。这个替换的方法被称为树形化（treeifyBin）

总之，JDK1.8之后哈希表的添加，查找，删除，扩容方法都增加了一种节点为TreeNode的情况。
- 添加时，当桶中链表个数超过8时会转换成红黑树。
- 删除，扩容时，如果桶中结构为红黑树，并且树中元素个数太少的话，会进行修剪或者直接直接还原成链表结构。
- 查找时即使哈希函数不优导致大量元素集中在一个桶中，由于有红黑树结构，性能也不会降低。

参考：  
[《Java 集合深入理解（17）：HashMap 在 JDK 1.8 后新增的红黑树结构》](https://blog.csdn.net/u011240877/article/details/53358305)  
[《HashMap底层实现原理》](https://blog.csdn.net/yinbingqiu/article/details/60965080)

**10. 多线程情况下HashMap死循环的问题**

HashMap采用数组链表来解决Hash冲突，因为是链表结构，就可能形成闭合的链路。  
在单线程情况下，只有一个线程对HashMap的数据结构进行操作，是不可能产生闭合的回路的。  
只有在多线程并发的情况下才会出现这种情况，那就是在执行put()操作时，如果size> initialCapacity*loadFactor，这时候HashMap就会进行ReHash操作，随之HashMap的结构就会发生变化。而如果多于一个线程同时出发了ReHash操作，就可能产生闭合的回路。  
在ReHash中，最关键的一步操作是transfer(ENtry[] newTable)，这个操作会把当前ENtry[] table数组的全部元素转移到新的table中。而这个transfer的过程在并发环境下会发生错误，导致数组链表中的链表形成循环链表，在后面的get操作时e = e.next操作无限循环，具体表现为CPU使用率100%。

``` java
void transfer(Entry[] newTable) {
        Entry[] src = table;
        int newCapacity = newTable.length;
        for (int j = 0; j < src.length; j++) {
            Entry<K,V> e = src[j];
            if (e != null) {
                src[j] = null;
                do {
                    Entry<K,V> next = e.next;//假设第一个线程执行到这里因为某种原因挂起
                    int i = indexFor(e.hash, newCapacity);
                    e.next = newTable[i];
                    newTable[i] = e;
                    e = next;
                } while (e != null);
            }
        }
    }
```
当线程1执行到注释点被挂起后，线程二执行完毕了ReHash；  
线程一被调度回来执行。

先是执行 newTalbe[i] = e;
然后是e = next，导致了e指向了key(7)，
而下一次循环的next = e.next导致了next指向了key(3)
把key(7)摘下来，放到newTable[i]的第一个，然后把e和next往下移。  
e.next = newTable[i] 导致  key(3).next 指向了 key(7)
注意：此时的key(7).next 已经指向了key(3)， 环形链表就这样出现了。

参考：  
[《疫苗：JAVA HASHMAP的死循环》](https://coolshell.cn/articles/9606.html)  
[《HashMap在并发下可能出现的问题分析》](https://yq.aliyun.com/articles/38431)  
[《深入理解JAVA集合系列三：HashMap的死循环解读》](https://www.cnblogs.com/dongguacai/p/5599100.html)
