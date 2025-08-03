---
layout: post
title: More Effective C++ 读书笔记
categories: [C++]
description: More Effective C++ 读书笔记
keywords: C++, More Effective C++, 读书笔记
---

# 第一章 基础议题

## 条款 01：仔细区别指针与引用

- 什么时候选择使用**引用**？
  - 当确定“总是会代表某个对象”而且“一旦代表了该对象就不能够再改变”
  - 如 `operator[]` 这类操作符
  - 效率会相较指针稍高，因为不需要测试其有效性（是否为 null）
- 什么时候用**指针**？
  - 需要考虑“不指向任何对象”的可能性时
  - 或是考虑“在不同时间指向不同对象”的能力时

## 条款 02：最好使用 C++ 转型操作符

C 风格的转型缺点有二：

1. 几乎允许将任何类型转换为任何其他类型。
2. 难以辨识。

因此 C++ 引入 4 个新型转型操作符：

1. static_cast
    - 基本上拥有和 C 旧式转型相同的威力和意义，以及相同的限制
2. const_cast
    - 最常见的用途是将某个对象的常量性去除
3. dynamic_cast
    - 将指向基类对象的指针或引用，转型为，指向派生类的指针或引用。并得知转型是否成功。
4. reinterpret_cast
    - 不太具有移植性
    - 常用于转换函数指针
    - 它主要用于处理指针、引用和整数之间的转换，不进行任何类型检查或数据重构，直接重新解释二进制位模式。

## 条款 03：绝对不要以多态方式处理数组

假如用多态方式，持有基类数组指针，然后操作派生类数组，那么会由于派生类占用空间过大，导致数组指针计算出现偏差。

## 条款 04：非必要不提供默认构造器

默认构造器的意思是在没有任何外来信息的情况下将对象初始化，如：

```cpp
class A {
  A() {} // 默认构造器
};
```

不提供默认构造器的原因是让对象不会创建**无意义的值**。

缺点有二：

1. 产生数组的时候会遇到问题
2. 不适用于许多基于模板的容器类
3. 虚基类如果缺少默认构造器，操作会很麻烦。

解决数组问题的方法有三：

1. 使用非堆数组，在定义数组时提供必要的自变量
2. 使用“指针数组”而非“对象数组”
3. 但是“指针数组”空间有点大怎么办？方法是先为此数组分配 raw memory，然后使用 placement new 在这块内存上构造类的对象。

# 第二章 操作符

## 条款 05：对定制的“类型转换函数”保持警觉

定制的类型转换函数包括单自变量构造器和隐式类型转换操作符。

这个隐式类型转换操作符大致是关键词 operator 后加上一个类型名称，如下：

```cpp
class Rational {
public:
  ...
  operator double const(); // 将 rational 转换为 double
};
```

但是，作者建议，**最好不要提供任何类型转换函数**。原因在于，在未预期的情况下，此类函数可能会被调用，而其结果可能是不正确、不直观的程序行为，很难调试。

- 对于隐式类型转换，解决方法是以对等的另一个函数取代类型转换操作符。
- 对于单自变量构造器，可以用 `explicit`。
  - 如果不支持 `explicit`，可以嵌套一个代理类（proxy class）来作为构造函数的形参。

## 条款 06：区别 ++/-- 操作符的前置和后置

处理用户定制类型时，应该尽可能使用前置式，因为它更轻便一点。

后置式的实现是以前置式为基础。如此核心内容请维护前置式的版本，后置式会自动调整为一致的行为。

示例代码如下：

```cpp
class UPInt {
public:
  UPInt& operator++();
  const UPInt operator++(int);
  ...
}

// 前置式 ++i
UPInt& UPInt::operator++() {
  *this += 1;
  return *this;
}

// 后置式 i++
const UPInt UPInt::operator++(int) {
  UPInt oldValue = *this;
  ++(*this);
  return oldValue;
}
```

## 条款 07：千万不要重载 `&&`, `||`, `,` 操作符

由于 C 对于真假表达式采取骤死式的评估方式（即前一个表达式出结果可能不会算下一个表达式），因此如果重载了这几个运算符可能会出大问题。

逗号则是因为其行为无法模仿，无法如逗号一样先算左再算右，最后再返回右。

## 条款 08：了解不同意义的 new 和 delete

