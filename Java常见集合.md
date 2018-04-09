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

**6. HashMap 和 Hashtable 的区别**
