# 15 Class第二部分

## 15.1 隐藏的“this”指针和成员函数调用

新程序员经常问的关于类的问题之一是，“当调用成员函数时，C++如何知道被调用的对象？”。

首先，定义一个要使用的简单类。此类封装整数值，并提供一些访问函数来获取和设置该值：

```C++
#include <iostream>

class Simple
{
private:
    int m_id{};
 
public:
    Simple(int id)
        : m_id{ id }
    {
    }

    int getID() const { return m_id; }
    void setID(int id) { m_id = id; }

    void print() const { std::cout << m_id; }
};

int main()
{
    Simple simple{1};
    simple.setID(2);

    simple.print();

    return 0;
}
```

当调用 simple.setID(2); 时，C++知道函数 setID() 应该在对象simple上操作，并且m_id实际上是指simple.m_id。

C++使用了一个名为this的隐藏指针！



### 15.1.1 隐藏的this指针

在每个成员函数内部，关键字this是保存当前隐式对象地址的const指针。

大多数时候，没有明确提到这一点，下面是一个说明示例：

```C++
#include <iostream>

class Simple
{
private:
    int m_id{};

public:
    Simple(int id)
        : m_id{ id }
    {
    }

    int getID() const { return m_id; }
    void setID(int id) { m_id = id; }

    void print() const { std::cout << this->m_id; } // 使用 `this` 访问隐式对象 使用 操作符-> 访问成员 m_id
};

int main()
{
    Simple simple{ 1 };
    simple.setID(2);
    
    simple.print();

    return 0;
}
```

请注意上述两个示例中的print()成员函数执行的操作完全相同：

```C++
    void print() const { std::cout << m_id; }       // 隐式使用 this
    void print() const { std::cout << this->m_id; } // 显示使用 this
```

原来，前者是后者的简写。当编译程序时，编译器将使用this->隐式地为引用隐式对象的任何成员添加前缀。前一种写法有助于保持代码更简洁，并防止冗余的必须反复显式地编写此->。



### 15.1.2 this 如何生效？

让我们仔细看看这个函数调用：

```c++
    simple.setID(2);
```

尽管对函数 setID(2) 的调用看起来只有一个参数，但实际上有两个！编译时，编译器重写表达式 simple.setID(2); 如下所示：

```c++
    Simple::setID(&simple, 2); // simple由前缀变换为函数参数！
```

注意，这现在只是一个标准函数调用，对象simple（以前是对象前缀）现在的地址作为函数的参数传递。

但这只是答案的一半。由于函数调用现在有一个添加的参数，因此还需要修改成员函数定义，以接受（并使用）该参数。下面是setID()的原始成员函数定义：

```c++
    void setID(int id) { m_id = id; }
```

编译器如何重写函数是特定于实现的细节，但最终结果如下：

```c++
    static void setID(Simple* const this, int id) { this->m_id = id; }
```

注意，setID函数有一个新的最左边的参数this，它是一个const指针（这意味着它不能被重新指向，但指向的内容可以修改）。m_id成员也被重写为this->m_id，使用this指针。

组合在一起：

1. 当调用 simple.setID(2) ，编译器实际会调用 Simple::setID(&simple, 2)，simple 的地址会传递给函数。
2. 函数有一个隐藏的 this 参数来接收 simple。
3. setID() 函数中的成员变量，被改写为带 this-> 前缀的形式，这指向传入的 simple。所以当执行到 this->m_id 时，实际计算的是 simple.m_id。

好消息是，所有这些都是自动发生的，是否记得它是如何工作的并不重要。需要记住的是，所有非静态成员函数都有一个this指针，该指针指向调用该函数的对象。

在这个上下文中，static关键字意味着函数不与类的对象相关联，被视为类的作用域内的普通函数。



### 15.1.3 this总是指向正在操作的对象

新程序员有时会对存在多少个指针感到困惑。每个成员函数都有一个指向隐式对象的this指针参数。考虑：

```C++
int main()
{
    Simple a{1}; // this = &a 在 Simple 的构造函数里
    Simple b{2}; // this = &b 在 Simple 的构造函数里
    a.setID(3); // this = &a 在成员函数 setID() 里
    b.setID(4); // this = &b 在成员函数 setID() 里

    return 0;
}
```

注意，this指针交替保存对象a或b的地址，这取决于调用的是对象a还是b的成员函数。

因为this是一个函数参数（而不是成员），所以它不会使类的实例在内存占用更大。



### 15.1.4 显式使用this

大多数情况下，不需要显式引用this指针。然而，在一些情况下，这样做是有用的：

首先，如果成员函数中具有与数据成员同名的参数，则可以通过使用以下命令来消除它们的歧义：

```c++
struct Something
{
    int data{}; // 这个结构体里没有使用 m_ 前缀

    void setData(int data)
    {
        this->data = data; // this->data 是成员变量, data 是参数
    }
};
```

这个Something类有一个名为data的成员。setData() 的函数参数也被命名为data。在 setData() 函数中，data是函数参数（因为函数参数遮挡了成员变量data），因此如果想要使用data成员，采用 this->data 的形式。

一些开发人员喜欢显式地将 this-> 添加到所有类成员中，以明确他们正在引用成员变量。建议您避免这样做，因为这样做往往会降低代码的可读性，而不会带来什么好处。使用“m_”前缀是区分私有成员变量和非成员（局部）变量的更简洁的方法。



### 15.1.5 返回 *this

其次，有时让成员函数将隐式对象作为返回值是有用的。这样做的主要原因是允许成员函数“链接”在一起，因此可以在单个表达式中对同一对象调用多个成员函数！这称为函数链接（或方法链接）。

考虑这个常见的示例，其中您使用std::cout 输出几次文本

```c++
std::cout << "Hello, " << userName;
```

编译器对上述代码段的求值如下：

```c++
(std::cout << "Hello, ") << userName;
```

首先，操作符«使用std::cout和字符串文字“Hello”，将“Hello“打印到控制台。然而，由于这是表达式的一部分，运算符«还需要返回值（或void）。如果运算符«返回void，则最终将此作为部分求值的表达式：

```c++
void{} << userName;
```

这显然没有任何意义（编译器会抛出错误）。相反，操作符«返回传入的流对象，在本例中为std::cout。这样，在计算完第一个运算符«后，我们得到：

```c++
(std::cout) << userName;
```

然后打印用户名。

这样，只需要指定一次std::cout，然后可以使用操作符«将任意多段文本链接在一起。每次调用operator«都返回std::cout，因此下一次调用operator«时使用std::cout作为左操作数。

也可以在成员函数中实现这种行为。考虑以下类：

```C++
class Calc
{
private:
    int m_value{};

public:

    void add(int value) { m_value += value; }
    void sub(int value) { m_value -= value; }
    void mult(int value) { m_value *= value; }

    int getValue() const { return m_value; }
};
```

如果你想加5，减3，再乘以4，你必须这样做：

```C++
#include <iostream>

int main()
{
    Calc calc{};
    calc.add(5); // 返回 void
    calc.sub(3); // 返回 void
    calc.mult(4); // 返回 void

    std::cout << calc.getValue() << '\n';

    return 0;
}
```

然而，如果通过引用使每个函数返回*this，可以将调用链接在一起。下面是具有“可链接”函数的Calc的新版本：

```C++
class Calc
{
private:
    int m_value{};

public:
    Calc& add(int value) { m_value += value; return *this; }
    Calc& sub(int value) { m_value -= value; return *this; }
    Calc& mult(int value) { m_value *= value; return *this; }

    int getValue() const { return m_value; }
};
```

注意，add()、sub() 和mult() 现在通过引用返回*this。因此，这允许我们执行以下操作：

```C++
#include <iostream>

int main()
{
    Calc calc{};
    calc.add(5).sub(3).mult(4); // 函数链接

    std::cout << calc.getValue() << '\n';

    return 0;
}
```

有效地将三行压缩为一个表达式！让我们仔细看看这是如何工作的。

首先，调用calc.add(5)，将5与m_value相加。然后，add() 返回对*this的引用，这是对隐式对象calc的引用，因此calc将是后续求值中使用的对象。接下来，calc.sub(3) 求值，它从m_value中减去3，然后再次返回calc。最后，calc.mult(4) 将m_value乘以4，并返回calc，它不再使用，因此被忽略。

由于每个函数在执行时都修改了calc，因此calc的m_value现在包含值（（（0+5）-3）*4），即8。

这可能是this最常见的显式用法，并且无论何时，具有可链接的成员函数都应该考虑使用 *this。

因为this总是指向隐式对象，所以在解引用它时，不需要检查它是否为空指针。



### 15.1.6 将类重置回默认状态

如果您的类具有默认构造函数，您可能会对提供一种将现有对象重置到其默认状态的方法感兴趣。

在前面讲解委托构造函数时，可以知道构造函数仅用于初始化新对象，不应直接调用。这样做将导致意外的行为。

将类重置回默认状态的最佳方法是创建 reset() 成员函数，让该函数创建新对象（使用默认构造函数），然后将该新对象分配给当前隐式对象，如下所示：

```C++
    void reset()
    {
        *this = {}; // 值初始化一个新对象，并赋值给当前的隐式对象
    }
```

下面是一个完整的程序，演示了此reset() 函数的实际使用：

```C++
#include <iostream>

class Calc
{
private:
    int m_value{};

public:
    Calc& add(int value) { m_value += value; return *this; }
    Calc& sub(int value) { m_value -= value; return *this; }
    Calc& mult(int value) { m_value *= value; return *this; }

    int getValue() const { return m_value; }

    void reset() { *this = {}; }
};


int main()
{
    Calc calc{};
    calc.add(5).sub(3).mult(4);

    std::cout << calc.getValue() << '\n'; // 打印 8

    calc.reset();
    
    std::cout << calc.getValue() << '\n'; // 打印 0

    return 0;
}
```


