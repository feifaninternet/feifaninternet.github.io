------------------------
title: HashMap 源码
tags: HashMap
categories: Origin Code
date: 2018-06-08 14:58:49
------------------------

### Description
HashMap的源码是很有代表性的，涉及的方面也很多，我们需要仔细的学习。   
关于HashMap的几个知识点：

1. HashMap是根据键值对存储的，并且存储时数据的键不能相同，如果相同，该键对应的值会被覆盖，如果想要保证HashMap能够正确的存储数据，要确保作为键的类已经正确覆写了equals()方法。
2. HashMap存储数据的位置与添加数据的键的hashCode()返回值有关。所以在元素使用HashMap存储的时候要确保已经正确重写了hashCode()。
3. HashMap最多只允许一条数据的键为null，可允许多条数据的值为null。
4. HashMap存储数据的顺序是不确定的，并且可能因为扩容导致存储的位置改变，因此遍历顺序是不确定的。
5. HashMap是线程不安全的，如果需要多线程的情况下使用，可以用Collections.synchronizedMap(Map map)方法使hashMap具有线程安全能力，或者使用ConcurrentHashMap。

### HashMap 的存储结构
HashMap的存储结构在JDK 1.7和1.8之间有很大的变化，JDK1.7中先添加的元素总是放在数组相应的角标位置，而原来处于该角标位置的节点作为next节点放到新节点的后面，在JDK1.8之后解决hash冲突就不单单是使用数组加上单链表的组合了，因为如果hash值冲突较多，链表的长度就会越来越长，时间复杂度会达到0(n)，在1.8之后，新增节点导致链表长度超过TREEIFY_THRESHOLD = 8的时候，就会在添加元素的同时将原来的单链表转化为红黑树。

<div class="wrap effect" style="box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	webkit-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	moz-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	o-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;">
	<img src="/picture/HashMapRedBlankTree.jpg" alt="HashMap的红黑树" title="HashMap的红黑树">
</div>

### HashMap 的重要参数

#### 1.哈希桶(buckets)
在 HashMap 的注释里使用哈希桶来形象的表示数组中每个地址位置。注意这里并不是数组本身，数组是装哈希桶的，他可以被称为哈希表。

#### 2.初始容量(initial capacity)
哈希表中哈希桶的初始数量。如果没有通过构造方法修改，这个容量值默认为DEFAULT_INITIAL_CAPACITY = 1<<4(即16)。值得注意的是为了保证 HashMap 添加和查找的高效性，HashMap 的容量总是 2^n 的形式。

#### 3.加载因子(load factor)
加载因子是哈希表（散列表）在其容量自动增加之前被允许获得的最大数量的度量。当哈希表中的条目数量超过负载因子和当前容量的乘积时，散列表就会被重新映射（即重建内部数据结构），重新创建的散列表容量大约是之前散列表哈系统桶数量的两倍。默认加载因子（0.75）在时间和空间成本之间提供了良好的折中。加载因子过大会导致很容易链表过长，加载因子很小又容易导致频繁的扩容。所以不要轻易试着去改变这个默认值。

#### 4.扩容阈值（threshold）
扩容阈值 = 哈希表容量 * 加载因子。哈希表的键值对总数 = 所有哈希桶中所有链表节点数的总和，扩容阈值比较的是是键值对的个数而不是哈希表的数组中有多少个位置被占了。

#### 5.树化阀值(TREEIFY_THRESHOLD)
哈希桶中的节点个数大于该值（默认为8）的时候将会被转为红黑树行存储结构。

#### 6.非树化阀值(UNTREEIFY_THRESHOLD)
当一个已经转化为树形存储结构的哈希桶中节点数量小于该值（默认为 6）的时候将再次改为单链表的格式存储。导致这种操作的原因可能有删除节点或者扩容。

#### 7.最小树化容量(MIN_TREEIFY_CAPACITY)
当链表的节点数超过8的时候就会转化为树化存储，其实对于转化还有一个要求就是哈希表的数量超过最小树化容量的要求（默认要求是 64）,且为了避免进行扩容、树形化选择的冲突，这个值不能小于 4 * TREEIFY_THRESHOLD);在达到该有求之前优先选择扩容。扩容因为因为容量的变化可能会使单链表的长度改变。

