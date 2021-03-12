[TOC]

## JAVA 基础

### HashMap

1.**HashMap实现原理**  

- 你看过HashMap源码吗，知道底层的原理吗
- 为什么使用数组+链表
- 用LinkedList代替数组可以吗
- 既然是可以的，为什么不用反而用数组。  

**重要变量介绍：**  

- `DEFAULT_INITIAL_CAPACITY` Table数组的初始化长度： `1 << 4` `2^4=16` （为什么要是 2的n次方？）
- `MAXIMUM_CAPACITY` Table数组的最大长度： `1<<30` `2^30=1073741824` 
- `DEFAULT_LOAD_FACTOR` 负载因子：默认值为`0.75`。 当元素的总个数>当前数组的长度 * 负载因子。数组会进行扩容，扩容为原来的两倍（todo：为什么是两倍？）
- `TREEIFY_THRESHOLD` 链表树化阙值： 默认值为 `8` 。表示在一个node（Table）节点下的值的个数大于8时候，会将链表转换成为红黑树。
- `UNTREEIFY_THRESHOLD` 红黑树链化阙值： 默认值为 `6` 。 表示在进行扩容期间，单个Node节点下的红黑树节点的个数小于6时候，会将红黑树转化成为链表。
- `MIN_TREEIFY_CAPACITY = 64` 最小树化阈值，当Table所有元素超过改值，才会进行树化（为了防止前期阶段频繁扩容和树化过程冲突）

**实现原理** ：数组+链表，HashMap采⽤Entry数组来存储key-value对，每⼀个键值对组成了⼀个Entry实体，Entry类实际上是⼀个单向的链表结 构，它具有Next指针，可以连接下⼀个Entry实体。 只是在JDK1.8中，链表⻓度⼤于8的时候，链表会转成红⿊树！  

**第一问: 为什么使用链表+数组：要知道为什么使用链表首先需要知道Hash冲突是如何来的：**     

答： 由于我们的数组的值是限制死的，我们在对key值进行散列取到下标以后，放入到数组中时，难免出现两个key值不同，但是却放入到下标相同的**格子**中，此时我们就可以使用链表来对其进行链式的存放。  

**第二问： 我⽤LinkedList代替数组结构可以吗？**  

答：可以  

**第三问： 那既然可以使用进行替换处理，为什么有偏偏使用到数组呢？**  

答：数组相比与链表查询效率高，在HashMap中，定位节点的位置是利⽤元素的key的哈希值对数组⻓度取模得到。此时，我们已得到节点的位置。显然数组的查 找效率⽐`LinkedList`⼤  

那`ArrayList`，底层也是数组，查找也快啊，为啥不⽤ArrayList? 因为采⽤基本数组结构，扩容机制可以⾃⼰定义，HashMap中数组扩容刚好是**2的次幂**，在做取模运算的效率⾼。 ⽽ArrayList的扩容机制是1.5倍扩容（这一点我相信学习过的都应该清楚），那ArrayList为什么是1.5倍扩容？。  

**Hash冲突：得到下标值：**  

hashMap对存放进来的key值进行了hashcode()，生成了一个值，但是这个值很大，我们不可以直接作为下标，此时我们想到了可以使用取余的方法，即可以得到对于任意的一个key值，进行这样的操作以后，其值都落在`0-Table.length-1` 中，例如这样：  

```key.hashcode()%Table.length；``` 

但是 HashMap的源码却不是这样做？
它对其进行了与操作，对Table的表长度减一再与生产的hash值进行相与：  

```
if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
```

![](/home/yaofc/Desktop/AndroidNote/Note/img/img_hashmap.jpg)

这里我们也就得知为什么Table数组的长度要一直都为`2的n次方`，只有这样，减一进行相与时候，才能够达到最大的`n-1`值。  

**举个栗子来反证一下：**
我们现在 数组的长度为 15 减一为 14 ，二进制表示 `0000 1110` 进行相与时候，最后一位永远是0，这样就可能导致，不能够完完全全的进行Table数组的使用。违背了我们最开始的想要对Table数组进行**最大限度的无序使用**的原则，因为HashMap为了能够存取高效，，要尽量较少碰撞，就是要尽量把数据分配均匀，每个链表⻓度⼤致相同。
**此时还有一点需要注意的是： 我们对key值进行hashcode以后，进行相与时候都是只用到了后四位，前面的很多位都没有能够得到使用,这样也可能会导致我们所生成的下标值不能够完全散列。**
`解决方案：`将生成的hashcode值的高16位于低16位进行异或运算，这样得到的值再进行相与，一得到最散列的下标值。  

```java
static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```

**二问讲一讲HashMap的get/put过程**

- 知道HashMap的put元素的过程是什么样吗？
- 知道get过程是是什么样吗？
- 你还知道哪些的hash算法？
- 说一说String的hashcode的实现

**Put方法** 

