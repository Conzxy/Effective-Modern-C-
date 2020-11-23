# Use the explicitly typed initializer idiom when auto deduces undesired types

## 代理类“骗过”了auto

```cpp
std::vector<bool> features(Widget const& w);

Widget w;
...
bool highPriority=feature(w)[5];
...
precessWidget(w,highPriority);
```

这个代码没问题

```cpp
auto highPriority=feature(w)[5];
...
precessWidget(w,highPriority);    //undefined behavior!
```

实际上，`std::vector<bool>`并不是严格的STL容器，因为它的`reference`并不符合标准，不是单纯的`T&`而是一个内置的代理类，这是因为委员会想优化分配空间，利用一个bit存储`bool`，然而C++的最小寻址为1B，因此作出妥协。

features返回临时对象`temp`，调用[]返回`reference`，其包含一个指针指向拥有`temp`管理的bits数据结构中的一个机器字，同时加上根据bit 5的偏移量，`highPriority`也拷贝了一份，但是由于临时对象的生存周期，`temp`被销毁，`highPriority`会持有悬垂指针，因此会产生未定义行为。

## 解决方法：explicitly typed initializer idiom

```cpp
auto highPriority=static_cast<bool>(features(w)[5]);
```

# Things to Remember

* "隐形"代理类型会造成`auto`推断出我不期望的“坏”类型

* 显式类型初始习惯可以强迫`auto`推断出我想要的类型
