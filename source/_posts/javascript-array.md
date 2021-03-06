---
title: Javascript 引用类型之 Array
date: 2016-02-18 10:54:08
updated:
tags: 前端
---

# Array

除了 `Object` 之外，`Array` 类型恐怕是 ECMAScript 中最常用的类型了。ECMAScript 的数组特点如下：

* 数组是有序列表；
* 数组的每一项可以保存不同类型的数据；
* 数组的大小可以动态调整，可以随着数据的添加自动增长以容纳新增数据。

## 创建方式

创建 `Array` 实例的方式有两种。第一种是使用 `Array` 构造函数：

```javascript
var colors = new Array();    // 创建一个空数组
var colors = new Array(20);    // 创建 length 值为 20 的数组
var colors = new Array("red", "blue", "green");    // 创建一个包含 3 个字符串值的数组
```

另一种方式是使用数组字面量表示法：

```javascript
var colors = [];    // 创建一个空数组
var colors = ["red", "blue", "green"];    // 创建一个包含 3 个字符串的数组
```

## 常用方法

针对数组有很多常用方法：

### 栈、队列方法

|方法|描述|
|---|---|
|`push()`|入栈（向数组的末尾添加一个或更多元素，并返回新的长度）|
|`pop()`|出栈（删除并返回数组的最后一个元素）|
|`unshift()`|向数组的开头添加一个或更多元素，并返回新的长度。|
|`shift()`|出队（删除并返回数组的第一个元素）|

### 重排序方法

|方法|描述|
|---|---|
|`sort()`|按升序排列数组项|
|`reverse()`|反转数组项的顺序|

### 操作方法

|方法|描述|
|---|---|
|`concat()`|拼接并返回新数组|
|`slice()`|裁剪并返回新数组|

### 拼接方法

| 方法               | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| `join()`           | 把数组的所有元素放入一个字符串。元素通过指定的分隔符进行分隔。 |
| `toString()`       | 把数组转换为字符串，并返回结果。                             |
| `toLocaleString()` | 把数组转换为本地字符串，并返回结果。                         |

### 位置方法

ECMAScript 5 新增的方法：

|方法|描述|
|---|---|
|`indexOf()`|查询特定项在数组的起始索引|
|`lastIndexOf()`|查询特定项在数组的结束索引|

### 迭代方法

ECMAScript 5 新增的方法：

`every()` 和 `some()` 是一组相似的方法，用于查询数组中的项是否满足某个条件：

|方法|描述|
|---|---|
|`every()`|对数组中的每一项运行给定函数，如果该函数对每一项都返回 `true`，则返回 `true`。|
|`some()`|对数组中的每一项运行给定函数，如果该函数对任一项返回 `true`，则返回 `true`。|

|方法|描述|
|---|---|
|`filter()`|对数组中的每一项运行给定函数，返回该函数会返回 `true` 的项组成的数组。|
|`map()`|对数组中的每一项运行给定函数，返回每次函数调用的结果组成的数组。|
|`forEach()`|对数组中的每一项运行给定函数。这个方法没有返回值。|

这些数组方法通过执行不同的操作，可以大大方便处理数组的任务。

### 归并方法

ECMAScript 5 新增的方法：

|方法|描述|
|---|---|
|`reduce()`|从数组的第一项开始，逐个遍历到最后，执行给定的归并函数。|
|`reduceRight()`|从数组的最后一项开始，向前遍历到第一项，执行给定的归并函数。|

# 参考

* 《JavaScript 高级程序设计》
* 《[ES6 数组](https://www.runoob.com/w3cnote/es6-array.html)》
* 《[JavaScript Array 对象](https://www.w3school.com.cn/jsref/jsref_obj_array.asp)》