### 15.1.7 this和const对象

对于非const成员函数，this是指向非常量值的const指针（这意味着它不能指向其他对象，但指向的对象可以修改）。对于const成员函数，这是指向常量值的const指针（意味着指针不能指向其他对象，也不能修改被指向的对象）。

尝试调用常量对象上的非常量成员时生成的错误可能有点神秘：

```C++
error C2662: 'int Something::getValue(void)': cannot convert 'this' pointer from 'const Something' to 'Something &'
error: passing 'const Something' as 'this' argument discards qualifiers [-fpermissive]
```

当对常量对象调用非常量成员函数时，隐式this函数参数是非常量对象的const指针。但传入的值是具有指向常量对象的const指针类型。将常量对象的指针转换为非常量对象的指针需要丢弃常量限定符，不能隐式完成。某些编译器生成的编译器错误反映了编译器无法执行这种转换。



### 15.1.8 为什么 this 是指针而不是引用

由于this指针始终指向隐式对象（并且永远不能是空指针，除非我们做了一些事情导致未定义的行为），因此您可能想知道为什么这是指针而不是引用。答案很简单：当它被添加到C++时，引用还不存在。

如果今天将 this 添加到C++语言中，它无疑将是引用而不是指针。在其他更现代的类C++语言（如Java和C#）中，this 被实现为引用。



## 15.2 类和头文件

到目前为止，编写的所有类都非常简单，以至于能够直接在类定义本身内实现成员函数。例如，下面是一个简单的Date类，其中所有成员函数都在Date类定义中进行实现：

```C++
#include <iostream>

class Date
{
private:
    int m_year{};
    int m_month{};
    int m_day{};
 
public:
    Date(int year, int month, int day)
        : m_year { year }
        , m_month { month }
        , m_day { day}
    {
    }

    void print() const { std::cout << "Date(" << m_year << ", " << m_month << ", " << m_day << ")\n"; };

    int getYear() const { return m_year; }
    int getMonth() const { return m_month; }
    int getDay() const { return m_day; }
};

int main()
{
    Date d { 2015, 10, 14 };
    d.print();

    return 0;
}
```

然而，随着类变得越来越长、越来越复杂，在类中包含所有成员函数定义可能会使类更难管理和使用。使用已经编写的类只需要理解其public接口（公共成员函数），而不需要理解类在引擎盖下的工作方式。成员函数放在类定义中实现，将公共接口与实际使用类无关的细节弄得乱七八糟。

为了帮助解决这个问题，C++允许在类定义之外定义成员函数来将类的“声明”部分与“实现”部分分开。

这里是与上面相同的Date类，构造函数和 print() 成员函数定义在类定义之外。请注意，这些成员函数的原型仍然存在于类定义中（因为这些函数需要声明为类类型定义的一部分），但实际实现已移到外部：

```C++
#include <iostream>

class Date
{
private:
    int m_year{};
    int m_month{};
    int m_day{};

public:
    Date(int year, int month, int day); // 构造函数声明

    void print() const; // print 函数声明

    int getYear() const { return m_year; }
    int getMonth() const { return m_month; }
    int getDay() const  { return m_day; }
};

Date::Date(int year, int month, int day) // 构造函数定义
    : m_year{ year }
    , m_month{ month }
    , m_day{ day }
{
}

void Date::print() const // print 函数定义
{
    std::cout << "Date(" << m_year << ", " << m_month << ", " << m_day << ")\n";
};

int main()
{
    const Date d{ 2015, 10, 14 };
    d.print();

    return 0;
}
```

成员函数可以像非成员函数一样在类定义之外定义。唯一的区别是，必须用类类型的名称（在本例中为Date::）作为成员函数名的前缀，以便编译器知道我们定义的是该类类型的成员，而不是非成员。

请注意，这里将定义的访问函数留在类定义中。由于访问函数通常只有一行，因此在类定义内定义这些函数混淆很小，而将它们移到类定义外将导致许多额外的代码行。由于这个原因，访问函数（和其他琐碎的单行函数）的定义通常留在类定义中。



### 15.2.1 将类定义放在头文件中

如果在源（.cpp）文件中定义类，则该类仅在该特定源文件中可用。在较大的程序中，通常希望使用在多个源文件中编写的类。

我们知道可以将函数声明放在头文件中。然后可以将这些函数声明#include到多个代码文件（甚至多个项目）中。类也不例外。类定义可以放在头文件中，然后#include在要使用类类型的任何其他文件中。

与只需要使用前向声明的函数不同，编译器通常需要查看类（或任何程序定义的类型）的完整定义，才能使用类型。这是因为编译器需要理解如何声明成员，以确保正确使用它们，并且它需要能够计算该类型的对象的大小，以便实例化它们。因此，头文件通常包含类的完整定义，而不仅仅是类的前向声明。



### 15.2.2 命名类的头文件和代码文件

通常，类在与类同名的头文件中定义，在类之外定义的任何成员函数都放在与类相同名称的.cpp文件中。

这里是我们的Date类，分为.cpp和.h文件：

date.h：

```C++
#ifndef DATE_H
#define DATE_H

class Date
{
private:
    int m_year{};
    int m_month{};
    int m_day{};
 
public:
    Date(int year, int month, int day);

    void print() const;

    int getYear() const { return m_year; }
    int getMonth() const { return m_month; }
    int getDay() const { return m_day; }
};

#endif
```

date.cpp：

```C++
#include "Date.h"

Date::Date(int year, int month, int day) // 构造函数定义
    : m_year{ year }
    , m_month{ month }
    , m_day{ day }
{
}

void Date::print() const // print 函数定义
{
    std::cout << "Date(" << m_year << ", " << m_month << ", " << m_day << ")\n";
};
```

现在，任何其他想要使用Date类的头文件或代码文件都可以简单地#include “Date.h” 。请注意，date.cpp还需要编译到任何使用date.h的项目中，以便链接器可以将对成员函数的调用连接到其定义。

优先将类定义放在与类同名的头文件中。琐碎的成员函数（例如访问函数、具有空函数体的构造函数等）可以在类定义中定义。

首选在与类同名的源文件中定义非平凡的成员函数。



### 15.2.3 如果头文件被#included多次，那么在头文件中定义类是否违反了单定义规则？

类不受单定义规则（ODR）的限制。因此，不存在将类定义包含到多个翻译单元中的问题。

将类定义多次包含到单个翻译单元中仍然是ODR冲突。需要使用头文件保护（或#pragma once）防止这种情况发生。



### 15.2.4 内联成员函数

成员函数不能免除ODR，因此您可能想知道，当成员函数在头文件中定义时（然后将其包含在多个翻译单元中），如何避免ODR冲突。

在类定义内定义的成员函数是隐式内联的。内联函数不受单定义规则的约束。

在类定义之外定义的成员函数不是隐式内联的（因此受制于单定义规则）。这就是为什么这些函数通常在代码文件中定义（在整个程序中它们只有一个定义）。

或者，如果在类定义之外定义的成员函数是内联的（使用inline关键字），则可以将它们保留在头文件中。这是样例的date.h头文件，其中在类外部定义的成员函数标记为内联：

data.h：

```C++
#ifndef DATE_H
#define DATE_H

#include <iostream>

class Date
{
private:
    int m_year{};
    int m_month{};
    int m_day{};
 
public:
    Date(int year, int month, int day);

    void print() const;

    int getYear() const { return m_year; }
    int getMonth() const { return m_month; }
    int getDay() const { return m_day; }
};

inline Date::Date(int year, int month, int day) // 标记为 inline
    : m_year{ year }
    , m_month{ month }
    , m_day{ day }
{
}

inline void Date::print() const // 标记为 inline
{
    std::cout << "Date(" << m_year << ", " << m_month << ", " << m_day << ")\n";
};

#endif
```

此date.h可以包含在多个翻译单元中，而不会出现问题。



### 15.2.5 成员函数的内联扩展

编译器必须能够看到函数的完整定义，才能执行内联扩展。通常，这样的函数（例如访问函数）在类定义中定义。然而，如果您希望在类定义之外定义成员函数，但仍然希望它符合内联扩展的条件，则可以将其定义为类定义下面的内联函数（在相同的头文件中）。这样，任何包含这个头文件的人都可以访问函数的定义。



### 15.2.6 那么，为什么不将所有内容都放在头文件中呢？

您可能会试图将所有成员函数定义放入头文件中，要么在类定义内，要么作为类定义下的内联函数。虽然这能通过编译，但这样做有几个缺点。

首先，如上所述，在类定义中定义成员会使类定义混乱。

其次，如果您更改了头文件中的任何代码，则需要重新编译包含该头文件的每个文件。这可能会产生连锁反应，其中一个微小的更改会导致整个程序需要重新编译。重新编译的成本可能相差很大：一个小项目可能只需要一分钟或更少的时间来构建，而一个大型商业项目可能需要几个小时。

相反，如果更改.cpp文件中的代码，则只需要重新编译该.cpp文件。因此，如果可以选择，通常最好将非平凡的代码放在.cpp文件中。

在一些情况下，将所有内容放在单个文件中可能是有意义的。

首先，对于仅在一个代码文件中使用且不打算进行一般重用的小类，您可能更喜欢直接在使用它的单个.cpp文件中定义类（和所有成员函数）。这有助于明确类仅在该单个文件中使用，而不是用于更广泛的用途。如果以后发现要在多个文件中使用该类，或者发现类和成员函数定义使源文件混乱，则始终可以将该类移动到单独的头/代码文件中。

