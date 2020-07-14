---
layout: post
title: "构造“点表示法”的 YAML 读取修改模块"
subtitle: "为实现“点表示法”对常用的模块以及读写 YAML 文件的方法进行了对比"
date: 2020-07-01 22:37:00
categories: [python, note]
tags: [python, yaml, ruamel.yaml]
---


# 构造“点表示法”的 YAML 读取修改模块

### 问题背景

为了让开发的机器人项目配置数据更加容易，在技术选型上决定采用YAML格式作为配置文件，其特点是综合INI与JSON的优点，语法不严格，可以混合JSON，层级丰富，数据类型可自行确定，可以注释。

但是Python所提供的YAML解析模块功能不是很好用，数据的修改方法是载入的class修改值后再写入，常用的`yaml`模块就会造成换行、顺序、注释等丢失，对于比较复杂的配置，需要注释的应用场景是无法适应的。可以转而采用`ruamel.yaml`模块，提供更多丰富的方法解决这些问题。其解决换行、注释的方法是使用自己构造的`ordereddict`（也叫`CommentMap`类型）数据类型（注意：区别于内置的`OrderedDict`类型）。

除此之外，访问数据的方式也是一直在讨论的焦点问题。理想的方式是以“点表示法”访问，如：

```python
cf = Config(yaml_path)
pid = cf.pid	# 读取PID
cf.pid = 9001	# 修改PID
```

这样表示方法清晰、优雅、不容易出错，方便精确到更深层级的配置信息。经研究考虑使用`addict`模块，该模块维护与使用上可靠，读取、修改方便，打印示例时直接以字典形式输出。但是问题在于`addict`是以类的`__dict__`实现点表示，多层访问是以递归构造`addict`类的方式实现，且`__dict__`只能以字典类型出现，而其它迂回方法实现的点表示又不够优雅与简洁。

```python
pid = cf.get('pid')
label_1 = cf.get('battle.option.10000.label')
```



### 矛盾点分析

**矛盾一：**以`ordereddict`为类型的`yaml`数据无法作为类或方法的属性值以实现点表示。

**矛盾二：**如果以`addict`等递归式模式构造配置数据，在引用、读取文件、写入文件还需要有包装类为其完成读写文件的操作，这样总体上增加了点表示的层级，失去简洁性。



### 解决方案尝试

#### 方案一

针对`addict`进行修改，将`addict`中涉及`dict`与创建空字典的均替换为`CommentedMap`

```python
from ruamel.yaml.comments import CommentedMap
```

经测试，会引发`Dict is not callable`的错误。

后续测试，同样的方法替换为`ordereddict`可以解决错误：

```python
from ruamel.yaml.compat import ordereddict
```

但是因为点表示的属性是通过`dict`构造出的，在赋值过程中同样会被转换，因此`orderddict`的信息还是会丢失。故无法通过直接替换`addict`的数据类型直接实现对`orderddict`的点表示访问、修改。

#### 方案二

配置数据仍采取访问的一份，交换用于写入的一份（必须是`ordereddict`），但不用同时修改，而是在需要写入前对二者遍历访问，如果有值不同的，则覆盖，最后写入数据。这样做的好处是读取与写入完全独立，并对原YAML不会造成任何破坏，缺点是遍历成本高，逻辑繁冗，代码复杂。更重要的是如果采用点表示，暂想不到合适的方法获得点访问的键名路径，当然也可以不考虑这些，仅仅是二者以`keys()`的方式获取键名，但是逻辑还是过于复杂。

#### 方案三

`addict`构造一个`self.['__cs']`字典，键名、键值分别为当前传入的`ordereddict`对象，在魔术方法`__setattr__`中同样给`__cs`赋值，这样可以正常读取、修改数据，在写入YAML时直接将`__cs`写入。但是经过测试发现数据的修改只局限于特定的访问层级构造的`__cs`字典，如：

```python
res = parse_yaml('config.yaml')
r = Dict(res)

print(r.taoba.pid)
r.taoba.pid = 1234

print(r.__cs)
# 9999

print(r.taoba.__cs)
# ordereddict([('pid', 1234), ('item_display', True), ('item_type', 'sell')])
```

所以该方案行不通。



### 结论

经过讨论，鉴于目前技术上的局限性，还是采用字符串式的点表示，逻辑简单，代码实现容易。



### 参考内容

https://github.com/cdgriffith/Box

https://github.com/mewwts/addict

https://www.cnblogs.com/alamZ/p/7054602.html

https://kb.kutu66.com/others/post_12714149