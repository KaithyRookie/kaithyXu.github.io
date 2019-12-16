---
layout:     post
title:      ConcurrentHashMap源码阅读(一)
subtitle:   ConcurrentHashMap源码阅读(一)
date:       2018-12-15
author:     KaithyXu
header-img: img/thunderstorm.jpg
catalog: true
tags:
    - Java
---
## ConcurrentHashMap源码阅读


### 介绍

ConcurrentHashMap是线程安全的容器；

* 读取操作是非阻塞的，所以可能出现读与写操作同时发生的情况
* 聚合操作的状态，如获取大小，判断是否为空操作仅在没有其他线程进行写操作时有效，否则方法返回的结果只适用于这一时刻的(瞬时的)
* 当表存在过多的冲突时，会自动扩容(每一个键都会对应一个独有的hashCode,根据 hashCode对表的大小取模获取key存放的位置)，默认的负载因子是0.75,由于扩容操作很慢，建议在创建时指定一个估计的大小
* ConcurrentHashMap中健值对是无序存放的

### 结构

![image](/img/concurrentHashMap.png)

### ConcurrentHashMap使用
ConcurrentHashMap中使用节点数组保存第一个节点，

#### 写操作
1. 初始化节点数组
当ConcurrentHashMap创建后，如果节点数组数组不存在或者大小为0，则会先利用CAS操作设置节点数组的大小
```
while ((tab = table) == null || tab.length == 0) {
            if ((sc = sizeCtl) < 0)
                Thread.yield(); // lost initialization race; just spin
            else if (U.compareAndSetInt(this, SIZECTL, sc, -1)) {
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
```

2. put
put方法的流程图如下：
![image](/img/ConcurrentHashMap_Put.jpg)

写入前会去获取hashCode对应位置的节点，存在一下几种情况：

* 若返回为空，则直接采用的是CAS操作将该节点放入数组对应位置并返回。
```
static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
                                        Node<K,V> c, Node<K,V> v) {
        return U.compareAndSetObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
    }
```

* 若返回不为空，而且返回的hash标识是MOVED标志，则表示当前Map正在进行扩容，会协助进行扩容操作。
如果返回的节点是ForwardingNode，由于ForwardingNode中，其属性nextTable保存的是节点数组,所以当程序判断ForwardingNode的nextTable不为空时，则会进入协助进行扩容操作
```
final Node<K,V>[] helpTransfer(Node<K,V>[] tab, Node<K,V> f) {
        Node<K,V>[] nextTab; int sc;
        if (tab != null && (f instanceof ForwardingNode) &&
            (nextTab = ((ForwardingNode<K,V>)f).nextTable) != null) {
            int rs = resizeStamp(tab.length);
            while (nextTab == nextTable && table == tab &&
                   (sc = sizeCtl) < 0) {
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                    sc == rs + MAX_RESIZERS || transferIndex <= 0)
                    break;
                if (U.compareAndSetInt(this, SIZECTL, sc, sc + 1)) {
                    transfer(tab, nextTab);
                    break;
                }
            }
            return nextTab;
        }
        return table;
    }
```

* 若返回的节点与要插入的节点的key相同，则会返回已存在的旧值，此判断条件中有onlyIfAbsent这一条件，Put操作时将此参数设置为false，所以不会返回旧值。
* 若返回的节点与插入的节点的key相同，则会更新key对应的value。
* 若当前的Map未树化，依旧采用链表的形式连接节点，则会找到链表中的末尾节点，并将插入的节点连在末尾节点后面，作为新的尾节点。
* 若当前的Map已经树化了，则会将新插入的节点插入树中，由于ConcurrentHashMap采用的是红黑树，所以具体来看一下整个插入流程

#### 红黑树概念

1. 根节点是黑色的
2. 每个叶子节点都是黑色的空节点，即叶子节点不存储数据
3. 任何相邻的节点都不能同时为红色，即红色节点是被黑色节点隔开的
4. 每一个节点，从改节点到达其可到达叶子节点的所有路径，都包含相同数目的黑色节点

红黑树插入后的调整过程一般存在三种情况(仅以根节点的左子树为例)：
设关注节点为x，父节点为xp，祖父节点为xpp，xppr为祖父节点的右子节点，xppl为祖父节点的左子节点。

1. CASE1: 若x的叔叔节点d是红色的，执行以下操作：
    * 将xp与d的颜色都设置为黑色；
    * 将xpp的颜色设置为红色；
    * 关注节点变为x的祖父节点xpp
    * 跳转到CASE2或CASE3
2. CASE2:若x的叔叔节点d是黑色的，x是xp的右子节点，执行以下操作：
    * 将关注节点变成x的父节点xp；
    * 围绕新的关注节点b左旋；
    * 跳到CASE3

3. CASE3:若关注节点x的叔叔节点d是黑色的，x是xp的左子节点，执行以下操作：
    * 围绕x的祖父节点xpp右旋
    * 将关注节点的父节点xp与兄弟节点c的颜色互换
    * 调整结束

具体的插入操作：
```
x.red = true;
for (TreeNode<K,V> xp, xpp, xppl, xppr;;) {
    // 如果关注的节点x是根节点，则直接改变节点的颜色
    if ((xp = x.parent) == null) {
        x.red = false;
        return x;
    }
    // 若关注节点X的父节点是黑色的，或者关注节点X的祖父节点为空，无需调整
    else if (!xp.red || (xpp = xp.parent) == null)
        return root;
    // 若关注节点x的父节点是节点x的祖父节点的左子节点
    if (xp == (xppl = xpp.left)) {
        // 若祖父节点xpp的右子节点buwzks并且是红色的，对应CASE1
        if ((xppr = xpp.right) != null && xppr.red) {
            xppr.red = false;
            xp.red = false;
            xpp.red = true;
            x = xpp;
        }
        // xpp的右子节点为空或是黑色，由于红黑树中空节点为黑色，所以x的叔叔节点是黑色的情况
        else {
            // 若x是xp的右子节点，对应CASE2
            if (x == xp.right) {
                root = rotateLeft(root, x = xp);
                xpp = (xp = x.parent) == null ? null : xp.parent;
            }
            // 对应CASE3
            if (xp != null) {
                xp.red = false;
                if (xpp != null) {
                    xpp.red = true;
                    root = rotateRight(root, xpp);
                }
            }
        }
    }
}
```