其次，如果类只有少量不太可能更改的成员函数，则创建仅包含一个或两个定义的.cpp文件可能不值得（因为它会使项目混乱）。在这种情况下，最好使成员函数内联，并将它们放在头文件中的类定义下。

第三，在现代C++中，类或库越来越多地作为“头文件”分发，这意味着类或库的所有代码都放在头文件中。这样做主要是为了使分发和使用这样的文件更容易，因为头文件只需要被#include，而代码文件需要显式添加到使用它的每个项目中才能编译它。如果有意为分发创建仅含头文件的类或库，则所有非平凡的成员函数都可以内联并放在类定义的头文件中。

最后，对于模板类，在类外部定义的模板成员函数几乎总是在头文件中的类定义下定义。就像非成员模板函数一样，编译器需要查看完整的模板定义才能实例化它。我们将在后续介绍这样的情况。



### 15.2.7 成员函数的默认参数

在学习默认参数时，我们讨论了非成员函数的默认参数的最佳实践：“如果函数具有前向声明（特别是头文件中的声明），请将默认参数放在那里。否则，将默认参数放到函数定义中。”

由于成员函数总是作为类定义的一部分声明（或定义），因此成员函数的最佳实践实际上更简单：始终将默认参数放在类定义中。



### 15.2.8 库

在编写程序时，通常都使用了标准库的一部分类，如std::string。要使用这些类，只需#include相关的头文件（如#incluse <string>）。请注意，您不需要将任何代码文件（如string.cpp或iostream.cpp）添加到项目中。

头文件提供编译器所需的声明，以验证您正在编写的程序的语法正确性。然而，属于C++标准库的类的实现包含在预编译文件中，该文件在链接阶段自动链接。你永远看不到代码。

许多开源软件包同时提供.h和.cpp文件，供您编译到程序中。然而，大多数商业库仅提供.h文件和预编译的库文件。这有几个原因：1）链接预编译库比每次需要时重新编译它更快，2）预编译库的单个副本可以由许多应用程序共享，而编译的代码被编译到使用它的每个可执行文件中，以及3）知识产权原因（您不希望人们窃取您的代码）。

虽然您可能暂时不会创建和分发自己的库，但将类分离为头文件和源文件不仅是一种良好的形式，它还使创建自己的自定义库变得更容易。创建自己的库超出了本教程的范围，但如果您希望分发预编译的二进制文件，则分离声明和实现是这样做的前提。



## 15.3 嵌套类型（成员类型）

考虑以下简短的程序：

```C++
#include <iostream>

enum class FruitType
{
	apple,
	banana,
	cherry
};

class Fruit
{
private:
	FruitType m_type { };
	int m_percentageEaten { 0 };

public:
	Fruit(FruitType type) :
		m_type { type }
	{
	}

	FruitType getType() { return m_type; }
	int getPercentageEaten() { return m_percentageEaten; }

	bool isCherry() { return m_type == FruitType::cherry; }

};

int main()
{
	Fruit apple { FruitType::apple };
	
	if (apple.getType() == FruitType::apple)
		std::cout << "I am an apple";
	else
		std::cout << "I am not an apple";
	
	return 0;
}
```

这个程序没有问题。但由于枚举类FruitType旨在与Fruit类一起使用，但它独立于类存在，这让我们不得不记住他们的关联。




### 15.3.1 嵌套类型（成员类型）

到目前为止，我们已经看到了类类型的两种不同成员：数据成员和成员函数。在上面的例子中，Fruit类同时具有这两个特性。

类类型支持另一种类型的成员：嵌套类型（也称为成员类型）。要创建嵌套类型，只需在类内的适当访问说明符下定义类型。

下面是与上面相同的程序，重写为使用在Fruit类中定义的嵌套类型：

```C++
#include <iostream>

class Fruit
{
public:
	// FruitType 移动到了 Fruit 内部, 在 public 下面
    // 同时重命名为Type，并定义为 enum 而不是 enum class
	enum Type
	{
		apple,
		banana,
		cherry
	};

private:
	Type m_type {};
	int m_percentageEaten { 0 };

public:
	Fruit(Type type) :
		m_type { type }
	{
	}

	Type getType() { return m_type;  }
	int getPercentageEaten() { return m_percentageEaten;  }

	bool isCherry() { return m_type == cherry; } // 在 Fruit 内, 不需要使用 FruitType:: 前缀
};

int main()
{
	// 注: 在 class 外部, 使用 Fruit:: 前缀访问
	Fruit apple { Fruit::apple };
	
	if (apple.getType() == Fruit::apple)
		std::cout << "I am an apple";
	else
		std::cout << "I am not an apple";
	
	return 0;
}
```

这里有几点值得指出。

首先，请注意，FruitType现在在类中定义，由于稍后将讨论的原因，它已被重命名为Type。

其次，在类的顶部定义了嵌套类型Type。嵌套类型名称在使用之前必须完全定义，因此通常首先定义它们。

第三，嵌套类型遵循正常的访问规则。类型是在public访问说明符下定义的，因此类型名称和枚举元素可以由外部直接访问。

第四，类类型充当中声明的名称的作用域，就像命名空间一样。因此，Type的完全限定名为Fruit::Type，而apple枚举元素的完全限定名称为Fruit::apple。

在类的成员中，不需要使用完全限定名。例如，在成员函数 isCherry() 中，在没有Fruit:: 作用域限定符的情况下访问cherry枚举元素。

在类之外，必须使用完全限定的名称（例如，Fruit::apple ）。我们将FruitType重命名为Type，以便可以使用 Fruit::Type（而不是更冗余的 Fruit::FruitType ）。

最后，将枚举类型从 enum class 更改为 enum。由于类本身现在充当作用域，因此使用 enum class 有些多余。更改为未限定作用域的枚举意味着可以用Fruit::apple的形式访问枚举元素，而不是必须使用的较长的 Fruit::Type::apple 。



### 15.3.2 嵌套的typedef和类型别名

类类型还可以包含嵌套的typedef或类型别名：

```C++
#include <iostream>
#include <string>
#include <string_view>

class Employee
{
public:
    using IDType = int;

private:
    std::string m_name{};
    IDType m_id{};
    double m_wage{};

public:
    Employee(std::string_view name, IDType id, double wage)
        : m_name { name }
        , m_id { id }
        , m_wage { wage }
    {
    }

    const std::string& getName() { return m_name; }
    IDType getId() { return m_id; } // 类内部可以使用不带限定名的类型
};

int main()
{
    Employee john { "John", 1, 45000 };
    Employee::IDType id { john.getId() }; // 类外部必须使用完全限定名

    std::cout << john.getName() << " has id: " << id << '\n';

    return 0;
}
```

请注意，在类内部，可以直接使用IDType，但在类外部，必须使用完全限定名Employee::IDType。

我们之前讨论过Typedef和类型别名，它们在这里的作用是相同的。C++标准库中的类通常使用嵌套的typedef。截至编写时，std::string定义了十个嵌套的typedef！



### 15.3.3 嵌套类和对外部类成员的访问

类将其他类作为嵌套类型是相当少见的，但这是可行的。在C++中，嵌套类不能访问外部类的this指针，嵌套类也不能直接访问外部类的成员。这是因为嵌套类可以独立于外部类进行实例化（在这种情况下，将没有可访问的外部类实例和成员！）

然而，由于嵌套类是外部类的成员，因此它们可以访问外部类的任何私有成员。

用一个例子来说明：

```C++
#include <iostream>
#include <string>
#include <string_view>

class Employee
{
public:
    using IDType = int;

    class Printer
    {
    public:
        void print(const Employee& e) const
        {
            // Printer 不能使用 Employee 的 `this` 指针
            // 所以不能直接使用 m_name 和 m_id
            // 代替的方案是, 可以传入一个 Employee 对象
            // 因为 Printer 是 Employee 的成员,
            // 所以可以直接访问私有成员 e.m_name 和 e.m_id
            std::cout << e.m_name << " has id: " << e.m_id << '\n';
        }
    };

private:
    std::string m_name{};
    IDType m_id{};
    double m_wage{};

public:
    Employee(std::string_view name, IDType id, double wage)
        : m_name{ name }
        , m_id{ id }
        , m_wage{ wage }
    {
    }

    // 这个例子中不提供访问函数
};

int main()
{
    const Employee john{ "John", 1, 45000 };
    const Employee::Printer p{}; // 实例化嵌套类的实例
    p.print(john);

    return 0;
}
```

有一种情况下，嵌套类更常用。在标准库中，大多数迭代器类都被实现为容器的嵌套类，它们被设计为在容器上迭代。



### 15.3.4 嵌套类型不能前向声明

嵌套类型还有一个值得一提的限制——嵌套类型不能被前向声明。在C++的未来版本中，可能取消此限制。



## 15.4 析构函数

### 15.4.1 清理问题

假设您正在编写一个程序，该程序需要通过网络发送一些数据。然而，建立与服务器的连接是昂贵的，因此您希望收集一组数据，然后一次性发送所有数据。这样的类可以如下结构：

```C++
class NetworkData
{
private:
    std::string m_serverName{};
    DataStore m_data{};

public:
	NetworkData(std::string_view serverName)
		: m_serverName { serverName }
	{
	}

	void addData(std::string_view data)
	{
		m_data.add(data);
	}

	void sendData()
	{
		// 连接服务器
		// 发送数据
		// 清理数据
	}
};

int main()
{
    NetworkData n("someipAddress");

    n.addData("somedata1");
    n.addData("somedata2");

    n.sendData();

    return 0;
}
```