### HashMap 的基本存储单元
HashMap在JDK1.7中只有Entry一种存储单元，而在JDK1.8中由于有了红黑树，就多了一种存储单元，而Entry也随之应景的改为Node。   
单链表节点的表现方法：
   ```java
    /**
     * 内部类 Node 实现基类的内部接口 Map.Entry<K,V>
     * 
     */
    static class Node<K,V> implements Map.Entry<K,V> {
       //此值是在数组索引位置
       final int hash;
       //节点的键
       final K key;
       //节点的值
       V value;
       //单链表中下一个节点
       Node<K,V> next;
        
       Node(int hash, K key, V value, Node<K,V> next) {
           this.hash = hash;
           this.key = key;
           this.value = value;
           this.next = next;
       }
    
       public final K getKey()        { return key; }
       public final V getValue()      { return value; }
       public final String toString() { return key + "=" + value; }
        //节点的 hashCode 值通过 key 的哈希值和 value 的哈希值异或得到，没发现在源码中中有用到。
       public final int hashCode() {
           return Objects.hashCode(key) ^ Objects.hashCode(value);
       }
    
       //更新相同 key 对应的 Value 值
       public final V setValue(V newValue) {
           V oldValue = value;
           value = newValue;
           return oldValue;
       }
     //equals 方法，键值同时相同才节点才相同
       public final boolean equals(Object o) {
           if (o == this)
               return true;
           if (o instanceof Map.Entry) {
               Map.Entry<?,?> e = (Map.Entry<?,?>)o;
               if (Objects.equals(key, e.getKey()) &&
                   Objects.equals(value, e.getValue()))
                   return true;
           }
           return false;
       }
    }
   ```
JDK1.8新增的红黑树节点 :
   ```java
    static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
       TreeNode<K,V> parent;  // red-black tree links
       TreeNode<K,V> left;
       TreeNode<K,V> right;
       TreeNode<K,V> prev;    // needed to unlink next upon deletion
       boolean red;
       TreeNode(int hash, K key, V val, Node<K,V> next) {
           super(hash, key, val, next);
       }
       
    }
   ```
   
### HashMap 的构造方法

#### 1.可以指定期望初始容量和加载因子的构造函数

   ```java
    public HashMap(int initialCapacity, float loadFactor) {
        // 指定期望初始容量小于0将会抛出非法参数异常
       if (initialCapacity < 0)
           throw new IllegalArgumentException("Illegal initial capacity: " +
                                              initialCapacity);
       // 期望初始容量不可以大于最大值 2^30  实际上我们也不会用到这么大的容量                                      
       if (initialCapacity > MAXIMUM_CAPACITY)
           initialCapacity = MAXIMUM_CAPACITY;
      // 加载因子必须大于0 不能为无穷大   
       if (loadFactor <= 0 || Float.isNaN(loadFactor))
           throw new IllegalArgumentException("Illegal load factor: " +
                                              loadFactor);
       this.loadFactor = loadFactor;//初始化全局加载因子变量
       this.threshold = tableSizeFor(initialCapacity);//根据初始容量计算计算扩容阈值
    }
   ```
   
这个函数没有初始化Node<K,V>[] table，事实上真正指定哈希表容量总是在第一次添加元素的时候，这点和ArrayList的机制不同
   ```java
    //根据期望容量返回一个 >= cap 的扩容阈值，并且这个阈值一定是 2^n 
    static final int tableSizeFor(int cap) {
       int n = cap - 1;
       n |= n >>> 1;
       n |= n >>> 2;
       n |= n >>> 4;
       n |= n >>> 8;
       n |= n >>> 16;
       //经过上述面的 或和位移 运算， n 最终各位都是1 
       //最终结果 +1 也就保证了返回的肯定是 2^n 
       return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
   ```
   
#### 2.仅指定期望初始容量的构造函数
这里比较简单，就是将指定的初始容量和默认加载因子传递给上述构造方法

   ```java
    public HashMap(int initialCapacity) {
       this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }
   ```