new operator 不可重载。其过程是：先调用 operator new（这个是可以重载的）分配足够内存，用于放置某类型的对象；第二步调用构造器，为分配的内存中那个对象设定初始值。

在这个过程中，我们只能改变用来容纳对象的那块内存的分配行为（即 operator new）。

## 条款 09：利用析构函数避免泄露资源

请最好遵循 RAII 原则，多用智能指针以及在析构函数内回收资源。

## 条款 10：在构造函数内阻止资源泄露

现假设类内有若干裸指针。如果在构造函数中发生了异常，已经要出内存的裸指针如何回收资源？要知道，析构函数只会在已经构造完成的对象中调用。

于是，在**裸指针**的前提下，不得不在构造函数中写一大堆 `try...catch...` 的形式，而如果是**常量裸指针**，你会发现你不得不在所有裸指针的初始化函数内，写一大堆 `try..catch...` 的内容。

最好的方法，其实还是引入**智能指针**来管理裸指针指向的资源。

## 条款 11：禁止异常流出析构函数外

两个理由：

1. 避免 terminate 函数在异常传播的栈展开机制中被调用
2. 可以协助确保析构函数完成其应该完成的所有事情

## 条款 12：了解“抛出一个 exception”与“传递一个参数”或“调用一个虚函数”之间的差异

“传递对象到函数去，或是以对象调用虚函数”和“将对象抛出成为一个异常”之间，有 3 个主要的差异：

1. 异常对象在 `throw exception` 的时候，总是会被复制（异常向外层传递时，防止局部变量被析构）
   1. 如果以 by value 的方式捕捉，甚至要被复制两次
   2. 在异常抛出转达的过程中，请仅使用 `throw`，不要带变量，防止再次复制
   3. 复制动作永远以对象的**静态类型**为本
   4. 千万不要抛出一个指向局部对象的指针
2. “被抛出成为异常”的对象，其被允许的类型转换动作，比“被传递到函数”的对象少。catch 中的隐式转换仅有两种：
   1. “继承架构中的类转换”
   2. 从一个“有型指针”转为“无型指针”，如 `catch (const void*)` 可以捕捉任何指针类型的异常
3. catch 子句总是依出现顺序做匹配尝试。遵循最先吻合的策略，因此，绝不要将“针对基类而设计的 catch 子句”放在“针对派生类而设计的 catch 子句”之前。

## 条款 13：以 by reference 方式捕捉 exceptions

传递 exception 方式有三种：pointer、value 和 reference。

pointer 的缺点在于：经常抛出一个指向局部变量的指针，回到上层函数会导致指向不存在的对象。而且 4 个标准的 exceptions `bad_alloc`, `bad_cast`, `bad_typeid` 和 `bad_exception` 都是对象而非对象指针。

value 的缺点在于：复制问题；捕获基类导致的切割问题。

## 条款 14：明智运用 exception specification（已废弃）

exception specification 是指在函数开头指定可能抛出的异常。如：

```cpp
void a() throw(int); // 这里声明抛出一个 int
```

`throw(A, B)` 在 C++03 中引入，C++17 中废除。

现代 C++ 放弃了 “指定异常类型范围” 的设计，原因是：
1. 实用性低：实践中，函数抛出的异常类型往往难以精确枚举（如间接调用其他函数可能引入新异常）。
2. 性能开销：运行时需检查异常类型是否在声明范围内，增加额外开销。
3. 维护成本高：若函数实现修改导致抛出新类型异常，需同步更新异常规范，否则行为未定义。

这节就当没看见吧……

## 条款 15：了解异常处理（exception handling）的成本

异常处理也有自身的成本：

1. 必须付出一些空间，放置异常的数据结构；付出时间，随时保证这些数据结构的正确性
2. try 语句块的成本。使用 try 语句块，代码长度和执行速度同时膨胀/变慢 5%-10%
3. 抛出异常的成本。在作者的时代，和正常的函数返回动作比较，由于抛出异常而导致函数返回，其速度可能比正常情况下慢 3 个数量级。

因此，作者认为请对 try 语句块的使用限制于非用不可的地点，并且在真正异常的情况下才抛出异常。

# 第三章：效率

## 条款 16：谨记 80-20 法则

80-20 法则：**一个程序 80% 的资源用于 20% 的代码身上**。这提示我们，软件的整体性能几乎总是由其构成要素（代码）的一小部分决定。

