---
title: C++ Primer 18.用于大型程序的工具
---

## 异常处理

- 异常使得我们能够将问题的检测与解决过程分离开
- 程序的一部分负责检测问题，然后将解决该问题的任务传递给程序的另一部分。检测环节无须知道问题处理模块的所有细节，反之亦然。

### 抛出异常

- 通过抛出一条表达式来引发异常
- 沿着调用链向上进行匹配，找到最近的处理代码，这个过程被称为栈展开
- 程序的控制权从 throw 转移到与之匹配的 catch 模块：
  - 可能是同一个函数中的局部 catch
  - 也可能是调用链中的某个函数的 catch
  - 如果没有找到匹配的 catch，程序执行 terminate 函数，终止程序
- throw 后面的语句将不再被执行
- 直到找到处理代码，链条上创建的局部对象将被调用析构函数销毁，因此析构函数不应该抛出异常

异常对象：

- 编译器使用异常抛出表达式来对异常对象进行拷贝初始化
- 当抛出一条表达式时，该表达式的静态编译时类型决定了异常对象的类型
- 如果 throw 表达式解引用一个基类指针，而该指针实际指向的是派生类对象，则抛出的对象将被切掉一部分，只有基类部分被抛出。
- 抛出指针要求在任何对应的处理代码存在的地方，指针所指的对象都必须存在。

```cpp
void f()
{
	try
	{
		throw runtime_error("error");
	}
	catch (runtime_error err)
	{
		cout << err.what() << endl;
	}
}

class Quote
{
public:
	virtual void print() const { cout << "Quote" << endl; }
};

class Bulk_quote : public Quote
{
public:
	void print() const override { cout << "Bulk_quote" << endl; }
};

void f2(){
	Quote *q = new Bulk_quote;
	try
	{
		throw *q;
	}
	catch (Quote q)
	{
		cout << q.print() << endl; // Quote
	}
}
```

### 捕获异常

- catch 子句中进行了异常声明，类似于函数形参
  ```cpp
  catch (exception e)
  {
  	cout << e.what() << endl;
  }
  ```
- 如果 catch 无须访问抛出的表达式的话，我们可以忽略捕获形参的名字。
- 异常声明的类型必须是完全类型，它可以是左值引用，但不能是右值引用。
- catch 声明的类型可以是值也可以是引用，同参数传递一样，值传递不会改变原对象，引用传递会改变原对象。
- catch 声明同样支持基类：
  - 如果是值传递，会切掉派生类部分，只保留基类部分。
  - 如果是引用传递，不会切掉派生类部分。
- 通常情况下，如果 catch 接受的异常与某个继承体系有关，则最好将该 catch 的参数定义成引用类型。

查找匹配的处理代码：

- catch 语句是按照其出现的顺序逐一进行匹配的，所以越是专门的 catch 越应该放在整个 catch 列表的前端
  ```cpp
  try
  {
  	// ...
  }
  catch (runtime_error e)
  {
  	// ...
  }
  catch (exception e)
  {
  	// ...
  }
  ```
- 与实参和形参的匹配规则相比，异常和 catch 异常声明的匹配规则受到更多限制。绝大多数类型转换都不被允许，必须是精确匹配：
  - 允许从非常量到常量的类型转换
  - 允许从派生类向基类的类型转换
  - 数组和函数类型被转换成对应指针类型

重新抛出：

- 如果当前的 catch 不足以完全处理异常，可以使用 throw 再次抛出异常
  ```cpp
  catch (my_error &eObj) {
  eObj.status = errCodes::serverErr;
  throw; // 重新抛出异常
  } catch (other_error eObj) {
  eObj.status = errCodes::badErr;
  throw;
  }
  ```
  捕获所有异常：

```cpp
try
{
	// ...
}
catch (...) // 使用省略号表示捕获所有异常
{
	cout << "catch all" << endl;
}
```

### 函数 try 语句块与构造函数

- 构造函数在进入函数体之前先执行初始值列表，要想处理构造函数初始值抛出的异常，必须将构造函数写成函数 try 语句块（也称为函数测试块）的形式。