然而，此NetworkData存在潜在问题。它依赖于在程序关闭之前显式调用 sendData()。如果NetworkData的用户忘记执行此操作，则数据将不会发送到服务器，并在程序退出时丢失。现在，你可能会说，“嗯，记住这样做并不难！”在这种情况下，你是对的。但考虑一个稍微复杂的示例，如此函数：

```C++
bool someFunction()
{
    NetworkData n("someipAddress");

    n.addData("somedata1");
    n.addData("somedata2");

    if (someCondition)
        return false;

    n.sendData();
    return true;
}
```

在这种情况下，如果someCondition为true，则函数将提前返回，并且不会调用 sendData() 。这是一个更容易犯的错误，因为 sendData() 调用存在，但程序并不是在所有情况下都调用它。

概括下这个问题，使用资源的类（通常是内存，但有时是文件、数据库、网络连接等…），它们的类对象在销毁之前必须显式地发送或关闭。或者在其他情况下，可能希望在销毁对象之前保留一些记录，例如将信息写入日志文件。术语“清理”通常用于指类在销毁类的对象之前必须执行的任何任务，以便符合预期行为。如果必须依赖类的用户来确保在销毁对象之前调用执行清理的函数，那么代码是非常容易出错的。

但为什么要求用户确保这一点？如果对象正在被销毁，则我们知道需要在该点执行清理。清理应该自动进行吗？



### 15.4.2 析构函数

在前面我们介绍了构造函数，它们是在创建非聚合类类型的对象时调用的特殊成员函数。构造函数用于初始化成员变量，并执行所需的任何其他设置任务，以确保类的对象可以使用。

类似地，类具有另一种类型的特殊成员函数，该函数在销毁非聚合类类型的对象时自动调用。该函数称为析构函数。析构函数被设计为允许类在销毁类的对象之前进行任何必要的清理。



### 15.4.3 析构函数命名

与构造函数一样，析构函数具有特定的命名规则：

1. 构造函数的名字需要与类名一致，同时需要带一个前缀波浪号（ ~ ）
2. 析构函数不能有参数
3. 析构函数不能有返回类型

类只能有一个析构函数。通常，您不应该显式调用析构函数（因为当对象被销毁时它将自动调用），因为很少有情况下您希望多次清理对象。

析构函数可以安全地调用其他成员函数，因为对象直到析构函数执行后才被销毁。



### 15.4.4 析构函数示例

```C++
#include <iostream>

class Simple
{
private:
    int m_id {};

public:
    Simple(int id)
        : m_id { id }
    {
        std::cout << "Constructing Simple " << m_id << '\n';
    }

    ~Simple() // 这里是析构函数
    {
        std::cout << "Destructing Simple " << m_id << '\n';
    }

    int getID() const { return m_id; }
};

int main()
{
    // 分配一个 Simple 对象
    Simple simple1{ 1 };
    {
        Simple simple2{ 2 };
    } // simple2 在这里销毁

    return 0;
} // simple1 在这里销毁
```

请注意，当销毁每个Simple对象时，都调用析构函数，该析构函数打印一条消息。“Destructing Simple 1”打印在“Destructing Simple 2”之后，因为simple2在代码块结束时被销毁，而simple1直到main()结束才被销毁。

记住，静态变量（包括全局变量和静态局部变量）在程序启动时构造，在程序关闭时销毁。



### 15.4.5 改进NetworkData程序

通过让析构函数调用 sendData() 函数，可以消除用户显式调用 sendData() 的需要：

```C++
class NetworkData
{
private:
    std::string m_serverName{};
    DataStore m_data{};

public:
	NetworkData(std::string_view serverName)
		: m_serverName { serverName }
	{
	}

	~NetworkData()
	{
		sendData(); // 确保对象被销毁时，所有的数据自动发送
	}

	void addData(std::string_view data)
	{
		m_data.add(data);
	}

	void sendData()
	{
		// 连接服务器
		// 发送数据
		// 清理数据
	}
};

int main()
{
    NetworkData n("someipAddress");

    n.addData("somedata1");
    n.addData("somedata2");

    return 0;
}
```

有了这样的析构函数，NetworkData对象将始终在销毁对象之前发送它所拥有的任何数据！清理会自动进行，这意味着出现错误的可能性更小，需要考虑的事情也更少。



### 15.4.6 隐式析构函数

如果非聚合类类型对象没有用户声明的析构函数，则编译器将生成具有空逻辑的析构函数。这个析构函数被称为隐式析构函数，它实际上只是一个占位符。

如果类不需要在销毁时进行任何清理，那么完全不定义析构函数是可以的，让编译器为类生成隐式析构函数。



### 15.4.7 关于std::exit() 函数的警告

在前面，我们讨论了std::exit() 函数，可以用于立即终止程序。当程序立即终止时，程序就结束了。局部变量不会被销毁，因此不会调用析构函数。在这种情况下，如果您依赖于析构函数来执行必要的清理工作，请谨慎。

未处理的异常也将导致程序终止，并且在执行此操作之前可能不会展开堆栈。如果堆栈展开没有发生，则在程序终止之前不会调用析构函数。



## 15.5 具有成员函数的类模板

函数模版：

```C++
template <typename T> // 模版参数声明
T max(T x, T y) // 函数模版 max<T> 定义
{
    return (x < y) ? y : x;
}
```

使用函数模板，可以定义类型模板参数（例如，类型名T），然后将它们用作函数参数 (T x, T y) 的类型。

同时我们也介绍了类模板，它允许为类类型（结构体、类和联合）的数据成员的类型使用类型模板参数：

```C++
#include <iostream>

template <typename T>
struct Pair
{
    T first{};
    T second{};
};

// 这里是我们自定义的 Pair 的推导 (需要在 C++17 以上版本使用)
// Pair 以两个参数 T 和 T初始化，会被推导为 Pair<T>
template <typename T>
Pair(T, T) -> Pair<T>;

int main()
{
    Pair<int> p1{ 5, 6 };        // 实例化 Pair<int> 并创建对象 p1
    std::cout << p1.first << ' ' << p1.second << '\n';

    Pair<double> p2{ 1.2, 3.4 }; // 实例化 Pair<double> 并创建对象 p2
    std::cout << p2.first << ' ' << p2.second << '\n';

    Pair<double> p3{ 7.8, 9.0 }; // 创建对象 p3，使用之前实例化的 Pair<double>
    std::cout << p3.first << ' ' << p3.second << '\n';

    return 0;
}
```



### 15.5.1 在成员函数中键入模板参数

模版参数，既可以作为成员变量的类型，也可以作为成员函数参数的类型。

在下面的示例中，重写了上面的Pair类模板，将其从结构体转换为类：

```C++
#include <ios>       // for std::boolalpha
#include <iostream>

template <typename T>
class Pair
{
private:
    T m_first{};
    T m_second{};

public:
    // 在类中定义成员函数
    // 模版参数是类声明时的模版参数
    Pair(const T& first, const T& second)
        : m_first{ first }
        , m_second{ second }
    {
    }

    bool isEqual(const Pair<T>& pair);
};

// 在类外部定义成员函数
// 需要重新提供模版参数声明
template <typename T>
bool Pair<T>::isEqual(const Pair<T>& pair)
{
    return m_first == pair.m_first && m_second == pair.m_second;
}

int main()
{
    Pair p1{ 5, 6 }; // 使用 CTAD 来推导 Pair<int>
    std::cout << std::boolalpha << "isEqual(5, 6): " << p1.isEqual( Pair{5, 6} ) << '\n';
    std::cout << std::boolalpha << "isEqual(5, 7): " << p1.isEqual( Pair{5, 7} ) << '\n';

    return 0;
}
```

上面的内容应该非常简单，但有几点值得注意。

首先，因为类具有私有成员，所以它不是聚合，因此不能使用聚合初始化。相反，必须使用构造函数初始化类对象。

由于类数据成员的类型为T，因此将构造函数类型的参数设置为const T&，因此用户可以提供相同类型的初始化值。由于T的复制成本可能很高，因此通过常量引用传递比通过值传递更安全。

注意，当在类模板定义中定义成员函数时，不需要为成员函数提供模板参数声明。这样的成员函数隐式使用类模板参数声明。

其次，可以使用CTAD，提供初始值匹配构造函数，让编译器来自动推断模板参数所需的信息。

让我们更仔细地看一下在类模板定义之外为类模板定义成员函数的情况：

```C++
template <typename T>
bool Pair<T>::isEqual(const Pair<T>& pair)
{
    return m_first == pair.m_first && m_second == pair.m_second;
}
```

由于该成员函数定义与类模板定义是分开的，因此需要重新提供模板参数声明（template <typename T>），以便编译器知道T是什么。

此外，当在类之外定义成员函数时，需要用类模板的完全模板化名称（Pair<T>::isEqual，而不是Pair::isEqual）来限定成员函数名称。



### 15.5.2 如何在类模板外部定义成员函数

对于类模板的成员函数，编译器需要查看类定义（以确保将成员函数模板声明为类的一部分）和模板成员函数定义（了解如何实例化模板）。因此，通常希望在同一位置定义类及其成员函数模板。

当在类内部中定义成员函数模板时，模板成员函数定义是类定义的一部分，因此，只要可以看到类定义，就可以看到模板成员函数的定义。这使得事情变得容易（以类定义比较乱为代价）。

当成员函数模板在类之外定义时，通常应在类定义的正下方定义它。这样，在任何可以看到类定义的地方，也将看到类定义下面的成员函数模板定义。

在类在头文件中定义的典型情况下，这意味着在类之外定义的任何成员函数模板也应该在类定义下面的相同头文件中进行定义。

在前面，我们知道从模板隐式实例化的函数是隐式内联的。这包括非成员和成员函数模板。因此，将头文件中定义的成员函数模板包含到多个代码文件中没有问题，因为从这些模板实例化的函数将隐式内联（并且链接器将消除它们的重复）。