有两种方法可以逼近软件的瓶颈，第一种方法是靠猜。

第二种方法则是完全根据观察或实验来识别出这 20% 的代码。例如假设程序太慢，则需要一个分析器告诉你程序的不同区段各花费多少时间，于是便可以专注在特别耗时的地方加以改善。

除运行时间外，知道语句被执行的频繁度，有时候也有作用。

而实验也需要尽可能多的数据来使你不至于估计错代码运行的瓶颈。

## 条款 17：考虑使用 lazy evaluation（缓式评估）

缓式评估类似线段树中 lazy tag 的思想，要等到用到某个变量才会去计算它。

作者描述了四种用途：

- 引用计数（Reference Counting）
- 区分读和写
- Lazy Fetching（缓式取出）
  - 但是可能会出现需要时才在 const 的成员函数获取其成员变量的情况，那么就必须把成员变量定义为 mutable 的指针。`null` 表示其未被赋值。
  - 如果不支持 mutable 的话，可以用一个指针把 this 的常量性 const_cast 滤除，然后再修改。
- Lazy Expression Evaluation（表达式缓评估）

事实上如果计算绝对必要，缓评估可能使程序更慢，增加内存用量。

## 条款 18：分期摊还预期的计算成本

这一节则是作者鼓励预处理行为：令程序超前进度地做“要求以外”的更多工作。此条款称为超急评估（over-eager evaluation）

其背后的观念是：如果预期程序常常会用到某个计算，可以降低每次计算的平均成本，办法是设计一份数据结构以便能够极有效率地处理需求。

其中最简单的一个做法就是将“已经计算好而有可能再被需要”的数值保留下来（caching）。

还有另一种做法是 Prefetching（预先取出）。此外，经验显示，如果某处的数据被需要，通常其邻近的数据也会被需要，这便是有名的 locality of reference 现象。

## 条款 19：了解临时对象的来源

C++ 真正的所谓临时对象是不可见的——不会在源代码中出现。只要你产生一个非堆对象而没有为它命名，便产生了一个临时对象。此临时匿名对象通常发生于两种情况：

1. 当隐式类型转换（implicit type conversions）被施行起来以求函数调用能成功（值传递或被传递给 reference-to-const 参数才会发生）
2. 当函数返回对象的时候

## 条款 20：协助完成“返回值优化（RVO）”

在现代 C++ 中，RVO（Return Value Optimization，返回值优化） 是编译器对函数返回对象时的一种关键优化，能避免不必要的对象复制或移动，显著提升性能。其核心是直接在调用方的内存空间构造返回对象，跳过中间临时对象的创建。

在 C++98 的时代，有一个技巧是：返回所谓的 constructor arguments 以取代对象。

```cpp
const Rational operator* (const Rational& lhs, const Rational& rhs) {
  return Rational(lhs.numerator() * rhs.numerator(), lhs.denominator() * rhs.denominator());
}
```

这是老版本的 RVO。C++ 允许编译器将临时对象优化，使他们不存在。

现代 C++ 中（主要指 C++17 后），一般有 RVO 和 NRVO（Named Return Value Optimization，命名返回值优化）两种。

- RVO：它作用于右值对象，即临时对象。
- NRVO：它作用于有名字的局部变量
  ```cpp
  vector<int> GetArray() {
    vector<int> array;   // 这里定义了一个有名字的对象，左值对象
    array.push_back(1);
    array.push_back(2);
    return array;   // 返回的对象是有名字的
  }
  ```

### 无法进行优化的场景

通常有这么几种情况，主要有：

1. 对象类型与返回值类型不一致时，编译器无法优化（比如常见的错误是对在函数内对返回值对象使用了 std::move 操作）
2. 返回对象依赖于运行时的分支依赖，比如：
   ```cpp
   vector<int> GetArray() {
     vector<int> obj1;
     vector<int> obj2;
     if (..) {
       return obj1;
     } else {
       return obj2;
     }
   }
   ```

其它场景还有：

- 返回全局变量
- 返回函数参数

## 条款 21：利用重载（overload）避免隐式类型转换（implicit type conversion）

本节条款的含义为：为了防止运算符操作中出现隐式类型转换，因此搞出一大堆重载相关运算符的内容。不过，有可能会变得更慢。