#### 3.无参数构造函数

   ```java
    public HashMap() {
       this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }
   ```
这也是我们最常用的一个构造函数，该方法初始化了加载因子为默认值，并没有调动其他的构造方法，跟我们之前说的一样，哈希表的大小以及其他参数都会在第一次调用扩容函数的时候初始化为默认值。

#### 4.传入一个 Map 集合的构造参数
这个构造函数比较复杂，在初始化的时候就涉及了添加元素，扩容这两大重要的方法。  
   ```java
    public HashMap(Map<? extends K, ? extends V> m) {
       this.loadFactor = DEFAULT_LOAD_FACTOR;
       putMapEntries(m, false);
    }
   ```
   
### HashMap 添加元素位置的确定
我们都知道HashMap的底层是哈希表，哈希表依靠hash值去确定元素存储位置，我们来看下HashMap的实现

JDK 1.7中我们得到hash值的扰动函数(键的 hashCode 函数返回值不一定满足哈希表长度的要求,所以要进行扰动处理)
   ```java
    //4次位运算 + 5次异或运算 
    //这种算法可以防止低位不变，高位变化时，造成的 hash 冲突
    static final int hash(Object k) {
       int h = 0;
       h ^= k.hashCode(); 
       h ^= (h >>> 20) ^ (h >>> 12);
       return h ^ (h >>> 7) ^ (h >>> 4);
    }
   ```
JDK 1.8中得到hash值的函数优化
   ```java
   /**
   * 把 key 的 hashCode 方法返回值右移16位，即丢弃低16位，高16位全为0 ，
   * 然后在于 hashCode 返回值做异或运算，即高 16 位与低 16 位进行异或运算，
   * 这么做可以在数组 table 的 length 比较小的时候，也能保证考虑到高低Bit
   * 都参与到 hash 的计算中，同时不会有太大的开销，扰动处理次数也从 4次位
   * 运算 + 5次异或运算 降低到 1次位运算 + 1次异或运算
   */
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
   ```
上面的扰动函数只是得到了hash的值，还没有确定在Node []数组中的角标，下面这个函数在JDK1.7中，JDK1.8中将这步运算放在了put函数中
   ```java
    static int indexFor(int h, int length) {
        // 与运算 优化取模运算(hash % length)
         return h & (length-1);  
    }
   ```
此函数确定最终元素储存的哈希桶角标位置

### HashMap 添加元素
puy(K key,V value)函数

   ```java
    // 可以看到具体的添加行为在 putVal 方法中进行
    public V put(K key, V value) {
       return putVal(hash(key), key, value, false, true);
    }
   ```
对于 putVal 前三个参数很好理解，第4个参数 onlyIfAbsent 表示只有当对应 key 的位置为空的时候替换元素，一般传 false，第 5 个参数 evict 如果是 false。那么表示是在初始化时调用的:

   ```java
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                  boolean evict) {
                  
       Node<K,V>[] tab; Node<K,V> p; int n, i;
       //如果是第一添加元素 table = null 则需要扩容
       if ((tab = table) == null || (n = tab.length) == 0)
           n = (tab = resize()).length;// n 表示扩容后数组的长度
       //  i = (n - 1) & hash 即上边讲得元素存储在 map 中的数组角标计算
       // 如果对应数组没有元素没发生 hash 碰撞 则直接赋值给数组中 index 位置   
       if ((p = tab[i = (n - 1) & hash]) == null)
           tab[i] = newNode(hash, key, value, null);
       else {// 发生 hash 碰撞了
           Node<K,V> e; K k;
            //如果对应位置有已经有元素了 且 key 是相同的则覆盖元素
           if (p.hash == hash &&
               ((k = p.key) == key || (key != null && key.equals(k))))
               e = p;
           else if (p instanceof TreeNode)//如果添加当前节点已经为红黑树，则需要转为红黑树中的节点
               e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
           else {// hash 值计算出的数组索引相同，但 key 并不同的时候，        // 循环整个单链表
               for (int binCount = 0; ; ++binCount) {
                   if ((e = p.next) == null) {//遍历到尾部
                        // 创建新的节点，拼接到链表尾部
                       p.next = newNode(hash, key, value, null); 
                       // 如果添加后 bitCount 大于等于树化阈值后进行哈希桶树化操作
                       if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                           treeifyBin(tab, hash);
                       break;
                   }
                   //如果遍历过程中找到链表中有个节点的 key 与 当前要插入元素的 key 相同，此时 e 所指的节点为需要替换 Value 的节点，并结束循环
                   if (e.hash == hash &&
                       ((k = e.key) == key || (key != null && key.equals(k))))
                       break;
                   //移动指针    
                   p = e;
               }
           }
           //如果循环完后 e!=null 代表需要替换e所指节点 Value
           if (e != null) { // existing mapping for key
               V oldValue = e.value//保存原来的 Value 作为返回值
               // onlyIfAbsent 一般为 false 所以替换原来的 Value
               if (!onlyIfAbsent || oldValue == null)
                   e.value = value;
                //这个方法在 HashMap 中是空实现，在 LinkedHashMap 中有关系   
               afterNodeAccess(e);
               return oldValue;
           }
       }
       //操作数增加
       ++modCount;
       //如果 size 大于扩容阈值则表示需要扩容
       if (++size > threshold)
           resize();
       afterNodeInsertion(evict);
       return null;
    }
   ```
