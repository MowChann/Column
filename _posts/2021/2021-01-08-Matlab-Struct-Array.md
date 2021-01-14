---
layout: post
title: MATLAB 中结构数组在灵活处理多段多维数据上的应用
subtitle: 如何用MATLAB实现编程语言上的“字典”数据类型
date: 2021-01-08 22:00:00
categories: [MatLab]
author: MowChan
latex: true
incomplete: true
tags: [MatLab, 字典, 结构数组]
---

### 问题描述

假设有若干个班的学生多科目成绩表，每个班的学生个数不等，考试科目都相同，要求使用 MATLAB 按班级存储成绩表，并遍历所有数据。

这个问题涉及到多个维度的多段数据存储，对于普通的编程语言来说，实现起来非常容易。

C/C++ 可以构造一个结构体储存各个科目的成绩

```cpp
struct {
  int course1, course2;
} score;
```

Python 可以使用二维数组与字典存储，如下所示

```python
[
  [{
      "course1": 90,
      "course2": 92
  },{
      "course1": 80,
      "course2": 85
  }], # 第1个班级
  [{
      "course1": 60,
      "course2": 55
  },{
      "course1": 70,
      "course2": 75
  }] # 第2个班级
]
```

可以看出，这个问题最终会转换成一个三维信息的问题。

### 解决方案

#### 多维数组

MATLAB 可以创建多维数组，如下采用直接赋值的方式创建一个三维数组

```matlab
>> a(3,3,3)=1 %创建3*3*3的数组
a(:,:,1) =
     0     0     0
     0     0     0
     0     0     0
a(:,:,2) =
     0     0     0
     0     0     0
     0     0     0
a(:,:,3) =
     0     0     0
     0     0     0
     0     0     1
```

那么用多维数组能实现最开始提出的问题吗？答案是否定的，因为 MATLAB 是将所有数据都看作“矩阵”，而问题中各个班级的学生个数不一样，构造三维矩阵时势必会按照最多的个数作为维度，维度不足的数组用0填充，这样就无法区分填充的数据与成绩为0的数据，而且给遍历正确的个数造成困难。此外，当考试科目更多时，用下标来选取对应科目的成绩在编程时很容易出错。

#### 结构数组

结构数组是 MATLAB 中的特色数据类型，相当于 Python 的“数组[字典]”结构。也就是说结构数组本质是一个“数组”，数组的每个元素可以指定若干个名称，每个名称对应一个值，而这个值可以是不同类型、不同维度的数据。

创建结构数组的方法

1. 先声明再赋值（也可以不用写声明语句，直接赋值）

```maltab
data = struct([]);
data(1).course1 = [90 80];
data(2).course2 = [55 75];
```

2. `struct()`函数创建

```
struct(field1,value1,...,fieldN,valueN)
```

```matlab
data(1) = struct('course1', [90 80], 'course2', [92 85]);
```

结构数组的遍历

```matlab
n = length(data);
for i=1:n
    disp( data(i).course1 )
    % [90 80]
    % [60 70]
end
```

#### 单位数组（元胞数组）

单元数组与结构数组类似，可以每个元素可以存储不同类型的数据。但单元数组是用下标来定位，使用起来不方便。

```matlab
>> a = cell(2,3)
a = 
    []    []    []
    []    []    []
```



### 参考来源

[matlab学习——多维数组_我有嘉宾德音孔昭的博客-CSDN博客_matlab 多维数组](https://blog.csdn.net/qq_43264642/article/details/88779949)

[如何将不同维度的数组放到同一个矩阵中？ – MATLAB中文论坛 (ilovematlab.cn)](https://www.ilovematlab.cn/thread-575663-1-1.html)

[matlab多维数组、结构体数组 - zhouerba - 博客园 (cnblogs.com)](https://www.cnblogs.com/zhouerba/p/8046108.html)

[Matlab基础之单元数组和结构数组_ 五仁月饼哭了的博客-CSDN博客_matlab 结构数组](https://blog.csdn.net/qq_29540745/article/details/52728335)