此外，请不要写成这种用两个其他类运算的代码：`const UPINT operator+ (int lhs, int rhs)`！

## 条款 22：考虑以操作符复合形式（op=）取代其独身形式（op）

首先，复合操作符比其对应的独身版本效率高。

第二，如果同时提供某个操作符的复合形式和独身形式，便允许代码编写者在效率与便利性之间做取舍。

第三，借由“以复合作为独身版本的实现基础”，可以确保二者操作语法不变。如：

```cpp
template<class T>
const T operator+ (const T& lhs, const T& rhs) {
  return T(lhs) += rhs;
}
```

## 条款 23：考虑使用其他程序库

> 不同的程序库即使提供相似的机能，也往往表现出不同的性能取舍策略，所以一旦找出程序的瓶颈。你应该思考是否有可能因为改用另一个程序库而移除了那些瓶颈。

## 条款 24：了解虚函数、多重继承、虚基类、运行时期类型辨识（virtual function, multiple inheritance, virtual base class, runtime type identification）的成本

### 虚函数

虚函数的成本：

1. 必须为每个拥有虚函数的类耗费一个 vtbl（虚表）空间，其大小视虚函数的个数（包括继承而来的）而定。
2. 必须在每一个拥有虚函数的对象内付出“一个额外指针”的代价，这个指针是指向虚表的指针 virtual table pointer(vptr)。
3. 虚函数无法 inline

### 多重继承 & 虚拟基类

多重继承往往导致 virtual base class（虚拟基类）的需求。虚拟基类可能导致另一个成本：因为其实现做法常常利用指针，指向“virtual base class”部分，以消除复制行为，而对象内可能出现一个（或多个）这样的指针。

### 运行时期类型辨识（runtime type identification, RTTI）

RTTI 让我们得以在运行时期获得对象和类的相关信息，它们被存放在类型为 type_info 的对象内。可以利用 `typeid` 操作符取得某个类对应的 type_info 对象。

RTTI 的设计理念是：根据 class 的 vtbl 来实现。比如可以在虚表中加入一个指针，指向这个 type_info 对象。

# 第四章：技术

## 条款 25：将构造函数和非成员函数虚化

### 构造函数间接虚化

虚化构造函数有一种用途是复制一个多态指针数组。英文写作 virtual copy constructor，例如：

```cpp
class NLComponent {
public:
  virtual NLComponent* clone() const = 0;
  ...
};
class TextBlock: public NLComponent {
public:
  virtual TextBlock* clone() const {
    return new TextBlock(*this);
  }
  ...
};
class Graphic: public NLComponent {
public:
  virtual Graphic* clone() const {
    return new Graphic(*this);
  }
  ...
};

class NewsLetter {
public:
  NewsLetter(const NewsLetter& rhs) {
    for (auto it = rhs.components.begin(); it != rhs.components.end(); ++it) {
      components.push_back((*it)->clone());
    }
  }
private:
  list<NLComponent*> components;
}
```

在上述代码中利用了虚函数返回类型规则的一个宽松点，当派生类重新定义基类的一个虚函数时，不需要一定声明与原本相同的返回类型。

### 将非成员函数的行为虚化

就像构造函数无法真正被虚化一样，非成员函数也无法真正被虚化。换言之，欲实现将非成员函数的行为虚化，就是要在形参为引用或指针的非成员函数中调用虚函数。

## 条款 26：限制某个类所能产生的对象数量

### 允许零个或一个对象

可以参考之前写的单例类的写法。

### 不同的对象构造状态

如果需要更多数量的对象，那么就把单例类中返回引用改为 new 一个新指针的函数，返回该指针（智能指针也可）即可。

### 允许对象生生灭灭

这里其实只需要将引用计数和之前的伪构造函数结合一下就行。把引用计数放到真正的构造和析构函数中，如果对象过多就抛出异常。

### 一个用来计算对象个数的基类

可以令计数器为“template 所产生的 classes”内的一个静态成员：