我们可以详细的分析一下添加元素的过程:

![HashMap的红黑树](/picture/HashMapInsert.jpg)

1. 如果 Node[] table 为null，则表示第一次添加元素，需要首次扩容
2. 计算对应的键值在table表中的索引位置，通过i = (n-1) & hash获得
3. 判断索引位置是否有元素，没有则直接插入到数组中。如果有元素且key相同，则覆盖value值，这里判断是用的是equals，这也体现了想要正确插入元素就要正确覆写equals的重要性
4. 如果索引位置的 key 不相同，则需要遍历单链表，如果遍历过如果有与 key 相同的节点，则保存索引，替换 Value；如果没有相同节点，则在但单链表尾部插入新节点。
5. 插入节点后，链表的长度大于树化阀值，则需要将单链表转换为红黑树
6. 成功加入节点(添加元素)后，判断是否大约扩容阀值，如果大于则需要扩容

### HashMap 的扩容过程
再添加元素的时候我们多次提及扩容，下面我们来仔细了解一下HashMap的扩容过程。

   ```java
    final Node<K,V>[] resize() {
       // oldTab 指向旧的 table 表
       Node<K,V>[] oldTab = table;
       // oldCap 代表扩容前 table 表的数组长度，oldTab 第一次添加元素的时候为 null 
       int oldCap = (oldTab == null) ? 0 : oldTab.length;
       // 旧的扩容阈值
       int oldThr = threshold;
       // 初始化新的阈值和容量
       int newCap, newThr = 0;
       // 如果 oldCap > 0 则会将新容量扩大到原来的2倍，扩容阈值也将扩大到原来阈值的两倍
       if (oldCap > 0) {
           // 如果旧的容量已经达到最大容量 2^30 那么就不在继续扩容直接返回，将扩容阈值设置到 Integer.MAX_VALUE，并不代表不能装新元素，只是数组长度将不会变化
           if (oldCap >= MAXIMUM_CAPACITY) {
               threshold = Integer.MAX_VALUE;
               return oldTab;
           }//新容量扩大到原来的2倍，扩容阈值也将扩大到原来阈值的两倍
           else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                    oldCap >= DEFAULT_INITIAL_CAPACITY)
               newThr = oldThr << 1; // double threshold
       }
       //oldThr 不为空，代表我们使用带参数的构造方法指定了加载因子并计算了
       //初始初始阈值 会将扩容阈值 赋值给初始容量这里不再是期望容量，
       //但是 >= 指定的期望容量
       else if (oldThr > 0) // initial capacity was placed in threshold
           newCap = oldThr;
       else {
            // 空参数构造会走这里初始化容量，和扩容阈值 分别是 16 和 12
           newCap = DEFAULT_INITIAL_CAPACITY;
           newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
       }
       //如果新的扩容阈值是0，对应的是当前 table 为空，但是有阈值的情况
       if (newThr == 0) {
            //计算新的扩容阈值
           float ft = (float)newCap * loadFactor;
           // 如果新的容量不大于 2^30 且 ft 不大于 2^30 的时候赋值给 newThr 
           //否则 使用 Integer.MAX_VALUE
           newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                     (int)ft : Integer.MAX_VALUE);
       }
       //更新全局扩容阈值
       threshold = newThr;
       @SuppressWarnings({"rawtypes","unchecked"})
        //使用新的容量创建新的哈希表的数组
       Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
       table = newTab;
       //如果老的数组不为空将进行重新插入操作否则直接返回
       if (oldTab != null) {
            //遍历老数组中每个位置的链表或者红黑树重新计算节点位置，插入新数组
           for (int j = 0; j < oldCap; ++j) {
               Node<K,V> e;//用来存储对应数组位置链表头节点
               //如果当前数组位置存在元素
               if ((e = oldTab[j]) != null) {
                    // 释放原来数组中的对应的空间
                   oldTab[j] = null;
                   // 如果链表只有一个节点，
                   //则使用新的数组长度计算节点位于新数组中的角标并插入
                   if (e.next == null)
                       newTab[e.hash & (newCap - 1)] = e;
                   else if (e instanceof TreeNode)//如果当前节点为红黑树则需要进一步确定树中节点位于新数组中的位置。
                       ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                   else { // preserve order
                       //因为扩容是容量翻倍，
                       //原链表上的每个节点 现在可能存放在原来的下标，即low位，
                       //或者扩容后的下标，即high位
                  //低位链表的头结点、尾节点
                  Node<K,V> loHead = null, loTail = null;
                  //高位链表的头节点、尾节点
                  Node<K,V> hiHead = null, hiTail = null;
                  Node<K,V> next;//用来存放原链表中的节点
                  do {
                      next = e.next;
                      // 利用哈希值 & 旧的容量，可以得到哈希值去模后，
                      //是大于等于 oldCap 还是小于 oldCap，
                      //等于 0 代表小于 oldCap，应该存放在低位，
                      //否则存放在高位（稍后有图片说明）
                      if ((e.hash & oldCap) == 0) {
                          //给头尾节点指针赋值
                          if (loTail == null)
                              loHead = e;
                          else
                              loTail.next = e;
                          loTail = e;
                      }//高位也是相同的逻辑
                      else {
                          if (hiTail == null)
                              hiHead = e;
                          else
                              hiTail.next = e;
                          hiTail = e;
                      }//循环直到链表结束
                  } while ((e = next) != null);
                  //将低位链表存放在原index处，
                  if (loTail != null) {
                      loTail.next = null;
                      newTab[j] = loHead;
                  }
                  //将高位链表存放在新index处
                  if (hiTail != null) {
                      hiTail.next = null;
                      newTab[j + oldCap] = hiHead;
                  }
               }
           }
       }
       return newTab;
    }
   ```
