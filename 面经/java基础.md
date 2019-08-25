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

HashMap的resize函数源码分析

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



但是因为ConcurrentHashMap不允许value为null，所以可以通过map.get(key)是否为null来判断该map中是否包含该key，这时就没有上面的并发问题了。



https://mp.weixin.qq.com/s?__biz=MzI3ODg2OTY1OQ==&mid=2247483957&idx=2&sn=05ac5a1544de26fe7f5946b9938333cd&chksm=eb5121c1dc26a8d7c20e99886b9533691f0f26b22489b51de1bb8786da8baf502d835bc7d033&scene=21#wechat_redirect




```