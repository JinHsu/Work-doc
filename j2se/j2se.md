## 面向对象的三大特征和六大原则

- 三大特征：封装、继承、多态
- 六大原则：单一职责原则、开闭原则、里氏替换原则、依赖倒置原则、接口隔离原则、迪米特法则



## 四种修饰符的限制范围

| 类内部    | 本包 | 子类 | 外部包 |      |
| --------- | ---- | ---- | ------ | ---- |
| public    | √    | √    | √      | √    |
| protected | √    | √    | √      | ×    |
| default   | √    | √    | ×      | ×    |
| private   | √    | ×    | ×      | ×    |

## 自动拆装箱如何实现



```java
Integer i = 8; // 自动装箱 Integer.valueOf(8)
int j = i; // 自动拆箱 Integer.intValue()
```



## 序列化与反序列化

### 序列化的主要作用包括两个方面：

- 实现网络中对象的传送，即在网络进程间传递对象。
- 实现内存中对象的持久化，即将对象保存到本地磁盘中。

### 主要应用场景：

- Java远程方法调用（RMI），即允许一个Java虚拟机上运行的Java程序调用其他虚拟机运行内存中对象的方法，即使这些虚拟机运行于物理隔离的不同主机上。
- 分布式系统中不同服务器间共享的JavaBean对象都需要先序列化为二进制数据，再在网络中传输。
- Session钝化机制：Web容器将一些session先序列化写入本地磁盘，在需要使用时再将对象从磁盘还原到内存中去，以减轻服务器的内存负担。

```java
OutputStream out = new FileOutputStream("D:\\Jerry.txt");
ObjectOutputStream oos = new ObjectOutputStream(out);
oos.writeObject(new Person("Jerry", 18));
oos.flush();

InputStream is = new FileInputStream("D:\\Jerry.txt");
ObjectInputStream ois = new ObjectInputStream(is);
Person jerry = (Person) ois.readObject();
```

### 自定义序列化策略

实现`java.io.Externalizable`接口，并重写`WriteExternal(ObjectOutput out)`和`readExternal(ObjectInput in)`方法。

> 被static和transient修饰的变量不会被序列化





## TreeMap

底层使用的是红黑树（），插入时自动排序，取出时需要中序遍历





## LinkedHashMap

保证插入的有序性，取出时的顺序就是插入时的顺序，可以应用于LRU。





## HashMap

#### 常量

```java
// 默认的初始容量，必须是2的幂。为了快速计算数组下标
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; 
// 最大容量（必须是 2 的幂且小于 2 的 30 次方，传入容量过大将被这个值替换）  
static final int MAXIMUM_CAPACITY = 1 << 30;  
// 装载因子：哈希表在其容量自动增加之前可以达到多满的一种饱和度百分比，其衡量了一个散列表的空间的使用程度，负载因子越大表示散列表的装填程度越高，反之愈小。
static final float DEFAULT_LOAD_FACTOR = 0.75f;  
// JDK1.8特有  
// 当 hash 值相同的记录超过 TREEIFY_THRESHOLD，会动态的使用一个专门的红黑树实现来代替链表结构，使得查找时间复杂度从 O(n) 变为 O(logn)  泊松分布算法
static final int TREEIFY_THRESHOLD = 8;  
// JDK1.8特有  
// 也是阈值，同上一个相反，当桶(bucket)上的链表数小于 UNTREEIFY_THRESHOLD 时红黑树转链表  
static final int UNTREEIFY_THRESHOLD = 6; 
// JDK1.8特有  
// 树的最小的容量，至少是 4 x TREEIFY_THRESHOLD = 32 然后为了避免(resizing 和 treeification thresholds) 设置成64  
static final int MIN_TREEIFY_CAPACITY = 64;
```

#### reHash

