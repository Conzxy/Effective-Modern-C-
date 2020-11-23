# Use constexpr whenever possible

## constexpr object

实际上，constexpr对象也是const，但其值是在编译器确定，存储在只读（read-only）内存中，你可以把它用于任何需要`完整常量表达式（integral const expression）`的上下文：

* 数组大小的指定

* 完整的模板实参

* 枚举值

* ....

## constexpr function

constexpr函数是用编译期常量来调用得到编译器常量的，如果是用运行期值带调用，得到的也会是运行期值，如同一般函数。

在C++11标准中，constexpr函数只允许有一条return语句，而C++14中解除了一部分限制，但是要求返回值为字面值（literal）类型，意味着它表示的值可以在编译期获得，除了void，一切基础类型都可以，甚至用户自定义类型也允许，只要可以编译器获得

还有两个限制也是在C++14解除的：

* constexpr在C++11声明成员函数是隐式const

* constexpr在C++11中不允许声明返回类型为void

也就是说C++14的标准允许修改this指向对象的数据成员同时可以声明void函数

## 书上代码

```cpp
class Point{
private:
    double x,y;
public:
    constexpr Point(double xv=0,double yv=0)noexcept
    :x(xv),y(yv){}

    constexpr void SetX(double newX)noexcept{ x=newX; }
    constexpr void SetY(double newY)noexcept{ y=newY; }

    constexpr double xvalue()const noexcept{return x;}
    constexpr double yvalue()const noexcept{return y;}

};

std::ostream& operator<<(std::ostream& os,Point const& p){
    os<<"("<<p.xvalue()<<","<<p.yvalue()<<")";
    return os;
}

constexpr Point midPoint(Point const& p1,Point const& p2)noexcept{
    return {(p1.xvalue()+p2.xvalue())/2,
            (p1.yvalue()+p2.yvalue())/2};
}

constexpr Point reflectionPoint(Point const& p)noexcept{
    Point result;

    result.SetX(-p.xvalue());
    result.SetY(-p.yvalue());

    return result;
}

int main(){
    Point p1(3,2);
    Point p2(5,29);
    std::cout<<"p1:"<<p1<<'\n';
    std::cout<<"p2:"<<p2<<'\n';
    std::cout<<"midpoint between p1 and p2:"<<midPoint(p1,p2)<<'\n';
    std::cout<<"reflection point of p1:"<<reflectionPoint(p1)<<std::endl;
}
```

## Things to Rememver

* constexpr对象是const并且初始化于编译期

* constexpr函数当用编译期值实参调用时产生编译期结果

* constexpr对象和函数相比non-constexpr对象和函数用途更宽泛

* constexpr可以作为对象和函数接口的一部分
