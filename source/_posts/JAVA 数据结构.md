---
title: JAVA 数据结构
tags: java
---

## JAVA 数据结构


### ArrayList：

在java中定义的数组，不需要定义数组的长度，我们可以添加或删除元素。

```java
import java.util.ArrayList;

ArrayList<T> objectName = new ArrayList<>();

// 添加元素
objectName.add("Google1");
objectName.add("Google2");
objectName.add("Google3");
// 访问元素
objectName.get(1);
// 修改元素
objectName.set(1, 'Google4');
// 删除元素
objectName.remove(1);
// 计算大小
objectName.size();
// 迭代数组
for (int i = 0; i < objectName.size(); i++) {
  System.out.println(objectName.get(i));
}

```



### LinkedList：

链表（Linked list）是一种常见的基础数据结构，是一种线性表，与 ArrayList 相比，LinkedList 的增加和删除对操作效率更高，而查找和修改的操作效率较低。

```javascript
import java.util.LinkedList;

LinkedList<T> objectName = new LinkedList<>();
// 添加元素
objectName.add("Google1");
objectName.add("Google2");
objectName.add("Google3");
// 首位添加元素
objectName.addFirst("Google4");
```



### HashSet：

HashSet 基于 HashMap 来实现的，是一个不允许有重复元素的集合。HashSet 允许有 null 值。HashSet 是无序的，即不会记录插入的顺序。

```java
import java.util.HashSet;

HashSet<T> objectName = new HashSet<>();
// 添加元素
objectName.add("Google1");
objectName.add("Google2");
objectName.add("Google3");
// 判断是否包含元素
objectName.contains("Google3");
// 清除集合
objectName.clear();
```



### HashMap：

HashMap 是一个散列表，它存储的内容是键值对(key-value)映射。

```java
import java.util.HashMap;

HashMap<Integer, String> objectName = new HashMap<Integer, String>();

objectName.put(1, 'hello')
objectName.put(2, 'world')

```