```cpp
template<class BeingCounted>
class Counted {
public:
  class TooManyObjects{}; // 此乃可能被抛出的异常类
  static int objectCount() { return numObjects;}
protected:
  Counted();
  Counted(const Counted& rhs);
  ~Counted() { --numObjects;}
private:
  static int numObjects;
  static const size_t maxObjects;
  void init();
}

template<class BeingCounted>
int Counted<BeingCounted>::numObjects = 0;

template<class BeingCounted>
Counted<BeingCounted>::Counted() { init();}

template<class BeingCounted>
Counted<BeingCounted>::Counted(const Counted<BeingCounted>&) { init();}

template<class BeingCounted>
void Counted<BeingCounted>::init() {
  if (numObjects >= maxObjects) throw TooManyObjects();
  ++numObjects;
}
```

于是被计数的类变成：

```cpp
class Printer: private Counted<Printer> {
public:
  static Printer* makePrinter();
  static Printer* makePrinter(const Printer& rhs);
  ~Printer();
  ...
  using Counted<Printer>::objectCount;
  using Counted<Printer>::TooManyObjects;
private:
  Printer();
  Printer(const Printer& rhs);
};
```

我们希望 Counted 大部分细节被隐藏起来，因此选用 private 继承。但是用户可能还是需要 `objectCount` 和 `TooManyObjects` 两个东西，因此需要在 public 中 using 一下。

但是还有个小尾巴需要处理，那就是关于 maxObjects 的定义，必须由用户在某个实现文件中来加上这一行：`const size_t Counted<Printer>::maxObjects = 10;`。

## 条款 27：要求（或禁止）对象产生于堆之中

### 要求对象产生于堆之中

限制的方法是让析构函数为 private，而构造函数为 public。

但是派生于这个基类的派生类以及内含这个类对象的类仍然存在问题。

1. 派生类问题：令基类的析构函数成为 protected 即可解决继承问题，但是如果继承出的派生类定义在栈上，基类也会在栈上而非堆上。
2. 内含问题：可以修改为“内涵一个指针，指向内含该类的对象”。

### 判断某个对象是否位于堆内

可以在 C++ 中使用 abstract mixin base class 满足这一需求。

```cpp
class HeapTracked {
public:
  class MissingAddress {}; // exception class
  virtual ~HeapTracked() = 0;
  static void *operator new(size_t size);
  static void operator delete(void *ptr);
  bool isOnHeap() const;
private:
  typedef const void* RawAddress;
  static list<RawAddress> addresses;
};

list<RawAddress> HeapTracked::addresses;
HeapTracked::~HeapTracked() {}

void* HeapTracked::operator new(size_t size) {
  void* memPtr = ::operator new(size);
  addresses.push_front(memPtr);
  return memPtr;
}

void HeapTracked::operator delete(void* ptr) {
  auto it = find(addresses.begin(), addresses.end(), ptr);
  if (it != addresses.end()) {
    addresses.erase(it);
    ::operator delete(ptr);
  } else {
    throw MissingAddress();
  }
}

bool HeapTracked::isOnHeap() const {
  const void* rawAddress = dynamic_cast<const void*>(this);
  auto it = find(addresses.begin(), addresses.end(), ptr);
  return it != addresses.end();
}
```

然后继承这个类即可。不过，这种 mixin class 有个确定就是不能用于内建类型上（如 int 和 char）。

### 禁止对象产生于堆上

简单做法就是直接把 operator new 和 operator delete 都声明为 private。不过这样只能防止直接实例化的情况，无法防止派生以及内含的情况。

## 条款 28：智能指针

当你要实现自己的智能指针时，需要注意以下问题：

1. 智能指针地构造、赋值、析构
2. 实现 Derefenrencing Operators（解引操作符，`*p` 和 `p->`）。这里面 `p->` 在 C++ 中有一个链式设计，智能指针调用 `->` 返回原指针即可，编译器会解释为原指针使用 `->` 的操作。
3. 测试智能指针是否为 NULL
4. 将智能指针转换为裸指针
5. 智能指针和“与继承有关的”类型转换
6. 智能指针与 const

总之，在原书的时代实现的智能指针与现代智能指针差异相当大，因此附上一个简单的 C++11 简化版的 `unique_ptr` 实现：