1.对key的hashCode()做hash运算，计算index;
2.如果没碰撞直接放到bucket⾥；
3.如果碰撞了，以链表的形式存在buckets后；
4.如果碰撞导致链表过⻓(⼤于等于TREEIFY_THRESHOLD)，就把链表转换成红⿊树(JDK1.8中的改动)；
5.如果节点已经存在就替换old value(保证key的唯⼀性)
6.如果bucket满了(超过load factor*current capacity)，就要resize

在得到下标值以后，可以开始put值进入到数组+链表中，会有三种情况：

1. 数组的位置为空。
2. 数组的位置不为空，且面是链表的格式。
3. 数组的位置不为空，且下面是红黑树的格式。

同时 对于`Key` 和`Value` 也要经历一下步骤

- 通过 Key 散列获取到对于的Table；’
- 遍历Table 下的Node节点，做更新/添加操作；
- 扩容检测；

**resise方法**  

HashMap 的扩容实现机制是将老table数组中所有的Entry取出来，重新对其Hashcode做`Hash`散列到新的Table中，可以看到注解`Initializes or doubles table size.` resize表示的是对数组进行初始化或进行Double处理。  

```java
final Node<K,V>[] resize() {
    //先将老的Table取别名，这样利于后面的操作。
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        //表示之前的数组容量不为空。
        if (oldCap > 0) {
        // 如果 此时的数组容量大于最大值
            if (oldCap >= MAXIMUM_CAPACITY) {
            // 扩容 阙值为 Int类型的最大值，这种情况很少出现
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }

            //表示 old数组的长度没有那么大，进行扩容，两倍（这里也是有讲究的）对阙值也进行扩容
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        //表示之前的容量是0 但是之前的阙值却大于零， 此时新的hash表长度等于此时的阙值
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
        //表示是初始化时候，采用默认的 数组长度* 负载因子
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        //此时表示若新的阙值为0 就得用 新容量* 加载因子重新进行计算。
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        // 开始对新的hash表进行相对应的操作。
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
        //遍历旧的hash表，将之内的元素移到新的hash表中。
            for (int j = 0; j < oldCap/***此时旧的hash表的阙值*/; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                //表示这个格子不为空
                    oldTab[j] = null;
                    if (e.next == null)
                    // 表示当前只有一个元素，重新做hash散列并赋值计算。
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                    // 如果在旧哈希表中，这个位置是树形的结果，就要把新hash表中也变成树形结构，
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                    //保留 旧hash表中是链表的顺序
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {// 遍历当前Table内的Node 赋值给新的Table。
                            next = e.next;
                            // 原索引
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            // 原索引+oldCap
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        // 原索引放到bucket里面
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        // 原索引+oldCap 放到bucket里面
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
```

**get方法**  

1.对key的hashCode()做hash运算，计算index;
2.如果在bucket⾥的第⼀个节点⾥直接命中，则直接返回；
3.如果有冲突，则通过key.equals(k)去查找对应的Entry;
4.若为树，则在树中通过key.equals(k)查找，O(logn)；
5.若为链表，则在链表中通过key.equals(k)查找，O(n)。    

```java
final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        // 判断 表是否为空，表重读是否大于零，并且根据此 key 对应的表内是否存在 Node节点。    
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                // 检查第一个Node 节点，若是命中则不需要进行do... whirle 循环。
                return first;
            if ((e = first.next) != null) {
                if (first instanceof TreeNode)
                //树形结构，采用 对应的检索方法，进行检索。
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                do {
                //链表方法 做while循环，直到命中结束或者遍历结束。
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }
```

**还知道哪些hash算法**  

先说⼀下hash算法⼲嘛的，Hash函数是指把⼀个⼤范围映射到⼀个⼩范围。把⼤范围映射到⼀个⼩范围的⽬的往往是为了 节省空间，使得数据容易保存。⽐较出名的有`MurmurHash`、`MD4`、`MD5`等等  

**三问 为什么hashmap的在链表元素数量超过8时候改为红黑树** 

- 知道jdk1.8中hashmap改了什么吗。
- 说一下为什么会出现线程的不安全性
- 为什么在解决hash冲突时候，不直接用红黑树，而是先用链表，再用红黑树
- 当链表转为红黑树，什么时候退化为链表  

**第一问**改动了什么
1.由数组+链表的结构改为数组+链表+红⿊树。
2.优化了⾼位运算的hash算法：h^(h>>>16)
3.扩容后，元素要么是在原位置，要么是在原位置再移动2次幂的位置，且链表顺序不变。
**注意**： 最后⼀条是重点，因为最后⼀条的变动，hashmap在1.8中，不会在出现死循环问题。 

**HashMap的线程不安全性**  

HashMap 在`jdk1.7`中 使用 数组加链表的方式，并且在进行链表插入时候使用的是头结点插入的方法。
**注** ：这里为什么使用 头插法的原因是我们若是在散列以后，判断得到值是一样的，使用头插法，不用每次进行遍历链表的长度。但是这样会有一个缺点，在进行扩容时候，会导致进入新数组时候出现倒序的情况，也会在多线程时候出现线程的不安全性。
但是对与 `jdk1.8` 而言，还是要进行阙值的判断，判断在什么时候进行红黑树和链表的转换。所以无论什么时候都要进行遍历，于是插入到尾部，防止出现扩容时候还会出现倒序情况。