```java
// key 的 hash　值的计算是通过hashCode()的高16位异或低16位实现的：(h = k.hashCode()) ^ (h >>> 16)
// 主要是从速度、功效、质量来考虑的，这么做可以在数组 table 的 length 比较小的时候，也能保证考虑到高低 Bit 都参与到 Hash 的计算中，同时不会有太大的开销。在混合了原始 hashCode 值的高位和低位后，加大了低位的随机性，而且混合后的低位掺杂了高位的部分特征，这样高位的信息也被变相保留下来，这就使得 hash 方法返回的值，具有更高的随机性，减少了冲突。
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```



#### HashMap优化

**JDK1.7：**

- 数组+链表
- 在扩容时，在数据复制转移到新的数组中时，链表采用头插法，会导致链表循环（转移前后链表倒序了）

**JDK1.8：**

- 数组+链表（红黑树）

- 在扩容时，在数据复制转移到新的数组中时，链表采用尾插法，不会导致链表循环（转移前后链表顺序不变）
- 当链表的长度大于8时，会转换为红黑树（平衡搜索二叉树），以提高查找效率（复杂度从 O(n) 变为 O(logn)  ）

```java
int index = hash(key) & (cap - 1); // 等价于hash(key) % cap
int newIndex = hash(key) & oldCap + oldIndex;// 数组转移过程中，key不需要rehash；等价于hash(key) & (newCap - 1)
```



#### HashMap非线程安全

扩容时，多线程导致链表插入重复数据；链表循环问题



## ConcurrentHashMap

JDK1.7

- ConcurrentHashMap（分段锁） 对整个桶数组进行了分割分段(Segment)，每一把锁只锁容器其中一部分数据，多线程访问容器里不同数据段的数据，就不会存在锁竞争，提高并发访问率。
- Segment是一种可重入锁ReentrantLock，在ConcurrentHashMap里扮演锁的角色，HashEntry则用于存储键值对数据。

JDK1.8

- 利用CAS+volatile来保证并发更新的安全，底层依然采用数组+链表+红黑树的存储结构。	

根据上述代码，对ConcurrentHashMap是如何解决HashMap并发问题这一疑问进行简要说明。



- 首先new一个新的hash表(nextTable)出来，大小是原来的2倍。后面的rehash都是针对这个新的hash表操作，不涉及原hash表(table)。
- 然后会对原hash表(table)中的每个链表进行rehash，此时会尝试获取头节点的锁。这一步就保证了在rehash的过程中不能对这个链表执行put操作。
- 通过sizeCtl控制，使扩容过程中不会new出多个新hash表来。
- 最后，将所有键值对重新rehash到新表(nextTable)中后，用nextTable将table替换。这就避免了HashMap中get和扩容并发时，可能get到null的问题。
- 在整个过程中，共享变量的存储和读取全部通过volatile或CAS的方式，保证了线程安全。



## Java SPI实现机制

SPI：Service Provider Interface，是Java提供的一套用来被第三方实现或者扩展的接口，它可以用来启用框架扩展和替换组件。 SPI的作用就是为这些被扩展的API**寻找**服务实现。

Java SPI：`META-INF/services/`，`java.util.ServiceLoader`去load具体的实现

Spring SPI：`"META-INF/spring.factories"`

Dubbo SPI：`META-INF/dubbo`，比Java SPI更强大，还支持AOP



## 常用集合

ArrayList，Vector，Collections.SynchronizedList，CopyOnWriteArrayList

LinkedList、Stack

HashSet、TreeSet、LinkedHashSet

HashMap、WeakHashMap、Hashtable、ConcurrentHashMap



Queue





- Vector在数据满时增长为原来的两倍，而 ArrayList在数据量达到容量的一半时,增长为原容量的1.5倍。



#### fail-fast机制

集合中的`modCount`和迭代器中的`modCount`。

在线程不安全的集合中，如果使用迭代器的过程中，发现集合被修改，会抛出`ConcurrentModificationException`错误，这就是fail-fast机制。对集合进行结构性修改时，`modCount`都会增加，在初始化迭代器时，`modCount`的值会赋给`expectedModCount`，在迭代的过程中，只要`modCount`改变了，`int expectedModCount = modCount`等式就不成立了，迭代器检测到这一点，就会抛出错误：`ConcurrentModificationException`。



### 