## 15.6 静态成员变量

在之前，我们介绍了全局变量和静态局部变量。这两种类型的变量都具有静态存储期，这意味着它们在程序开始时创建，在程序结束时销毁。这样的变量保持其值，即使它们超出变量作用域。例如：

```C++
#include <iostream>

int generateID()
{
    static int s_id{ 0 }; // 静态局部变量
    return ++s_id;
}

int main()
{
    std::cout << generateID() << '\n';
    std::cout << generateID() << '\n';
    std::cout << generateID() << '\n';

    return 0;
}
```

请注意，静态局部变量s_id在多个函数调用中保持了其值。

类类型为static关键字带来了另外两种用法：静态成员变量和静态成员函数。



### 15.6.1 静态成员变量

在研究应用于成员变量的static关键字之前，首先考虑以下类：

```C++
#include <iostream>

struct Something
{
    int value{ 1 };
};

int main()
{
    Something first{};
    Something second{};
    
    first.value = 2;

    std::cout << first.value << '\n';
    std::cout << second.value << '\n';

    return 0;
}
```

当实例化一个类对象时，每个对象的存储都是完全互相隔离的。在这种情况下，因为声明了两个Something类对象，所以最后得到了value的两个副本：first.value和second.value。first.value与second.value不同。

通过使用static关键字，可以使类的成员变量成为静态的。与普通成员变量不同，静态成员变量由类的所有对象共享。考虑以下程序，类似于上面的程序：

```C++
#include <iostream>

struct Something
{
    static int s_value; // 现在是 static
};

int Something::s_value{ 1 }; // 初始化 s_value 为 1

int main()
{
    Something first{};
    Something second{};

    first.s_value = 2;

    std::cout << first.s_value << '\n';
    std::cout << second.s_value << '\n';
    return 0;
}
```

因为s_value是一个静态成员变量，所以s_value在类的所有对象之间共享。因此，first.s_value与second.s_value是相同的变量。上面的程序显示，使用first设置的值可以使用second访问！



### 15.6.2 静态成员不与类对象关联

尽管可以通过类的对象访问静态成员（如上面的示例中的first.s_value和second.s_value所示），但即使类的对象没有被实例化，静态成员也存在！这是有意义的：它们在程序开始时创建，在程序结束时销毁，因此它们的生命周期不像普通成员那样绑定到类对象。

本质上，静态成员是存在于类的作用域内的全局变量。类的静态成员和命名空间内的普通变量之间没有什么区别。

由于静态成员s_value独立于任何类对象存在，因此可以使用类名和域解析操作符（在本例中为Something::s_value）直接访问它：

```C++
class Something
{
public:
    static int s_value; // 声明静态成员变量
};

int Something::s_value{ 1 }; // 定义静态成员变量 (下面进行讨论)

int main()
{
    // 注: 这里不再实例化任何 Something 的对象

    Something::s_value = 2;
    std::cout << Something::s_value << '\n';
    return 0;
}
```

在上面的片段中，s_value由类名Something来进行使用，而不是通过对象使用。请注意，甚至还没有实例化Something类型的对象，但仍然能够访问和使用Something::s_value。这是访问静态成员的首选方法。

静态成员是位于类的作用域内的全局变量。



### 15.6.3 定义和初始化静态成员变量

当在类类型中声明静态成员变量时，告诉编译器静态成员变量的存在，但不是实际定义它（很像前向声明）。由于静态成员变量本质上是全局变量，因此必须在全局作用域内显式定义（并可选地初始化）类外部的静态成员。

在上面的示例中，我们通过这一行来执行此操作：

```c++
int Something::s_value{ 1 }; // 定义静态成员变量
```

这一行有两个用途：它实例化静态成员变量（就像全局变量一样），并对其进行初始化。在本例中，提供的是初始化值1。如果未提供初始值设定项，则默认情况下静态成员变量为零初始化。

请注意，此静态成员定义不受访问控制的约束：您可以定义和初始化该值，即使它在类中声明为 private（或 protected）。

如果类在头（.h）文件中定义，则静态成员定义通常放在类的关联代码文件中（例如Something.cpp）。如果类是在源（.cpp）文件中定义的，则静态成员定义通常直接放在类的下面。不要将静态成员定义放在头文件中（就像全局变量一样，如果该头文件被多次包含，则最终将得到多个定义，这将导致编译错误）。



### 15.6.4 类定义内静态成员变量的初始化

有几个快捷方式。首先，当静态成员是常量整型（包括char和bool）或常量枚举时，可以在类定义内初始化静态成员：

```C++
class Whatever
{
public:
    static const int s_value{ 4 }; // 静态 const int 可以在类内部直接定义和初始化
};
```

在上面的示例中，由于静态成员变量是常量int。因此允许使用此快捷方式，因为这些特定的常量类型是编译时常量。

在前面讲解跨多个文件共享全局常量（使用内联变量）中，引入了内联变量，这是允许具有多个定义的变量。C++17允许静态成员成为内联变量：

```C++
class Whatever
{
public:
    static inline int s_value{ 4 }; // 静态内联变量可以直接定义和初始化
};
```

无论这些变量是否为常量，都可以在类定义内初始化它们。这是定义和初始化静态成员的首选方法。

由于constexpr成员是隐式内联的（从C++17开始），因此也可以在类定义内初始化静态constexpr成员，而无需显式使用inline关键字：

```C++
#include <string_view>

class Whatever
{
public:
    static constexpr double s_value{ 2.2 }; // ok
    static constexpr std::string_view s_view{ "Hello" }; // 甚至支持类类型的 静态 constexpr 定义和初始化
};
```

将静态成员设为内联或constexpr，以便可以在类定义内初始化它们。



### 15.6.5 静态成员变量的示例

为什么在类中使用静态变量？一种用法是为类的每个实例分配唯一的ID。下面是一个示例：

```C++
#include <iostream>

class Something
{
private:
    static inline int s_idGenerator { 1 };
    int m_id {};

public:
    // 获取下一个id值
    Something() : m_id { s_idGenerator++ } 
    {    
    }

    int getID() const { return m_id; }
};

int main()
{
    Something first{};
    Something second{};
    Something third{};

    std::cout << first.getID() << '\n';
    std::cout << second.getID() << '\n';
    std::cout << third.getID() << '\n';
    return 0;
}
```

由于s_idGenerator由所有Something对象共享，因此在创建新的Somethine对象时，构造函数使用s_idGenerator的当前值初始化m_id，然后s_idGenerator加一。这确保每个实例化的Something对象接收唯一的id（按创建顺序递增）。

在调试时为每个对象提供唯一的ID会有所帮助，因为它可以用于区分在其他情况下具有相同数据的对象。当使用数据数组时，这一点尤其明显。

当类需要使用查找表（例如，用于存储一组预计算值的数组）时，静态成员变量也很有用。通过将查找表设置为静态，所有对象只存在一个副本，而不是为每个实例化的对象创建一个副本。这可以节省大量内存。



### 15.6.6 只有静态成员可以使用类型演绎（auto和CTAD）

静态成员可以使用auto从初始值设定项推断其类型，或者使用类模板参数推断（CTAD）从初始值设置项推断模板类型参数。

非静态成员不能使用auto或CTAD。

做出这种区分的原因相当复杂，但归根结底，非静态成员可能会发生某些情况，从而导致歧义或非直观的结果。静态成员不会发生这种情况。因此，非静态成员不能使用这些功能。

```C++
#include <utility> // for std::pair<T, U>

class Foo
{
private:
    auto m_x { 5 };           // 非静态成员不能使用 auto
    std::pair m_v { 1, 2.3 }; // 非静态成员不能使用 CTAD

    static inline auto s_x { 5 };           // 静态成员可以使用 auto
    static inline std::pair s_v { 1, 2.3 }; // 静态成员可以使用 CTAD

public:
    Foo() {};
};

int main()
{
    Foo foo{};
    
    return 0;
}
```



## 15.7 静态成员函数

在上一课中，我们了解到静态成员变量是属于类的成员变量，而不属于类的对象。如果静态成员变量是public的，则可以使用类名和域解析操作符直接访问它：

```C++
#include <iostream>

class Something
{
public:
    static inline int s_value { 1 };
};

int main()
{
    std::cout << Something::s_value; // s_value 是 public, 可以直接访问
}
```

但如果静态成员变量是私有的呢？考虑以下示例：

```C++
#include <iostream>

class Something
{
private: // 现在是 private
    static inline int s_value { 1 };
};

int main()
{
    std::cout << Something::s_value; // error: s_value 是 private，无法在类的外部直接访问
}
```

在这种情况下，不能直接从 main() 访问Something::s_value，因为它是私有的。通常，通过公共成员函数访问私有成员。虽然可以创建一个普通的公共成员函数来访问s_value，但这需要实例化类类型的对象才能使用该函数！

```C++
#include <iostream>

class Something
{
private:
    static inline int s_value { 1 };

public:
    int getValue() { return s_value; }
};

int main()
{
    Something s{};
    std::cout << s.getValue(); // 可以, 但必须先实例化一个对象才能调用 getValue()
}
```



### 15.7.1 静态成员函数

成员变量不是唯一可以设置为静态的成员类型。成员函数也可以设置为静态。下面是具有静态成员函数访问器的示例：

```C++
#include <iostream>

class Something
{
private:
    static inline int s_value { 1 };

public:
    static int getValue() { return s_value; } // static 成员函数
};

int main()
{
    std::cout << Something::getValue() << '\n';
}
```

