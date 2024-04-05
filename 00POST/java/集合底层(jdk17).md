# ArrayList
## 继承结构和层次关系
```java
public class ArrayList<E> extends AbstractList<E>  
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```
1. 可以看出ArratList是继承了AbstractList，这是因为抽象类中可以有抽象方法，还可以有具体的实现方法，AbstractList是实现List接口中一些通用的方法，而ArrayList就继承这个AbstractList类，拿到一些通用的方法，然后自己再实现一些自己特有的方法，这样一来，让代码更简洁，并且如果需要实现其它List，也可以使其继承AbstractList从而实现代码复用
2. 实现List接口作用不明，可能是为了查看代码方便，使观看者了解到其是List的一种实现（尽管是通过继承AbstractList间接实现）
3. RandomAccess 是一个标记接口，用于表明一个类的实例支持随机访问。具体来说，实现了 `RandomAccess` 接口的类可以通过索引直接访问其元素，而不需要进行迭代`ArrayList` 实现了 `RandomAccess` 接口，因为它内部使用数组实现，可以快速随机访问元素。
4. Cloneable 是一个标记接口，用于表示类的实例可以被克隆。实现了 `Cloneable` 接口的类，可以通过 `Object` 类中的 `clone()` 方法创建该类的副本。`ArrayList` 实现了 `Cloneable` 接口，因此可以通过 `clone()` 方法复制 `ArrayList` 的实例。
5. Serializable是一个标记接口，用于表示类的实例可以被序列化。序列化是将对象转换为字节流的过程，可以用于对象的存储和网络传输。`ArrayList` 实现了 `Serializable` 接口，因此可以通过 Java 序列化机制将 `ArrayList` 的实例转换为字节流进行存储和传输。
## 构造方法
ArrayList提供了三种构造方法：
1. public ArrayList()
2. public ArrayList(int initialCapacity)
3. public ArrayList(Collection\<? extends E> c)
```java
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

public ArrayList() {  
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;  
}
```
通过无参构造可以发现ArrayList底层就是一个**Object\[]数组**，并且初始化时是空的，这说明它其实是**懒加载**
## 添加元素：add
```java
// `E e`：要添加到 `ArrayList` 的元素。
// `Object[] elementData`：`ArrayList` 内部用于存储元素的数组。
// `int s`：当前 `ArrayList` 中已经存储的元素数量（即大小）。
private void add(E e, Object[] elementData, int s) {  
    if (s == elementData.length)  
        elementData = grow();  
    elementData[s] = e;  
    size = s + 1;  
}
```
添加元素的add方法按如下流程执行：
1. 方法检查当前 `ArrayList` 中是否有足够的容量来存储新的元素。如果内部数组 `elementData` 的长度 `s` 等于 `elementData.length`，意味着数组已经满了，需要调用 `grow()` 方法来扩容。
2. 将元素 `e` 添加到内部数组的第 `s` 个位置（数组索引从0开始）。
3. 更新 `ArrayList` 的大小 `size`，将其设为 `s + 1`，表示添加了一个新元素。
## 动态扩容：grow
通过上述观察add源码，我们可以发现ArrayList动态扩容的关键方法就是grow方法，其实现：
```java
private Object[] grow(int minCapacity) {  
    int oldCapacity = elementData.length;  
    if (oldCapacity > 0 || elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {  
        int newCapacity = ArraysSupport.newLength(oldCapacity,  
                minCapacity - oldCapacity, /* minimum growth */  
                oldCapacity >> 1           /* preferred growth */);  
        return elementData = Arrays.copyOf(elementData, newCapacity);  
    } else {  
        return elementData = new Object[Math.max(DEFAULT_CAPACITY, minCapacity)];  
    }  
}
```
该方法按以下流程对ArrayList进行扩容：
1. 获取当前内部数组 `elementData` 的旧容量 `oldCapacity`。
2. 检查旧容量是否大于0，或者 `elementData` 是否等于 `DEFAULTCAPACITY_EMPTY_ELEMENTDATA`（即之前提到的懒加载初始化的空数组）
3. 如果上述条件成立，计算出新容量 `newCapacity`，这里使用了一个辅助方法 `ArraysSupport.newLength()` 来确定新容量的大小。该方法会根据旧容量、最小增长量和首选增长量来计算新的容量。
4. 使用 `Arrays.copyOf()` 方法创建一个新的数组，并将旧数组中的元素拷贝到新数组中，最后返回新数组。
5. 如果上述条件不成立（即当前内部数组为空），则直接创建一个新的 `Object` 类型数组，其长度为 `DEFAULT_CAPACITY` 和 `minCapacity` 中的较大值，并返回该数组。 `DEFAULT_CAPACITY` 是 `ArrayList` 内部的默认初始容量10。
也就是说，**当第一次添加元素时，ArrayList会给一个初始容量为10的数组；后续每次添加元素时如果数组长度不够，会动态计算出新的数组长度，并采用复制的方法进行元素转移完成添加元素时的扩容**
## 计算新数组长度：newLength和hugeLength
它们都是ArraysSupport中的静态方法，作用是根据旧数组的长度以及最小增长量和首选增长量来确定新数组的长度。
- **最小增长量（minGrowth）**：这个值表示在确定新数组长度时，至少需要增加的元素数量。即使在当前情况下并不需要这么多的增长，但至少会增加这么多的容量。这个值通常用于确保扩容操作不会频繁地触发，以提高性能。
-  **首选增长量（prefGrowth）**：这个值表示在确定新数组长度时的首选增长数量。当需要扩容时，新数组的长度会增加至少这个值，但如果需求更大，那么新数组的长度会相应地增加更多。这个值通常用于平衡内存的使用和性能之间的权衡，以及减少扩容操作的次数。
```java
public static int newLength(int oldLength, int minGrowth, int prefGrowth) {  
    int prefLength = oldLength + Math.max(minGrowth, prefGrowth); 
    if (0 < prefLength && prefLength <= SOFT_MAX_ARRAY_LENGTH) {  
        return prefLength;  
    } else {  
        // put code cold in a separate method  
        return hugeLength(oldLength, minGrowth);  
    }  
}

private static int hugeLength(int oldLength, int minGrowth) {  
    int minLength = oldLength + minGrowth;  
    if (minLength < 0) { // overflow  
        throw new OutOfMemoryError(  
            "Required array length " + oldLength + 
            " + " + minGrowth + " is too large");  
    } else if (minLength <= SOFT_MAX_ARRAY_LENGTH) {  
        return SOFT_MAX_ARRAY_LENGTH;  
    } else {  
        return minLength;  
    }  
}
```
newLength方法中的步骤如下：
1. 计算出首选长度 `prefLength`，它等于旧长度加上最小增长量和首选增长量中的较大值。这里的 `prefLength` 可能会溢出，但是因为这个方法是内联的，所以预先条件没有被检查。
2. 检查 `prefLength` 是否在 0 到 `SOFT_MAX_ARRAY_LENGTH` 之间（`SOFT_MAX_ARRAY_LENGTH` 是一个软限制，表示数组的最大长度）。
```java
//在保留了足够大的数组长度的同时，为额外的元数据留出了一些空间。这样做的目的是为了在不同的 JVM 实现中都能保证数组长度不会超出最大值并且不会发生溢出
public static final int SOFT_MAX_ARRAY_LENGTH = Integer.MAX_VALUE - 8;
```
3. 如果 `prefLength` 符合条件，则直接返回该值作为新数组的长度。
4. 如果 `prefLength` 超出范围，则调用另一个方法 `hugeLength()`，这个方法会针对超出软限制的情况进行处理，并返回新数组的长度。
hugeLength方法中的步骤如下：
1. 计算出最小长度 `minLength`，它等于旧长度加上最小增长量。这里的 `minLength` 可能会溢出，但是在溢出之前会先进行判断。
2. 检查 `minLength` 是否小于0，如果小于0，则表示溢出了，抛出 `OutOfMemoryError` 异常。
3. 如果 `minLength` 小于等于软限制 `SOFT_MAX_ARRAY_LENGTH`，则返回软限制作为新数组的长度。
4. 如果 `minLength` 超出了软限制，则直接返回 `minLength` 作为新数组的长度。
**综上我们可以总结出ArrayList扩容后新数组与旧数组之间的长度关系，有几种可能：**
1. **如果原数组长度的1.5倍仍在软限制范围内，则直接返回1.5倍**
2. **如果原数组长度的1.5倍已经超过限制，比较现在所需的最小长度与软限制之间的关系**
	- **若最小长度已经为负数，表明元素数目已经超出int最大值，抛异常**
	- **若最小长度小于软限制，返回软限制（无法达到1.5倍，就尽可能大）**
	- **若最小长度大于软限制，返回最小长度**
