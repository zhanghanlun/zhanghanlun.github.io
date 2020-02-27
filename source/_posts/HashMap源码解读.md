---
title: HashMap源码解读—Java8
date: 2018-04-25 17:24:45
tags:
	- Java
	- HashMap
categories: Java
---

# 前言
&emsp;&emsp;在我们面试过程中，经常会遇到要求说HashMap的底层实现，在JDK源码中，Oracle公司给出了我们HashMap的源码，通过阅读HashMap的源码，我们可以很清楚的知道HashMap是怎么实现的。下面我们开始阅读HashMap的源码吧。
<!-- more -->
# 关于HashMap的类的继承与实现
```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
```
通过上面的代码我们知道了HashMap继承了AbstractMap这个类，实现了Map，Cloneable还有Serializable这个接口
# HashMap类中的常量

```java
    private static final long serialVersionUID = 362498820763181265L;//序列化ID
  	/**
     * 默认的初始容量一定是2的倍数，在这里是16
     */
    static final int DEFAULT_INITIAL_CAPACITY = 1 <<4;

    /**
     * 最大容量一定是2的倍数，而且小于2的30次方
     */
    static final int MAXIMUM_CAPACITY = 1 << 30;

    /**
     * 默认的负载因子是0.75
     */
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    /**
    * 当HashMap桶中的箱子的数量达到8的时候就会将
    * 箱子的数据结构由链表转换成红黑树
    */
    static final int TREEIFY_THRESHOLD = 8;
    /**
    * 由树转换成链表的阈值
    * 在HashMap进行扩容的时候，当树中结点的个数小于6的时候有，由树转换成链表
    */
    static final int UNTREEIFY_THRESHOLD = 6;
    /**
    * 桶可能被树化的最小容量，至少为4*TREEIFY_THRESHOLD，
    * 避免扩容和树化阈值之间的冲突。
    * 只有当表的容量达到64之后才能进行树化
    */
    static final int MIN_TREEIFY_CAPACITY = 64;
      /**
     * 表，数组实现的结构
     */
    transient Node<K,V>[] table;

    /**
     * 保存缓存的entrySet
     */
    transient Set<Map.Entry<K,V>> entrySet;

    /**
     * HashMap中存在的键值对数量
     */
    transient int size;

    /**
     * HashMap结构被修改的次数
     */
    transient int modCount;

    /**
     * 下一个调整大小的值，容量*负载因子
     *
     * @serial
     */
    int threshold;

    /**
     * Hash Table的负载因子
     *
     * @serial
     */
    final float loadFactor;
```
# HashMap中的数据结构

HashMap中在底层的实现主要是基于数组和链表实现的，当然还有红黑树。在处理冲突的时候应用的是链表和红黑树相结合的方法，具体的实现，请看下面的图片：
![这里写图片描述](https://img-blog.csdn.net/20180420210741559?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5naGFubHVu/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
我们知道HashMap存储的数据都存放在Entry这个数据结构中。Entry是HashMap的存储结点。
具体的JDK源码如下：
```java
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;//final类型
        final K key;   //final类型
        V value;
        Node<K,V> next;
        /**
        * 结点构造函数
        */
        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }
        /**
        * 计算HashCode
        * 是由key的hashCode和value的HashCode进行异或产生的。
        */
        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }
        /**
        * 结点比较，地址相同则相同，Key和Value相同则相同。
        */
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

# HashMap构造函数
HashMap一共4个构造函数，分别如下：
```java
	/**
    * 默认构造函数，负载因子为默认的0.75
    */
	public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }
    /**
    * 构造函数初始化 初始容量和负载因子
    */
	public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        this.threshold = tableSizeFor(initialCapacity);
    }
    /**
    * 构造函数初始化 初始容量
    */
    public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }
    /**
    * 构造函数，对Map进行复制构造HashMap
    */
    public HashMap(Map<? extends K, ? extends V> m) {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        putMapEntries(m, false);
    }
    //承接上面的方法
    /**
     * Implements Map.putAll and Map constructor.
     *
     * @param m the map
     * @param evict false，当刚开始构造的时候
     */
    final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
        int s = m.size();
        if (s > 0) {
        	//判断原先的table是否为空，初始化参数
            if (table == null) { // pre-size
            	//计算需要的表的长度
                float ft = ((float)s / loadFactor) + 1.0F;
                int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                         (int)ft : MAXIMUM_CAPACITY);
                if (t > threshold)
                	//设置阈值
                    threshold = tableSizeFor(t);
            }
            //超过阈值，进行扩容
            else if (s > threshold)
                resize();
            //遍历Map插入Map的结点进入HashMap
            for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
                K key = e.getKey();
                V value = e.getValue();
                putVal(hash(key), key, value, false, evict);
            }
        }
    }
