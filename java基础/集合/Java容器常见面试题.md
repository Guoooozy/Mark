# Java容器常见面试题

## 1.集合概述

### 1.1 说说List，Set，Map三者的区别

- `List`（顺序）：存储的元素的是有序的，可重复的
- `Set`（不可重复）：存储的元素是无序的，不可重复的
- `Map`（使用Key来搜索）：使用键值对来存储，key是无序不可重复，value是无序，可重复的

### 1.2集合框架底层数据结构

#### 1.2.1 List

- `ArrayList`：`Object[]`数组
- `Vector`：`Object[]`数组
- `LinkedList`：双向链表（1.6之前为循环链表，1.7之后取消了循环）

#### 1.2.2 Set

- `HashSet`(无序，唯一)：基于`HashMap`实现，底层使用`HashMap`来保存元素
- `LinkedHashSet`：`LinkedHashSet`是`HashSet`的子类，并且内部是通过`LinkedHashMap`实现的，类似于`LinkedHashMap` 内部是基于`HashMap`实现一样
- `TreeSet`（有序，唯一）：红黑树（自平衡的排序二叉树）

#### 1.2.3 Map

- `HashMap`： JDK1.8 之前 `HashMap` 由数组+链表组成的，数组是 `HashMap` 的主体，链表则是主要为了解决哈希冲突而存在的（“拉链法”解决冲突）。JDK1.8 以后在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为 8）（将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树）时，将链表转化为红黑树，以减少搜索时间
- `LinkedHashMap`： `LinkedHashMap` 继承自 `HashMap`，所以它的底层仍然是基于拉链式散列结构即由数组和链表或红黑树组成。另外，`LinkedHashMap` 在上面结构的基础上，增加了一条双向链表，使得上面的结构可以保持键值对的插入顺序。同时通过对链表进行相应的操作，实现了访问顺序相关逻辑。
- `Hashtable`： 数组+链表组成的，数组是 `HashMap` 的主体，链表则是主要为了解决哈希冲突而存在的
- `TreeMap`： 红黑树（自平衡的排序二叉树）

## 2.Collection子接口之List

### 2.1ArrayList和Vector的区别？

- ArrayList 是List 的主要实现类，底层使用数组存储，适用于频繁的查找工作，线程不安全
- Vector 是List 古老的实现类，底层使用数组存贮，线程安全

### 2.2 ArrayList和LinkedList的区别？

1. 是否保证线程安全？：两者都是不同步的，都不保证线程安全
2. 底层数据结构：ArrayList使用的是数组，LinkedList使用的是双向链表（不循环）
3. 插入和删除是否受元素位置的影响：
   - ArrayList 采用数组存储，所以插入和删除手元素位置的影响，
   - LinkedList 采用链表，所以插入删除不收元素位置影响
4. 是否支持快速随机方位：LinkedList不支持高效的随机访问，ArrayList支持。快速随机访问就是通过元素的序号，快速获取元素对象（对应于get（int index）方法）
5. 内存空间占用：ArrayList的空间浪费主要体现在list列表的结尾会预留一些空间，而LinkedList的空间花费体现在它的每一个元素都需要消耗比ArrayList更多的空间（要存放直接后继和直接前驱以及数据）

## 3.Collection子接口之Set

### 3.1.无序性和不可重复性的含义是什么

1. 什么是无序性？无序性不等于随机性，无序性是指存储的数据在底层数组中并非按照数组索引的顺序添加，而是根据数据的哈希值决定的
2. 什么是不可重复性？不可重复性是值，添加元素按照equals判断时，返回false，需要同时重写equals方法和HashCode方法

## 4.Map接口

### 4.1 HashMap 和 Hashtable的区别

1. **线程是否安全：** `HashMap` 是非线程安全的，`HashTable` 是线程安全的,因为 `HashTable` 内部的方法基本都经过`synchronized` 修饰。（如果你要保证线程安全的话就使用 `ConcurrentHashMap` 吧！）；
2. **效率：** 因为线程安全的问题，`HashMap` 要比 `HashTable` 效率高一点。另外，`HashTable` 基本被淘汰，不要在代码中使用它；
3. **对 Null key 和 Null value 的支持：** `HashMap` 可以存储 null 的 key 和 value，但 null 作为键只能有一个，null 作为值可以有多个；HashTable 不允许有 null 键和 null 值，否则会抛出 `NullPointerException`。
4. **初始容量大小和每次扩充容量大小的不同 ：** ① 创建时如果不指定容量初始值，`Hashtable` 默认的初始大小为 11，之后每次扩充，容量变为原来的 2n+1。`HashMap` 默认的初始化大小为 16。之后每次扩充，容量变为原来的 2 倍。② 创建时如果给定了容量初始值，那么 Hashtable 会直接使用你给定的大小，而 `HashMap` 会将其扩充为 2 的幂次方大小（`HashMap` 中的`tableSizeFor()`方法保证，下面给出了源代码）。也就是说 `HashMap` 总是使用 2 的幂作为哈希表的大小,后面会介绍到为什么是 2 的幂次方。
5. **底层数据结构：** JDK1.8 以后的 `HashMap` 在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为 8）（将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树）时，将链表转化为红黑树，以减少搜索时间。Hashtable 没有这样的机制。

### 4.2 HashMap 和 HashSet 的区别 

`HashSet` 底层就是基于 `HashMap` 实现的。（`HashSet` 的源码非常非常少，因为除了 `clone()`、`writeObject()`、`readObject()`是 `HashSet` 自己不得不实现之外，其他方法都是直接调用 `HashMap` 中的方法。

| `HashMap`                              | `HashSet`                                                    |
| -------------------------------------- | ------------------------------------------------------------ |
| 实现了 `Map` 接口                      | 实现 `Set` 接口                                              |
| 存储键值对                             | 仅存储对象                                                   |
| 调用 `put()`向 map 中添加元素          | 调用 `add()`方法向 `Set` 中添加元素                          |
| `HashMap` 使用键（Key）计算 `hashcode` | `HashSet` 使用成员对象来计算 `hashcode` 值，对于两个对象来说 `hashcode` 可能相同，所以` equals()`方法用来判断对象的相等性 |

### 4.3 HashMap 和TreeMap的区别

主要区别就是**`HashMap`来说 `TreeMap` 主要多了对集合中的元素根据键排序的能力以及对集合内元素的搜索的能力。**

### 4.4 HashSet 如何检查重复

当你把对象加入`HashSet`时，`HashSet` 会先计算对象的`hashcode`值来判断对象加入的位置，同时也会与其他加入的对象的 `hashcode` 值作比较，如果没有相符的 `hashcode`，`HashSet` 会假设对象没有重复出现。但是如果发现有相同 `hashcode` 值的对象，这时会调用`equals()`方法来检查 `hashcode` 相等的对象是否真的相同。如果两者相同，`HashSet` 就不会让加入操作成功。