**所以当在多线程的使用场景中，尽量使用线程安全的ConcurrentHashMap。至于`Hashtable`而言，使用效率太低。**  

**第三问**为什么不一开始就使用红黑树，不是效率很高吗?
因为红⿊树需要进⾏左旋，右旋，变⾊这些操作来保持平衡，⽽单链表不需要。
当元素⼩于8个当时候，此时做查询操作，链表结构已经能保证查询性能。
当元素⼤于8个的时候，此时需要红⿊树来加快查 询速度，但是新增节点的效率变慢了。
因此，如果⼀开始就⽤红⿊树结构，元素太少，新增效率⼜⽐较慢，⽆疑这是浪费性能的。  

**第四问**什么时候退化为链表
为6的时候退转为链表。中间有个差值7可以防⽌链表和树之间频繁的转换。
假设⼀下，如果设计成链表个数超过8则链表转 换成树结构，链表个数⼩于8则树结构转换成链表，
如果⼀个HashMap不停的插⼊、删除元素，链表个数在8左右徘徊，就会 频繁的发⽣树转链表、链表转树，效率会很低。

**四问HashMap的并发问题**  

- HashMap在并发环境下会有什么问题
- 一般是如何解决的

**问题的出现**  

(1)多线程扩容，引起的死循环问题
(2)多线程put的时候可能导致元素丢失
(3)put⾮null元素后get出来的却是null

**不安全性的解决方案**  

1. 在之前使用hashtable。 在每一个函数前面都加上了synchronized 但是 效率太低 我们现在不常用了。
2. 使用 ConcurrentHashmap函数，对于这个函数而言 我们可以每几个元素共用一把锁。用于提高效率。

**五问你一般用什么作为HashMap的key值**  

- key可以是null吗，value可以是null吗
- 一般用什么作为key值
- 用可变类当Hashmap1的Key会有什么问题
- 让你实现一个自定义的class作为HashMap的Key该如何实现

**key可以是null吗，value可以是null吗**  

当然都是可以的，但是对于 key来说只能运行出现一个key值为null，但是可以出现多个value值为null

**一般用什么作为key值**  

⼀般⽤Integer、String这种不可变类当HashMap当key，⽽且String最为常⽤。
(1)因为字符串是不可变的，所以在它创建的时候hashcode就被缓存了，不需要重新计算。 这就使得字符串很适合作为Map中的键，字符串的处理速度要快过其它的键对象。 这就是HashMap中的键往往都使⽤字符串。
(2)因为获取对象的时候要⽤到equals()和hashCode()⽅法，那么键对象正确的重写这两个⽅法是⾮常重要的,这些类已 经很规范的覆写了hashCode()以及equals()⽅法。

**用可变类当Hashmap1的Key会有什么问题**  

hashcode可能会发生变化，导致put进行的值，无法get出来，如下代码所示：

```java
HashMap<List<String>,Object> map=new HashMap<>();
        List<String> list=new ArrayList<>();
        list.add("hello");
        Object object=new Object();
        map.put(list,object);
        System.out.println(map.get(list));
        list.add("hello world");
        System.out.println(map.get(list));
```

输出值如下：

```java
java.lang.Object@1b6d3586
null
```

**实现一个自定义的class作为Hashmap的key该如何实现**  

对于这个问题考查到了下面的两个知识点

- 重写hashcode和equals方法需要注意什么？
- 如何设计一个不变的类。
  **针对问题⼀，记住下⾯四个原则即可**
  (1)两个对象相等，hashcode⼀定相等
  (2)两个对象不等，hashcode不⼀定不等
  (3)hashcode相等，两个对象不⼀定相等
  (4)hashcode不等，两个对象⼀定不等
  **针对问题⼆，记住如何写⼀个不可变类**
  (1)类添加final修饰符，保证类不被继承。 如果类可以被继承会破坏类的不可变性机制，只要继承类覆盖⽗类的⽅法并且继承类可以改变成员变量值，那么⼀旦⼦类 以⽗类的形式出现时，不能保证当前类是否可变。
  (2)保证所有成员变量必须私有，并且加上final修饰 通过这种⽅式保证成员变量不可改变。但只做到这⼀步还不够，因为如果是对象成员变量有可能再外部改变其值。所以第4 点弥补这个不⾜。
  (3)不提供改变成员变量的⽅法，包括setter 避免通过其他接⼝改变成员变量的值，破坏不可变特性。
  (4)通过构造器初始化所有成员，进⾏深拷⻉(deep copy)
  (5) 在getter⽅法中，不要直接返回对象本⾝，⽽是克隆对象，并返回对象的拷⻉ 这种做法也是防⽌对象外泄，防⽌通过getter获得内部可变成员对象后对成员变量直接操作，导致成员变量发⽣改变