```

# put方法
```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
		boolean evict) {
	Node<K, V>[] tab;
    Node<K, V> p;
	int n, i;
	// 判断HashMap的table是否是空或者长度为0，执行resize操作
	if ((tab = table) == null || (n = tab.length) == 0)
		n = (tab = resize()).length;
	// (n - 1) & hash 计算在table表中的下标的值
	// 此处的table为空
	if ((p = tab[i = (n - 1) & hash]) == null)
		tab[i] = newNode(hash, key, value, null);
	else {
		Node<K, V> e;
		K k;
		// hash冲突，key相等
		if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
			e = p;
		// p是TreeNode的，调用putTreeVal方法添加TreeNode
		else if (p instanceof TreeNode)
			e = ((TreeNode<K, V>) p).putTreeVal(this, tab, hash, key, value);
		else {
			for (int binCount = 0;; ++binCount) {
				// 尾插法插入结点
				if ((e = p.next) == null) {
					p.next = newNode(hash, key, value, null);
					// 如果binCount大于TREEIFY_THRESHOLD，进行树化操作
					if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
						treeifyBin(tab, hash);
					break;
				}
				if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
					break;
				p = e;
			}
		}
			// 要插入的结点和之前的结点有相同的key值，覆盖原先的结果
		if (e != null) { // existing mapping for key
			V oldValue = e.value;
			if (!onlyIfAbsent || oldValue == null)
				e.value = value;
			afterNodeAccess(e);
			return oldValue;
		}
	}
    ++modCount;
	// 如果超过阈值，执行resize操作
	if (++size > threshold)
		resize();
	afterNodeInsertion(evict);
	return null;
}
```
通过上面的代码我们知道，put方法主要的实现是putVal方法，putVal方法的具体过程如下
>* 判断HashMap的table的长度是否为0还是null,是的话执行resize操作
>* 接着计算下该Hash值在table表中的下标，是否后结点Node，没有结点Node，直接调用new Node()方法生成Node结点插入该表中。
>* 如果在该下标处存在Node结点，则判断是否是TreeNode，如果是TreeNode，则执行putTreeVal方法。
>* 尾插法插入结点，如果改下标处的Node的结点的数量超过TREEIFY_THRESHOLD，则把Node结点树化成TreeNode结点。
>* 如果插入的结点和之前的结点有相同的key值，覆盖原先的结果。
>* 如果size超过阈值则进行resize操作。

在put函数中我们提到了resize操作，resize操作究竟是什么呢？让我们继续进行源码的阅读：
```java
 	/**
     * HashMap的扩容方法，两次幂次展开，每箱的元素放在相同索引出或者以旧表长度为偏移量移动新表
     */
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    //判断oldCap的容量
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        //扩容两倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // 初始容量被放入阈值
        newCap = oldThr;
    else {               // 零初始化阈值用默认值
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
    //重建table
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                //如果结点是TreeNode结点，那么执行split方法
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // 将该链表分成两个链表，低下标链表，高下标链表
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                //如果e.hash&oldCap==0,原来容量新增的那个bit为0，保持原索引
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                          //如果e.hash&oldCap==1,原来容量新增的那个bit为1，原索引+oldCap为新位置
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                        //原索引放到bucket中
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    //原索引+oldCap放到bucket中
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
关于resize的扩容部分的具体解释，参照了美团技术团队的一篇文章《Java 8系列之重新认识HashMap》具体如图所示：
![这里写图片描述](https://img-blog.csdn.net/20180425132027214?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5naGFubHVu/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
根据上图我们知道：
原来两个的下标都是9，后来经过扩容后Key1的下标还是9，而Key2的下标变为9+16=25。
# get方法
我们在使用HashMap的时候，除了用put方法来放置我们的键值对，而且我们还使用get方法来根据我们的key值来取相关的值。首先来看代码：
```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}
 /**
 * HashMap的getNode方法
 * @param hash hash for key
 * @param key the key
 * @return the node, or null if none
 * 判断Hash值，然后判断key值，最后再判断key.equal
 */
 final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        if (first.hash == hash && // 总是返回第一个结点
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        //第一个结点不符合的话，判断后面的结点。
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
    //所有结点都不符合的话，返回null
    return null;
}
```
相比于put方法，get方法在我们看来纯粹就是小巫见大巫了，其中的基本思想如下：
>* 先计算key对应的Hash值
>* 然后找到在table中的下标
>* 如果第一个结点的hash值相等的话，比对key值，如果相等返回该结点
>* 根据链表从后往前找，找到hash值和key值都相等的就返回
>* 如果都没有找到返回null

# 计算Hash的方法

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```
它的思想也比较简单：
>* 如果key==null，返回0
>* 计算key的HashCode记为h
>* 取h的高16位与h本身进行异或运算，即为key的Hash值

# 总结
关于hashMap的讲解，美团点评技术团队有一篇博客讲的很好，[
Java 8系列之重新认识HashMap](https://tech.meituan.com/2016/06/24/java-hashmap.html)