```cpp
template <typename T>
Blob<T>::Blob(std::initializer_list<T> il) try:
    data(std::make_shared<std::vector<T>>(il)) {
        /* 空函数体 */
    } catch (const std::bad_alloc &e) {
        handle_out_of_memory(e);
    }
```

- 在初始化构造函数的参数时也可能发生异常，则该异常属于调用表达式的一部分，并将在调用者所在的上下文中处理。

### noexcept 说明符

- 知道函数不会抛出异常有助于简化调用该函数的代码
- 如果编译器确认函数不会抛出异常，它就能执行某些特殊的优化操作
- C++11 新标准引入了 noexcept 说明符，用于指出函数不会抛出异常
  ```cpp
  void f() noexcept;
  ```
- noexcept：
  - 对一个函数来说，noexcept 说明要么出现在该函数的所有声明语句和定义语句中，要么一次也不出现
  - 说明应该在函数的尾置返回类型之前
  - 可以在函数指针的声明和定义中指定 noexcept
  - 在 typedef 或类型别名中不能出现 noexcept
  - 在成员函数中，noexcept 说明符需要跟在 const 及引用限定符之后，在 final、override、虚函数的=0 之前

```cpp
void f() noexcept;
void f() noexcept{
	// ...
}
auto f() noexcept -> void;
class A {
	void f() const noexcept;
	void f() & noexcept;
	void f() noexcept final;
	void f() noexcept override;
	virtual void f() noexcept = 0;
};
```

- 编译器并不会在编译时检查 noexcept 说明，一旦一个 noexcept 函数抛出了异常，程序就会调用 terminate 以确保遵守不在运行时抛出异常的承诺。
- 因此 noexcept 可以用在两种情况下：
  - 确认函数不会抛出异常
  - 根本不知道该如何处理异常

向后兼容：

```cpp
void recoup(int) noexcept;
void recoup(int) throw(); // 等价
```

异常说明的实参：

- noexcept 说明符接受一个可选的实参，该实参必须能转换为 bool 类型
  ```cpp
  void recoup(int) noexcept(true);  // 不会抛出异常
  void alloc(int) noexcept(false);  // 可能抛出异常
  ```

noexcept 运算符：

- noexcept 运算符常与异常说明实参一起使用
- noexcept 运算符是一个一元运算符
- 和 sizeof 类似，noexcept 也不会求其运算对象的值。
- 用于确定一个表达式是否会抛出异常

```cpp
void f() noexcept(noexcept(g()));  // f和g的异常说明一致，noexcept(g())表示g是否会抛出异常
```

异常说明与指针、虚函数和拷贝控制：

- 尽管 noexcept 说明符不属于函数类型的一部分，但是函数的异常说明仍然会影响函数的使用。
- 如果指针声明了不抛出异常，则该指针只能指向不抛出异常的函数；反之就能指向任何函数

  ```cpp
  void recoup(int) noexcept(true);
  void alloc(int) noexcept(false);

  void (*pf1)(int) noexcept = recoup;
  void (*pf2)(int) = recoup

  pf1 = alloc; // ERROR!
  pf2 = alloc;
  ```

- 如果一个虚函数承诺了它不会抛出异常，则后续派生出来的虚函数也必须做出同样的承诺。如果基类的虚函数允许抛出异常，则派生类对应函数既可以允许抛出异常，也可以不允许抛出异常

```cpp
class Base {
public:
	virtual double f1(double) noexcept;//不会抛出异常
	virtual int f2() noexcept (false);//可能抛出异常
	virtual void f3();//可能抛出异常
};
class Derived : public Base {
public:
	double f1(double); //错误:Base: :f1承诺不会抛出异常
	int f2()noexcept (false) ; //正确:与 Base: : f2的异常说明一致
	void f3()noexcept; //正确:Derived 的f3做了更严格的限定，这是允许的
};
```