整个扩容过程可以概括为两点：
1. 寻找扩容后数组的大小以及新的扩容阀值
2. 将原有哈希表拷贝到新的哈希表中

### 其他添加元素的方法
1.批量添加元素，默认加载因子

   ```java
    public HashMap(Map<? extends K, ? extends V> m) {
       this.loadFactor = DEFAULT_LOAD_FACTOR;
       putMapEntries(m, false);
    }
   ```
真正批量添加元素的方法为putAll()

   ```java
    public void putAll(Map<? extends K, ? extends V> m) {
       putMapEntries(m, true);
    }
   ```
 
   ```java
    //同样第二参数代表是否初次创建 table 
     final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
       int s = m.size();
       if (s > 0) {
            //如果哈希表为空则初始化参数扩容阈值
           if (table == null) { // pre-size
               float ft = ((float)s / loadFactor) + 1.0F;
               int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                        (int)ft : MAXIMUM_CAPACITY);
               if (t > threshold)
                   threshold = tableSizeFor(t);
           }
           else if (s > threshold)//构造方法没有计算 threshold 默认为0 所以会走扩容函数
               resize();
            //将参数中的 map 键值对依次添加到 HashMap 中
           for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
               K key = e.getKey();
               V value = e.getValue();
               putVal(hash(key), key, value, false, evict);
           }
       }
    }
   ```
   