```cpp
#include <iostream>
#include <type_traits>  // 用于 SFINAE 检查
#include <utility>      // 用于 std::move、std::forward
#include <memory>

// 简化版 unique_ptr 实现（C++11 兼容）
template <typename T, typename Deleter = std::default_delete<T>>
class unique_ptr {
public:
    // 构造函数：接管原始指针所有权
    explicit unique_ptr(T* ptr = nullptr) noexcept
        : raw_ptr(ptr) {}

    // 移动构造函数：转移所有权
    unique_ptr(unique_ptr&& other) noexcept
        : raw_ptr(other.release()),
          deleter(std::move(other.deleter)) {}

    // 模板移动构造函数：支持从 unique_ptr<U> 转换（C++11 用 SFINAE 替代 requires）
    template <typename U, typename E>
    unique_ptr(unique_ptr<U, E>&& other,
               // SFINAE：仅当 U* 可转换为 T* 时才启用此构造函数
               typename std::enable_if<std::is_convertible<U*, T*>::value>::type* = nullptr) noexcept
        : raw_ptr(other.release()),
          deleter(std::forward<E>(other.get_deleter())) {}

    // 析构函数：释放资源
    ~unique_ptr() {
        if (raw_ptr) {
            deleter(raw_ptr);
        }
    }

    // 移动赋值运算符
    unique_ptr& operator=(unique_ptr&& other) noexcept {
        if (this != &other) {
            reset(other.release());
            deleter = std::move(other.deleter);
        }
        return *this;
    }

    // 禁止拷贝（C++11 语法）
    unique_ptr(const unique_ptr&) = delete;
    unique_ptr& operator=(const unique_ptr&) = delete;

    // 指针操作符重载
    T& operator*() const { return *raw_ptr; }
    T* operator->() const { return raw_ptr; }

    // 获取原始指针
    T* get() const noexcept { return raw_ptr; }

    // 释放所有权
    T* release() noexcept {
        T* temp = raw_ptr;
        raw_ptr = nullptr;
        return temp;
    }

    // 重置指针
    void reset(T* new_ptr = nullptr) noexcept {
        if (raw_ptr) {
            deleter(raw_ptr);
        }
        raw_ptr = new_ptr;
    }

    // 获取删除器
    Deleter& get_deleter() noexcept { return deleter; }
    const Deleter& get_deleter() const noexcept { return deleter; }

private:
    T* raw_ptr = nullptr;    // 管理的原始指针
    Deleter deleter;         // 删除器（默认使用 std::default_delete）
};

// 辅助函数：创建 unique_ptr（C++11 风格）
template <typename T, typename... Args>
unique_ptr<T> make_unique(Args&&... args) {
    return unique_ptr<T>(new T(std::forward<Args>(args)...));
}   
```

那个比较长的模板移动构造函数可以保证智能指针关于继承以及 const 的转换正确。

## 条款 29：引用计数

引用计数计数允许多个等值对象共享同一个实值。其有两个动机：一是为了简化堆上对象周边的记录工作；二是最好让所有等值对象共享一份实值。

下面有两个实现，一个是带引用计数的字符串，另一个是将引用计数加到既有的类上。

### 引用计数字符串

原书中实现这个引用计数字符串，使用了四个类，RCObject、RCPtr、String 和 StringValue。使用 `RCPtr<T>` 有两个要求：一是 `T` 的拷贝构造函数执行深层复制，二是其对象所指对象，类型为 `T`，而非 T 的派生类。