- 当编译器合成拷贝控制成员时，同时也生成一个异常说明：
  - 如果对所有成员和基类的所有操作都承诺了不会抛出异常，则合成的成员是 noexcept 的。
  - 如果合成成员调用的任意一个函数可能抛出异常，则合成的成员是 noexcept(false)。
  - 如果定义了一个析构函数但是没有为它提供异常说明，则编译器将合成一个（与合成的析构函数异常说明一致）。

### 异常类层次

![Alt text](image.png)

- 类型 exception 仅仅定义了拷贝构造函数、拷贝赋值运算符、一个虚析构函数和一个名为 what 的虚成员。
- 其中 what 函数返回 c 风格字符串，并且确保不会抛出任何异常。
- 类 exception、bad_cast 和 bad_alloc 定义了默认构造函数。
- 类 runtime_error 和 logic_error 没有默认构造函数，但是有一个可以接受 C 风格字符串或者标准库 string 类型实参的构造函数，这些实参负责提供关于错误的更多信息。

定义自己的异常类：

```cpp
class isbn_mismatch: public std::logic_error {
public:
    explicit isbn_mismatch(const std::string &s): std::logic_error(s) {}
    isbn_mismatch(const std::string &s, const std::string &lhs, const std::string &rhs):
        std::logic_error(s), left(lhs), right(rhs) {}

    const std::string left, right;
};

Sales_data::Sales_data(std::istream &is) {
	// ...
	if (is) {
		throw isbn_mismatch("wrong data for Sales_data object", bookNo, s);
	}
}
Sales_data item1, item2,sum;
while (std::cin >> item1 >> item2) {
	try {
		sum = item1 + item2;
	} catch (const isbn_mismatch &e) {
		std::cerr << e.what() << ": left isbn(" << e.left << ") right isbn(" << e.right << ")" << std::endl;
	}
}
```

## 命名空间

- 多个库将名字放置在全局命名空间中将引发命名空间污染
- 命名空间分割了全局命名空间，其中每个命名空间是一个作用域。

### 命名空间定义

- 命名空间既可以定义在全局作用域内，也可以定义在其他命名空间中，但是不能定义在函数或类的内部
- 命名空间作用域后面无须分号
- 访问命名空间之外的代码，需要使用作用域运算符 ::
  ```cpp
  namespace cpp_primer {
  	class Query {};
  }
  cpp_primer::Query q;
  ```
- 命名空间可以是不连续的，可以在不同的文件中使用同一个命名空间
- 通常情况下，不把#include 放在命名空间内部
- 也可以在命名空间定义的外部定义该命名空间的成员。但声明必须在作用域内，定义需要明确指出其所属的命名空间：
  ```cpp
  cplusplus_primer::Sales_data cplusplus_primer::operator+(
  	const Sales_data& lhs, const Sales_data& rhs) {
  	/* ... */
  }
  ```
- 成员可以定义在命名空间外部，但只能是命名空间的外层空间中，而不能在一个不相关的作用域中定义这个运算符。

模板特例化：

- 模板特例化必须定义在原始模板所属的命名空间中
- 不过同样的，定义可以放在命名空间的外部，只要声明在命名空间内部即可

```cpp
namespace std {
    template <> struct hash<Sales_data>;
}

template <> struct std::hash<Sales_data> {
    size_t operator()(const Sales_data& s) const {
        return hash<string>()(s.bookNo) ^ hash<string>()(s.units_sold) ^
               hash<string>(s.revenue);
    }
}
```

全局作用空间：

- 全局作用域中定义的名字被隐式地添加到全局命名空间中
- 全局作用域是隐式的，所以它并没有名字，记作：`::member name`

嵌套命名空间：

```cpp
namespace cplusplus_primer {
	namespace QueryLib{
		class Query {};
		Query operator&(const Query&, const Query&);
		// ...
	}
	namespace Bookstore {
		class Quote {};
		class Bulk_quote {};
	}
}
cplusplus_primer::QueryLib::Query q;
```

内联命名空间：

- 内联命名空间中的名字可以被外层命名空间直接使用
- 可以用来区分不同版本的接口，比如当前版本放在内联命名空间，旧版本放在非内联命名空间