### HashMap 查询元素
put和get往往是成对存在的，下面我们来看看hashMap的get方法

1.根据键值对的key去获取对应的value

   ```java
    public V get(Object key) {
       Node<K,V> e;
       //通过 getNode寻找 key 对应的 Value 如果没找到，或者找到的结果为 null 就会返回null 否则会返回对应的 Value
       return (e = getNode(hash(key), key)) == null ? null : e.value;
    }
    
    final Node<K,V> getNode(int hash, Object key) {
       Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
       //现根据 key 的 hash 值去找到对应的链表或者红黑树
       if ((tab = table) != null && (n = tab.length) > 0 &&
           (first = tab[(n - 1) & hash]) != null) {
           // 如果第一个节点就是,那么直接返回
           if (first.hash == hash && // always check first node
               ((k = first.key) == key || (key != null && key.equals(k))))
               return first;
            //如果 对应的位置为红黑树调用红黑树的方法去寻找节点   
           if ((e = first.next) != null) {
               if (first instanceof TreeNode)
                   return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                //遍历单链表找到对应的 key 和 Value   
               do {
                   if (e.hash == hash &&
                       ((k = e.key) == key || (key != null && key.equals(k))))
                       return e;
               } while ((e = e.next) != null);
           }
       }
       return null;
    }
   ```
2.JDK 1.8新增get方法，在未找到key对应的value值时返回默认值

   ```java
    @Override
    public V getOrDefault(Object key, V defaultValue) {
       Node<K,V> e;
       return (e = getNode(hash(key), key)) == null ? defaultValue : e.value;
    }
   ```

### HashMap 的删操作

   ```java
    public V remove(Object key) {
       Node<K,V> e;
       return (e = removeNode(hash(key), key, null, false, true)) == null ?
           null : e.value;
    }
   ```
   
   ```java
    @Override
    public boolean remove(Object key, Object value) {
       //这里传入了value 同时matchValue为true
       return removeNode(hash(key), key, value, true, true) != null;
    }
   ```
删除时有两个需要注意的参数
- matchValue : 如果这个值为true则表示只有当Value与第三个Value相同的时候才删除一个节点
- movable : 按树的方式删除时的参数

   ```java
    final Node<K,V> removeNode(int hash, Object key, Object value,
                                   boolean matchValue, boolean movable) {
       Node<K,V>[] tab; Node<K,V> p; int n, index;
       //判断哈希表是否为空，长度是否大于0 对应的位置上是否有元素
       if ((tab = table) != null && (n = tab.length) > 0 &&
           (p = tab[index = (n - 1) & hash]) != null) {
           
           // node 用来存放要移除的节点， e 表示下个节点 k ，v 每个节点的键值
           Node<K,V> node = null, e; K k; V v;
           //如果第一个节点就是我们要找的直接赋值给 node
           if (p.hash == hash &&
               ((k = p.key) == key || (key != null && key.equals(k))))
               node = p;
           else if ((e = p.next) != null) {
                // 遍历红黑树找到对应的节点
               if (p instanceof TreeNode)
                   node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
               else {
                    //遍历对应的链表找到对应的节点
                   do {
                       if (e.hash == hash &&
                           ((k = e.key) == key ||
                            (key != null && key.equals(k)))) {
                           node = e;
                           break;
                       }
                       p = e;
                   } while ((e = e.next) != null);
               }
           }
           // 如果找到了节点
           // !matchValue 是否不删除节点
           // (v = node.value) == value ||
                                (value != null && value.equals(v))) 节点值是否相同，
           if (node != null && (!matchValue || (v = node.value) == value ||
                                (value != null && value.equals(v)))) {
               //删除节点                 
               if (node instanceof TreeNode)
                   ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
               else if (node == p)
                   tab[index] = node.next;
               else
                   p.next = node.next;
               ++modCount;
               --size;
               afterNodeRemoval(node);
               return node;
           }
       }
       return null;
    }
   ```