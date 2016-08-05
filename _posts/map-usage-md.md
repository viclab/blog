title: map使用实践
date: 2016-03-10 03:39:16
tags:
---
map作为容器，使用比较方便频繁，下面主要就其赋值与插入新的数据进行总结。
# map容器添加新值
map容器中储存的是pair，一个key-value二元组。通常插入方式有两种，如下所示：
```cpp
typedef std::map<int, std::string> STRING_MAP;
```
- insert 方法
```cpp
std::pair<STRING_MAP::iterator, bool> rs = im.insert(std::make_pair(1, "insert after"));
```
由于普通的map同一个key值只允许存在一个value，所以，对于key=1,如果已经有value与之对应，则insert失败，rs.second值为false
- [] 赋值
```cpp
STRING_MAP sm;
sm[1] = "victors";     
std::cout << "before im[1]:" << im[1] << std::endl;
sm[1] = "hello";
```
同样，对于key=1的value已经存在，再次对key=1进行赋值，则【】操作符会更新key=1的值，以对于【】操作符而言，不存在失败的说法。
如果map中已经存在该key，则更新该key的value值；如果不存在，则直接插入，效果同insert成功。
> 注: 通过操作符map_[key]来取值的话，如果key不存在，则会返回一个默认的Value，并不会报错