```cpp
// FifthEd.h
inline namespace FifthEd {
	class Query_base {};
}
namespace FifthEd { // 除了第一次inline，后续的定义都可以不加inline，隐式inline
	class Query : public Query_base {};
}

// FourthEd.h
namespace FourthEd {
	class Item_base {};
	class Query_base {};
}

// main.cpp
namespace cplusplus_primer {
	#include "FourthEd.h"
	#include "FifthEd.h"
}
cplusplus_primer::Query_base q; // 直接使用最新版本
cplusplus_primer::FourthEd::Query_base q; // 使用旧版本
```

未命名的命名空间：

- 定义的变量具有静态生命周期
- 可以在某个给定的文件内不连续，但是不能跨越多个文件。每个文件定义自己的未命名的命名空间，两个文件的未命名的命名空间相互无关。
- 如果一个头文件定义了未命名的命名空间，则该命名空间中定义的名字将在每个包含了该头文件的文件中对应不同的实体。
- 推荐使用未命名空间的变量取代 static 声明的变量（保持文件外部变量不可见）

```cpp
int i;
namespace {
    int i;
}
// 二义性：i的定义既可以是全局的，又可以是未命名命名空间的
i = 10;
```

```cpp
namespace local {
    namespace {
        int i;
    }
}

local::i = 42;
```

### 使用命名空间成员

- 有多种方法可以简写命名空间中的成员：
  - 命名空间别名
  - using 声明
  - using 指示

```cpp
// 别名
namespace cplusplus_primer {
	namespace QueryLib {
	int i = 16, j = 15, k = 23;
	}
}
namespace query = cplusplus_primer::QueryLib;
cout<<query::i<<endl;

// using 声明，一次只引入一个成员
using std::vector;

// using 指示
using namespace std;
```

- using 声明：
  - using 声明的有效范围从 using 声明的地方开始，一直到 using 声明所在的作用域结束为止。
  - 声明语句可以出现在全局作用域、局部作用域、命名空间作用域以及类的作用域中
  - 在类的作用域中，这样的声明语句只能指向基类成员。

```cpp
using std::vector;
void f() {
	using std::string;
	string s;
}
class Derived : public Base {
public: // 改变成员的可访问性
	using Base::fcn;
};
```

- using 指示：
  - using 指示使得某个特定的命名空间中所有的名字都可见
  - using 指示可以出现在全局作用域、局部作用域和命名空间作用域中，但是不能出现在类的作用域中。
  - using 指示容易导致二义性
  - using 指示一般只在命名空间本身的实现文件中使用

```cpp
namespace A {
    int i, j;
}

void f() {
    using namespace A;
    cout << i * j << endl;
}
```

```cpp
namespace blip {
    int i = 16, j = 15, k = 23;
}

int j = 0;
void manip() {
    using namespace blip;
    ++i;
    ++j;  // 二义性错误
    ++::j;
    ++blip::j;
    int k = 97;  // 局部k隐藏了blip::k
    ++k;  // 98
}
```

头文件与 using 声明或指示：

- using 指示或 using 声明如果是在头文件，则会将名字注入到所有包含了该头文件的文件中。
- 通常，头文件最多只能在它的函数或命名空间内使用 using 指示或 using 声明

### 命名空间与作用域

- 对命名空间内部名字的查找遵循常规的查找规则：由内向外依次查找每个外层作用域。
- 有一个例外：当函数参数是类类型的对象时，还会查找该类所属的命名空间。对于类的引用或指针同样有效。这使得我们可以直接访问输出运算符。
  ```cpp
  std::string s;
  std::cin >> s; // 等价于 operator>>(std::cin, s);通过查找 string 类所在的命名空间，这样可以直接使用operator>>
  // 如果没有这个规则，我们不得不写作：std::operator>>(std::cin, s);
  ```

```cpp
namespace A {
    class C {
        friend void f2();
        friend void f(const C&);
    };
}

int main() {
    A::C cobj;
    f(cobj);  // 正确，通过实参查找找到A::f
    f2();     // 错误，A::f2没有被声明
}
```

