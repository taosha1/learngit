<!-- TOC -->

- [HashMap源码解析](#hashmap%e6%ba%90%e7%a0%81%e8%a7%a3%e6%9e%90)
- [ArrayList和LinkedList 源代码](#arraylist%e5%92%8clinkedlist-%e6%ba%90%e4%bb%a3%e7%a0%81)

<!-- /TOC -->

### HashMap源码解析
```
HashMap的元素存储在Node数组中，这个数组的大小这里称为“桶”的大小。另外还有一个参数size指的是我们往HashMap中put了多少个元素。当size>桶的数量*DEFAULT_LOAD_FACTOR的时候，这时HashMap要进行扩容操作，也就是桶不能装满。DEFAULT_LOAD_FACTOR是衡量桶的利用率:

当链表长度>=8的时候并且Node数组的大小>=64，链表会变为红黑树结构（因为红黑树的增删改查复杂度是logn，链表是n，红黑树结构比链表代价更小）。

DEFAULT_LOAD_FACTOR是空间和时间的一个平衡点;
DEFAULT_LOAD_FACTOR较小时，需要的空间较大，但是put和get的代价较小;
DEFAULT_LOAD_FACTOR较大时，需要的空间较小，但是put和get的代价较大）。

Node的hash属性：保存key的hashcode的值：key的hashcode ^ (key的hashcode>>>16)。这样做主要是为了减少hash冲突当我们往map中put(k,v)时，这个k,v键值对会被封装为Node，那么这个Node放在Node数组的哪个位置呢：index=hash&(n-1),n为Node数组的长度。那为什么这样计算hash可以减少冲突呢？如果直接使用hashCode&(n-1)来计算index，此时hashCode的高位随机特性完全没有用到，因为n相对于hashcode的值很小，计算index的时候只能用到低16位。基于这一点，把hashcode高16位的值通过异或混合到hashCode的低16位，由此来增强hashCode低16位的随机性。

HashMap允许key为null，null的hash为0,从构造器中我们可以看到：HashMap是“懒加载”，在构造器中值保留了相关保留的值，并没有初始化table<Node>数组，当我们向map中put第一个元素的时候，map才会进行初始化！

HashMap  tableSizeFor函数源码分析:在new HashMap的时候，如果我们传入了大小参数，这是HashMap会对我们传入的HashMap容量进行传到tableSizeFor函数处理：这个函数主要功能是：返回一个数：这个数是大于等于cap并且是2的整数次幂的所有数中最小的那个，即返回一个最接近cap(>=cap)，并且是2的整数次幂的数。

创建HashMap时，该变量的值是：初始容量（2的整数次幂）,之后threshold的值是HashMap扩容的门限值：即当前Nodetable数组的长度* loadfactor。举个例子而言，如果我们传给HashMap构造器的容量大小为9，那么threshold初始值为16，在向HashMap中put第一个元素后，内部会创建长度为16的Node数组，并且threshold的值更新为16*0.75=12。具体而言，当我们一直往HashMap put元素的时候，如果put某个元素后，Node数组中元素个数为13，此时会触发扩容（因为数组中元素个数>threshold了，即13>threshold=12），具体扩容操作之后会详细分析，简单理解就是，扩容操作将Node数组长度*2；并且将原来的所有元素迁移到新的Node数组中。

get函数实质就是进行链表或者红黑树遍历搜索指定key的节点的过程；另外需要注意到HashMap的get函数的返回值不能判断一个key是否包含在map中，get返回null有可能是不包含该key；也有可能该key对应的value为null。HashMap中允许key为null，允许value为null。
    public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }

    final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            if ((e = first.next) != null) {
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }
getNode的主要逻辑：如果table[index]处节点的key就是要找的key则直接返回该节点； 否则：如果在table[index]位置进行搜索，搜索是否存在目标key的Node：这里的搜索又分两种：链表搜索和红黑树搜索，具体红黑树的查找就不展开了，红黑树是一种弱平衡(相对于AVL)BST，红黑树查找、删除、插入等操作都能够保证在O(lon(n))时间复杂度内完成，

put函数
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
     //链表查找 table[i]处存放的是链表，接下来和TreeNode类似在遍历链表过程中先判断当前的key是否已经存在，如果存在则令 e指向该Node；否则将该Node插入到链表末尾，插入后判断链表长度是否>=8，是的话要进行额外操作
          //binCountt最后的值是链表的长度
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                         /*
                            TREEIFY_THRESHOLD值是8,
                            binCount>=7,然后又插入了一个新节
                            点，链表长度>=8，这时要么进行扩容
                            操作，要么把链表结构转为红黑树结构。
                                         */
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
             /*
    这个函数的默认实现是“空”，
    即这个函数默认什么操作都
    不执行，那为什么要有它呢？
    这其实是个hook/钩子函数，
    主要要在LinkedHashMap
    （HashMap子类）中使用，
    LinkedHashMap重写了这
    个函数。以后会有讲解
    LinkedHashMap的文章。
             */
        afterNodeInsertion(evict);
        return null;
    }

HashMap的resize函数源码分析
    final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
                     /*j这个桶位置只有一个元素，直接rehash到table数组 */
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                     /*如果是红黑树：也是将红黑树拆分为两个链表，这里主要看链表的拆分，两者逻辑一样*/
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order

                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }

重点中的重点，面试谈到HashMap必考resize相关知识，整体思路介绍：

有两种情况会调用当前函数：

1.之前说过HashMap是懒加载，第一次hHashMap的put方法的时候table还没初始化，这个时候会执行resize，进行table数组的初始化，table数组的初始容量保存在threshold中（如果从构造器中传入的一个初始容量的话），如果创建HashMap的时候没有指定容量，那么table数组的初始容量是默认值：16。即，初始化table数组的时候会执行resize函数

2.扩容的时候会执行resize函数，当size的值>threshold的时候会触发扩容，即执行resize方法，这时table数组的大小会翻倍。
注意我们每次扩容之后容量都是翻倍（ *2），所以HashMap的容量一定是2的整数次幂，那么HashMap的容量为什么一定得是2的整数次幂呢？（面试重点）。位运算，减少hash冲突

 要知道原因，首先回顾我们put key的时候，每一个key会对应到一个桶里面，桶的索引是这样计算的： index = hash & (n-1)，index的计算最为直观的想法是：hash%n，即通过取余的方式把当前的key、value键值对散列到各个桶中；那么这里为什么不用取余(%)的方式呢？

原因是CPU对位运算支持较好，即位运算速度很快。另外,当n是2的整数次幂时：hash&(n-1)与hash%(n-1)是等价的，但是两者效率来讲是不同的，位运算的效率远高于%运算。

这里举一个例子来来说明这个特性：下面以Hash初始容量n=16，默认loadfactor=0.75举例（其他2的整数次幂的容量也是类似的），默认容量：n=16，二进制：10000；n-1：15，n-1二进制：01111。某个时刻，map中元素大于16*0.75=12，即size>12。此时会发生扩容，即会新建了一个数组，容量为扩容前的两倍，newtab，len=32。

接下来我们需要把table中的Node搬移(rehash)到newtab。从table的i=0位置开始处理,假设我们当前要处理table数组i索引位置的node，那这个node应该放在newtab的那个位置呢？下面的hash表示node.key对应的hash值，也就等于node.hash属性值,另外为了简单，下面的hash只写出了8位（省略的高位的0），实际上hash是32位：node在newtab中的索引：
index = hash & (0x0001_1111)
= hash & (0x0000_1111) 
| hash & (0x0001_0000) 
= hash & (0x0000_1111) | hash & n)
= i + ( hash & n)

hash&n要么等于n要么等于0;也就是：inde要么等于i，要么等于i+n;再具体一点：当hash&n==0的时候，index=i;

当hash&n==n的时候，index=i+n;这有什么用呢？当我们把table[i]位置的所有Node迁移到newtab中去的时候：

这里面的node要么在newtab的i位置（不变），要么在newtab的i+n位置；也就是我们可以这样处理：把table[i]这个桶中的node拆分为两个链表l1和类：如果hash&n==0，那么当前这个node被连接到l1链表；否则连接到l2链表。这样下来，当遍历完table[i]处的所有node的时候，我们得到两个链表l1和l2，这时我们令newtab[i]=l1,newtab[i+n]=l2,这就完成了table[i]位置所有node的迁移/rehash，这也是HashMap中容量一定的是2的整数次幂带来的方便之处。
下面的resize的逻辑就是上面讲的那样。将table[i]处的Node拆分为两个链表，这两个链表再放到newtab[i]和newtab[i+n]位置.


treeifyBin源码解析
//将链表转换为红黑树结构，在链表的
//插入操作后调用
    /*MIN_TREEIFY_CAPACITY值
    是64,也就是当链表长度>8的
    时候，有两种情况：如果table
    数组的长度<64,此时进行扩容
    操作;如果table数组的长度>64，
    此时进行链表转红黑树结构的操作.
    具体转细节在面试中几乎没有问的，
    这里不细讲了，大部同学认为链表长度
    >8一定会转换成红黑树，这是不对的！
    */
    final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
            resize();
        else if ((e = tab[index = (n - 1) & hash]) != null) {
            TreeNode<K,V> hd = null, tl = null;
            do {
                TreeNode<K,V> p = replacementTreeNode(e, null);
                if (tl == null)
                    hd = p;
                else {
                    p.prev = tl;
                    tl.next = p;
                }
                tl = p;
            } while ((e = e.next) != null);
            if ((tab[index] = hd) != null)
                hd.treeify(tab);
        }
    }



HashMap面试“明星”问题汇总，及答案 

你知道HashMap吗，请你讲讲HashMap？

这个问题不单单考察你对HashMap的掌握程度，也考察你的表达、组织问题的能力。个人认为应该从以下几个角度入手（所有常见HashMap的考点问题总结）：
size必须是2的整数次方原因
get和put方法流程
resize方法
影响HashMap的性能因素（key的hashCode函数实现、loadFactor、初始容量）
HashMap key的hash值计算方法以及原因（见上面hash函数的分析）
HashMap内部存储结构：Node数组+链表或红黑树
table[i]位置的链表什么时候会转变成红黑树（上面源码中有讲）
HashMap主要成员属性：threshold、loadFactor、HashMap的懒加载
HashMap的get方法能否判断某个元素是否在map中
HashMap线程安全吗，哪些环节最有可能出问题，为什么？
HashMap的value允许为null，但是HashTable和ConcurrentHashMap的valued都不允许为null，试分析原因？
HashMap中的hook函数（在后面讲解LinkedHashMap时会讲到，这也是面试时拓展的一个点）

上面问题的答案都可以在上面的源码分析中找到，下面在给三点补充：

HashMap的初始容量是怎样影响HashMap的性能的？

假如你预先知道最多往HashMap中存储64个元素，那么你在创建HashMap的时候：如果选用无参构造器：默认容量16，在存储16*loadFactor个元素之后就要进行扩容（数组扩容涉及到连续空间的分配，Node节点的rehash，代价很高，所以要尽量避免扩容操作）；如果给构造器传入的参数是64，这时HashMap中在存储64*loadFactor个元素之后就要进行扩容；但是如果你给构造器传的参数为：(int)(64/0.75)+1，此时就可以保证HashMap不用进行扩容，避免了扩容时的代价。


HashMap线程安全吗，哪些环节最有可能出问题，为什么？

我们都知道HashMap线程不安全，那么哪些环节最优可能出问题呢，及其原因：没有参照这个问题有点不好直接回答，但是我们可以找参照啊，参照：ConcurrentHashMap，因为大家都知道HashMap不是线程安全的，ConcurrentHashMap是线程安全的，对照ConcurrentHashMap，看看ConcurrentHashMap在HashMap的基础之上增加了哪些安全措施，这个问题就迎刃而解了。后面会有分析ConcurrentHashMap的文章，这里先简要回答这个问题：HashMap的put操作是不安全的，因为没有使用任何锁；HashMap在多线程下最大的安全隐患发生在扩容的时候，想想一个场合：HashMap使用默认容量16，这时100个线程同时往HashMap中put元素，会发生什么？扩容混乱，因为扩容也没有任何锁来保证并发安全，另外,后面的博文会讲到ConcurrentHashMap的并发扩容操作是ConcurrentHashMap的一个核心方法。


HashMap的value允许为null，但是HashTable和ConcurrentHashMap的value 都不允许为null，试分析原因？

首先要明确ConcurrentHashMap和Hashtable从技术从技术层面讲是可以允许value为null；但是它是实际是不允许的，这肯定是为了解决一些问题，为了说明这个问题，我们看下面这个例子（这里以ConcurrentHashMap为例，HashTable也是类似）。

HashMap由于允value为null，get方法返回null时有可能是map中没有对应的key；也有可能是该key对应的value为null。所以get不能判断map中是否包含某个key，只能使用contains判断是否包含某个key。

看下面的代码段，要求完成这个一个功能：如果map中包含了某个key则返回对应的value，否则抛出异常：

if (map.containsKey(k)) {
   return map.get(k);
} else {
   throw new KeyNotPresentException();
}
 如果上面的map为HashMap，那么没什么问题，因为HashMap本来就是线程不安全的，如果有并发问题应该用ConcurrentHashMap，所以在单线程下面可以返回正确的结果

如果上面的map为ConcurrentHashMap，此时存在并发问题：在map.containsKey(k)和map.get之间有可能其他线程把这个key删除了，这时候map.get就会返回null，而ConcurrentHashMap中不允许value为null，也就是这时候返回了null，一个根本不允许出现的值？

但是因为ConcurrentHashMap不允许value为null，所以可以通过map.get(key)是否为null来判断该map中是否包含该key，这时就没有上面的并发问题了

https://mp.weixin.qq.com/s?__biz=MzI3ODg2OTY1OQ==&mid=2247483957&idx=2&sn=05ac5a1544de26fe7f5946b9938333cd&chksm=eb5121c1dc26a8d7c20e99886b9533691f0f26b22489b51de1bb8786da8baf502d835bc7d033&scene=21#wechat_redirect

```

### ArrayList和LinkedList 源代码
在面试中经常碰到：ArrayList和LinkedList的特点和区别？

个人认为这个问题的回答应该分成这几部分：

介绍ArrayList底层实现

介绍LinkedList底层实现

两者个适用于哪些场合

本文也是按照上面这几部分组织的。



ArrayList的源码解析

成员属性源码解析

public class ArrayList<E> 
    extends AbstractList<E>
    implements List<E>, RandomAccess
    ,Cloneable, java.io.Serializable
{
    private static final long 
       serialVersionUID 
       = 8683452581122892189L;

    //默认容量是10
    private static final int 
             DEFAULT_CAPACITY = 10;


    //当传入ArrayList构造器的容量为0时
    //用这个数组表示：容器的容量为0
    private static final Object[] 
           EMPTY_ELEMENTDATA = {};


接上面

/*
主要作为一个标识位，在扩容时区分：
默认大小和容量为0，使用默认容量时采取的
是“懒加载”:即等到add元素的时候才进行实际
容量的分配，后面扩容函数讲解还会提到这点
*/
private static final Object[] 
DEFAULTCAPACITY_EMPTY_ELEMENTDATA={};

//ArrayList底层使用Object数组保存的元素
transient Object[] elementData; 

//记录当前容器中有多少元素
private int size;


构造器源码解析

/*
最常用的构造器之一，实际上就是创建了一个
指定大小的Object数组来保存之后add的元素
*/
public ArrayList(int initialCapacity){
    if (initialCapacity > 0) {
       //初始化保存数据的Object数组
        this.elementData
        =new Object[initialCapacity];
    } else if(initialCapacity==0) {
      //标识容量为0：EMPTY_ELEMENTDATA
        this.elementData 
                  = EMPTY_ELEMENTDATA;
    } else {
        throw new 
        IllegalArgumentException(
        "Illegal Capacity: "+
                initialCapacity);
    }
}


/*
无参构造器，指向的是默认容量大小的Object
数组，注意使用无参构造函数的时候并没有
直接创建容量 为10(默认容量是10)的Object
数组,而是采取懒加载的策略：使用
DEFAULTCAPACITY_EMPTY_ELEMENTDATA,
这个默认数组的容量是0,所以得区分是
默认容量，还是你传给构造器的容量参数大小
本身就是0。在真正执行add操作时才会创建
Object数组，即在扩容函数中有处理默认容量
的逻辑，后面会有详细分析。
*/
    public ArrayList() {
        //这个赋值操作仅仅是标识作用
       this.elementData = 
     DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

   //省略一部分不常用代码函数
      

add方法源码解析

/*
add是ArrayList最常用的接口，逻辑很简单
*/
public boolean add(E e) {
 /*
 主要用于标识线程安全，即ArrayList只能
在单线程环境下使用，在多线程环境下会出现并发
安全问题，modCount主要用于记录对ArrayList的
修改次数，如果一个线程操作ArrayList期间
modCount发生了变化即，有多个线程同时修改当前
这个ArrayList，此时会抛出
“ConcurrentModificationException”异常，
这又被称为“failFast机制”，在很多非线程安全的
类中都有failFast机制：HashMap、 LinkedList
等。这个机制主要用于迭代器、加强for循环等相关
功能，也就是一个线程在迭代一个有failfast机制
容器的时候，如果其他线程改变了容器内的元素，
迭代的这个线程会抛 
出“ConcurrentModificationException”异常
*/
    modCount++;

/*
add操作的核心函数，当使用无参构造器时并没有
直接分配大小为10的Object数组，这里面有对应
的处理逻辑。
*/   
    //进入该函数
    add(e, elementData, size);
    return true;
}


private void add(E e,Object[] elementData
                , int s) {
    /*
    如果使用无参构造器：开始时length为0，
    s也为0.grow()核心函数,扩容/初始化操作
    */
    if (s == elementData.length)
        elementData = grow();
    elementData[s] = e;
    size = s + 1;
}


grow相关方法源码解析

private Object[] grow() {
    //继续追踪
    return grow(size + 1);
}


private Object[] grow(int minCapacity){
    /*
  使用数组复制的方式，扩容：将elementData
  所有元素复制到一个新数组中，这个新数组的
  长度是newCapacity()函数的返回值，之后再
  把这个新数组赋值给elementData，完成扩容
  操作
    */
    //进入newCapacity()函数
    return elementData = 
    Arrays.copyOf(elementData,
          newCapacity(minCapacity));
}


//返回的是扩容后数组的长度
private int newCapacity(int minCapacity){
    int oldCapacity=elementData.length;
    //扩容后的容量为原来容量的1.5倍
    int newCapacity = oldCapacity 
            + (oldCapacity >> 1);
    if (newCapacity-minCapacity <=0){
        if (elementData ==
        DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
             //默认容量的处理
             return Math.max(
           DEFAULT_CAPACITY, minCapacity);

      /*
    minCapacity是int类型，有溢出的可能，也就
    是ArrayList最大大小是Integer.MAX_VALUE
      */
        if (minCapacity<0) //overflow
           throw new OutOfMemoryError();
        
        //返回新容量
        return minCapacity;
    }

  /*
  MAX_ARRAY_SIZE=Integer.MAX_VALUE-8,
  当扩容后大于MAX_ARRAY_SIZE ，返回 
  hugeCapacity(minCapacity),
  其实就是Integer.MAX_VALUE
    */
    return (newCapacity-MAX_ARRAY_SIZE
         <= 0)? newCapacity
        : hugeCapacity(minCapacity);
}


private static int hugeCapacity
                (int minCapacity){
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity>MAX_ARRAY_SIZE)
        ? Integer.MAX_VALUE
        : MAX_ARRAY_SIZE;
}


ArrayList的failfast机制

//最后看下ArrayList的failFast机制
private class Itr implements 
                Iterator<E>{
    //index of next element to return
    int cursor;      
    // index of last element returned;  
    int lastRet = -1; -1 if no such
    /*
    在迭代之前先保存modCount的值，
    modCount在改变容器元素、容器
    大小时会自增加1
    */
    int expectedModCount=modCount;

    // prevent creating a synthetic
    // constructor
    Itr() {}

    public boolean hasNext() {
        return cursor != size;
    }

    
    @SuppressWarnings("unchecked")
    public E next() {
       /*
       使用迭代器遍历元素的时候先检查
       modCount的值是否等于预期的值，
       进入该函数
       */
        checkForComodification();
        int i = cursor;
        if (i >= size)
            throw new 
            NoSuchElementException();
        Object[] elementData =
          ArrayList.this.elementData;
        if (i >= elementData.length)
            throw new 
          ConcurrentModificationException();
        cursor = i + 1;
        return (E)elementData[lastRet=i];
    }

    
  /*
    可以发现：在迭代期间如果有线程改变了
    容器，此时会抛出
    “ConcurrentModificationException”
    */
   final void checkForComodification(){
        if (modCount!=expectedModCount)
            throw new 
          ConcurrentModificationException();
    }


ArrayList的其他操作，比如：get、remove、indexOf其实就很简单了，都是对Object数组的操作：获取数组某个索引位置的元素，删除数组中某个元素，查找数组中某个元素的位置......所以说理解原理很重要。



上面注释的部分就是ArrayList的考点，主要有：初始容量、最大容量、使用Object数组保存元素（数组与链表的异同）、扩容机制（1.5倍）、failFast机制等。



LinkedList源码分析

成员属性源码分析

public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>
    ,Cloneable, java.io.Serializable
{
  MAX_ARRAY_SIZE=Integer.MAX_VALUE-8,
    /*
    LinkedList的size是int类型，但是后面
    会看到LinkedList大小实际只受内存大小
    的限制也就是LinkedList的size大小可能
    发生溢出，返回负数
    transient int size = 0;

    //LinkedList底层使用双向链表实现，
    //并保留了头尾两个节点的引用
    transient Node<E> first;//头节点


    transient Node<E> last;//尾节点
    //省略一部分无关代码

    //下面分析LinkedList内部类Node


内部类Node源码分析

    private static class Node<E> {
        E item;//元素值
        Node<E> next;//后继节点

        //前驱节点,即Node是双向链表
        Node<E> prev;

        Node(Node<E> prev, E element
        , Node<E> next) {//Node的构造器
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }


构造器源码分析

//LinkedList无参构造器：什么都没做
public LinkedList() {}


其他核心辅助接口方法源码分析

/*
LinkedList的大部分接口都是基于
                这几个接口实现的：
1.往链表头部插入元素
2.往链表尾部插入元素
3.在指定节点的前面插入一个节点
4.删除链表的头结点
5.删除除链表的尾节点
6.删除除链表中的指定节点
*/

//1.往链表头部插入元素
private void linkFirst(E e) {
    final Node<E> f = first;
    final Node<E> newNode = 
            new Node<>(null, e, f);
    first = newNode;
    if (f == null)
        last = newNode;
    else
        f.prev = newNode;
    size++;
    modCount++;//failFast机制
}

  
//2.往链表尾部插入元素
void linkLast(E e) {
    final Node<E> l = last;
    final Node<E> newNode = 
        new Node<>(l, e, null);
    last = newNode;
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    size++;
    modCount++;//failFast机制
}

//3.在指定节点(succ)的前面插入一个节点
void linkBefore(E e, Node<E> succ) {
    // assert succ != null;
    final Node<E> pred = succ.prev;
    final Node<E> newNode 
        = new Node<>(pred, e, succ);
    succ.prev = newNode;
    if (pred == null)
        first = newNode;
    else
        pred.next = newNode;
    size++;
    modCount++;//failFast机制
}

//4.删除链表的头结点
private E unlinkFirst(Node<E> f){
    //assert f==first && f!=null;
    final E element = f.item;
    final Node<E> next = f.next;
    f.item = null;
    f.next = null; //help GC
    first = next;
    if (next == null)
        last = null;
    else
        next.prev = null;
    size--;
    modCount++;//failFast机制
    return element;
}


//5.删除除链表的尾节点
private E unlinkLast(Node<E> l) {
    //assert l==last && l!=null;
    final E element = l.item;
    final Node<E> prev = l.prev;
    l.item = null;
    l.prev = null; // help GC
    last = prev;
    if (prev == null)
        first = null;
    else
        prev.next = null;
    size--;
    modCount++;//failFast机制
    return element;
}


//6.删除除链表中的指定节点
E unlink(Node<E> x) {
    // assert x != null;
    final E element = x.item;
    final Node<E> next = x.next;
    final Node<E> prev = x.prev;

    if (prev == null) {
        first = next;
    } else {
        prev.next = next;
        x.prev = null;
    }

    if (next == null) {
        last = prev;
    } else {
        next.prev = prev;
        x.next = null;
    }

    x.item = null;
    size--;
    modCount++;//failFast机制
        return element;
    }


常用API源码分析

//LinkedList常用接口的实现
public E removeFirst() {
    final Node<E> f = first;
    if (f == null)
        throw 
        new NoSuchElementException();
    //调用 4.删除链表的头结点 实现
        return unlinkFirst(f);
  }

public E removeLast() {
    final Node<E> l = last;
    if (l == null)
        throw 
        new NoSuchElementException();
     //调用 5.删除除链表的尾节点 实现
         return unlinkLast(l);
    }

   public void addFirst(E e) {
       //调用 1.往链表头部插入元素 实现
       linkFirst(e);
    }

   public void addLast(E e) {
       //调用 2.往链表尾部插入元素 实现
       linkLast(e);
}

public boolean add(E e) {
    //调用 2.往链表尾部插入元素 实现
    linkLast(e);
    return true;
    }

public boolean remove(Object o) {
    if (o == null) {
        for (Node<E> x = first;
         x != null; x = x.next) {
            if (x.item == null) {
         //调用 6.删除除链表中的
         //指定节点 实现
                unlink(x);
                return true;
            }
        }
    } else {
        for (Node<E> x = first
        ; x != null; x = x.next) {
            if (o.equals(x.item)) {
                //调用 6.删除除链表中的
                //指定节点 实现
                unlink(x);
                return true;
            }
        }
    }
    return false;
    }


//省略其他无关函数


failfast机制

//迭代器中的failFast机制
private class ListItr 
        implements ListIterator<E> {
    private Node<E> lastReturned;
    private Node<E> next;
    private int nextIndex;

    /*
    在迭代之前先保存modCount的值，
    modCount在改变容器元素、容器大小时
    会自增加1
    */
    private int expectedModCount
                 = modCount;

    ListItr(int index) {
        next = (index == size) 
            ? null : node(index);
        nextIndex = index;
    }

    public boolean hasNext() {
        return nextIndex < size;
    }

    public E next() {
        /*
        使用迭代器遍历元素的时候先检查
        modCount的值是否等于预期的值，
        进入该函数
        */
        checkForComodification();
        if (!hasNext())
            throw 
            new NoSuchElementException();

        lastReturned = next;
        next = next.next;
        nextIndex++;
        return lastReturned.item;
    }

     /*
    可以发现：在迭代期间如果有线程改变了容器，
    此时会抛出
    “ConcurrentModificationException”
    */
     final void checkForComodification(){
          if (modCount!=expectedModCount)
              throw new 
           ConcurrentModificationException();
        }
  

LinkedList的实现较为简单：底层使用双向链表实现、保留了头尾两个指针、LinkedList的其他操作基本都是基于上面那六个函数实现的，另外LinkedList也有failFast机制，这个机制主要在迭代器中使用。



数组和链表各自的特性    

     数组和链表的特性差异，本质是：连续空间存储和非连续空间存储的差异。主要有下面两点：     

ArrayList：底层是Object数组实现的：由于数组的地址是连续的，数组支持O(1)随机访问；数组在初始化时需要指定容量；数组不支持动态扩容，像ArrayList、Vector和Stack使用的时候看似不用考虑容量问题（因为可以一直往里面存放数据）；但是它们的底层实际做了扩容；数组扩容代价比较大，需要开辟一个新数组将数据拷贝进去，数组扩容效率低；适合读数据较多的场合。

LinkedList：底层使用一个Node数据结构，有前后两个指针，双向链表实现的。相对数组，链表插入效率较高，只需要更改前后两个指针即可；另外链表不存在扩容问题，因为链表不要求存储空间连续，每次插入数据都只是改变last指针；另外，链表所需要的内存比数组要多，因为他要维护前后两个指针；它适合删除，插入较多的场景LinkedList还实现了Deque接口。

https://mp.weixin.qq.com/s?__biz=MzI3ODg2OTY1OQ==&mid=2247483981&idx=1&sn=20a66be96d2f37f2cca3083a93fb2a81&scene=19#wechat_redirect