```cpp
#include <cstring>

template<class T>
class RCPtr {
public:
    RCPtr(T* realPtr = 0);
    RCPtr(const RCPtr& rhs);
    ~RCPtr();

    RCPtr& operator=(const RCPtr& rhs);

    T* operator->() const;
    T& operator*() const;

private:
    T *pointee;
    void init();
};

template<class T>
void RCPtr<T>::init() {
    if (pointee == 0) return;
    if (pointee->isShareable() == false) {
        pointee = new T(*pointee);
    }
    pointee->addReference();
}

template<class T>
RCPtr<T>::RCPtr(T* realPtr): pointee(realPtr) { init();}

template<class T>
RCPtr<T>::RCPtr(const RCPtr& rhs): pointee(rhs.pointee) {
    init();
}

template<class T>
RCPtr<T>::~RCPtr() {
    if (pointee) pointee->removeReference();
}

template<class T>
RCPtr<T>& RCPtr<T>::operator=(const RCPtr& rhs) {
    if (pointee != rhs.pointee) {
        if (pointee) pointee->removeReference(0);
        pointee = rhs.pointee;
        init();
    }
    return *this;
}

template<class T>
T* RCPtr<T>::operator->() const { return pointee;}

template<class T>
T& RCPtr<T>::operator*() const { return *pointee;}

class RCObject {
public:
    void addReference();
    void removeReference();
    void markUnshareable();
    bool isShareable() const;    
    bool isShared() const;
protected:
    RCObject();
    RCObject(const RCObject& rhs);
    RCObject& operator=(const RCObject& rhs);
    virtual ~RCObject() = 0;
private:
    int refCount;
    bool shareable;
};

RCObject::RCObject(): refCount(0), shareable(true) {}
RCObject::RCObject(const RCObject&): refCount(0), shareable(true) {}
RCObject& RCObject::operator=(const RCObject&) { return *this;}
RCObject::~RCObject() {}
void RCObject::addReference() { ++refCount;}
void RCObject::removeReference() {
    if (--refCount == 0) delete this;
}
void RCObject::markUnshareable() { shareable = false; }
bool RCObject::isShareable() const { return shareable;}
bool RCObject::isShared() const { return refCount > 1;}

class String {
public:
    String(const char *value="");
    const char& operator[](int index) const;
    char& operator[](int index);
private:
    struct StringValue: public RCObject {
        char *data;
        StringValue(const char *initValue);
        StringValue(const StringValue& rhs);
        void init(const char *initValue);
        ~StringValue();
    };
    RCPtr<StringValue> value;
};

void String::StringValue::init(const char* initValue) {
    data = new char[strlen(initValue) + 1];
    strcpy(data, initValue);
}

String::StringValue::StringValue(const char *initValue) {
    init(initValue);
}

String::StringValue::StringValue(const StringValue& rhs) {
    init(rhs.data);
}

String::StringValue::~StringValue() { delete [] data;}

String::String(const char *initValue):value(new StringValue(initValue)) {}

const char& String::operator[](int index) const {
    return value->data[index];
}

char& String::operator[](int index) {
    if (value->isShared()) {
        value = new StringValue(value->data);
    }
    value->markUnshareable();
    return value->data[index];
}
```

### 泛化版；适用于其他类的引用计数

这里新引入一个类 `RCIPtr`，这个类与之前的 `RCPtr` 的区别在于，RCPtr 直接指向实值，而 RCIPtr 通过中介层 CountHolder 指向实值。第二，RCIPtr 将 `->` 和 `*` 重载了，这样只要有非 const 权限操作发生，写时进行复制就会自动进行。第三，需要做引用计数的类无需再派生自 `RCObject` 了。

```cpp
template<class T>
class RCIPtr {
public:
    RCIPtr(T* realPtr = 0);
    RCIPtr(const RCIPtr& rhs);
    ~RCIPtr();

    RCIPtr& operator=(const RCIPtr& rhs);
    const T* operator->() const;
    T* operator->();
    const T& operator*() const;
    T& operator*();

private:
    struct CountHolder: public RCObject {
        ~CountHolder() { delete pointee;}
        T *pointee;
    }
    CountHolder *counter;
    void init();
    void makeCopy();
};

template<class T>
void RCIPtr<T>::init() {
    if (counter->isShareable() == false) {
        T *oldValue = counter->pointee;
        counter = new CountHolder;
        counter->pointee = new T(*oldValue);
    }
    pointee->addReference();
}

template<class T>
RCIPtr<T>::RCIPtr(T* realPtr): counter(new CountHolder) {
    counter->pointee = realPtr;
    init();
}

template<class T>
RCIPtr<T>::RCIPtr(const RCIPtr& rhs): counter(rhs.counter) {
    init();
}

template<class T>
RCIPtr<T>::~RCIPtr() {
    counter -> removeReference();
}

template<class T>
RCIPtr<T>& RCIPtr<T>::operator=(const RCIPtr& rhs) {
    if (counter != rhs.counter) {
        counter->removeReference(0);
        counter = rhs.counter;
        init();
    }
    return *this;
}

template<class T>
const T* RCIPtr<T>::operator->() const { 
    return counter->pointee;
}

template<class T>
const T& RCIPtr<T>::operator*() const {
    return *(counter->pointee);
}

template<class T>
void RCIPtr<T>::makeCopy() {
    if (counter->isShared()) {
        T *oldValue = counter->pointee;
        counter->removeReference();
        counter = new CountHolder;
        counter->pointee = new T(*oldValue);
        counter->addReference();
    }
}

template<class T>
T* RCIPtr<T>::operator->() {
    makeCopy(); return counter->pointee;
}

template<class T>
T& RCIPtr<T>::operator*() {
    makeCopy(); return *(counter->pointee);
}
```