### 重载与命名空间

- 如前面所述，名字查找将在实参类（及其基类）所属的命名空间中进行。该规则影响确定候选函数集。在这些命名空间中所有与被调用函数同名的函数都将被添加到候选集当中，即使其中中某些函数在调用语句处不可见：

  ```cpp
  namespace NS {
  	class Quote { /* ... */ };
  	void display(const Quote&) { /* ... */ }
  }
  class Bulk_item: public NS::Quote { /* ... */ };

  int main() {
  	Bulk_item book1;
  	display(book1);  // 通过类类型实参对应的基类所在的命名空间进行查找
  	return 0;
  }
  ```

重载和 using 声明：

- using 声明语句声明的是一个名字，而非一个特定的函数
- 当为函数书写 using 声明时，该函数的所有版本都会被引入到当前作用域中
- 如果 usina 声明出现在局部作用域中，则引入的名字将隐藏外层作用域的相关声明
- 如果 using 声明所在的作用域中已经有一个函数与新引入的函数同名且形参列表相同，则发生错误。

```cpp
namespace ns {
	void f(int){
		cout << "ns::f(int)" << endl;
	}
}
void f(int){
	cout << "f(int)" << endl;
}
void f(const char*){
	cout << "f(const char*)" << endl;
}
int main()
{
	using ns::f;
	f(42);  // ns::f(int)
	f("42");  // 错误：f(const char*)被隐藏
    return 0;
}
```

重载和 using 指示：

- 与 using 声明不同，using 指示引入与已有函数形参列表完全相同的函数并不会产生错误。此时，只要我们指明调用的是哪个版本即可。

```cpp
namespace ns {
	void f(int){
		cout << "ns::f(int)" << endl;
	}
}
void f(int){
	cout << "f(int)" << endl;
}
using ns::f; // 错误，两个函数冲突
int main()
{

	f(42);
	return 0;
}
```

```cpp
namespace ns {
	void f(int){
		cout << "ns::f(int)" << endl;
	}
}
void f(int){
	cout << "f(int)" << endl;
}
using namespace ns; // OK
int main()
{

	::f(42);
	ns::f(42);
	return 0;
}
```

跨越多个 using 指示的重载：

- 如果存在多个 using 指示,则来自每个命名空间的名字都会成为候选函数集的一部分

```cpp
namespace AW{
	int print(int);
}
namespace Primer{
	double print(double);
}
using namespace AW;
using namespace Primer;
long double print(long double);
int main(){
	print(1); // AW::print(int)
	print(1.0); // Primer::print(double)
}
```

## 多重继承和虚继承

### 多重继承

- 多重继承是指从多个直接基类中产生派生类的能力
- 如果访问说明符被忽略掉了，对于 class 来说默认是 private，对于 struct 来说默认是 public
- 多重继承的派生列表也只能包含已经被定义过的类，而且这些类不能是 final 的。
- 多重继承的派生类从每个基类中继承状态

![Alt text](image-1.png)

派生类构造函数初始化所有基类：

- 基类的构造顺序与派生列表中基类的出现顺序保持一致（内存中的顺序），与初始化列表中的顺序无关

继承的构造函数与基类：

- C++11 允许派生类从它的基类中继承构造函数，但如果从多个基类中继承了相同的构造函数，则程序将产生错误：

```cpp
struct Base1 {
    Base1() = default;
    Base1(const std::string&);
    Base1(std::shared_ptr<int>);
};
struct Base2 {
    Base2() = default;
    Base2(const std::string&);
    Base2(int);
};

// 错误，D1试图从两个基类中都继承D1::D1(const std::string&)
struct D1: public Base1, public Base2 {
    using Base1::Base1;
    using Base2::Base2;
};

// 正确
struct D2: public Base1, public Base2 {
    using Base1::Base1;
    using Base2::Base2;
    D2(const std::string& s): Base1(s), Base2(s) {} // D2必须定义一个接受string的构造函数
    D2() = default;  // 一旦D2定义了构造函数，就必须定义默认构造函数
};
```

