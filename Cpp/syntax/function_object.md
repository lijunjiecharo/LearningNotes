
# 函数对象

## 函数调用运算符重载

函数调用运算符必须是成员函数，可以定义多个不同版本的调用运算符，相互之间类似函数重载，应该在参数上有所区别

如果类定义了调用运算符，则该类的对象称作函数对象（function object），因为可以调用这种对象行为像函数一样

```cpp
struct AbsInt {
    int operator()(int val) const {
        return val<0 ? -val : val;
    }
};

int main() {
    int a = -100;
    AbsInt ai;
    auto res = ai(a);
}
```

## lambda 表达式

```cpp
[capture](parameters)->return-type {body}
```
![](../../Picture/Cpp/syntax/lambda/01.png)

lambda 表达式使得可以使用匿名函数

使用 `auto` 关键字保存 lambda 表达式，之后就可以当做正常的函数使用

lambda 表达式是 inline 函数

- `[...]`：标识一个 lambda 表达式的开始，不能省略；表达式可以引用在它之外声明的变量，但只能使用定义 lambda 为止时所在作用范围内可见的变量，这些变量的集合成为闭包，被定义在 `[]` 内
1. `[]`：没有任何参数
2. `[=]`：所在范围内所有可见的局部变量按值传递
3. `[&]`：所在范围内所有可见的局部变量按引用传递
4. `[this]`：所在类中的成员变量
5. `[a]`：将 a 按值进行传递
6. `[&a]`：将 a 按引用进行传递

- `()`：标识重载 `()` 操作符参数，接受参数可以按值和按引用传递

- `->`：标识返回值类型；当只有一个返回语句或返回类型为 void 时可以自动推导返回类型，故可以省略

- `{}`：函数体

- lambda 表达式默认是 const 函数，表达式内无法修改捕获的任何变量，但可以指定为 `mutable`，修改为非 const 函数


## std::function

类模板 `std::function` 是一个通用的多态函数包装器，`std::function` 的实例可以存储、复制和调用任何可调用的目标，包括：lamdba 表达式，bind 表达式或其他函数对象，以及指向成员函数和指定数据成员的指针

`std::function` 可以取代函数指针的作用，因为它可以延迟函数的执行，特别适合作为回调函数使用

```cpp
#include <functional>
template< class R, class... Args >
class function<R(Args...)>;
/*
R: 被调用函数的返回类型
Args…：被调用函数的形参
*/

// 1. 普通函数
#include <iostream>
using namespace std;

int f(int a, int b)
{
    return a + b;
}
int main()
{
    function<int(int, int)>func = f;
    cout<<f(1, 2)<<endl;
    return 0;
}

// 2. 函数对象
struct functor
{
public:
    int operator() (int a, int b)
    {
        return a + b;
    }
};
int main()
{
   functor ft;
   function<int(int,int)> func = ft();
   cout<<func(1,2)<<endl;   
   return 0;
}

// 3. 模板函数对象
template<class T>
struct functor
{
public:
    T operator() (T a, T b)
    {
        return a + b;
    }
};
int main()
{
   functor ft;
   function<int(int,int)> func = ft<int>();
   cout<<func(1,2)<<endl;    
   return 0;
}

// 4. lambda表达式
int main()
{
	auto f = [](const int a, const int b) {return a + b; };
	std::function<int(int, int)>func = f;
	cout << func(1, 2) << endl;   
	return 0;
}

// 5. 类静态成员函数
class Plus
{
public:
    static int plus(int a, int b)
    {
        return a + b;
    }
};
int main()
{
    function<int(int, int)> f = &Plus::plus;
    cout << f(1, 2) << endl;
    return 0;
}

// 6. 类成员函数
class Plus
{
public:
    int plus(int a, int b)
    {
        return a + b;
    }
};
int main()
{
    Plus p;
    function<int(Plus&,int, int)> f = &Plus::plus;
    //function<int(const Plus,int, int)> f = &Plus::plus;
    cout << f(p,1, 2) << endl;
    return 0;
}

// 7. 类共有数据
class Plus
{
    Plus(int num_):num(num_){}
	public:
	    int plus(int a, int b)
	    {
	        return a + b;
	    }
	   int  num;
};
int main()
{
    const Plus p(2);
    function<int(const Plus&)> f = &Plus::num;
    //function<int(const Plus)> f = &Plus::num;
    cout << f(p) << endl;                       
    return 0;
}

// 8. 通过bind函数调用类成员函数
class Plus
{
public:
    int plus(int a, int b)
    {
        return a + b;
    }
};
int main()
{
   Plus p;
   // 指针形式调用成员函数
   function<int(int, int)> f = bind(&Plus::plus, &p, placeholders::_1, placeholders::_2);// placeholders::_1是占位符
   // 对象形式调用成员函数
   function<int(int, int)> f1 = bind(&Plus::plus, p, placeholders::_1, placeholders::_2);// placeholders::_1是占位符
   cout << f(1, 2) << endl;  
   cout << f1(1, 2) << endl;                              
   return 0;
}
```

## std::bind()

`std::bind()` 函数看作一个通用的函数适配器，它接受一个可调用对象，生成一个新的可调用对象来适应原对象的参数列表；将可调用对象与其参数一起进行绑定，绑定后的结果可以使用 `std::function` 保存

作用：将可调用对象和其参数绑定成一个仿函数；只绑定部分参数，减少可调用对象传入的参数


```cpp

template< class F, class... Args >
/*unspecified*/ bind( F&& f, Args&&... args );
 
template< class R, class F, class... Args >
/*unspecified*/ bind( F&& f, Args&&... args );

// 1. 绑定普通函数
double my_divide (double x, double y) {return x/y;}
auto fn_half = std::bind (my_divide,_1,2);  
std::cout << fn_half(10) << '\n';    // 5
// 第一个参数是函数名，普通函数做实参时，会隐式转换成函数指针，等价于 std::bind (&my_divide,_1,2)
// _1表示占位符，位于 <functional> 中，std::placeholders::_1，表示第一个被传入的参数


// 2. 绑定一个成员函数
struct Foo 
{
    void print_sum(int n1, int n2)
    {
        std::cout << n1+n2 << '\n';
    }
    int data = 10;
};
int main() 
{
    Foo foo;
    auto f = std::bind(&Foo::print_sum, &foo, 95, std::placeholders::_1);
    f(5); // 100
}
// 第一个参数表示对象的成员函数的指针，第二个参数表示对象的地址
// 必须显示的指定 &Foo::print_sum，因为编译器不会将对象的成员函数隐式转换成函数指针
// 必须在 Foo::print_sum 前添加&
// 使用对象成员函数的指针时，必须要知道该指针属于哪个对象，因此第二个参数为对象的地址 &foo


// 3. 绑定一个引用参数 
// 不是占位符的参数被拷贝到bind返回的可调用对象中
// 但可以使用 std::ref 来传递引用

ostream & print(ostream &os, const string& s, char c)
{
    os << s << c;
    return os;
}

int main()
{
    vector<string> words{"helo", "world", "this", "is", "C++11"};
    ostringstream os;
    char c = ' ';
    for_each(words.begin(), words.end(), 
                   [&os, c](const string & s){os << s << c;} );
    cout << os.str() << endl;

    ostringstream os1;
    // ostream不能拷贝，若希望传递给bind一个对象，
    // 而不拷贝它，就必须使用标准库提供的ref函数
    for_each(words.begin(), words.end(),
                   bind(print, ref(os1), _1, c));
    cout << os1.str() << endl;
}

```