由于静态成员函数不与特定对象关联，因此可以使用类名和域解析操作符（ 例如Something::getValue() ）直接调用它们。与静态成员变量一样，它们也可以通过类类型的对象调用，但不建议这样做。



### 15.7.2 静态成员函数没有this指针

静态成员函数有两个值得注意的奇怪之处。首先，因为静态成员函数没有附加到对象，所以它们没有this指针！这是有意义的——this指针始终指向成员函数正在处理的对象。静态成员函数不在对象上工作，因此不需要this指针。

其次，静态成员函数可以直接访问其他静态成员（变量或函数），但不能访问非静态成员。这是因为非静态成员必须属于类对象，并且静态成员函数没有可使用的类对象！



### 15.7.3 在类定义外部定义的静态成员

静态成员函数也可以在类声明之外定义。这与普通成员函数的工作方式相同。

```C++
#include <iostream>

class IDGenerator
{
private:
    static inline int s_nextID { 1 };

public:
     static int getNextID(); // 这里是静态成员函数声明
};

// 这里是类外部的静态成员函数定义，注意这里没有加 static 关键字
int IDGenerator::getNextID() { return s_nextID++; } 

int main()
{
    for (int count{ 0 }; count < 5; ++count)
        std::cout << "The next ID is: " << IDGenerator::getNextID() << '\n';

    return 0;
}
```

注意，因为这个类中的所有数据和函数都是静态的，所以不需要实例化类的对象来使用它的函数！该类使用静态成员变量来保存要分配的下一个ID的值，并提供一个静态成员函数来返回该ID并对其进行递增。

在类定义中定义的成员函数隐式内联。在类定义外部定义的成员函数不是隐式内联的，但可以使用inline关键字内联。因此，在头文件中定义的静态成员函数应该内联，以便在随后将该头文件包含在多个翻译单元中时不违反“单定义规则”（ODR）。



### 15.7.4 关于纯静态类的警告

编写全是静态成员的类时要小心。尽管这种“纯静态类”可能有用，但它们也有一些潜在的缺点。

首先，因为所有静态成员都只实例化一次，所以没有办法拥有纯静态类的多个副本（无法拷贝类对象并重命名它）。例如，如果您需要两个独立的IDGenerator，这对于纯静态类是不可能的。

其次，在关于全局变量的课程中，了解到全局变量是危险的，因为任何代码片段都可以更改全局变量的值，并最终破坏另一段看似无关的代码。对于纯静态类也是如此。由于所有成员都属于类（而不是类的对象），并且类声明通常具有全局范围，因此纯静态类本质上相当于在全局可访问的命名空间中声明函数和全局变量，具有全局变量所具有的所有缺点。

与其编写全是静态成员的类，不如考虑编写一个普通类并实例化它的全局实例（全局变量具有静态存储期）。这样，可以在适当的时候使用全局实例，但仍然可以进行实例化其它实例。



### 15.7.5 纯静态类与命名空间

纯静态类与命名空间有许多重叠。两者都允许您定义具有静态存储期变量，并在其作用域内定义函数。然而，一个显著的区别是类具有访问控制，而命名空间没有。

通常，当您有静态数据成员和/或需要访问控制时，最好使用静态类。否则，首选命名空间。



### 15.7.6 C++不支持静态构造函数

如果可以通过构造函数初始化普通成员变量，那么您应该能够通过静态构造函数初始化静态成员变量。虽然一些现代语言确实支持静态构造函数来实现这一目的，但不幸的是C++不是其中之一。

如果可以直接初始化静态变量，则不需要构造函数：您可以在定义点初始化静态成员变量（即使它是私有的）。在上面的IDGenerator示例中这样做。下面是另一个示例：

```c++
#include <iostream>

struct Chars
{
    char first{};
    char second{};
    char third{};
    char fourth{};
    char fifth{};
};

struct MyClass
{
	static inline Chars s_mychars { 'a', 'e', 'i', 'o', 'u' }; // 在定义点初始化静态成员变量
};

int main()
{
    std::cout << MyClass::s_mychars.third; // 打印 i

    return 0;
}
```

如果初始化静态成员变量需要执行代码（例如循环），那么有许多不同的、有些迟钝的方法可以做到这一点。处理所有变量（静态或非静态）的一种方法是使用函数创建对象，用数据填充它，并将其返回给调用方。该返回值可以拷贝到正在初始化的对象中。

```C++
#include <iostream>

struct Chars
{
    char first{};
    char second{};
    char third{};
    char fourth{};
    char fifth{};
};

class MyClass
{
private:
    static Chars generate()
    {
        Chars c{}; // 创建一个对象
        c.first = 'a'; // 填充值
        c.second = 'e';
        c.third = 'i';
        c.fourth = 'o';
        c.fifth = 'u';
        
        return c; // 返回对象
    }

public:
	static inline Chars s_mychars { generate() }; // 将返回的对象填充到 s_mychars
};

int main()
{
    std::cout << MyClass::s_mychars.third; // 打印 i

    return 0;
}
```



## 15.8 友元非成员函数

在之前，我们一直在宣扬访问控制的优点，它提供了一种机制来控制谁可以访问类的各个成员。私有成员只能由类的其他成员访问，而公共成员可以由每个人访问。我们讨论了保持数据私有化的好处，以及创建供非成员使用的公共接口。

然而，在某些情况下，这种安排要么是不充分的，要么是不理想的。

例如，考虑一个专注于管理某些数据集的存储类。现在假设您也想显示该数据，但处理显示的代码将有许多选项，因此很复杂。您可以将存储管理功能和显示管理功能放在同一个类中，但这会使事情变得混乱，并形成复杂的界面。您还可以将它们分开：存储类管理存储，而其他一些显示类管理所有的显示功能。这创造了一个很好的责任分离。但显示类随后将无法访问存储类的私有成员，并且可能无法执行其工作。

或者，在语法上，我们可能更喜欢使用非成员函数而不是成员函数（我们将在下面展示一个例子）。重载运算符时通常是这种情况，这是我们将在以后的课程中讨论的主题。但非成员函数也有相同的问题——它们不能访问类的私有成员。

如果访问函数（或其他公共成员函数）已经存在，并且对于我们试图实现的任何功能都足够，那么很好——可以（并且应该）只使用它们。但在某些情况下，这些函数并不存在。然后呢？

一种选择是向类中添加新的成员函数，以允许其他类或非成员函数执行它们在其他情况下无法执行的任何工作。但我们可能不希望允许公共访问这些东西——也许这些东西高度依赖于实现，或者容易被滥用。

真正需要的是在个案基础上颠覆访问控制系统的某种方法。



### 15.8.1 friend

这些挑战的答案是friend。

在类的主体中，可以使用友元声明（使用friend关键字）来告诉编译器，其他一些类或函数现在是友元。在C++中，友元是一个类或函数（成员或非成员），它被授予对另一个类的私有和受保护成员的完全访问权限。通过这种方式，类可以有选择地授予其他类或函数对其成员的完全访问权限，而不会影响其他任何内容。

例如，如果我们的存储类使display类成为朋友，那么display类将能够直接访问存储类的所有成员。display类可以使用这种直接访问来实现存储类的显示，同时在结构上保持独立。

友元声明不受访问控制的影响，因此它在类主体中的位置并不重要。

既然我们知道了什么是友元，那么让我们看一看将友谊授予非成员函数、成员函数和其他类的特定示例。在本课中，将讨论友元非成员函数，然后在下一课学习友元类和友元成员函数。

友谊总是由其成员将被访问的类授予（而不是由希望访问的类或函数授予）。在访问控制和授予友谊之间，类始终保留控制谁可以访问其成员的能力。



### 15.8.2 友元非成员函数

友元函数是一个函数（成员或非成员），可以访问类的私有成员和受保护成员，就像它是该类的成员一样。在所有其他方面，友元函数是正常的函数。

让我们看一个简单类的示例，该类使非成员函数成为友元：

```C++
#include <iostream>

class Accumulator
{
private:
    int m_value { 0 };

public:
    void add(int value) { m_value += value; }

    // 这里是友元声明，授予非成员函数  void print(const Accumulator& accumulator) 对 Accumulator 的访问能力
    friend void print(const Accumulator& accumulator);
};

void print(const Accumulator& accumulator)
{
    // 因为 print() 是 Accumulator 的友元
    // 因此可以访问 Accumulator 的私有变量
    std::cout << accumulator.m_value;
}

int main()
{
    Accumulator acc{};
    acc.add(5); // 将 5 加到 accumulator

    print(acc); // 调用 print() 非成员函数

    return 0;
}
```

在这个例子中，声明了一个名为 print() 的非成员函数，该函数接受Accumulator类的对象。因为 print() 不是Accumulator类的成员，所以它通常不能访问私有成员m_value。然而，Accumulator类有一个友元声明，使「void print(const Accumulator& accumulator)」成为友元。

请注意，因为 print() 是非成员函数（因此没有隐式对象），所以必须显式地将Accumulator对象传递给 print() 。



### 15.8.3 在类内定义友元非成员

类似于成员函数可以在类内定义（如果需要），友元非成员函数也可以在类中定义。下面的示例在Accumulator类中定义友元非成员函数 print() ：

```C++
#include <iostream>

class Accumulator
{
private:
    int m_value { 0 };

public:
    void add(int value) { m_value += value; }

    // 类中定义的友元非成员函数
    friend void print(const Accumulator& accumulator)
    {
    // 因为 print() 是 Accumulator 的友元
    // 因此可以访问 Accumulator 的私有变量
        std::cout << accumulator.m_value;
    }
};

int main()
{
    Accumulator acc{};
    acc.add(5); // 将 5 加到 accumulator

    print(acc); // 调用 print() 非成员函数

    return 0;
}
```

