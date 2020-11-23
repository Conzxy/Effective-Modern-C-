# Prefer auto to explicit type declaration

`auto`有着诸多优点，记录一下

## 避免未初始化对象的产生

`auto `因为需要类型推断，因此必须初始化，提供了初始化检测的一种手段

```cpp
int x;        //如果为local object，则没有初始化，但不会报错
auto x;       //error：auto必须初始化
auto x=2;     //well-formed
```

## 避免冗长的类型声明

```cpp
typename std::iterator_traits<It>::value_type currValue=*b;
auto currValue=*b;
```

## 可以表示闭包类型

你必须要知道，闭包类型只有编译器才知道，你是不能够写出来的，但是`auto`通过类型推断可以表示只有编译器知道的类型：

```cpp
auto dereFUPLess=
    [](std::unique_ptr<Widget> const& p1,
       std::unique_ptr<Widget> const& p2){
           return *p1<*p2;
       }
```

C++14允许lambda参数也可包含`auto`，这种lambda被称作`generic lambda expression`

```cpp
auto dereFUPLess=
    [](auto const &p1,auto const&p2){
        return *p1<*p2;
    }
```

### std::function与auto的对比

```cpp
std::function<bool(std::unique_ptr<Widget> const&,
                   std::unique_ptr<Widget> const&)>
dereFUPLess=[](std::unique_ptr<Widget> const& p1,
               std::unique_ptr<Widget> const& p2){
                   return *p1<*p2;
                }                   
```

`auto`声明仅仅为闭包类型分配它所需要的，但是对于`std::function`模板的实例来说，对于任何签名式都有个固定大小，这个可能不能够存储要求的闭包类型，这时候，`std::function`构造函数将会为其分配`heap`内存存储该闭包类型。这意味着`std::function`往往比`auto`声明的对象需要更多的内存空间。

而且实现细节会限制内联，这样通过`std::function`对象的不直接调用，往往比`auto`更慢。

所以`std::function`相比auto有这两点劣势：

* 生成更大，更慢

* 可能产生`out-of-memory`异常

## 避免不恰当的隐式转换

```cpp
std::vector<int> v;
···
unsigned sz=v.size();
```

这是个`type shortcuts`问题，因为在64位的Windows下，`unsigned`是32位，而`std::vector<int>::size_type`是64位。

如果我们`auto`声明，将会得到恰当的类型

```cpp
std::unordered_map<std::string,int> m;
···
for(std::pair<std::string,int>& p:m){}
```

对于`std::unordered_map`，其键类型为const，因此m的元素类型是`std::pair<string const,int>`，因此`std::pair<string const,int>`会隐式转换为`std::pair<string,int>`，即产生一个临时对象进行绑定，且每次循环结束都会销毁该临时对象，大大损失了效率。

```cpp
for(auto const& p:m){}
```

## 可以自动变化以促进重构

假如初始化表达式改变了，auto声明的类型也会随之改变。

这样的话在重构的时候，节省了很多功夫。

e.g.:一个函数返回`int`，之后改为`long`，`auto`会在下次编译时自动更新。但是如果是显式`int`声明则需要找到所有调用点去修改它。

## auto不是完美的

* 关于类型推断这个详见`item 2`和`item 6`

* 大量的`auto`会造成可读性下降

# Things to Remember

* `auto`变量必须被初始化，通常免疫会导致效率和兼容性问题的类型不匹配，简化重构过程，还比显式指定类型简短

* `auto`的坑请认真看`item 2`和`item 6`