# LinkedList
## 继承结构和层次关系
```java
public class LinkedList<E> extends AbstractSequentialList<E> 
	implements List<E>, Deque<E>, Cloneable, java.io.Serializable {
```
- `List` : 表明它是一个列表，支持添加、删除、查找等操作，并且可以通过下标进行访问。
- `Deque` ：继承自 `Queue` 接口，具有双端队列的特性，支持从两端插入和删除元素，方便实现栈和队列等数据结构。
- `Cloneable` ：表明它具有拷贝能力，可以进行深拷贝或浅拷贝操作。
- `Serializable` : 表明它可以进行序列化操作，也就是可以将对象转换为字节流进行持久化存储或网络传输，非常方便。
## 节点：node
```java
private static class Node<E> {  
    E item;  
    Node<E> next;  
    Node<E> prev;  
  
    Node(Node<E> prev, E element, Node<E> next) {  
        this.item = element;  
        this.next = next;  
        this.prev = prev;  
    }  
}
```
可以看出，LinkedList每个元素都保存着前一个和后一个元素的指针，因此它是一个双向链表
除此之外，它还维护了头节点和尾节点：
```java
transient Node<E> first;  

transient Node<E> last;
```
## 构造方法
```java
//创建一个空的链表对象
public LinkedList() { } 
// 接收一个集合类型作为参数，会创建一个与传入集合相同元素的链表对象 
public LinkedList(Collection<? extends E> c) { 
	this(); 
	addAll(c); 
}
```
## 添加、插入元素：add
LinkedList提供了重载的两个add方法，分别支持直接在尾部添加元素和在指定索引前面插入元素
```java
//添加元素
public boolean add(E e) {  
    linkLast(e);  
    return true;  
}
//插入元素
public void add(int index, E element) {  
    checkPositionIndex(index);  
  
    if (index == size)  
        linkLast(element);  
    else  
	    //node(index) 用于指向第index个节点并返回它
        linkBefore(element, node(index));  
}
```
## 获取元素
1. `getFirst()`：获取链表的第一个元素。
2. `getLast()`：获取链表的最后一个元素。
3. `get(int index)`：获取链表指定位置的元素。
## 删除元素
- `removeFirst()`：删除并返回链表的第一个元素。
- `removeLast()`：删除并返回链表的最后一个元素。
- `remove(E e)`：删除链表中首次出现的指定元素，如果不存在该元素则返回 false。
- `remove(int index)`：删除指定索引处的元素，并返回该元素的值。
- `void clear()`：移除此链表中的所有元素。
# HashMap
## 继承结构和层次关系
```java
public class HashMap<K,V> extends AbstractMap<K,V>  
    implements Map<K,V>, Cloneable, Serializable 
```
## 底层数据结构
HashMap底层的数据结构是数组+链表+红黑树
### 数组
HashMap维护了成员变量Node数组来存储元素
```java
transient Node<K,V>[] table;
```
### 链表
数组中存放链表的头节点，各节点组成链表
```java
static class Node<K,V> implements Map.Entry<K,V> {  
    final int hash;  
    final K key;  
    V value;  
    Node<K,V> next;  
  
    Node(int hash, K key, V value, Node<K,V> next) {  
        this.hash = hash;  
        this.key = key;  
        this.value = value;  
        this.next = next;  
    }
	    ···（其它方法）
}
```
### 红黑树
jdk8之后的优化，当链表长度过长时，会将链表转化为红黑树以增加元素插入和查询的效率
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
  
    /**  
     * Returns root of tree containing this node.     
     * */    
    final TreeNode<K,V> root() {  
        for (TreeNode<K,V> r = this, p;;) {  
            if ((p = r.parent) == null)  
                return r;  
            r = p;  
        }  
    }
    ···（其它方法）
}
```
## 构造方法
HashMap提供了四种构造方法：
1. public HashMap(int initialCapacity, float loadFactor)
2. public HashMap(int initialCapacity)
3. public HashMap()
4. public HashMap(Map\<? extends K, ? extends V> m) 
使用构造方法时，为初始容量赋值时，尽量赋为2的次幂，这有利于提升底层hash的效率；
不过就算没有这么做，HashMap进行初始化时也会取小于给定容量的最大2的次幂，例如，赋值20，最终得到的是长度为16的HashMap
负载因子（loadFactor）的默认值为0.75，当已用空间/总空间大于等于该值时，会进行扩容为原来的两倍，例如，当长度为16的HashMap存放了12个以上元素时，会扩容为32
## 添加元素
HashMap中的元素是无序的（添加顺序和存储顺序不一致），这与它添加元素时的操作有关
### hash
```java
static final int hash(Object key) {  
    int h;  
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);  
}
```
通过该方法可以看出，当传入元素（key）不为null时，进行了两步来进行hash
1. 调用对象的 `hashCode()` 方法获取哈希码，并将结果保存到变量 `h` 中。
2. 执行一个位运算，将 `h` 右移 16 位，并与 `h` 进行异或操作（^）。这一步是为了增加哈希值的随机性，以减少哈希冲突的可能性。
### putVal
```java
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
            for (int binCount = 0; ; ++binCount) {  
                if ((e = p.next) == null) {  
                    p.next = newNode(hash, key, value, null);  
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st  
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
    afterNodeInsertion(evict);  
    return null;  
}
```
1. 首先，获取当前 HashMap 的数组 `table`，并获取数组长度 `n`。
2. 判断数组是否为空，若为空，调用 `resize()` 方法进行初始化，并重新获取数组长度 `n`。
3. 计算键在数组中的位置 `i`。（hash方法计算出的值与数组长度-1进行`&`运算）
4. 判断该位置是否为空，如果为空，则直接在该位置插入新节点。
5. 如果该位置不为空，且节点的哈希值和键与要插入的相同，则更新该节点的值。
6. 如果该位置的节点是树节点，则调用树节点的 `putTreeVal` 方法进行插入。
7. 否则，遍历链表，查找是否存在相同的键。**如果存在，则更新值；如果不存在，则将新节点插入到链表尾部**，并根据链表长度进行相应的处理（是否需要转化为红黑树，调用treeifyBin方法转换，这里的阈值TREEIFY_THRESHOLD - 1为7，由于长度从0开始计数，因此**当链表长度为8时进行转换**）。
8. 如果找到了相同的键，则更新值，并返回旧值；否则，返回 null。
9. 更新 HashMap 的修改次数 `modCount`，并根据大小阈值判断是否需要进行 resize 操作。
10. 调用 `afterNodeInsertion` 方法，进行插入后的操作。
11. 返回旧值（如果存在）或 null。
## 扩容：resize
resize方法源码过长，这里不再展示，仅进行说明
1. 数组的初始化以及扩容均通过resize进行，都是通过创建一个新的数组，长度为默认长度或原数组的两倍
2. 获取旧的数组 `oldTab`、旧的数组长度 `oldCap` 和阈值 `oldThr`。
3. 根据旧数组的情况确定新数组的大小 `newCap` 和新的阈值 `newThr`。
	- 如果当前数组长度为0，创建默认长为16的数组
	- 如果当前数组长度大于2的30次方（扩大两倍后超出int最大值），创建长为int最大值的数组
	- 创建长度为原数组长度两倍的数组
4. 将旧数组中的元素重新分配到新数组中（复制），涉及到链表节点、红黑树节点等情况的处理。
5. 扩容过程会伴随着一次重新 hash 分配，并且会遍历 hash 表中所有的元素，是非常耗时的。在编写程序中，要尽量避免 resize。
# ConcurrentHashMap
ConcurrentHashMap1.7与1.8的实现有点区别，下面主要是解析1.8之后的实现
## 继承结构和层次关系
```java
public class ConcurrentHashMap<K,V> extends AbstractMap<K,V>  
    implements ConcurrentMap<K,V>, Serializable 
```
ConcurrentMap 接口是一个扩展了 Map 接口的并发安全的接口，它提供了一些**支持并发访问**的方法。
## 构造方法
1. public ConcurrentHashMap() 
2. public ConcurrentHashMap(int initialCapacity)
3. public ConcurrentHashMap(Map\<? extends K, ? extends V> m)
4. public ConcurrentHashMap(int initialCapacity, float loadFactor)
5. public ConcurrentHashMap(int initialCapacity,  float loadFactor, int concurrencyLevel)
```java
/**
* @param initialCapacity 初始容量  
* @param loadFactor 负载因子  
* @param concurrencyLevel 并发级别
*/
public ConcurrentHashMap(int initialCapacity,  
                         float loadFactor, int concurrencyLevel) {  
    if (!(loadFactor > 0.0f) || initialCapacity < 0 || concurrencyLevel <= 0)  
        throw new IllegalArgumentException();  
    if (initialCapacity < concurrencyLevel)   // Use at least as many bins  
        initialCapacity = concurrencyLevel;   // as estimated threads  
    long size = (long)(1.0 + (long)initialCapacity / loadFactor);  
    int cap = (size >= (long)MAXIMUM_CAPACITY) ?  
        MAXIMUM_CAPACITY : tableSizeFor((int)size);  
    this.sizeCtl = cap;  
}
```
有几个细节需要注意：
1. 并发级别指的是 ConcurrentHashMap 在内部使用的分段锁的数量，也就是内部数据结构中的桶（bins）的数量。在 ConcurrentHashMap 中，数据被分成多个段（segments），每个段上都有一把锁，这样不同的线程可以同时对不同的段进行操作，从而提高了并发性能。
2. 初始容量必须大于并发级别
3. 参数initialCapacity并不是指定最终的数组大小，内部经过了计算，最终的值一定是2的幂（通过tableSizeFor方法保证）
### 关于sizeCtl
`sizeCtl` 的作用是作为一个控制参数，用于监控 ConcurrentHashMap 的状态并在需要时触发扩容、收缩或其他调整大小的操作。
1. **负数值**:
    - 当 `sizeCtl` 的值为负数时，表示 ConcurrentHashMap 正在进行初始化，扩容或收缩等操作。具体的值代表了当前操作的状态和进度。
    - 例如，如果 `sizeCtl` 的值为 `-1`，表示 ConcurrentHashMap 正在进行初始化。
    - 如果 `sizeCtl` 的值为 `-N`（N 大于 1），表示 ConcurrentHashMap 正在进行扩容操作，其中 `-N` 代表当前正在进行扩容的阶段（比如 `-2` 表示第二阶段扩容）。
2. **零值**:
    - 当 `sizeCtl` 的值为零时，表示 ConcurrentHashMap 的初始化已经完成，且当前没有扩容或收缩操作在进行。
3. **正数值**:
    - 当 `sizeCtl` 的值为正数时，表示 ConcurrentHashMap 中的有效元素数量的估计值。这个值是通过哈希表中的节点数量和状态信息计算得出的，并且可能会用于触发扩容操作。
    - 如果 `sizeCtl` 的值大于等于 `MIN_TREEIFY_CAPACITY`，则表示需要将桶（bucket）转换为红黑树结构。
    - 如果 `sizeCtl` 的值小于零但不是负数，并且不是 `MIN_TREEIFY_CAPACITY`，则表示 ConcurrentHashMap 正在进行收缩操作。
## 初始化initTable
```java
/**
 * Initializes table, using the size recorded in sizeCtl.
 */
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    while ((tab = table) == null || tab.length == 0) {
        //　如果 sizeCtl < 0 ,说明另外的线程执行CAS 成功，正在进行初始化。
        if ((sc = sizeCtl) < 0)
            // 让出 CPU 使用权
            Thread.yield(); // lost initialization race; just spin
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            try {
                if ((tab = table) == null || tab.length == 0) {
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
                    sc = n - (n >>> 2);
                }
            } finally {
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}
```
1. 初始化是通过**自旋和 CAS** 操作完成的。
2. 返回的结果是node数组（并且这里的node实现自Map.Entry）
3. 通过sizeCtl控制当前的初始化状态
## 加入元素putVal
```java
public V put(K key, V value) {  
    return putVal(key, value, false);  
}  
  
/** Implementation for put and putIfAbsent */  
final V putVal(K key, V value, boolean onlyIfAbsent) {  
    if (key == null || value == null) throw new NullPointerException();  
    int hash = spread(key.hashCode());  
    int binCount = 0;  
    for (Node<K,V>[] tab = table;;) {  
        Node<K,V> f; int n, i, fh; K fk; V fv;  
        if (tab == null || (n = tab.length) == 0)  
	        //数组桶为空，初始化数组桶（自旋+CAS)
            tab = initTable();  
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {  
			// 桶内为空，CAS 放入，不加锁，成功了就直接 break 跳出
            if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value)))  
                break;                // no lock when adding to empty bin  
        }  
        else if ((fh = f.hash) == MOVED)  
            tab = helpTransfer(tab, f);  
        else if (onlyIfAbsent // check first node without acquiring lock  
                 && fh == hash  
                 && ((fk = f.key) == key || (fk != null && key.equals(fk)))  
                 && (fv = f.val) != null)  
            return fv;  
        else {  
            V oldVal = null;  
			//使用 synchronized 加锁加入节点
            synchronized (f) {  
                if (tabAt(tab, i) == f) {  
                    if (fh >= 0) {  
                        binCount = 1;  
                        for (Node<K,V> e = f;; ++binCount) {  
                            K ek;  
                            if (e.hash == hash &&  
                                ((ek = e.key) == key ||  
                                 (ek != null && key.equals(ek)))) {  
                                oldVal = e.val;  
                                if (!onlyIfAbsent)  
                                    e.val = value;  
                                break;  
                            }  
                            Node<K,V> pred = e;  
                            if ((e = e.next) == null) {  
                                pred.next = new Node<K,V>(hash, key, value);  
                                break;  
                            }  
                        }  
                    }  
                    else if (f instanceof TreeBin) {  
                        Node<K,V> p;  
                        binCount = 2;  
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,  
                                                       value)) != null) {  
                            oldVal = p.val;  
                            if (!onlyIfAbsent)  
                                p.val = value;  
                        }  
                    }  
                    else if (f instanceof ReservationNode)  
                        throw new IllegalStateException("Recursive update");  
                }  
            }  
            if (binCount != 0) {  
                if (binCount >= TREEIFY_THRESHOLD)  
                    treeifyBin(tab, i);  
                if (oldVal != null)  
                    return oldVal;  
                break;  
            }  
        }  
    }  
    addCount(1L, binCount);  
    return null;  
}
```
putVal方法中的步骤如下:
1. 检查键值对中的键和值是否为空，如果为空则抛出 NullPointerException 异常。
2. 计算键的哈希值，并调用 `spread` 方法对哈希值进行扩散。
3. 进入无限循环，每次循环都会进行插入操作，直到成功插入元素或者发生了扩容操作。
4. 在循环中，首先检查哈希表是否为空，如果为空则进行初始化操作，确保哈希表的正确创建。
5. 然后，通过哈希值找到对应的桶（bucket）。如果桶为空，则尝试原子地(CAS)插入新节点到桶中。如果成功插入，则跳出循环。
6. 如果桶不为空，进一步判断桶的状态：
    - 如果桶状态为 `MOVED`，则帮助进行数据迁移操作，并继续循环。
    - 如果插入操作为 `putIfAbsent`，则尝试在不获取锁的情况下检查第一个节点是否匹配，如果匹配则返回节点值。
    - 否则，使用同步块锁定桶，进行插入操作。具体插入操作取决于桶的状态：
        - 如果桶中为普通节点，则遍历链表进行插入。
        - 如果桶中为树节点，则调用树节点的插入方法进行插入。
        - 如果桶中为 `ReservationNode`，则抛出异常。
    - 插入完成后，如果插入的节点数超过了 `TREEIFY_THRESHOLD`，则将链表转换为树结构。
7. 最后，更新计数器并返回插入前的旧值（如果存在）。
有几个值得注意的点：
1. ConcurrentHashMap也是与hashMap一样，使用了数组+链表+红黑树三种数组结构
2. ConcurrentHashMap使用了CAS和Sychornized两种方法来支持并发
	- CAS操作用于数组桶为空时，无论哪个线程先进行操作都是可以的，不会出现并发问题，并且它是乐观锁的思想，更轻量级
	- Sychornized是悲观锁的思想，针对每个桶（bucket），即node数组的一个索引位置，的操作都是在获取锁后进行的。具体来说，使用了同步块来锁定当前操作的桶，以确保在多线程环境下对桶内元素的修改操作是线程安全的。
## 获取元素get
```java
public V get(Object key) {
    Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
    // key 所在的 hash 位置
    int h = spread(key.hashCode());
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (e = tabAt(tab, (n - 1) & h)) != null) {
        // 如果指定位置元素存在，头结点hash值相同
        if ((eh = e.hash) == h) {
            if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                // key hash 值相等，key值相同，直接返回元素 value
                return e.val;
        }
        else if (eh < 0)
            // 头结点hash值小于0，说明正在扩容或者是红黑树，find查找
            return (p = e.find(h, key)) != null ? p.val : null;
        while ((e = e.next) != null) {
            // 是链表，遍历查找
            if (e.hash == h &&
                ((ek = e.key) == key || (ek != null && key.equals(ek))))
                return e.val;
        }
    }
    return null;
} 
```
get方法中的步骤如下:
1. 通过 `spread(key.hashCode())` 方法计算出 key 的哈希值，以便确定该 key 在哈希表中的位置。
2. 检查哈希表是否已经初始化并且不为空，以及指定位置的桶是否包含元素。
3. 如果指定位置的桶不为空，则进一步遍历该桶中的元素：
    - 如果头结点的哈希值与给定的 key 的哈希值相等，并且头结点的 key 与给定的 key 相等，则直接返回头结点的值。
    - 如果头结点的哈希值小于 0，说明该桶正在进行扩容或者已经是一个红黑树结构，则调用头结点的 find 方法来进行查找操作。
    - 如果不满足以上两种情况，则遍历该桶中的链表，查找与给定 key 相等的元素。
4. 如果遍历结束仍然没有找到匹配的元素，则返回 null。
# 总结
1. ArratList：动态数组，可扩容
2. LinkedList：链表
3. HashMap：数组+链表+红黑树，可扩容
4. ConcurrentHashMap：数组+链表+红黑树，可扩容，通过CAS+Sychornized支持并发
# 补充
## 问题
Q：HashMap数组中存放指针指向链表或红黑树头节点，为什么数组可以存放两种指针呢？
A：可以看到TreeNode继承自LinkedHashMap.Entry<K,V>，稍微跟进一下就能发现，LinkedHashMap.Entry<K,V>继承自HashMap.Node<K,V>！也就是说，TreeNode实际上是Node的孙子，这也就解释了为什么数组中可以同时存放Node和TreeNode两种对象

Q：为什么HashMap计算元素索引时使用(n - 1) & hash而非n&hash？
A：用一个具体的例子来说明。
假设数组长度 `n` 是 16，即 `n = 16`，而键的哈希值 `hash` 是 20，即 `hash = 20`。
如果使用 `n & hash` 来计算键在数组中的位置，那么计算结果是16，但是数组的长度只有 16，超出了数组的范围；
现在使用 `(n - 1) & hash` 来计算键在数组中的位置，即 `(16 - 1) & 20`：结果是4，这样计算可以确保结果在数组范围内，并且能够均匀地分布在数组的不同位置。