尽管您可能会假设，由于 print() 是在Accumulator中定义的，这使得print() 成为Accumulator的成员，但事实并非如此。因为 print() 被定义为友元，所以它被视为非成员函数（就像它是在Accumulator外部定义的一样）。



### 15.8.4 语法上优先使用友元非成员函数

有时我们可能更喜欢使用非成员函数而不是成员函数。现在展示一个例子。

```C++
#include <iostream>

class Value
{
private:
    int m_value{};

public:
    explicit Value(int v): m_value { v }  { }

    bool isEqualToMember(const Value& v) const;
    friend bool isEqualToNonmember(const Value& v1, const Value& v2);
};

bool Value::isEqualToMember(const Value& v) const
{
    return m_value == v.m_value;
}

bool isEqualToNonmember(const Value& v1, const Value& v2)
{
    return v1.m_value == v2.m_value;
}

int main()
{
    Value v1 { 5 };
    Value v2 { 6 };

    std::cout << v1.isEqualToMember(v2) << '\n';
    std::cout << isEqualToNonmember(v1, v2) << '\n';

    return 0;
}
```

在这个例子中，定义了两个类似的函数，用于检查两个Value对象是否相等。isEqualToMember()是成员函数，isEquallToNonmember() 是非成员函数。让我们专注于如何定义这些函数。

在isEqualToMember()中，隐式传递一个对象，显式传递另一个对象。函数的实现反映了这一点，必须在思想上协调m_value属于隐式对象，而v.m_value则属于显式参数。

在isEqualToNonmember()中，两个对象都是显式传递的。这导致函数实现中更好的对称性，因为m_value成员有一个显式的前缀。

您可能仍然更喜欢调用语法v1.isEqualToMember(v2)，而不是isEquallToNonmember(v1, v2)。但当我们讨论操作符重载时，将看到这个主题再次出现。



### 15.8.5 多个友元

一个函数可以同时是多个类的友元。例如，考虑以下示例：

```C++
#include <iostream>

class Humidity; // 前向声明 Humidity

class Temperature
{
private:
    int m_temp { 0 };
public:
    explicit Temperature(int temp) : m_temp { temp } { }

    friend void printWeather(const Temperature& temperature, const Humidity& humidity); // 这一行需要 Humidity 的前向声明
};

class Humidity
{
private:
    int m_humidity { 0 };
public:
    explicit Humidity(int humidity) : m_humidity { humidity } {  }

    friend void printWeather(const Temperature& temperature, const Humidity& humidity);
};

void printWeather(const Temperature& temperature, const Humidity& humidity)
{
    std::cout << "The temperature is " << temperature.m_temp <<
       " and the humidity is " << humidity.m_humidity << '\n';
}

int main()
{
    Humidity hum { 10 };
    Temperature temp { 12 };

    printWeather(temp, hum);

    return 0;
}
```

关于这个例子，有三点值得注意。首先，由于printWeather() 平等地使用湿度和温度，因此将其作为其中一个的成员是没有意义的。非成员函数工作得更好。其次，因为printWeather() 是湿度和温度的友元，所以它可以从这两个类的对象访问私有数据。最后，请注意示例顶部的以下行：

```c++
class Humidity;
```

这是湿度类的前向声明。类前向声明的作用与函数前向声明相同——它们将稍后定义的标识符告知编译器。然而，与函数不同，类没有返回类型或参数，因此类前向声明总是简单的类名（除非它们是类模板）。

如果没有这一行，编译器将在解析Temperature中的友元声明时告诉我们它不知道Humidity是什么。



### 15.8.6 友元违反数据隐藏的原则了吗？

不。友元是由进行数据隐藏的类授予的，该类期望友元访问其私有成员。将友元视为类本身的扩展，具有相同的访问权限。因此，访问是预期的，而不是违规。

如果使用得当，友元可以使程序更易于维护，因为它允许在从设计角度看有意义时分离函数（而不是出于访问控制的原因必须将其保持在一起）。或者当使用非成员函数。

然而，因为友元可以直接访问类的实现，所以对类的实现的更改通常也需要对友元进行更改。如果一个类有许多友元（或者那些友元有友元），这可能会导致连锁反应。

在实现友元函数时，尽可能使用公共接口而不是直接访问成员。这将有助于将您的友元函数与未来的实现更改隔离开来，并减少需要在以后修改和/或重新测试的代码。

友元函数应该尽可能使用类接口而不是直接访问。



### 15.8.7 与友元函数相比，优先使用非友元函数

在讨论数据隐藏（封装）的好处中，我们提到应该更喜欢非成员函数，而不是成员函数。出于同样的原因，应该更喜欢非友元函数而不是友元函数。

例如，在下面的示例中，如果Accumulator的实现被更改（例如，我们重命名m_value），那么print()的实现也需要更改：

```C++
#include <iostream>

class Accumulator
{
private:
    int m_value { 0 }; // 如果重名这里

public:
    void add(int value) { m_value += value; } // 这里需要修改

    friend void print(const Accumulator& accumulator);
};

void print(const Accumulator& accumulator)
{
    std::cout << accumulator.m_value; // 这里也需要修改
}

int main()
{
    Accumulator acc{};
    acc.add(5); // 将 5 加到 accumulator

    print(acc); // 调用 print() 非成员函数

    return 0;
}
```

更好的方式如下：

```C++
#include <iostream>

class Accumulator
{
private:
    int m_value { 0 };

public:
    void add(int value) { m_value += value; }
    int value() const { return m_value; } // 添加合理的访问函数
};

void print(const Accumulator& accumulator) // 不再是 Accumulator 的友元
{
    std::cout << accumulator.value(); // 使用访问函数而不是直接访问
}

int main()
{
    Accumulator acc{};
    acc.add(5); // 将 5 加到 accumulator

    print(acc); // 调用 print() 非成员函数

    return 0;
}
```

在本例中，print() 使用访问函数 value() 获取m_value的值，而不是直接访问m_value。现在，如果Accumulator的实现发生了更改，则不需要更新print() 。

在向现有类的公共接口添加新成员时要小心，因为每个函数（即使是微不足道的函数）都会增加一定程度的混乱和复杂性。在上述Accumulator的情况下，具有访问函数来获取当前累积值是完全合理的。在更复杂的情况下，最好使用友元，而不是向类的接口添加许多新的访问函数。



## 15.9 友元类和友元成员函数

### 15.9.1 友元类

友元类是可以访问另一个类的私有成员和受保护成员的类。

下面是一个示例：

```C++
#include <iostream>

class Storage
{
private:
    int m_nValue {};
    double m_dValue {};
public:
    Storage(int nValue, double dValue)
       : m_nValue { nValue }, m_dValue { dValue }
    { }

    // 设置 Display 是 Storage 的友元类
    friend class Display;
};

class Display
{
private:
    bool m_displayIntFirst {};

public:
    Display(bool displayIntFirst)
         : m_displayIntFirst { displayIntFirst }
    {
    }

    // 因为 Display 是 Storage 的友元, Display 中可以访问 Storage 的所有成员
    void displayStorage(const Storage& storage)
    {
        if (m_displayIntFirst)
            std::cout << storage.m_nValue << ' ' << storage.m_dValue << '\n';
        else // 首先显示 double 值
            std::cout << storage.m_dValue << ' ' << storage.m_nValue << '\n';
    }

    void setDisplayIntFirst(bool b)
    {
         m_displayIntFirst = b;
    }
};

int main()
{
    Storage storage { 5, 6.7 };
    Display display { false };

    display.displayStorage(storage);

    display.setDisplayIntFirst(true);
    display.displayStorage(storage);

    return 0;
}
```

由于Display类是Storage的友元，因此Display中可以访问任何Storage对象的私有成员。

首先，尽管Display是Storage的友元，但Display不能访问Storage对象的*this指针（因为*this实际上是一个函数参数）。

第二，友元不是对等的。仅仅因为Display是Storage的友元，并不意味着Storage也是Display的朋友。如果希望两个类成为彼此的友元，则两者都必须将对方声明为友元。

友元也是不可传递的。如果A类是B的友元，而B是C的友元，这并不意味着A是C的友元。

友元类可以充当前向声明。这意味着不需要在将类绑定到友元之前去向前声明它。在上面的示例中，「friend class Display;」同时充当Display的向前声明和友元声明。

友元也是不可继承的。如果B是A的友元，则从B派生的类不是A的友元。



### 15.9.2 友元成员函数

可以将单个成员函数设置为友元，而不是将整个类设置为友元。这类似于使非成员函数成为友元，只是改为使用成员函数。

然而，在现实中，这可能比预期的要复杂一些。让我们转换前面的示例，使Display::displayStorage成为友元成员函数。您可以尝试这样的操作：

```C++
#include <iostream>

class Display; // 前向声明 Display

class Storage
{
private:
	int m_nValue {};
	double m_dValue {};
public:
	Storage(int nValue, double dValue)
		: m_nValue { nValue }, m_dValue { dValue }
	{
	}

	// 让 Display::displayStorage 成员函数成为 Storage 的友元函数
	friend void Display::displayStorage(const Storage& storage); // 编译失败: Storage 这里看不到 Display 类的定义
};

class Display
{
private:
	bool m_displayIntFirst {};

public:
	Display(bool displayIntFirst)
		: m_displayIntFirst { displayIntFirst }
	{
	}

	void displayStorage(const Storage& storage)
	{
		if (m_displayIntFirst)
			std::cout << storage.m_nValue << ' ' << storage.m_dValue << '\n';
		else
			std::cout << storage.m_dValue << ' ' << storage.m_nValue << '\n';
	}
};

int main()
{
    Storage storage { 5, 6.7 };
    Display display { false };
    display.displayStorage(storage);

    return 0;
}
```