析构函数与多重继承：

- 析构函数的调用顺序与构造函数正好相反。合成的拷贝/移动操作的顺序与构造函数一致。

### 类型转换与多个基类

- 多基类与单基类类似，派生类的指针或引用能自动转换成一个可访问基类的指针或引用
  ```cpp
  void Print(const Bear& bear) {
  	cout << bear << endl;
  }
  Panda panda;
  Print(panda); // Panda -> Bear
  ```
- 编译器不会在派生类向基类的几种转换中进行比较和选择，在它看来转换到任意一种基类都一样好。

  ```cpp
  void print(const Bear&);
  void print(const Endangered&);

  Panda ying_yang("ying_yang");
  print(ying_yang);  // 二义性错误
  ```

### 多重继承下的类作用域

- 在单基类的情况下，派生类的作用域嵌套在直接基类和间接基类的作用域中。查找过程沿着继承体系自底向上进行，直到找到所需的名字。
- 派生类的名字将隐藏基类的同名成员。
- 在多重继承的情况下，相同的查找过程在所有直接基类中同时进行。如果名字在多个基类中都被找到，则对该名字的**使用**将具有二义性（指定基类名字解决二义性）。
- 要想避免潜在的二义性，最好的办法是在派生类中为该函数定义一个新版本。

```cpp
class Base1 {
public:
    int val = 1;
};

class Base2 {
public:
    int val = 2;
};

class Derived: public Base1, public Base2 {};

int main() {
    Derived d;
    cout << d.val << endl;  // 二义性错误
    cout << d.Base1::val << endl;  // 1
    return 0;
}
```

```cpp
double Panda::max_weight() const {
    return std::max(ZooAnimal::max_weight(), Endangered::max_weight());
}
```

### 虚继承

- 菱形继承或者，基类间有继承关系会导致派生类多次继承同一个基类，这种情况下，基类的多个实例将出现在派生类对象中。
- 虚继承用来解决这样的问题，共享的基类子对象称为虚基类，它能够保证派生类中只包含唯一一个共享的虚基类子对象。
- 虚派生只影响从指定了虚基类的派生类中进一步派生出的类,它不会影响派生类本身。
- 在实际的编程过程中，位于中间层次的基类将其继承声明为虚继承一般不会带来什么问题。通常情况下，使用虚继承的类层次是由一个人或一个项目组一次性设计完成的。

![Alt text](image-2.png)

```cpp
class Raccon : public virtual ZooAnimal {};
class Bear : virtual public ZooAnimal {}; // virtual 和 public 的顺序可以互换
class Panda : public Raccon, public Bear, public Endangered {};
```

虚继承的可见性：

- 假定 B 定义了名为 x 的成员，D1 和 D2 都是从 B 虚继承得到，D 继承了 D1 和 D2，则在 D 的作用域中，x 通过 D 的两个基类都是可见的。此时通过 D 使用 x，有三种情况：
  - D1/D2 都没定义 x，则 x 被解析为 B 的成员，不存在二义性
  - 如果 x 是 B 的成员，同时是 D1 和 D2 中某一个的成员，不存在二义性，派生类的 x 比共享虚基类 B 的 x 优先级更高
  - 如果 D1 和 D2 中都有 x 的定义，则直接访问 x 将产生二义性问题

```cpp
class B {
public:
	int x;
};
class D1 : public virtual B {};
class D2 : public virtual B {};
class D : public D1, public D2 {};
D d;
d.x = 10;
```

- 与非虚的多重继承体系一样，解决二义性问题的最好方法是在派生类中为成员自定义新的实例。

### 构造函数与虚继承

- 在虚派生中，虚基类是由最底层的派生类负责初始化的
- 先执行虚基类的构造，然后是按照声明的顺序执行非虚基类的构造，最后执行派生类的构造

```cpp
Derived::Derived():
    Obj(), Base1(), Base2() {}
```