## 条款 30：Proxy class（代理类）

代理类用途很多，比如实现二维数组，区分读写（左右值）等。

### 实现二维数组

具体操作大致是实现 `Array2D` 和 `Array1D` 两个类，其中都重载 `[]` 运算符即可。

### 区分 `operator[]` 的读写动作

```cpp
#include <vector>
#include <iostream>

template <typename T>
class MyContainer {
private:
    std::vector<T> data;

    // 代理对象：区分读写操作
    class Proxy {
    private:
        MyContainer& container;  // 引用容器
        size_t index;            // 元素索引
    public:
        Proxy(MyContainer& c, size_t i) : container(c), index(i) {}

        // 写操作：对代理对象赋值时调用
        Proxy& operator=(const T& value) {
            std::cout << "写操作：index " << index << " = " << value << std::endl;
            container.data[index] = value;  // 实际写入
            return *this;
        }

        // 读操作：将代理对象转换为 T 类型时调用
        operator T() const {
            std::cout << "读操作：index " << index << " = " << container.data[index] << std::endl;
            return container.data[index];  // 实际读取
        }
    };

public:
    MyContainer(size_t size, const T& init) : data(size, init) {}

    // operator[] 返回代理对象
    Proxy operator[](size_t index) {
        return Proxy(*this, index);  // 传递容器自身和索引
    }
};

int main() {
    MyContainer<int> obj(5, 0);  // 初始化 5 个元素为 0

    obj[2] = 3;       // 触发写操作：operator=
    int x = obj[2];   // 触发读操作：operator int()
    return 0;
}
```

### 缺陷

代理类也有其缺陷（需要加入的内容），如：

1. 取地址问题。需要重载 `operator&`。
2. `operator+=` 等符号缺失，需要重载。
3. 通过 proxy 调用真实对象的成员对象。必须把每一个函数重载。

无法完全取代真实对象的原因有二：

1. 用户把它们传递给接受非 const 引用的函数
2. 隐式类型转换。假如有连续多次的隐式类型转换会失败。

## 条款 31：让函数根据一个以上的对象类型来决定如何虚化

### 虚函数 + RTTI

这种方式并不好，大致是一堆 if-else 判断 typeid，然后调用相应的函数。它会造成程序难以维护。

### 只使用虚函数

在每一个类中定义一大堆虚函数，每个虚函数名称相同，但形参为不同的类。

```cpp
class SpaceShip: public GameObject {
public:
  virtual void collide(GameObject& otherObject);
  virtual void collide(SpaceShip& otherObject);
  virtual void collide(SpaceStation& otherObject);
  virtual void collide(Asteroid& otherObject);
  ...
};

void SpaceShip::collide(GameObject& otherObject) {
  otherObject.collide(*this);
}
```

这种方法比上面的 虚函数+RTTI 的方法安全一些，但如果你对头文件权力不够，会束缚系统的扩充性。

### 自行仿真虚函数表

与只使用虚函数方法相比，本方法使用不同名称作为虚函数，如下：

```cpp
class SpaceShip: public GameObject {
public:
  virtual void collide(GameObject& otherObject);

  virtual void hitSpaceShip(GameObject& spaceShip);
  virtual void hitSpaceStation(GameObject& spaceStation);
  virtual void hitAsteroid(GameObject& asteroid);
  ...
};

void SpaceShip::hitSpaceShip(GameObject& spaceShip) {
  SpaceShip& otherShip = dynamic_cast<SpaceShip&>(spaceShip); // 为防止出问题，必须在所有虚函数顶端使用 dynamic_cast 转换指针
  ...
}
```

然后使用一个 map 记录函数地址即可。

### 使用非成员函数的碰撞处理函数

可以在一个匿名 namespace 中（其中每样东西只对其包含文件私有）把所有需要处理的函数定义出来。然后用 map 记录函数地址即可。

### 继承 + 自行仿真的虚函数表

如果新增了新的继承类，而之前的匿名 namespace 中没有相应函数，那么就不得不使用之前的只使用虚函数的双虚函数方法。