然而，这是行不通的。为了使单个成员函数成为友元，编译器必须看到友元成员函数类的完整定义（而不仅仅是前向声明）。由于类Storage尚未看到类Display的完整定义，因此编译器将在尝试使成员函数成为友元时出错。

幸运的是，通过将类Display的定义移动到类Storage的定义之前（在同一文件中，或者将Display的含义移动到头文件中，并在定义Storage之前引用它），可以很容易地解决这个问题。

```C++
#include <iostream>

class Display
{
private:
	bool m_displayIntFirst {};

public:
	Display(bool displayIntFirst)
		: m_displayIntFirst { displayIntFirst }
	{
	}

	void displayStorage(const Storage& storage) // 编译失败: 编译器不知道 Storage 是啥东西
	{
		if (m_displayIntFirst)
			std::cout << storage.m_nValue << ' ' << storage.m_dValue << '\n';
		else // display double first
			std::cout << storage.m_dValue << ' ' << storage.m_nValue << '\n';
	}
};

class Storage
{
private:
	int m_nValue {};
	double m_dValue {};
public:
	Storage(int nValue, double dValue)
		: m_nValue { nValue }, m_dValue { dValue }
	{
	}

	// 让 Display::displayStorage 成员函数成为 Storage 的友元函数
	friend void Display::displayStorage(const Storage& storage); // okay now
};

int main()
{
    Storage storage { 5, 6.7 };
    Display display { false };
    display.displayStorage(storage);

    return 0;
}
```

然而，现在有另一个问题。因为成员函数Display::displayStorage() 使用Storage作为引用参数，并且刚刚将Storage的定义移到Display的定义之下，编译器将抱怨它不知道Storage是什么。不能通过重新排列定义顺序来修复此问题，因为这样将撤消以前的修复。

幸运的是，通过几个简单的步骤，这也是可以修复的。首先，可以添加类Storage作为前向声明，以便编译器在看到类的完整定义之前可以引用Storage。

其次，可以在Storage类的完整定义之后，将 Display::displayStorage() 的定义移出该类。

下面是它的样子：

```C++
#include <iostream>

class Storage; // 前向声明 Storage

class Display
{
private:
	bool m_displayIntFirst {};

public:
	Display(bool displayIntFirst)
		: m_displayIntFirst { displayIntFirst }
	{
	}

	void displayStorage(const Storage& storage); // 这一行需要看到 Storage 的前向声明
};

class Storage // Storage 的完整定义
{
private:
	int m_nValue {};
	double m_dValue {};
public:
	Storage(int nValue, double dValue)
		: m_nValue { nValue }, m_dValue { dValue }
	{
	}

	// 让 Display::displayStorage 成员函数成为 Storage 的友元函数
	// 需要看到 Display 类的完整定义
	friend void Display::displayStorage(const Storage& storage);
};

// 现在来定义 Display::displayStorage
// 需要看到 Storage 的完整定义 (因为要访问 Storage 的成员)
void Display::displayStorage(const Storage& storage)
{
	if (m_displayIntFirst)
		std::cout << storage.m_nValue << ' ' << storage.m_dValue << '\n';
	else
		std::cout << storage.m_dValue << ' ' << storage.m_nValue << '\n';
}

int main()
{
    Storage storage { 5, 6.7 };
    Display display { false };
    display.displayStorage(storage);

    return 0;
}
```

现在一切都将正确编译：类Storage的前向声明足以满足Display类中Display::displayStorage()的声明。Display的完整定义满足了将Display::displayStorage()声明为Storage的友元。类Storage的完整定义足以满足成员函数Display::displayStorage() 的定义。

如果这有点令人困惑，请参阅上面程序中的注释。关键点是类前向声明满足对类的引用。然而，访问类的成员需要编译器看到完整的类定义。

这看起来像是一种痛苦。幸运的是，因为这是我们试图在单个文件中完成所有事情，才不得不这样做。更好的解决方案是将每个类定义放在单独的头文件中，成员函数定义放在相应的.cpp文件中。这样，所有的类定义都将在.cpp文件中可用，并且不需要重新排列类或函数！



## 15.10 引用限定符

在前面学习返回对数据成员的引用的成员函数中，讨论了当隐式对象是右值时，返回对数据成员的引用是危险的。下面简要回顾一下：

```C++
#include <iostream>
#include <string>
#include <string_view>

class Employee
{
private:
	std::string m_name{};

public:
	Employee(std::string_view name): m_name { name } {}
	const std::string& getName() const { return m_name; } //  返回 const 引用
};

// createEmployee() 按值返回一个 Employee (意味着返回的是一个右值)
Employee createEmployee(std::string_view name)
{
	Employee e { name };
	return e;
}

int main()
{
	// Case 1: okay: 在同一个表达式中使用右值的成员的引用
	std::cout << createEmployee("Frank").getName() << '\n';

	// Case 2: 有问题: 保存右值返回的成员的引用，稍后使用
	const std::string& ref { createEmployee("Garbo").getName() }; // 悬空引用，createEmployee() 创建的临时对象已经被销毁
	std::cout << ref << '\n'; // 未定义的行为

	return 0;
}
```

在案例2中，从createEmployee(“Garbo”) 返回的右值对象在初始化ref后被销毁，使ref引用刚被销毁的数据成员。ref的后续使用造成未定义的行为。

这有点棘手。

1. 如果getName()函数按值返回，会生成昂贵且不必要的副本。
2. 如果getName()函数通过常量引用返回，则这是高效的（因为没有生成std::string的副本），但当调用的对象是右值时，可能会被误用（导致未定义的行为）。

由于成员函数通常在左值对象上调用，因此传统的选择是通过常量引用返回，并在隐式对象是右值的情况下简单地避免误用返回的引用。



### 15.10.1 引用限定符

上述挑战的根源是，希望一个函数服务于两种不同的情况（一种是隐式对象是左值，另一种是隐式对象是右值）。一种情况下的最佳方案对另一种情况并不理想。

为了帮助解决这些问题，C++11引入了一个鲜为人知的特性，称为引用限定符，它允许根据是在左值还是右值对象上调用成员函数来重载它。使用这个特性，可以创建getName()的两个版本——一个用于对象是左值的情况，另一个用于对象为右值的情况。

首先，从getName() 的非引用限定版本开始

```c++
std::string& getName() const { return m_name; } // 在左值和右值对象上均可调用
```

为了引用限定此函数，将一个「&」限定符添加到只匹配左值对象的重载中，并将一个「&&」限定符加到只匹配右值对象的重载中：

```C++
const std::string& getName() const &  { return m_name; } //  & 限定只匹配左值隐式对象, 按引用返回
std::string        getName() const && { return m_name; } // && 限定只匹配右值隐式对象, 按值返回
```

因为这些函数是不同的重载，所以它们可以有不同的返回类型！左值限定重载通过常量引用返回，而右值限定重载则通过值返回。

下面是上面的完整示例：

```C++
#include <iostream>
#include <string>
#include <string_view>

class Employee
{
private:
	std::string m_name{};

public:
	Employee(std::string_view name): m_name { name } {}

	const std::string& getName() const &  { return m_name; } //  & 限定只匹配左值隐式对象, 按引用返回
	std::string        getName() const && { return m_name; } // && 限定只匹配右值隐式对象, 按值返回
};

// createEmployee() 按值返回一个 Employee (意味着返回的是一个右值)
Employee createEmployee(std::string_view name)
{
	Employee e { name };
	return e;
}

int main()
{
	Employee joe { "Joe" };
	std::cout << joe.getName() << '\n'; // Joe 是 左值, 调用的是 std::string& getName() & (返回引用)
    
	std::cout << createEmployee("Frank").getName() << '\n'; // Frank 是 右值, 调用的是 std::string getName() && (返回拷贝)

	return 0;
}
```

这允许我们在隐式对象是左值时做高效率的事情，而在隐式目标是右值时做安全的事情。

当隐式对象是非常量临时对象时，从性能角度来看，上面的getName()的上述右值重载可能是次优的。在这种情况下，隐式对象无论如何都将在表达式末尾死亡。因此，可以让它尝试移动成员（使用std::move），而不是返回成员的副本（可能很昂贵）。

这可以通过为非常量值添加以下重载getter来实现：

```C++
        // 如果隐式对象是 非 const 右值, 使用 std::move 去尝试移动 m_name
	std::string getName() && { return std::move(m_name); }
```

这既可以与const 右值 getter共存，也可以直接使用它（因为const 右值相当少见）。



### 15.10.2 关于引用限定成员函数的一些注释

首先，对于给定的函数，非引用限定重载和引用限定过载不能共存。只用使用一个或另一个。

其次，如果仅提供左值限定重载（即未定义右值限定版本），则对具有右值隐式对象的函数的任何调用都将导致编译错误。这提供了一种有用的方法，可以完全防止将函数与右值隐式对象一起使用。



### 15.10.3 不建议使用引用限定符

虽然引用限定符有用，但以这种方式使用它们有一些缺点。

1. 向每个返回引用的getter添加右值重载会给类增加混乱，而只是为了解决不常见的情况，通过良好的习惯很容易避免问题。
2. 通过值返回右值重载意味着必须支付复制（或移动）的成本，即使在可以安全使用引用的情况下（例如，在课程顶部的示例的情况1）。

此外：

1. 大多数C++开发人员都不知道该功能（这可能会导致错误或使用效率低下）。
2. 标准库通常不使用此功能。

基于以上所有内容，不建议将引用限定符用作最佳实践。相反，建议始终立即使用访问函数的结果，而不要保存返回的引用以供以后使用。
