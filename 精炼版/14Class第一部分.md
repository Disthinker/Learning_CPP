# 14 Class第一部分

## 14.1 面向对象编程简介

### 14.1.1 面向过程编程

我们在C++中将对象定义为“一块可用于存储值的内存”。具有名称的对象称为变量。C++程序由发送到计算机的指令的顺序列表组成，这些指令定义数据（通过对象）和对该数据执行的操作（通过语句和表达式组成的函数）。

到目前为止，我们一直在做一种称为面向过程编程的行为。在其中，重点是创建实现程序逻辑的“过程”（在C++中称为函数）。将数据对象传递给这些函数，这些函数对数据执行操作，然后可能返回一个结果供调用方使用。

在过程编程中，函数和这些函数操作的数据是单独的实体。程序员负责将函数和数据组合在一起，以产生所需的结果。这导致代码如下所示：

```c++
eat(you, apple);
```

现在，看看你周围的一切——你看到的每一个地方都是物体：书籍、建筑物、食物，甚至你。这类对象有两个主要组成部分：1）一些相关的属性（例如重量、颜色、大小、坚固性、形状等），以及2）它们可以表现的一些行为（例如打开、使其他东西变热等）。这些属性和行为是不可分割的。

在编程中，属性由对象表示，行为由函数表示。因此，过程编程比较糟糕地描述了现实，因为它分离了属性（对象）和行为（函数）。



### 14.1.2 面向对象编程

在面向对象编程（object-oriented programming，通常缩写为OOP）中，重点是创建合适的数据类型，里面包含对应的属性以及一组良好的行为函数。OOP中的术语“对象”是指可以从此类型中实例化的对象。

这导致代码看起来更像：

```c++
you.eat(apple);
```

这使得行为主体（you）、正在调用什么行为（eat）以及哪些对象是该行为的附件（apple）变得更清楚。

因为属性和行为不再是分开的，所以对象更容易模块化，这使得程序更容易编写和理解，并且还提供了更高程度的代码重用性。通过定义对象之间如何互相交互，这提供了一种更直观的处理数据的方法。

在下一课中，将讨论如何创建此类对象。



### 14.1.3 面向过程与面向对象程序示例

下面是一个面向过程风格的程序，打印动物的名称和腿的数量：

```C++
#include <iostream>
#include <string_view>

enum AnimalType
{
    cat,
    dog,
    chicken,
};

constexpr std::string_view animalName(AnimalType type)
{
    switch (type)
    {
    case cat: return "cat";
    case dog: return "dog";
    case chicken: return "chicken";
    default:  return "";
    }
}

constexpr int numLegs(AnimalType type)
{
    switch (type)
    {
    case cat: return 4;
    case dog: return 4;
    case chicken: return 2;
    default:  return 0;
    }
}


int main()
{
    constexpr AnimalType animal{ cat };
    std::cout << "A " << animalName(animal) << " has " << numLegs(animal) << " legs\n";

    return 0;
}
```

在这个程序中，编写了一些函数，这些函数允许我们做一些事情，如获取动物的腿数，以及获取动物的名称。

虽然这很好，但考虑一下当想更新这个程序时会发生什么，如果想添加动物蛇。向代码中添加蛇，需要修改AnimalType、numLegs()和animalName()。如果这是一个大的代码库，还需要更新使用AnimalType的任何其他相关函数——如果AnimalType在许多地方使用，那么可能有许多代码需要修改（很容易引入错误）。

现在，让我们使用OOP思维来编写相同的程序（产生相同的输出）：

```C++
#include <iostream>
#include <string_view>

struct Cat
{
    std::string_view name{ "cat" };
    int numLegs{ 4 };
};

struct Dog
{
    std::string_view name{ "dog" };
    int numLegs{ 4 };
};

struct Chicken
{
    std::string_view name{ "chicken" };
    int numLegs{ 2 };
};

int main()
{
    constexpr Cat animal;
    std::cout << "a " << animal.name << " has " << animal.numLegs << " legs\n";

    return 0;
}
```

在本例中，每个动物都是其自己的类型，并且该类型管理与该动物相关的所有内容。

现在考虑这样一个情况，如果想添加动物蛇。我们所要做的就是创建一个蛇类型。几乎不需要更改现有代码，这意味着破坏已经工作的代码的风险要小得多。

上面的猫、狗和鸡示例有许多重复（因为每个都定义了完全相同的成员集）。在这种情况下，创建通用的Animal结构并为每个动物创建实例可能更可取。但如果我们想向Chicken添加一个不适用于其他动物的新成员（例如，每天下蛋的个数），该怎么办？在后面会介绍，通过使用OOP模型，我们可以将该成员限制为只属于Chicken对象。



### 14.1.4 OOP带来了其他好处

在学校，当你提交编程作业时，你的工作基本上已经完成。您的老师或助教将运行您的代码，以查看它是否产生正确的结果。根据运行结果，你会相应地被评分。然后您的代码大概率会被丢弃。

在实际工作中，当您将代码提交到其他开发人员使用的代码库中，或提交到真实用户使用的应用程序中时，这是一种完全不同的情况。一些新的操作系统或软件版本将破坏您的代码。用户将发现您所犯的一些逻辑错误。业务合作伙伴将需要一些新的功能。其他开发人员将需要在不破坏代码的情况下扩展您的代码。您的代码需要能够进化，并且它需要能够以最小的时间投入、最小的头痛和最小的破坏来做到这一点。

解决这些问题的最佳方法是尽可能保持代码的模块化（和非冗余）。为了帮助实现这一点，OOP还引入了许多其他有用的概念：继承、封装、抽象和多态。

我们将在适当的时候讨论所有这些都是什么，以及它们如何帮助减少代码的冗余，并更容易修改和扩展。一旦您正确地熟悉了OOP并使用了它，您可能永远不会想再回到纯过程编程。

也就是说，OOP并不能取代过程编程——相反，它在编程工具带中为您提供了额外的工具，以帮助在需要时管理复杂性。



### 14.1.5 术语“对象”

请注意，术语“对象”在不同地方有不同的含义，这会导致一定程度的混淆。在传统编程中，对象是存储值的存储空间。在面向对象编程中，“对象”意味着它既是传统编程意义上的对象，又是属性和行为的组合。在本教程中，将倾向于术语对象的传统含义，并在特别提到OOP对象时更倾向于术语“类对象”。



## 14.2 class简介

在前一章中，我们介绍了结构体。它可以将多个成员变量绑定到单个对象（可以作为一个单元进行初始化和传递）。换句话说，结构体提供了一个方便的封装形式，来存储和移动相关的数据值。

考虑以下结构：

```C++
#include <iostream>

struct Date
{
    int day{};
    int month{};
    int year{};
};

void printDate(const Date& date)
{
    std::cout << date.day << '/' << date.month << '/' << date.year; // 设置 DMY 的格式
}

int main()
{
    Date date{ 4, 10, 21 }; // 使用聚合初始化
    printDate(date);        // 讲对象传递给函数

    return 0;
}
```

尽管结构体很有用，但有许多缺陷，在试图构建大型复杂程序（特别是由多个开发人员处理的程序）时，这些缺陷可能会带来挑战。



### 14.2.1 数据状态有效性问题

结构体最大的问题，是没有办法确保结构体内的数据一定是有效的。在前面，我们学习过不变量的定义，即“在某个组件执行时必须为真的条件”。

在类类型（包括结构体、类和联合）的情况下，类不变量是一个条件，必须在对象的整个生存期内为真，以便对象保持有效状态。违反的类不变量的对象被称为处于无效状态，使用该对象可能会导致意外或未定义的行为。

首先，考虑以下结构体：

```C++
struct Pair
{
    int first {};
    int second {};
};
```

第一个和第二个成员可以独立设置为任何值，因此Pair结构没有不变量。

现在考虑以下表示分数的几乎相同的结构体：

```C++
struct Fraction
{
    int numerator { 0 };
    int denominator { 1 };
};
```

分母为0的分数在数学上是无效的（因为分数的值是其分子除以分母——除以0在数学上没有定义）。因此，需要确保Fraction对象的denominator成员不会设置为0。

例如：

```C++
#include <iostream>

struct Fraction
{
    int numerator { 0 };
    int denominator { 1 }; // 默认初始化: 初始化为有效值
};

void printFractionValue(const Fraction& f)
{
     std::cout << f.numerator / f.denominator << '\n';
}

int main()
{
    Fraction f { 5, 0 };   // 创建一个分母为0的分数
    printFractionValue(f); // 会导致除0错误

    return 0;
}
```

在上面的例子中，使用注释来记录Fraction的不变量。还提供了一个默认的成员初始值，以确保在用户不提供初始化值的情况下将分母设置为1。这确保了当用户决定对Fraction对象进行值初始化时，Fraction object将有效。这是一个好的开始。

但没有什么可以阻止显式地违反这个类不变量：当创建分数f时，可以使用聚合初始化来显式地将分母初始化为0。虽然这不会立即导致问题，但对象现在处于无效状态，进一步使用该对象可能会导致意外或未定义的行为。

当调用printFractionValue(f)时：程序由于除零错误而终止。

考虑到Fraction示例的相对简单性，简单地避免创建无效的Fraction对象应该不会太困难。然而，在使用许多结构体、具有许多成员的结构体或其成员具有复杂关系的更复杂的代码库中，理解哪些值组合可能违反某些类不变量可能不是那么明显。

一个小的改进是在printFractionValue函数的头部使用断言 assert(f.denominator != 0); 。然而，从行为上来说，这并没有真正改变任何事情。我们应该做到的是避免出错。



### 14.2.2 更复杂的类不变量

Fraction的类不变量是简单的——分母成员不能为0。这在概念上很容易理解，也不太难避免。

当结构体的成员必须具有相关值时，这变得更具挑战性。

```C++
#include <string>

struct Employee
{
    std::string name { };
    char firstInitial { }; // 应当永远是 `name` 的第一个字母 (or `0`)
};
```

在上面的（设计不佳）结构体中，存储在成员firstInitial中的字符值应始终与name的第一个字符匹配。

初始化Employee对象时，用户负责确保维护类不变量。如果name被分配了一个新值，还必须确保firstInitial也被更新。对于使用Employee对象的开发人员来说，这种相关性可能并不明显，也可能忘记维护这种关系。

即使我们编写函数来帮助我们创建和更新Employee对象（确保始终从name的第一个字符设置firstInitial），仍然依赖于用户了解和使用这些函数。

简而言之，依赖开发人员手动维护类不变量可能会导致有问题的代码。

理想情况下，希望有一种机制，对象要么不能被置于无效状态，要么可以立即发出异常信号（而不是让未定义的行为在未来的某个随机点发生）。

结构体（聚合样式）没有解决这种问题的优雅机制。



### 14.2.3 class简介

在开发C++时，Bjarne Stroustrup希望引入一些功能，允许开发人员创建可以更直观地使用的程序定义类型。他还对为困扰大型复杂程序的一些常见缺陷和维护挑战（如前面提到的类不变量问题）寻找优雅的解决方案感兴趣。

根据他在其他编程语言（特别是Simula，第一个面向对象的编程语言）方面的经验，Bjarne确信，开发一种程序定义的类型是可能的，它是通用的，功能强大，足以用于几乎任何事情。在向Simula学习时，他将这种类型称为类（class）。

就像结构体一样，类是程序定义的复合类型，可以有许多具有不同类型的成员变量。

从技术角度来看，结构体和类几乎是相同的——因此，使用结构体实现的任何示例都可以使用类实现，反之亦然。然而，从实践的角度来看，我们使用结构体和类的方式不同。



### 14.2.4 定义类

由于类是程序定义的数据类型，因此必须在使用之前定义它。类的定义类似于结构体，只是我们使用class关键字而不是struct。例如，下面是Employee类的定义：

```C++
class Employee
{
    int m_id {};
    int m_age {};
    double m_wage {};
};
```

为了演示相似的类和结构体可以有多相似，下面的程序等效于在课程顶部介绍的程序，但Date现在是一个类，而不是结构：

```C++
#include <iostream>

class Date       // 将 struct 替换为 class
{
public:          // 这里行，是一个访问说明符
    int m_day{}; // 为成员变量，添加 "m_" 前缀
    int m_month{};
    int m_year{};
};

void printDate(const Date& date)
{
    std::cout << date.m_day << '/' << date.m_month << '/' << date.m_year;
}

int main()
{
    Date date{ 4, 10, 21 };
    printDate(date);

    return 0;
}
```



### 14.2.5 大多数C++标准库都是类

您已经在使用过类对象。std::string和std::string_view都被定义为类。事实上，标准库中的大多数非别名类型都定义为类！

类确实是C++的核心和灵魂——它们是如此基础，以至于C++最初被命名为“带类的C”！



## 14.3 成员函数

在前面，我们介绍了结构体是程序定义的类型，它可以包含成员变量。下面是用于保存日期的结构体的示例：

```C++
struct Date
{
    int year {};
    int month {};
    int day {};
};
```

现在，如果想将日期打印到屏幕上，需要编写一个函数来完成这项工作。下面是一个完整的程序：

```C++
#include <iostream>

struct Date
{
    // 这里是成员变量
    int year {};
    int month {};
    int day {};
};

void print(const Date& date)
{
    // 使用 (.) 来访问成员变量
    std::cout << date.year << '/' << date.month << '/' << date.day;
}

int main()
{
    Date today { 2020, 10, 14 }; // 聚合初始化结构体

    today.day = 16; // 使用 (.) 来访问成员变量
    print(today);   // 使用普通函数来访问结构体

    return 0;
}
```



### 14.3.1 属性和动作的分离

看看我们周围的一切——看到的每一个地方都是物体：书籍、建筑物、食物，甚至是我们自己。现实生活中的对象有两个主要组成部分：1）一些可观察的属性（例如重量、颜色、大小、坚固性、形状等……），以及2）它们可以执行的基于这些属性的一些操作（例如被打开、损坏其它）。这些属性和动作是不可分割的。

在编程中，用变量表示属性，用函数表示动作。

在上面的Date示例中，请注意，分别定义了属性（Date的成员变量）和使用这些属性执行的操作（函数print()）。只需根据print()的const Date&参数来推断Date和print()之间的连接。

虽然可以将Date和print()放在一个名称空间中（以便更清楚地知道这两个是要打包在一起的），但这会在程序中添加更多的名称和更多的名称空间前缀，从而使代码混乱。

如果有某种方法可以将属性和操作一起定义为一个整体，则可以解决这种分离问题。



### 14.3.2 成员函数

除了有成员变量之外，类类型（包括结构体、类和联合）也可以有自己的函数！属于类类型的函数称为成员函数。

不是成员函数的函数被称为非成员函数，以将它们与成员函数区分开来。上面的print()函数是一个非成员函数。

成员函数必须在类类型定义内部声明，可以在类类型内部或外部定义。提醒一下，定义也是声明，因此如果在类中定义成员函数，它将被视为声明。

在其他面向对象语言（如Java和C#）中，这些被称为方法。尽管术语“方法”在C++中没有使用，但首先学习其他语言之一的程序员仍然可以使用该术语。



### 14.3.3 成员函数示例

让我们重写课程顶部的Date示例，将print()从非成员函数转换为成员函数：

```C++
// 成员函数版本
#include <iostream>

struct Date
{
    int year {};
    int month {};
    int day {};

    void print() // 定义了成员函数 print
    {
        std::cout << year << '/' << month << '/' << day;
    }
};

int main()
{
    Date today { 2020, 10, 14 }; // 结构体聚合初始化

    today.day = 16; // 使用 (.) 来访问成员变量
    today.print();  // 使用 (.) 来访问成员函数

    return 0;
}
```

成员函数与非成员函数示例之间有三个关键区别：

1. 声明与定义 print() 函数的位置
2. 如何调用 print() 函数
3. 如果在 print() 函数中访问成员变量



### 14.3.4 成员函数在类类型定义内声明

在非成员函数示例中，print() 非成员函数在Date结构体外部的全局命名空间中定义。默认情况下，它具有外部链接，因此可以从其他源文件调用它（使用适当的前向声明）。

在成员函数示例中，print() 成员函数在Date结构体定义中声明。因此 print() 被声明为Date的一部分，所以这告诉编译器print() 是一个成员函数。

在类类型定义内定义的成员函数是隐式内联的，因此如果类类型定义被包含在多个代码文件中，它们不会导致违反单定义规则。



### 14.3.5 调用成员函数（以及隐式对象）

在非成员函数示例中，调用print(today)，其中today（显式）作为参数传递。

在成员函数示例中，调用today.print() 。此语法使用成员选择运算符（.）选择要调用的成员函数，与访问成员变量的方式一致（例如，today.day=16; ）。

必须使用对应类型的对象调用（非静态）成员函数。在这种情况下，today 是调用 print() 的对象。

注意，在成员函数的情况下，不需要 today 作为参数传递。调用成员函数的对象隐式传递给成员函数。由于这个原因，调用成员函数的对象通常称为隐式对象。

换句话说，当调用 today.print() 时，today是隐式对象，它隐式传递给print() 成员函数。



### 14.3.6 成员函数内访问成员变量使用隐式对象

在成员函数内部，未以（.）为前缀的成员变量都与隐式对象相关联。

换句话说，当调用today.print()时，today是隐式对象，year、month和day（没有前缀）的值分别为today.year、today.month和today.day。



### 14.3.7 另一个成员函数示例

下面是一个稍微复杂一些的成员函数的示例：

```C++
#include <iostream>
#include <string>

struct Person
{
    std::string name{};
    int age{};

    void kisses(const Person& person)
    {
        std::cout << name << " kisses " << person.name << '\n';
    }
};

int main()
{
    Person joe{ "Joe", 29 };
    Person kate{ "Kate", 27 };

    joe.kisses(kate);

    return 0;
}
```

来看看这是如何工作的。首先，定义了两个Person变量，joe和kate。接下来，调用 joe.kisses(kate); 。joe是这里的隐式对象，kate作为显式参数传递。

当 kisses() 成员函数执行时，name 不使用成员选择操作符（.），因此它引用隐式对象，即joe，所以解析为joe.name。person.name 使用成员选择操作符，因此它不引用隐式对象。由于person是kate的引用，因此解析为kate.name。



### 14.3.8 成员变量和函数可以按任何顺序定义

C++编译器通常从上到下编译代码。对于遇到的每个名称，编译器会检查它是否已经看到该名称的声明，以便它可以进行适当的类型检查。

这意味着在非成员函数内部，不能使用未在之前声明的变量或函数：

```C++
void x()
{
    y(); // error: y 没有声明, 编译器无法得知y是什么
}
 
int y()
{
    return 5;
}
```

然而，在类类型内部，对于成员函数和成员变量，这个限制不适用，可以按自己喜欢的顺序定义成员。例如：

```C++
struct Foo
{
    int m_x{ y() };   // 这里可以调用 y()，即使 y 在这里仍未被定义

    void x() { y(); } // 这里可以调用 y()，即使 y 在这里仍未被定义
    int y()  { return 5; }
};
```

对于非成员函数，可以前向声明变量或函数，以便在编译器看到完整定义之前使用它们。类内的实际定义有隐式的前向声明。



### 14.3.9 成员函数可以重载

就像非成员函数一样，成员函数也可以重载，只要每个成员函数之间可以区分。



### 14.3.10 结构体和成员函数

在C中，结构体只能有成员变量，没有成员函数。

在C++中，在设计class时，Bjarne Stroustrup花费了一些时间考虑是否应授予结构体（从C继承）具有成员函数的能力。经过考虑，决定应该这样做。

在现代C++中，结构体具有成员函数。

这一决定引发了一系列其他问题，即结构体应该有哪些其他新的C++功能。Bjarne担心，让结构体访问有限的功能子集最终会增加语言的复杂性和边缘情况。为了简单起见，他最终决定结构体和类将具有统一的规则集（这意味着结构体可以做类可以做的一切，反之亦然）。

成员函数可以与结构体和类一起使用。

然而，结构体应该避免定义构造函数，因为这样做会使它们成为非聚合函数。



### 14.3.11 没有数据变量的类类型

可以创建没有数据成员的类类型（例如，仅具有成员函数）。也可以实例化此类类型的对象：

```C++
#include <iostream>

struct Foo
{
    void printHi() { std::cout << "Hi!\n"; }
};

int main()
{
    Foo f{};
    f.printHi(); // requires object to call

    return 0;
}
```

然而，如果类类型没有任何数据成员，那么使用类类型可能是多余的。在这种情况下，请考虑改用名称空间。可以更清楚地看到，没有管理的数据（并且不需要实例化对象来调用函数）。

```C++
#include <iostream>

namespace Foo
{
    void printHi() { std::cout << "Hi!\n"; }
};

int main()
{
    Foo::printHi(); // 不需要实际的对象

    return 0;
}
```

如果类类型没有数据成员，则首选使用命名空间。



## 14.4 Const类对象和Const成员函数

在前面，我们知道基本数据类型（int、double、char等）的对象可以通过const关键字设置为常量。所有常变量都必须在创建时初始化。

类似地，通过使用const关键字，类类型对象（结构体、类和联合）也可以成为const。这样的对象也必须在创建时初始化。就像普通变量一样，当需要确保类类型对象在创建后不会被修改时，通常将它们设置为const（或constexpr）。



### 14.4.1 不允许修改常量对象的数据成员

一旦初始化了常量类类型对象，就不允许任何修改它的数据成员的操作，因为这将违反const属性。这包括直接更改成员变量（如果它们是public的），或调用设置成员变量的成员函数。



### 14.4.2 Const对象不能调用非const成员函数

您可能会惊讶地发现，此代码也会导致编译错误：

```C++
#include <iostream>

struct Date
{
    int year {};
    int month {};
    int day {};

    void print()
    {
        std::cout << year << '/' << month << '/' << day;
    }
};

int main()
{
    const Date today { 2020, 10, 14 }; // const

    today.print();  // 编译失败: 不能调用非const成员函数

    return 0;
}
```

即使print()不尝试修改成员变量，对today.print()的调用仍然是无法编译。这是因为print()成员函数本身没有声明为const。编译器不允许对常量对象调用非const成员函数。



### 14.4.3 Const成员函数

为了解决上述问题，需要使print()成为const成员函数。来保证不会修改对象或调用任何非const成员函数（因为它们可能会修改对象）。

使print()成为const成员函数很容易——只需将const关键字附加到函数原型中，在参数列表之后，在函数体之前：

```C++
#include <iostream>

struct Date
{
    int year {};
    int month {};
    int day {};

    void print() const // 现在是个const成员函数
    {
        std::cout << year << '/' << month << '/' << day;
    }
};

int main()
{
    const Date today { 2020, 10, 14 }; // const

    today.print();  // ok: const 对象可以调用 const 成员函数

    return 0;
}
```

在上面的例子中，print()是一个const成员函数，这意味着可以在const对象上调用它（例如today）。

试图更改成员变量或调用非const成员函数的const成员函数将导致发生编译器错误。例如：

```C++
struct Date
{
    int year {};
    int month {};
    int day {};

    void incrementDay() const // 设置为 const
    {
        ++day; // 编译失败: const 成员函数不能修改成员变量
    }
};

int main()
{
    const Date today { 2020, 10, 14 }; // const

    today.incrementDay();

    return 0;
}
```

构造函数不能设置为const，因为它们需要初始化或修改对象的成员。



### 14.4.4 可以在非常量对象上调用const成员函数

也可以在非常量对象上调用const成员函数：

```C++
#include <iostream>

struct Date
{
    int year {};
    int month {};
    int day {};

    void print() const // const
    {
        std::cout << year << '/' << month << '/' << day;
    }
};

int main()
{
    Date today { 2020, 10, 14 }; // non-const

    today.print();  // ok: 可以调用

    return 0;
}
```

由于const成员函数可以在常量和非常量对象上调用，因此如果成员函数不修改对象的状态，则应将其设置为常量。

一旦成员函数成为const，就可以在常量对象上调用该函数。

不（并且永远不会）修改对象状态的成员函数应成为const，以便可以在常量和非常量对象上调用它。



### 14.4.5 成员函数常量和非常量重载

最后，尽管不经常这样做，但可以重载成员函数，使其具有同一函数的const版本和非const版本。这是因为const限定符被认为是函数签名的一部分，所以两个仅在const上不同的函数被认为是不同的。

```C++
#include <iostream>

struct Something
{
    void print()
    {
        std::cout << "non-const\n";
    }

    void print() const
    {
        std::cout << "const\n";
    }
};

int main()
{
    Something s1{};
    s1.print(); // 调用 print()

    const Something s2{};
    s2.print(); // 调用 print() const
    
    return 0;
}
```

当返回值的const需要不同时，通常会使用const和非const版本重载函数。这是相当罕见的。



## 14.5 公共和私有成员以及访问说明符

### 14.5.1 成员访问权限

类的每个成员都有一个称为访问级别的属性，该属性确定谁可以访问该成员。

C++有三种不同的访问级别：公共（public）、私有（private）和受保护（protected）。在本课中，将介绍两个常用的访问级别：public和private。

每当访问成员时，编译器都会检查该成员的访问级别，是否允许访问该成员。如果不允许访问，编译器将生成编译错误。



### 14.5.2 默认情况下，结构体的成员是public的

具有公共访问级别的成员称为公共成员。公共成员，对如何访问它们没有任何限制。就像前面类比中的公园一样，任何人都可以访问公共成员。

公共成员可以被同一类的其他成员访问，也可以被同一类之外的其它地方访问到。

默认情况下，结构体的所有成员都是公共成员。

考虑以下结构：

```C++
#include <iostream>

struct Date
{
    // 结构体成员默认都是public，意味着可以被任何人访问
    int year {};       // public 默认
    int month {};      // public 默认
    int day {};        // public 默认

    void print() const // public 默认
    {
        // public成员可以被同一类中的其它成员函数访问
        std::cout << year << '/' << month << '/' << day;
    }
};

// 非成员函数 main，也意味着 "public"
int main()
{
    Date today { 2020, 10, 14 }; // 聚合初始化

    // public成员可以在此访问
    today.day = 16; // okay: day 是 public
    today.print();  // okay: print() 是public

    return 0;
}
```

在此示例中，可以在三个位置访问public成员：

1. 在成员函数print()中，访问隐式对象的year、month和day成员。
2. 在main()中，直接访问 today.day 来设置其值。
3. 在main()中，调用成员函数 today.print()。

所有这三种访问都是允许的，因为可以从任何地方访问pubilc成员。

因为main()不是Date的成员，所以它被认为是公共的一部分。然而，由于public中可以访问public成员，main() 可以直接访问Date的成员（包括对today.print()的调用）。

术语“public”用于给定的类的成员之外的代码。这包括非成员函数以及其他类的成员。



### 14.5.3 默认情况下，类的成员是私有的

私有成员只能由同一类的其他成员访问。

考虑下面的示例，它几乎与上面的示例相同：

```C++
#include <iostream>

class Date // 现在是 class 而不是 struct
{
    // class 成员默认是是 private, 只能被其它本类的成员访问
    int m_year {};     // 默认 private
    int m_month {};    // 默认 private
    int m_day {};      // 默认 private

    void print() const // 默认 private
    {
        // 私有成员可以在成员函数中访问到
        std::cout << m_year << '/' << m_month << '/' << m_day;
    }
};

int main()
{
    Date today { 2020, 10, 14 }; // 编译失败: 无法再使用聚合初始化

    // 私有成员不能在 public 域内访问
    today.m_day = 16; // 编译失败: m_day 是 private
    today.print();    // 编译失败: print() 是 private

    return 0;
}
```

在此示例中，成员在相同的三个位置进行访问：

1. 在成员函数print()中，访问隐式对象的m_year、m_month和m_day成员。
2. 在main()中，直接访问 today.m_day来设置其值。
3. 在main()中，调用成员函数 today.print()。

然而，如果您编译这个程序，您将注意到生成了三个编译错误。

在main() 中，语句 today.m_day = 16 和 today.print() 现在都会生成编译错误。这是因为main() 是公共域的一部分，并且不允许公共域直接访问私有成员。

在print()中，允许访问成员m_year、m_month和m_day。这是因为 print() 是类的成员，并且类的成员可以访问私有成员。

那么，第三个编译错误是从哪里来的呢？也许令人惊讶的是，today 的初始化现在会导致编译错误。在前面学习结构体聚合初始化中，可以注意到聚合“没有私有或受保护的非静态数据成员”。Date类具有私有数据成员（因为默认情况下类的成员是私有的），因此Date类不符合聚合的条件。因此，不能再使用聚合初始化来初始化它。



### 14.5.4 命名私有成员变量

在C++中，以“m_”前缀开头命名私有数据成员是一种常见的约定。这样做有几个重要的原因。

考虑某个类的以下成员函数：

```C++
// 某个成员函数，将成员变量 m_name 设置为参数 name
void setName(std::string_view name)
{
    m_name = name;
}
```

首先，“m_”前缀允许轻松地将数据成员与成员函数中的函数参数或局部变量区分开来。可以很容易地看到，“m_name”是成员，而“name”不是。这有助于明确此函数正在更改类的状态。这很重要，因为当更改数据成员的值时，它会持续存在于成员函数的作用域之外（而对函数参数或局部变量的更改通常不会）。

这与建议对局部静态变量使用“s_”前缀，对全局变量使用“g_”前缀的原因相同。

其次，“m_”前缀有助于防止私有成员变量与局部变量、函数参数和成员函数的名称之间的命名冲突。

如果私有成员名是name而不是m_name，则：

1. name函数参数将隐藏名称私有数据成员。
2. 若有一个名为name的成员函数，那个么由于重新定义标识符名称，将得到一个编译错误。

考虑以“m_”前缀开头命名私有成员，以帮助将它们与局部变量、函数参数和成员函数的名称区分开来。



### 14.5.5 通过访问说明符设置访问级别

默认情况下，结构体的成员是公共的，类的成员是私有的。

然而，可以通过使用访问说明符来显式设置成员的访问级别。访问说明符设置说明符之后的所有成员的访问级别。C++提供了三个访问说明符：“public:”、“private:”和“protected:”。

在下面的示例中，使用“public:”访问说明符来确保public域可以使用print()成员函数，使用“private:”访问说明符来使数据成员私有。

```C++
class Date
{
// 这里定义的所有成员都是 private

public: // 这里是 public: 访问说明符

    void print() const // public
    {
        // 可以访问 private 成员
        std::cout << m_year << '/' << m_month << '/' << m_day;
    }

private: // 这里是 private: 访问说明符

    int m_year { 2020 };  // private
    int m_month { 14 }; // private
    int m_day { 10 };   // private
};

int main()
{
    Date d{};
    d.print();  // okay, main() 可以访问 public 成员

    return 0;
}
```

此示例编译。由于print() 是“public:”访问说明符的公共成员，因此允许main()（它是public的一部分）访问它。

因为有私有成员，所以无法聚合初始化。对于本例，使用默认成员初始化（作为临时解决方案）。

由于类默认为private访问，因此可以省略前导的“private:”访问说明符：

```C++
class Foo
{
// 默认 private 访问级别
    int m_something {};  // 默认 private
};
```

然而，由于类和结构体具有不同的访问级别默认值，许多开发人员更喜欢显式声明：

```C++
class Foo
{
private: // 多余, 但是更清晰
    int m_something {};  // 默认 private
};
```

尽管这在技术上是多余的，但使用显式“private:”说明符可以清楚地表明以下成员是私有的，而不必根据Foo是定义为类还是结构体来推断默认访问级别。



### 14.5.6 访问级别摘要

下面是不同访问级别的快速摘要表：

| 访问级别 | 访问说明符 | 成员可访问 | 子类可访问 | public可访问 |
| -------- | ---------- | ---------- | ---------- | ------------ |
| 公共     | public:    | 是         | 是         | 是           |
| 受保护   | protected: | 是         | 是         | 否           |
| 私有     | private:   | 是         | 否         | 否           |

允许类类型以任何顺序使用任意数量的访问说明符，并且它们可以重复使用（例如，可以有一些公共成员，然后有一些私有成员，然后是更多公共成员）。

大多数类都为各种成员使用私有和公共访问说明符。



### 14.5.7 结构体和类的访问级别最佳实践

结构体应该完全避免访问说明符，这意味着默认情况下所有结构体成员都是公共的。我们希望结构体是聚合，并且聚合只能具有公共成员。使用“public:”访问说明符在默认情况下是多余的，使用“private:”或“protected:”将使该结构体成为非聚合结构。

类通常应仅具有私有（或受保护）数据成员（通过使用默认的“private:”或“protected:”）。

类通常具有公共成员函数（因此这些成员函数可以在创建对象后由公共使用）。如果成员函数不打算供公众使用，则有时会将其设为私有（或受保护）。



### 14.5.8 访问级别按类工作

C++访问级别的一个细微差别是，对成员的访问是在每个类的基础上定义的，而不是在每个对象的基础上。

成员函数可以直接访问（隐式对象的）私有成员。然而，由于访问级别是按类的，而不是按对象的，因此成员函数还可以直接访问作用域中同一类类型的任何其他对象的私有成员。



### 14.5.9 结构体和类之间的技术和实践差异

类将其成员默认设置为private，而结构体将其成员默认设置为public。

在实践中，以不同的方式使用结构体和类。

根据经验法则，在满足以下所有条件时使用结构体：

1. 有一个简单的数据集合，不能从限制访问中受益。
2. 聚合初始化已足够。
3. 没有类不变量、设置限制或清理需要。

可以在何处使用结构体的几个示例：constexpr全局程序数据、简单结构（例如int成员的简单集合，不能从私有化中受益）、用于从函数返回一组数据的结构。

否则使用类。



## 14.6 访问函数

考虑以下Date类：

```C++
#include <iostream>

class Date
{
private:
    int m_year{ 2020 };
    int m_month{ 10 };
    int m_day{ 14 };

public:
    void print() const
    {
        std::cout << m_year << '/' << m_month << '/' << m_day << '\n';
    }
};

int main()
{
    Date d{};  // 创建一个 Date 对象
    d.print(); // 打印 date

    return 0;
}
```

如果date对象的用户想要获得年份，该怎么办？或者将年份更改为不同的值？他们将无法这样做，因为m_year是私有的（因此不能被公共直接访问）。

对于某些类，（在类所处上下文中）需要适当地获取或设置私有成员变量的值。



### 14.6.1 访问函数

访问函数是一个普通的公共成员函数，其任务是检索或更改私有成员变量的值。

访问函数有两种：getter和setter。Getter（有时也称为访问器）是返回私有成员变量值的public成员函数。Setter（有时也称为mutator）是设置私有成员变量值的public成员函数。

Getter通常被设置为const，因此可以在常量和非常量对象上调用它们。Setter应该是非const，因此它们需要修改数据成员。

为了便于说明，更新Date类，以拥有一整套getter和setter：

```C++
#include <iostream>

class Date
{
private:
    int m_year { 2020 };
    int m_month { 10 };
    int m_day { 14 };

public:
    void print()
    {
        std::cout << m_year << '/' << m_month << '/' << m_day << '\n';
    }

    int getYear() const { return m_year; }        // getter for year
    void setYear(int year) { m_year = year; }     // setter for year

    int getMonth() const  { return m_month; }     // getter for month
    void setMonth(int month) { m_month = month; } // setter for month

    int getDay() const { return m_day; }          // getter for day
    void setDay(int day) { m_day = day; }         // setter for day
};

int main()
{
    Date d{};
    d.setYear(2021);
    std::cout << "The year is: " << d.getYear() << '\n';

    return 0;
}
```



### 14.6.2 访问函数命名

访问函数没有通用命名约定。当然，有一些命名约定比其他约定更受欢迎。

1. 前缀为“get”和“set”；
2. 没有前缀；
3. 仅限“set”前缀。



### 14.6.3 Getter应通过值或常量值引用返回

Getter应提供对数据的“只读”访问。因此，最佳实践是，它们应该通过值（如果制作成员的副本成本不高）或常量值引用（如果制作该成员的副本的成本很高）返回。



### 14.6.4 访问函数的问题

创建类时，请考虑以下事项：

1. 如果您的类没有不变量约束，并且需要大量访问函数，请考虑使用结构体（其数据成员是公共的），提供对成员的直接访问。
2. 优先实现行为或操作，而不是访问函数。例如，不是实现setAlive(bool)这样的setter，而是实现kill() 和 revive() 函数。
3. 仅在public区域需要合理获取或设置单个成员的值的情况下提供访问函数。



## 14.7 成员函数返回对数据成员的引用

```C++
// 接受两个std::string , 返回字典序最小的
const std::string& firstAlphabetical(const std::string& a, const std::string& b)
{
	return (a < b) ? a : b; // 可以对 std::string 使用 operator< ，按字典序来比较两个字符串
}

int main()
{
	std::string hello { "Hello" };
	std::string world { "World" };

	std::cout << firstAlphabetical(hello, world); // 按引用传递，按引用返回

	return 0;
}
```

成员函数也可以通过引用返回数据，并且也遵循与非成员函数相同的规则。然而，成员函数还有一个额外的情况需要讨论：通过引用返回数据成员。

这在getter访问函数中最常见，因此将使用getter成员函数来说明这个主题。但请注意，该主题适用于返回数据成员引用的任何成员函数。



### 14.7.1 按值返回数据成员可能很昂贵

考虑以下示例：

```C++
#include <iostream>
#include <string>

class Employee
{
	std::string m_name{};

public:
	void setName(std::string_view name) { m_name = name; }
	std::string getName() const { return m_name; } //  getter 按值返回
};

int main()
{
	Employee joe{};
	joe.setName("Joe");
	std::cout << joe.getName();

	return 0;
}
```

在本例中，getName() 访问函数按值返回std::string m_name。

虽然这是最安全的做法，但这也意味着每次调用 getName() 时都会生成m_name的昂贵副本。由于访问函数往往被大量调用，因此这通常不是最佳选择。



### 14.7.2 通过左值引用返回数据成员

成员函数还可以通过（常量）左值引用返回数据成员。

数据成员与包含它们的对象具有相同的生存期。由于成员函数总是在对象上调用，并且该对象必须存在于调用方的作用域中，因此成员函数通过（常量）左值引用返回数据成员通常是安全的（因为当函数返回时，通过引用返回的成员仍然存在于调用者的作用域）。

让我们更新上面的示例，以便getName()通过常量左值引用返回m_name：

```C++
#include <iostream>
#include <string>

class Employee
{
	std::string m_name{};

public:
	void setName(std::string_view name) { m_name = name; }
	const std::string& getName() const { return m_name; } //  getter 返回 const 左值引用
};

int main()
{
	Employee joe{}; // joe 在函数结束前都会存在
	joe.setName("Joe");

	std::cout << joe.getName(); // 获取 joe.m_name 的引用

	return 0;
}
```

现在，当调用joe.getName() 时，joe.m_name通过引用返回调用方，避免了复制。然后，调用者使用该引用将joe.m_name打印到控制台。

由于在main()函数结束之前，joe一直存在于调用方的作用域中，因此对joe.m_name的引用在相同的持续时间内也是有效的。



### 14.7.3 返回引用的类型应与数据成员的类型匹配

通常，通过引用返回的成员函数，其返回类型应与返回的数据成员的类型匹配。在上面的示例中，m_name的类型为std::string，因此getName()返回const std::string&。

对于getter，使用auto可以让编译器从返回的成员推断返回类型，这是确保不发生转换的有用方法：

```C++
#include <iostream>
#include <string>

class Employee
{
	std::string m_name{};

public:
	void setName(std::string_view name) { m_name = name; }
	const auto& getName() const { return m_name; } // 使用 `auto` 让编译器从 m_name 自动推导返回类型
};

int main()
{
	Employee joe{}; // joe 在函数结束前都会存在
	joe.setName("Joe");

	std::cout << joe.getName(); // 获取 joe.m_name 的引用

	return 0;
}
```

然而，从文档的角度来看，使用auto返回类型会模糊getter的返回类型。因此，通常更推荐显式返回类型。



### 14.7.4 右值隐式对象并通过引用返回

有一种情况需要小心一点。在上面的例子中，joe是一个左值对象，它一直存在到函数结束。因此，joe.getName()返回的引用在函数结束之前也是有效的。

但是，如果隐式对象是一个右值（例如某个按值返回的函数的返回值），该怎么办？右值对象在创建它们的完整表达式的末尾被销毁。当右值对象被破坏时，对该右值成员的任何引用都将无效并悬空，并且使用这种引用将产生未定义的行为。

因此，对右值对象成员的引用只能在创建右值对象的完整表达式中安全使用。

让我们探讨一些与此相关的案例：

```C++
#include <iostream>
#include <string>
#include <string_view>

class Employee
{
	std::string m_name{};

public:
	void setName(std::string_view name) { m_name = name; }
	const std::string& getName() const { return m_name; } //  getter 返回cosnt 引用
};

// createEmployee() 创建一个 Employee 值 (返回的是右值)
Employee createEmployee(std::string_view name)
{
	Employee e;
	e.setName(name);
	return e;
}

int main()
{
	// Case 1: okay: 在相同表达式中获取右值对象成员的引用
	std::cout << createEmployee("Frank").getName();

	// Case 2: 有误: 保存右值对象成员的引用在之后使用
	const std::string& ref { createEmployee("Garbo").getName() }; // createEmployee() 创建的对象被销毁后，返回的引用将会悬空
	std::cout << ref; // 未定义的行为

	// Case 3: okay: 将引用的值，拷贝到其它对象
	std::string val { createEmployee("Hans").getName() };
	std::cout << val; // okay: val 是一个独立的对象

	return 0;
}
```

当调用createEmployee()时，它将按值返回Employee对象。这个返回的Employee对象是一个右值，它将一直存在到包含对createEmployee()的调用的完整表达式的末尾。当该右值对象被销毁时，对该对象成员的任何引用都将成为悬空的。

在Case 1中，调用createEmployee(“Frank”)，它返回一个右值Employee对象。然后对这个右值对象调用getName()，它返回对m_name的引用。然后立即使用该引用将名称打印到控制台。此时，包含对createEmployee(“Frank”)的调用的完整表达式结束，右值对象及其成员被销毁。由于右值对象或其成员都没有在这一处之外使用，因此这种情况可以正常运行。

在Case 2中，遇到了问题。首先，createEmployee(“Garbo”)返回一个右值对象。然后调用getName()来获取对该右值的m_name成员的引用。然后使用该m_name成员初始化ref。此时，包含对createEmployee(“Garbo”)的调用的完整表达式结束，右值对象及其成员被销毁。这使得ref悬而未决。因此，当在后续语句中使用ref时，访问的是悬空引用和未定义的行为结果。

但如果想保存函数中的值，该函数通过引用返回成员以供以后使用，该怎么办？可以使用返回的引用来初始化非引用局部变量。

在Case 3中，使用返回的引用来初始化非引用局部变量val.这将导致被引用的成员被复制到val。初始化后，val独立于引用而存在。因此，当右值对象随后被销毁时，val不受此影响。因此，val可以在未来的语句中输出，而不会出现问题。



### 14.7.5 安全使用成员函数返回的引用

尽管右值隐式对象存在潜在的危险，但getter通常返回常量引用，而不是复制对象。

鉴于此，让我们讨论一下如何安全地使用这些函数的返回值。上述示例中的三个案例说明了三个关键点：

1. 优先立即使用成员函数返回的引用（如情况1所示）。由于这对左值和右值对象都有效，因此如果总是这样做，将避免麻烦。
2. 不要“保存”返回的引用以供以后使用（如案例2所示），除非确定隐式对象是左值。如果对右值隐式对象执行此操作，则悬空的引用时将导致未定义的行为。
3. 如果确实需要持久化返回的引用以供以后使用，并且不确定隐式对象是左值，则使用返回的引用来初始化一个新的对象，这将制作一个新的副本（如案例3所示）。



### 14.7.6 不返回对私有数据成员的非常量引用

因为引用的操作就像操作被引用的对象，所以返回非常量引用的成员函数提供对该成员的直接访问（即使该成员是私有的）。

例如：

```C++
#include <iostream>

class Foo
{
private:
    int m_value{ 4 }; // private 成员

public:
    int& value() { return m_value; } // 返回了非常量引用 (不要这样做)
};

int main()
{
    Foo f{};                // f.m_value 初始化是 4
    f.value() = 5;          // 将于 m_value = 5
    std::cout << f.value(); // 打印 5

    return 0;
}
```

因为 value() 返回对m_value的非常量引用，所以调用方可以使用该引用直接访问（并更改）m_value。

这允许调用者破坏访问控制系统。



### 14.7.7 Const成员函数不能返回对数据成员的非常量引用

不允许const成员函数返回对成员的非常量引用。这是有意义的——不允许常量成员函数修改对象的状态，也不允许它调用将修改对象状态的函数。它不应该做任何可能导致修改对象的事情。

如果允许const成员函数返回对成员的非常量引用，则它将为调用方提供一种直接修改该成员的方法。这违反了const成员函数的意图。



## 14.8 数据隐藏（封装）的好处

### 14.8.1 类中的实现和接口

类的接口，定义类的用户将如何与类的对象交互。由于只能从类外部访问公共成员，因此类的公共成员形成其接口。因此，由公共成员组成的接口有时称为公共接口。

接口是类的作者和类的用户之间的隐式契约。如果现有接口被更改，则使用它的任何代码都可能会无法使用。因此，确保类的接口设计良好且稳定（不要改变太多）是很重要的。

类的实现，由实际使类按预期行为的代码组成。这包括存储数据的成员变量，以及包含程序逻辑和操作成员变量的成员函数。



### 14.8.2 数据隐藏（封装）

在编程中的数据隐藏，通过对用户隐藏程序定义的数据类型的实现，来强制实现接口和实现分离。

术语封装有时也用于指代数据隐藏。然而，该术语也用于指将数据和功能捆绑在一起（不考虑访问控制），因此其使用可能是不明确的。



### 14.8.3 如何实现数据隐藏

在C++类类型中实现数据隐藏很简单。

首先，确保类的数据成员是私有的（因此用户不能直接访问它们）。成员函数体中的语句已经不能被用户直接访问。

其次，确保成员函数是公共的，以便用户可以调用它们。

通过遵循这些规则，强制类的用户使用公共接口操作对象，并防止他们直接访问实现细节。

在C++中定义的类应该使用数据隐藏。事实上，标准库提供的所有类都是这样做的。另一方面，结构体不应使用数据隐藏，因为具有非公共成员会阻止它们被视为聚合。

以这种方式定义类需要类作者做一些额外的工作。并且要求类的用户使用公共接口，这似乎比直接提供对成员变量的公共访问更麻烦。但这样做提供了大量有用的好处，有助于鼓励类的可重用性和可维护性。



### 14.8.4 数据隐藏使类更易于使用，并降低了复杂性

要使用封装的类，不需要知道它是如何实现的。只需要理解它的接口：哪些成员函数是公开可用的，它们采用什么参数，以及它们返回什么值。

例如：

```C++
#include <iostream>
#include <string_view>

int main()
{
    std::string_view sv{ "Hello, world!" };
    std::cout << sv.length();

    return 0;
}
```

在这个简短的程序中，没有看到如何实现std::string_view的详细信息。也无法看到一个std::string_view有多少数据成员，它们的名称是什么，或者它们是什么类型。也不知道length()成员函数如何返回正在查看的字符串的长度。

最重要的是，不必知道！这个程序效果符合预期。需要知道的只是如何初始化类型为std::string_view的对象，以及length()成员函数返回的内容。



### 14.8.5 数据隐藏允许我们维护不变量

之前我们介绍了类不变量的概念，这些条件在对象的整个生命周期中都必须为真，以便对象保持有效状态。

考虑以下程序：

```C++
#include <iostream>
#include <string>

struct Employee // 成员默认是 public
{
    std::string name{ "John" };
    char firstInitial{ 'J' }; // 需要是name的首字母

    void print() const
    {
        std::cout << "Employee " << name << " has first initial " << firstInitial << '\n';
    }
};

int main()
{
    Employee e{}; // 默认是 "John" 和 'J'
    e.print();

    e.name = "Mark"; // name 是 "Mark"
    e.print(); // 但是首字母是错的

    return 0;
}
```

Employee结构体具有一个类不变量，即firstInitial应始终等于name的第一个字符。不满足的话print()函数将打印出错误的结果。

因为名称name是公共的，所以main()中的代码能够将e.name设置为“Mark”，并且firstInitial成员不会更新。不变量被破坏了，对print()的第二次调用没有按预期工作。

当为用户提供对类实现的直接访问时，他们将负责维护所有不变量——他们可能不会这样做（可能根本不会）。将此负担放在用户身上会增加许多复杂性。

让我们重写此程序，使成员变量私有化，并公开成员函数以设置Employee的名称：

```C++
#include <iostream>
#include <string>
#include <string_view>

class Employee // 成员默认是 private
{
    std::string m_name{};
    char m_firstInitial{};

public:
    void setName(std::string_view name)
    {
        m_name = name;
        m_firstInitial = name.front(); // 使用 std::string::front() 获取 `name` 的第一个字母
    }

    void print() const
    {
        std::cout << "Employee " << m_name << " has first initial " << m_firstInitial << '\n';
    }
};

int main()
{
    Employee e{};
    e.setName("John");
    e.print();

    e.setName("Mark");
    e.print();

    return 0;
}
```



### 14.8.6 数据隐藏允许更好地进行错误检测（和处理）

在上面的程序中，m_firstInitial必须与m_name的第一个字符匹配。因为m_firstInitial独立于m_name存在，通过将数据成员m_firstInitial替换为返回第一个初始值的成员函数，可以删除该变量：

```C++
#include <iostream>
#include <string>

class Employee
{
    std::string m_name{ "John" };

public:
    void setName(std::string_view name)
    {
        m_name = name;
    }

    // 使用 std::string::front() 获取 `m_name` 的首字母
    char firstInitial() const { return m_name.front(); }

    void print() const
    {
        std::cout << "Employee " << m_name << " has first initial " << firstInitial() << '\n';
    }
};

int main()
{
    Employee e{}; // 默认初始化为 "John"
    e.setName("Mark");
    e.print();

    return 0;
}
```

如果将m_name设置为空字符串，则不会立即发生任何错误。但如果随后调用firstInitial()，则std::string的front()成员将尝试获取空字符串的第一个字母，这将导致未定义的行为。

理想情况下，希望防止m_name为空。

如果用户对m_name成员具有公共访问权限，他们可以自行设置m_name = “"，我们无法防止这种情况发生。

然而，因为我们强制用户通过公共接口函数 setName() 设置m_name，所以可以让 setName() 验证用户是否传入了有效的名称。如果名称不为空，则可以将其分配给m_name。如果名称是空字符串，可以做许多事情来响应：

1. 忽略请求
2. assert检查
3. 抛出异常
4. 等等



### 14.8.7 数据隐藏使得可以在不破坏现有程序的情况下更改实现细节

考虑这个简单的例子：

```C++
#include <iostream>

struct Something
{
    int value1 {};
    int value2 {};
    int value3 {};
};

int main()
{
    Something something;
    something.value1 = 5;
    std::cout << something.value1 << '\n';
}
```

虽然该程序工作良好，但如果决定更改类的实现细节，如下所示，会发生什么情况？

```C++
#include <iostream>

struct Something
{
    int value[3] {}; // 使用长度为3的数组
};

int main()
{
    Something something;
    something.value1 = 5;
    std::cout << something.value1 << '\n';
}
```

该程序不再能够编译，因为名为value1的成员不再存在，并且main() 中的语句仍在使用该标识符。

数据隐藏使我们能够在不破坏使用类的程序的情况下更改类的实现方式。

下面是使用函数访问m_value1的封装版本：

```C++
#include <iostream>

class Something
{
private:
    int m_value1 {};
    int m_value2 {};
    int m_value3 {};

public:
    void setValue1(int value) { m_value1 = value; }
    int getValue1() const { return m_value1; }
};

int main()
{
    Something something;
    something.setValue1(5);
    std::cout << something.getValue1() << '\n';
}
```

现在，将类的实现改回数组：

```C++
#include <iostream>

class Something
{
private:
    int m_value[3]; // 注: 更改了类的实现方式!

public:
    // 更新函数的实现，以匹配对应的成员变量
    void setValue1(int value) { m_value[0] = value; }
    int getValue1() const { return m_value[0]; }
};

int main()
{
    // 使用到类的地方不用做改动
    Something something;
    something.setValue1(5);
    std::cout << something.getValue1() << '\n';
}
```



### 14.8.8 具有接口的类更容易调试

最后，封装可以帮助您在出现问题时调试程序更容易。通常，当程序不能正常工作时，是因为一个成员变量被赋予了不正确的值。如果每个人都能够直接设置成员变量，那么跟踪哪段代码实际将成员变量修改为错误的值可能会很困难。这可能涉及断点监控修改成员变量的每个语句——有许多这样的语句。

然而，如果成员只能通过单个成员函数更改，那么可以简单地中断该单个函数，并观察每个调用方更改值。这可以更容易地确定谁是罪魁祸首。



### 14.8.9 优先使用非成员函数

在C++中，如果函数可以实现为非成员函数，请考虑将其实现为非成员函数，而不是成员函数。

这有许多好处：

1. 非成员函数不是类接口的一部分。因此，类的接口将更小、更直观，使类更容易理解。
2. 非成员函数强制封装，因为这样的函数必须通过类的公共接口工作。不存在仅仅因为方便而直接访问成员变量的诱惑。
3. 在更改类的实现时，不需要考虑非成员函数（只要接口没有以不兼容的方式更改）。
4. 非成员函数往往更容易调试。
5. 包含特定于应用程序的数据和逻辑的非成员函数可以与类的可重用部分分离。

```C++
#include <iostream>
#include <string>

class Yogurt
{
    std::string m_flavor{ "vanilla" };

public:
    void setFlavor(std::string_view flavor)
    {
        m_flavor = flavor;
    }

    std::string_view getFlavor() const { return m_flavor; }
};

// 最佳: 非成员函数 print() 不是类接口的一部分
void print(const Yogurt& y)
{
        std::cout << "The yogurt has flavor " << y.getFlavor() << '\n';
}

int main()
{
    Yogurt y{};
    y.setFlavor("cherry");
    print(y);

    return 0;
}
```

以上版本是最好的。print() 现在是一个非成员函数。它不直接访问任何成员。即使类成员发生更改，也不需要更改print。此外，每个应用程序都可以提供自己的print() 函数，可以自行控制打印方式。



### 14.8.10 类成员声明的顺序

在类之外编写代码时，需要声明变量和函数，然后才能使用它们。然而，在类内部，这种限制不存在。可以按自己喜欢的顺序排列成员。

那么应该如何排列它们呢？

这里有两个流派：

1. 首先列出私有成员，然后列出公共成员函数。这遵循了使用前声明的传统风格。任何查看您的类代码的人都会看到您在使用数据成员之前是如何定义它们的，这可以使阅读和理解实现细节变得更加容易。
2. 首先列出您的公共成员，并将您的私人成员放在底部。因为使用您的类的人对公共接口感兴趣，所以将您的公共成员放在首位会使他们需要的信息放在最前面，并将实现细节（最不重要的）放在最后。

在现代C++中，更通常推荐第二种方法（公共成员优先），特别与其他开发人员共享的代码。



## 14.9 构造函数简介

当类类型是聚合时，可以使用聚合初始化：

```C++
struct Foo // Foo 是聚合
{
    int x {};
    int y {};
};

int main()
{
    Foo foo { 6, 7 }; // 使用聚合初始化

    return 0;
}
```

聚合初始化执行成员级初始化（成员按定义顺序初始化）。因此，当在上面的示例中实例化foo时，foo.x被初始化为6，而foo.y被初始化为7。

然而，一旦将任何成员变量设置为私有（以隐藏数据），类类型就不再是聚合（因为聚合不能有私有成员）。这意味着不再能够使用聚合初始化：

```C++
class Foo // Foo 不再是聚合 (有私有成员)
{
    int m_x {};
    int m_y {};
};

int main()
{
    Foo foo { 6, 7 }; // 编译失败: 不能使用聚合初始化

    return 0;
}
```

由于以下几个原因，不允许通过聚合初始化来初始化具有私有成员的类类型：

1. 聚合初始化需要知道类的实现（因为必须知道成员是什么，以及它们的定义顺序），这是在隐藏数据成员时有意避免的。
2. 如果类具有某种不变量，将依赖于用户以保证不变量的方式初始化类。

那么，如何初始化有私有成员变量的类呢？编译器为上例给出的错误消息提供了一条线索：“error:no matching constructor for initialization of’Foo’”

我们必须需要一个匹配的构造函数。但那到底是什么？



### 14.9.1 构造函数（constructor）

构造函数是一个特殊的成员函数，在创建非聚合类类型对象后会被自动调用。

定义非聚合类类型对象时，编译器会查看是否可以找到与调用方提供的初始化值（如果有）匹配的可访问构造函数。

1. 如果找到可访问的匹配构造函数，则为对象分配内存，然后调用构造函数。
2. 如果找不到可访问的匹配构造函数，则将提示编译错误。

除了确定如何创建对象之外，构造函数通常还执行两个功能：

1. 它们通常执行成员变量的初始化（通过成员初始化列表）
2. 它们可以执行其他设置功能（通过构造函数主体中的语句）。这可能包括检查初始化值、打开文件或数据库等…

在构造函数完成执行后，我们说对象已经“构造”好，对象现在应该处于一致的可用状态。

请注意，聚合不允许具有构造函数——因此，如果将构造函数添加到聚合中，它就不再是聚合。



### 14.9.2 构造函数的命名

与普通成员函数不同，构造函数有特定的命名规则：

1. 构造函数必须与类具有相同的名称（具有相同的大小写）。对于模板类，此名称不包括模板参数。
2. 构造函数没有返回类型。

由于构造函数通常是类接口的一部分，因此它们通常是public的。



### 14.9.3 基本构造函数示例

```C++
#include <iostream>

class Foo
{
private:
    int m_x {};
    int m_y {};

public:
    Foo(int x, int y) // 有两个初始值设定项的构造函数
    {
        std::cout << "Foo(" << x << ", " << y << ") constructed\n";
    }

    void print() const
    {
        std::cout << "Foo(" << m_x << ", " << m_y << ")\n";
    }
};

int main()
{
    Foo foo{ 6, 7 }; // 调用 Foo(int, int) 构造函数
    foo.print();

    return 0;
}
```

当编译器看到定义Foo Foo{6,7}时，它会查找将接受两个int参数的匹配Foo构造函数。Foo(int，int)是匹配项，因此编译器将允许定义。

在运行时，当foo被实例化时，为foo分配内存，然后调用Foo(int，int)构造函数，参数x为6，参数y为7。然后，构造函数的主体执行并打印 “Foo(6, 7) constructed”。

当调用print()成员函数时，成员m_x和m_y的值为0。这是因为尽管调用了Foo(int，int)构造函数，但它并没有实际初始化成员。



### 14.9.4 构造函数参数的隐式转换

在隐式类型转换中，可以注意到编译器将在函数调用中执行参数的隐式转换（如果需要），以匹配参数为不同类型的函数定义：

```C++
void foo(int, int)
{
}

int main()
{
    foo('a', true); // 可以匹配 foo(int, int)

    return 0;
}
```

对于构造函数来说，这没有什么不同：Foo(int，int) 构造函数将匹配隐式可转换为int的任何调用：

```C++
class Foo
{
public:
    Foo(int x, int y)
    {
    }
};

int main()
{
    Foo foo{ 'a', true }; // 可以匹配 Foo(int, int) 构造函数

    return 0;
}
```



### 14.9.5 构造函数不应为const

构造函数需要能够初始化正在构造的对象——因此，构造函数不能是const。

```C++
#include <iostream>

class Something
{
private:
    int m_x{};

public:
    Something() // 构造函数不应为const
    {
        m_x = 5; // 在 non-const 构造函数才能修改成员变量
    }

    int getX() const { return m_x; } // const
};

int main()
{
    const Something s{}; // const 对象, 隐式调用 (non-const) 构造函数

    std::cout << s.getX(); // 打印 5
    
    return 0;
}
```

通常，不能在常量对象上调用非const成员函数。然而，由于构造函数是隐式调用的，因此可以在常量对象上调用非const构造函数。



### 14.9.6 构造函数 vs Setter访问函数

构造函数被设计为在实例化点初始化整个对象。Setter访问函数旨在将值分配给现有对象的单个成员。



## 14.10 构造函数成员初始化列表

### 14.10.1 通过成员初始化列表进行成员初始化

为了让构造函数初始化成员，可以使用成员初始化列表。不要将其与类似名称的“初始值设定项列表”混淆，那是用于值列表初始化聚合。

在下面的示例中，Foo(int，int) 构造函数已更新为使用成员初始化列表来初始化m_x和m_y：

```C++
#include <iostream>

class Foo
{
private:
    int m_x {};
    int m_y {};

public:
    Foo(int x, int y)
        : m_x { x }, m_y { y } // 这里是 成员初始化列表
    {
        std::cout << "Foo(" << x << ", " << y << ") constructed\n";
    }

    void print() const
    {
        std::cout << "Foo(" << m_x << ", " << m_y << ")\n";
    }
};

int main()
{
    Foo foo{ 6, 7 };
    foo.print();

    return 0;
}
```

成员初始化列表在构造函数参数之后定义。它以冒号（：）开头，然后列出要初始化的每个成员以及该变量的初始化值，用逗号分隔。这里必须使用直接形式的初始化（最好使用大括号，但括号也可以）——这里不能使用复制初始化（等号）。还要注意，成员初始化列表不会以分号结尾。

实例化foo时，使用指定的值来设置初始化列表中的成员。在这种情况下，将m_x初始化为x的值（6），将m_y初始化为y的值（7）。然后运行构造函数的主体。

当调用print()成员函数时，可以看到m_x仍然具有值6，m_y仍然具有值7。



### 14.10.2 成员初始化列表的缩进格式

以下样式都有效（在实践中您可能会看到这三种样式）：

```C++
    Foo(int x, int y) : m_x { x }, m_y { y }
    {
    }
```

```C++
    Foo(int x, int y) :
        m_x { x },
        m_y { y }
    {
    }
```

```C++
  	Foo(int x, int y)
        : m_x { x }
        , m_y { y }
    {
    }
```

我们建议使用上面的第三种样式：

1. 将冒号放在构造函数名称后面的行上，因为这将成员初始化列表与函数原型清晰地分开。
2. 缩进成员初始化列表，以便更容易看到函数名。

如果成员初始化列表简短/琐碎，则所有初始值设定项都可以放在一行上，否则（或者如果愿意），每个成员和初始值设定项对可以放在单独的一行上（以逗号开头以保持对齐）。



### 14.10.3 成员初始化顺序

所有成员初始化列表中的成员总是按照它们在类中定义的顺序进行初始化（而不是按照列表中定义的次序）。

在上面的示例中，由于m_x在类定义中定义在m_y之前，因此m_x将首先初始化（即使它没有在列表中首先列出）。

因为我们直观地期望从左到右初始化变量，这可能会导致发生细微的错误。考虑以下示例：

```C++
#include <algorithm> // for std::max
#include <iostream>

class Foo
{
private:
    int m_x{};
    int m_y{};

public:
    Foo(int x, int y)
        : m_y{ std::max(x, y) }, m_x{ m_y } // 这一行有问题
    {
    }

    void print() const
    {
        std::cout << "Foo(" << m_x << ", " << m_y << ")\n";
    }
};

int main()
{
    Foo foo{ 6, 7 };
    foo.print();

    return 0;
}
```

在上面的示例中，意图是计算传入的初始化值中较大的一个（通过std::max（x，y）），然后使用该值初始化m_x和m_y。尽管m_y在成员初始化列表中列在第一位，但由于m_x是在类中首先定义的，因此m_x首先被初始化。m_x被初始化为m_y的值，该值尚未初始化。最后，m_y被初始化为较大的初始化值。

为了帮助防止这种错误，成员初始化列表中的成员应该按照它们在类中定义的顺序列出。如果成员的初始化顺序不正确，某些编译器将发出警告。

最好避免使用其他成员的值初始化成员变量（如果可能）。这样，即使您确实在初始化顺序中出错，也不重要，因为初始化值之间没有依赖关系。



### 14.10.4 成员初始化列表与默认成员初始值设置项

可以用几种不同的方法初始化成员：

1. 如果成员在初始化列表中列出，则使用该初始化值
2. 否则，如果成员具有默认的初始值设定项，则使用该初始化值
3. 否则，该成员将默认初始化。

这意味着，如果成员既有默认的初始值设定项，又列在构造函数的成员初始化列表中，则列表中的值优先。

下面是一个显示所有三种初始化方法的示例：

```C++
#include <iostream>

class Foo
{
private:
    int m_x{};    // 默认成员初始值 (会被忽略)
    int m_y{ 2 }; // 默认成员初始值 (被使用)
    int m_z;      // 无默认值

public:
    Foo(int x)
        : m_x{ x } // 成员初始化列表
    {
        std::cout << "Foo constructed\n";
    }

    void print() const
    {
        std::cout << "Foo(" << m_x << ", " << m_y << ", " << m_z << ")\n";
    }
};

int main()
{
    Foo foo{ 6 };
    foo.print();

    return 0;
}
```

下面是正在发生的事情。构造foo时，初始化列表中只有m_x，因此m_x首先初始化为6。m_y不在初始化列表中，但它具有默认值，因此它被初始化为2。m_z既不在成员初始化列表中，也没有默认值，因此它是默认初始化的（对于基本类型，这意味着它未被初始化）。因此，当打印m_z的值时，得到未定义的行为。



### 14.10.5 构造函数体

构造函数的主体通常为空。这是因为我们主要使用构造函数进行初始化，这是通过成员初始化列表完成的。如果这就是需要做的所有事情，那么不需要在构造函数的主体中使用任何语句。

构造函数体中的语句在成员初始化列表执行后执行，因此可以添加语句来执行所需的任何其他设置任务。在上面的示例中，将一些内容打印到控制台，以显示构造函数已执行，也可以执行其他操作，如打开文件或数据库、分配内存等…

新程序员有时使用构造函数的主体将值分配给成员：

```C++
#include <iostream>

class Foo
{
private:
    int m_x{};
    int m_y{};

public:
    Foo(int x, int y)
    {
        m_x = x; // 这是赋值，而不是初始化
        m_y = y; // 这是赋值，而不是初始化
    }

    void print() const
    {
        std::cout << "Foo(" << m_x << ", " << m_y << ")\n";
    }
};

int main()
{
    Foo foo{ 6, 7 };
    foo.print();

    return 0;
}
```

尽管在这种简单的情况下，这将产生预期的结果，但在需要初始化成员的情况下（例如，属性为常量或引用的数据成员），赋值将不起作用。



## 14.11 默认构造函数和默认参数

默认构造函数是不接受参数的构造函数。通常，这是一个没有参数定义的构造函数。

下面是具有默认构造函数的类的示例：

```C++
#include <iostream>

class Foo
{
public:
    Foo() // 默认构造函数
    {
        std::cout << "Foo default constructed\n";
    }
};

int main()
{
    Foo foo{}; // 未设置初始参数, 调用 Foo 的默认构造函数

    return 0;
}
```



### 14.11.1 类类型的值初始化与默认初始化

如果类类型具有默认构造函数，则值初始化和默认初始化都将调用默认构造函数。因此，对于上述示例中的Foo类这样的类，以下内容本质上是等效的：

```C++
    Foo foo{}; // 值初始化, 调用 Foo 的默认构造函数
    Foo foo2;  // 默认初始化, 调用 Foo 的默认构造函数
```

然而，正如之前讲解（默认成员初始化）中所述，值初始化对于聚合更安全。由于很难区分类类型是聚合还是非聚合，因此推荐对所有类类型使用值初始化。



### 14.11.2 具有默认参数的构造函数

与所有函数一样，构造函数的最右侧参数可以具有默认参数。

例如：

```C++
#include <iostream>

class Foo
{
private:
    int m_x { };
    int m_y { };

public:
    Foo(int x=0, int y=0) // 有默认参数
        : m_x { x }
        , m_y { y }
    {
        std::cout << "Foo(" << m_x << ", " << m_y << ") constructed\n";
    }
};

int main()
{
    Foo foo1{};     // 调用 Foo(int, int) 使用默认参数
    Foo foo2{6, 7}; // 调用 Foo(int, int)

    return 0;
}
```

如果构造函数中的所有参数都有默认值，则构造函数是默认构造函数（因为它可以在没有参数的情况下调用）。



### 14.11.3 构造函数重载

由于构造函数是函数，因此可以重载它们。也就是说，可以有多个构造函数，以便可以按不同的方式构造对象：

```C++
#include <iostream>

class Foo
{
private:
    int m_x {};
    int m_y {};

public:
    Foo() // 默认构造函数
    {
        std::cout << "Foo constructed\n";
    }

    Foo(int x, int y) // 普通构造函数
        : m_x { x }, m_y { y }
    {
        std::cout << "Foo(" << m_x << ", " << m_y << ") constructed\n";
    }
};

int main()
{
    Foo foo1{};     // 调用 Foo()
    Foo foo2{6, 7}; // 调用 Foo(int, int)

    return 0;
}
```

上面的一个推论是，一个类应该只有一个默认构造函数。如果提供了多个默认构造函数，编译器将无法知道应使用哪个构造函数：

```C++
#include <iostream>

class Foo
{
private:
    int m_x {};
    int m_y {};

public:
    Foo() // 默认
    {
        std::cout << "Foo constructed\n";
    }

    Foo(int x=1, int y=2) // 默认
        : m_x { x }, m_y { y }
    {
        std::cout << "Foo(" << m_x << ", " << m_y << ") constructed\n";
    }
};

int main()
{
    Foo foo{}; // 编译失败: 不知道该调用哪个构造函数

    return 0;
}
```

在上面的示例中，实例化没有参数的foo，因此编译器将查找默认构造函数。将找到两个，这是有歧义的。将导致编译错误。



### 14.11.4 隐式默认构造函数

如果非聚合类类型对象没有用户声明的构造函数，编译器将生成public默认构造函数（以便类可以是值初始化的或默认初始化的）。此构造函数称为隐式默认构造函数。

考虑以下示例：

```C++
#include <iostream>

class Foo
{
private:
    int m_x{};
    int m_y{};

    // 注: 未声明构造函数
};

int main()
{
    Foo foo{};

    return 0;
}
```

该类没有用户声明的构造函数，因此编译器将为生成隐式默认构造函数。该构造函数将用于实例化foo{}。

隐式默认构造函数等效于在构造函数体中没有参数、没有成员初始化列表和语句的构造函数。换句话说，对于上面的Foo类，编译器生成：

```C++
public:
    Foo() // 隐式生成的默认构造函数
    {
    }
```

当类没有数据成员时，隐式默认构造函数最有用。如果类具有数据成员，则可能希望使用用户提供的值来初始化它们，而隐式默认构造函数不足以实现这一点。



### 14.11.5 使用=default生成显式默认构造函数

在编写等效于隐式生成的默认构造函数时，可以告诉编译器为我们生成默认构造函数。该构造函数称为显式默认构造函数，可以通过使用=default语法来生成：

```C++
#include <iostream>

class Foo
{
private:
    int m_x {};
    int m_y {};

public:
    Foo() = default; // 生成显式默认构造函数

    Foo(int x, int y)
        : m_x { x }, m_y { y }
    {
        std::cout << "Foo(" << m_x << ", " << m_y << ") constructed\n";
    }
};

int main()
{
    Foo foo{}; // 调用 Foo() 默认构造函数

    return 0;
}
```

在上面的示例中，由于有一个用户声明的构造函数 Foo(int, int)，因此通常不会生成隐式默认构造函数。然而，因为已经告诉编译器生成这样的构造函数，所以会生成。该构造函数随后将由foo{}实例化使用。



### 14.11.6 显式默认构造函数与空的用户定义构造函数

至少有两种情况下，显式默认构造函数的行为与空的用户定义构造函数不同。

```C++
#include <iostream>

class User
{
private:
    int m_a; // 注: 无默认值
    int m_b {};

public:
    User() {} // 用户定义空的构造函数

    int a() const { return m_a; }
    int b() const { return m_b; }
};

class Default
{
private:
    int m_a; // 注: 无默认值
    int m_b {};

public:
    Default() = default; // 显式默认构造函数

    int a() const { return m_a; }
    int b() const { return m_b; }
};

class Implicit
{
private:
    int m_a; // 注: 无默认值
    int m_b {};

public:
    // 隐式默认构造函数

    int a() const { return m_a; }
    int b() const { return m_b; }
};

int main()
{
    User user{}; // 默认初始化
    std::cout << user.a() << ' ' << user.b() << '\n';

    Default def{}; // 零值初始化, 然后默认初始化
    std::cout << def.a() << ' ' << def.b() << '\n';

    Implicit imp{}; // 零值初始化, 然后默认初始化
    std::cout << imp.a() << ' ' << imp.b() << '\n';

    return 0;
}
```

请注意，在默认初始化之前，user.a不是零初始化的，因此未初始化。



### 14.11.7 仅在有意义时创建默认构造函数

默认构造函数，允许在业务不提供初始值时，创建非聚合类类型的对象。因此，当不需要用户提供初始值，类的默认值有意义时，才需要提供默认构造函数。

例如：

```C++
#include <iostream>

class Fraction
{
private:
    int m_numerator{ 0 };
    int m_denominator{ 1 };

public:
    Fraction() = default;
    Fraction(int numerator, int denominator)
        : m_numerator{ numerator }
        , m_denominator{ denominator }
    {
    }

    void print() const
    {
        std::cout << "Fraction(" << m_numerator << ", " << m_denominator << ")\n";
    }
};

int main()
{
    Fraction f1 {3, 5};
    f1.print();

    Fraction f2 {}; // 生成 Fraction 0/1
    f2.print();

    return 0;
}
```

对于表示分数的类，允许用户创建没有初始值设定项的fraction对象是有意义的（在这种情况下，用户将获得分数0/1）。

现在考虑这个类：

```C++
#include <iostream>
#include <string>
#include <string_view>

class Employee
{
private:
    std::string m_name{ };
    int m_id{ };

public:
    Employee(std::string_view name, int id)
        : m_name{ name }
        , m_id{ id }
    {
    }

    void print() const
    {
        std::cout << "Employee(" << m_name << ", " << m_id << ")\n";
    }
};

int main()
{
    Employee e1 { "Joe", 1 };
    e1.print();

    Employee e2 {}; // 编译失败: 无匹配的构造函数
    e2.print();

    return 0;
}
```

对于表示雇员的类，允许创建没有名字的雇员是没有意义的。因此，这样的类不应该具有默认构造函数，因此如果类的用户尝试这样做，将出现编译错误。



## 14.12 委托构造函数

考虑以下功能：

```C++
void A()
{
    // 完成任务 A 的语句
}

void B()
{
    // 完成任务 A 的语句
    // 完成任务 B 的语句
}
```

这两个函数都有一组执行完全相同的操作的语句（任务A）。在这种情况下，可以这样重构：

```C++
void A()
{
    // 完成任务 A 的语句
}

void B()
{
    A();
    // 完成任务 B 的语句
}
```

通过这种方式，删除了函数A() 和B() 中存在的冗余代码。这使得代码更容易维护，因为更改只需要在一个地方进行。

当一个类包含多个构造函数时，每个构造函数中的代码即使不相同，也很相似，并且有大量重复。类似地，希望在可能的情况下删除构造函数的冗余。

考虑以下示例：

```C++
#include <iostream>
#include <string>
#include <string_view>

class Employee
{
private:
    std::string m_name{};
    int m_id{ 0 };

public:
    Employee(std::string_view name)
        : m_name{ name }
    {
        std::cout << "Employee " << m_name << " created\n";
    }

    Employee(std::string_view name, int id)
        : m_name{ name }, m_id{ id }
    {
        std::cout << "Employee " << m_name << " created\n";
    }
};

int main()
{
    Employee e1{ "James" };
    Employee e2{ "Dave", 42 };
}
```

每个构造函数的主体打印相同的内容。

构造函数可以调用其他函数，包括类的其他成员函数。因此，可以这样重构：

```C++
#include <iostream>
#include <string>
#include <string_view>

class Employee
{
private:
    std::string m_name{};
    int m_id{ 0 };

    void printCreated() const
    {
        std::cout << "Employee " << m_name << " created\n";
    }

public:
    Employee(std::string_view name)
        : m_name{ name }
    {
        printCreated();
    }

    Employee(std::string_view name, int id)
        : m_name{ name }, m_id{ id }
    {
        printCreated();
    }
};

int main()
{
    Employee e1{ "James" };
    Employee e2{ "Dave", 42 };
}
```

虽然这比以前的版本更好，但它需要引入一个新函数，这并不理想。



### 14.12.1 明显的解决方案不起作用

类似于上面的示例中如何让函数B() 调用函数A() ，显而易见的解决方案是让一个Employee构造函数调用另一个构造函数。但这不会按预期工作，因为构造函数不是设计为,可以直接从另一个函数体（包括其他构造函数）调用的！

例如，您可能会认为尝试以下操作：

```C++
#include <iostream>
#include <string>
#include <string_view>

class Employee
{
private:
    std::string m_name{};
    int m_id{ 0 };

public:
    Employee(std::string_view name)
        : m_name{ name }
    {
        std::cout << "Employee " << m_name << " created\n";
    }

    Employee(std::string_view name, int id)
        : m_name{ name }, m_id{ id }
    {
        Employee(name); // 编译失败
    }
};

int main()
{
    Employee e1{ "James" };
    Employee e2{ "Dave", 42 };
}
```

这不起作用，并将导致编译错误。

当试图在没有任何参数的情况下显式调用构造函数时，会发生更危险的情况。这不会对默认构造函数执行函数调用——相反，它会创建一个临时（未命名）对象，并对其进行值初始化！下面是一个愚蠢的示例：

```C++
#include <iostream>
struct Foo
{
    int x{};
    int y{};

public:
    Foo()
    {
        x = 5;
    }

    Foo(int value): y { value }
    {
        // 期望: 调用 Foo() 函数
        // 实际: 值初始化了一个未命名的 Foo 临时对象 (马上又会被销毁)
        Foo(); // 注: 等价于 Foo{}
    }
};

int main()
{
    Foo f{ 9 };
    std::cout << f.x << ' ' << f.y; // 打印 0 9
}
```

在本例中，Foo(int) 构造函数里有语句Foo() ，期望调用Foo() 构造函数并将值5分配给成员x。然而，该语法实际上创建了一个未命名的临时Foo，然后对其进行值初始化（就像编写了Foo{}一样）。执行 x = 5语句时，为临时Foo的x成员分配一个值。由于未使用临时对象，因此一旦完成构造，就会丢弃它。

Foo(int) 构造函数的隐式对象的x成员从未被赋值。因此，当稍后在main（）中打印出它的值时，得到的是0，而不是预期的5。

注意，这种情况不会生成编译错误——相反，它只是默默地无法产生预期的结果！



### 14.12.2 委托构造函数

允许构造函数将初始化委托（转移责任）给同一类类型的另一个构造函数。这个过程有时被称为构造函数链接，这样的构造函数被称为委托构造函数。

要将一个构造函数的初始化委托给另一个构造函数，只需将另一个构造函数放在成员初始化列表里。应用于我们上面的示例：

```C++
#include <iostream>
#include <string>
#include <string_view>

class Employee
{
private:
    std::string m_name{};
    int m_id{ 0 };

public:
    Employee(std::string_view name)
        : Employee{ name, 0 } // 委托给 Employee(std::string_view, int) 构造函数
    {
    }

    Employee(std::string_view name, int id)
        : m_name{ name }, m_id{ id } // 实际初始化成员
    {
        std::cout << "Employee " << m_name << " created\n";
    }

};

int main()
{
    Employee e1{ "James" };
    Employee e2{ "Dave", 42 };
}
```

初始化 e1{“James”} 时，将调用匹配的构造函数Employee(std::string_view) ，参数name设置为“James.”。该构造函数将初始化委托给其他构造函数，因此随后调用Employee(std::string_view, int) 。name（“James”）的值作为第一个参数传递，字面值0作为第二个参数传递。然后运行委托构造函数的主体执行cout语句。然后，返回到初始构造函数，其（空）主体运行。最后，控制权返回给调用者。

这种方法的缺点是，它有时需要重复初始化值。在对Employee(std::string_view, int) 构造函数的委托中，需要int参数的初始化值。必须硬编码字面值0，因为没有办法引用默认的成员初始值设定项。

关于委托构造函数的一些附加说明。首先，委托给另一个构造函数的构造函数不允许自己进行任何成员初始化。构造函数可以委托或执行初始化，但不能同时委托和初始化。

其次，一个构造函数可以委托给另一个构造函数，后者又委托回第一个构造函数。这形成了一个无限循环，并将导致程序耗尽堆栈空间并崩溃。通过确保所有构造函数都解析为非委托构造函数，可以避免这种情况。



### 14.12.3 使用默认参数减少构造函数

默认值有时也可以用于将多个构造函数减少为较少的构造函数。例如，通过在id参数上放置默认值，可以创建单个Employee构造函数，该构造函数需要name参数，但可以选择接受id参数：

```C++
#include <iostream>
#include <string>
#include <string_view>

class Employee
{
private:
    std::string m_name{};
    int m_id{ 0 }; // 默认成员初始化

public:

    Employee(std::string_view name, int id = 0) // id 参数的默认值
        : m_name{ name }, m_id{ id }
    {
        std::cout << "Employee " << m_name << " created\n";
    }
};

int main()
{
    Employee e1{ "James" };
    Employee e2{ "Dave", 42 };
}
```

由于默认值必须附加到函数调用中最右侧的参数，因此定义类时的一个良好实践，是定义用户必须首先为其提供初始化值的成员（然后将这些参数设置为构造函数的最左侧参数）。有默认值的参数设置为构造函数的最右侧参数。

用户必须为其提供初始化值的成员应首先定义（并作为构造函数的最左侧参数）。用户可以选择为其提供初始化值（因为有默认值）的成员应其次定义（并作为构造函数的最右侧参数）。



## 14.13 临时类对象

考虑以下示例：

```C++
#include <iostream>

int add(int x, int y)
{
    int sum{ x + y }; // 将 x + y 的结果存储到sum中
    return sum;       // 返回sum的值
}

int main()
{
    std::cout << add(5, 3) << '\n';

    return 0;
}
```

在 add() 函数中，变量sum用于存储表达式x+y的结果。然后在return语句中对该变量求值，产生要返回的值。虽然这有时可能对调试有用（因此如果需要，可以检查sum的值），但它实际上通过定义一个对象（该对象随后仅使用一次）使函数比需要的更复杂。

在大多数情况下，变量只使用一次，实际上不需要这个变量。相反，可以直接将表达式放到需要变量的位置。下面是以这种方式重写的 add() 函数：

```C++
#include <iostream>

int add(int x, int y)
{
    return x + y; // 直接返回 x + y 计算的结果
}

int main()
{
    std::cout << add(5, 3) << '\n';

    return 0;
}
```

这不仅适用于返回值，也适用于大多数函数参数。例如：

```C++
#include <iostream>

void printValue(int value)
{
    std::cout << value;
}

int main()
{
    int sum{ 5 + 3 };
    printValue(sum);

    return 0;
}
```

可以改成这样：

```C++
#include <iostream>

void printValue(int value)
{
    std::cout << value;
}

int main()
{
    printValue(5 + 3);

    return 0;
}
```

请注意，这使代码干净了点。不需要定义变量并为其命名。不需要阅读整个函数来确定该变量是否实际用于其他地方。因为5+3是一个表达式，它只用于这一行。

请注意，这仅在接受右值表达式的情况下有效。在需要左值表达式的情况下，必须有一个对象：

```C++
#include <iostream>

void addOne(int& value) // 传递 non-const 引用需要左值输入
{
    ++value;
}

int main()
{
    int sum { 5 + 3 };
    addOne(sum);   // okay, sum 是左值

    addOne(5 + 3); // 编译错误: 5 + 3 不是左值

    return 0;
}
```



### 14.13.1 临时类对象

同样的问题也适用于类类型。

下面的示例类似于上面的示例，但使用程序定义的类类型IntPair而不是int：

```C++
#include <iostream>

class IntPair
{
private:
    int m_x{};
    int m_y{};

public:
    IntPair(int x, int y)
        : m_x { x }, m_y { y }
    {}

    int x() const { return m_x; }
    int y() const { return m_y; }
};

void print(IntPair p)
{
    std::cout << "(" << p.x() << ", " << p.y() << ")\n";        
}
        
int main()
{
    // 情况 1: 传递变量
    IntPair p { 3, 4 };
    print(p); // 打印 (3, 4)
    
    return 0;
}
```

在情况1中，实例化变量IntPair p，然后将p传递给函数print()。

然而，p只使用一次，函数print()将接受右值，因此实际上没有理由在这里定义变量。所以让我们去掉p。

可以通过传递临时对象而不是命名变量来实现这一点。临时对象（有时称为匿名对象或未命名对象）是没有名称且仅在单个表达式期间存在的对象。

有两种常用的方法来创建临时类类型对象：

```C++
#include <iostream>

class IntPair
{
private:
    int m_x{};
    int m_y{};

public:
    IntPair(int x, int y)
        : m_x { x }, m_y { y }
    {}

    int x() const { return m_x; }
    int y() const{ return m_y; }
};

void print(IntPair p)
{
    std::cout << "(" << p.x() << ", " << p.y() << ")\n";        
}
        
int main()
{
    // 情况 1: 传递变量
    IntPair p { 3, 4 };
    print(p);

    // 情况 2: 构造临时 IntPair 然后传递给函数
    print(IntPair { 5, 6 } );

    // 情况 3: 隐式将 { 7, 8 } 转换为 Intpair 然后传递给函数
    print( { 7, 8 } );
    
    return 0;
}
```

在案例2中，告诉编译器构造一个IntPair对象，并用{5,6}初始化它。因为该对象没有名称，所以它是临时的。然后将临时对象传递给函数 print() 的参数p。当函数调用返回时，临时对象被销毁。

在案例3中，也创建了一个临时IntPair对象，以传递给函数print()。然而，由于没有显式地指定要构造的类型，编译器将从函数参数中推断出必要的类型（IntPair），然后隐式地将{7,8}转换为IntPair对象。

总结如下：

```C++
IntPair p { 1, 2 }; // 创建命名对象 p 初始值为 { 1, 2 }
IntPair { 1, 2 };   // 创建临时对象 初始值为 { 1, 2 }
{ 1, 2 };           // 编译器尝试将 { 1, 2 } 转换问临时对象
```

可能更常见的是看到与返回值一起使用的临时对象：

```C++
#include <iostream>

class IntPair
{
private:
    int m_x{};
    int m_y{};

public:
    IntPair(int x, int y)
        : m_x { x }, m_y { y }
    {}

    int x() const { return m_x; }
    int y() const { return m_y; }
};

void print(IntPair p)
{
    std::cout << "(" << p.x() << ", " << p.y() << ")\n";        
}

// 情况 1: 创建命名对象并返回
IntPair ret1()
{
    IntPair p { 3, 4 };
    return p;
}

// 情况 2: 创建临时对象并返回
IntPair ret2()
{
    return IntPair { 5, 6 };
}

// 情况 3: 隐式将 { 7, 8 } 转换为 IntPair 并返回
IntPair ret3()
{
    return { 7, 8 };
}
     
int main()
{
    print(ret1());
    print(ret2());
    print(ret3());

    return 0;
}
```



## 14.14 拷贝构造函数简介

考虑以下程序：

```C++
#include <iostream>
 
class Fraction
{
private:
    int m_numerator{ 0 };
    int m_denominator{ 1 };
 
public:
    // 默认构造函数
    Fraction(int numerator=0, int denominator=1)
        : m_numerator{numerator}, m_denominator{denominator}
    {
    }

    void print() const
    {
        std::cout << "Fraction(" << m_numerator << ", " << m_denominator << ")\n";
    }
};

int main()
{
    Fraction f { 5, 3 };  // 调用 Fraction(int, int) 构造函数
    Fraction fCopy { f }; // 这里会调用什么构造函数?

    f.print();
    fCopy.print();

    return 0;
}
```

让我们仔细看看这个程序是如何工作的。

变量f的初始化是调用Fraction(int, int)构造函数的标准列表初始化。

但下一行呢？变量fCopy的初始化显然也是一种初始化，构造函数用于初始化类。那么这一行调用的是什么构造函数呢？

答案是：拷贝构造函数。



### 14.14.1 拷贝构造函数

拷贝构造函数，可以使用相同类型的现有对象来做初始化。在拷贝构造函数执行之后，新创建的对象应该是传入对象的副本。




### 14.14.2 隐式拷贝构造函数

如果不为类提供拷贝构造函数，C++将为您创建public隐式拷贝构造函数。在上面的示例中，语句Fraction fCopy{f}；调用隐式拷贝构造函数以使用f初始化fCopy。

默认情况下，隐式拷贝构造函数将执行成员级初始化。这意味着将使用传入的对象的相应成员来初始化每个成员。在上面的示例中，使用f.m_numerator（值为5）初始化fCopy.m_numelator，并使用f.m_denominator（值为3）初始化fCopy.m_denoginator。

执行拷贝构造函数后，f和fCopy的成员具有相同的值，因此fCopy是f的副本。因此，对两者调用print()具有相同的结果。



### 14.14.3 定义自己的拷贝构造函数

还可以显式定义自己的拷贝构造函数。在本课中，将使拷贝构造函数打印一条消息，以便可以向您展示在进行拷贝时它确实在执行。

拷贝构造函数看起来就像您期望的那样：

```C++
#include <iostream>
 
class Fraction
{
private:
    int m_numerator{ 0 };
    int m_denominator{ 1 };
 
public:
    // 默认构造函数
    Fraction(int numerator=0, int denominator=1)
        : m_numerator{numerator}, m_denominator{denominator}
    {
    }

    // 拷贝构造函数
    Fraction(const Fraction& fraction)
        // 使用传入对象的成员来初始化对应的成员变量
        : m_numerator{ fraction.m_numerator }
        , m_denominator{ fraction.m_denominator }
    {
        std::cout << "Copy constructor called\n"; // 这里打印，是为了证明本函数确实被执行了
    }

    void print() const
    {
        std::cout << "Fraction(" << m_numerator << ", " << m_denominator << ")\n";
    }
};

int main()
{
    Fraction f { 5, 3 };  // 调用 Fraction(int, int) 构造函数
    Fraction fCopy { f }; // 调用 Fraction(const Fraction&) 拷贝构造函数

    f.print();
    fCopy.print();

    return 0;
}
```

在上面定义的拷贝构造函数在功能上等同于默认情况下获得的构造函数，只是我们添加了一个输出语句来证明拷贝构造函数实际上被调用。当用f初始化fCopy时，调用该拷贝构造函数。

拷贝构造函数除了复制对象之外，不应执行任何其他操作。这是因为编译器在某些情况下可能会优化拷贝构造函数。如果您依赖拷贝构造函数来执行某些行为，而不仅仅是复制，则该行为可能会发生，也可能不会发生。我们在后续类初始化讲解中进一步讨论了这一点。

访问控制以每个类为基础（而不是以每个对象为基础）。这意味着类的成员函数可以访问同一类型的任何类对象的私有成员。

在上面的Fraction拷贝构造函数中，可以直接访问参数fraction的private成员。否则，将不得不为每个成员设置访问函数。



### 14.14.4 首选隐式拷贝构造函数

与隐式默认构造函数不做任何事情（因此很少是我们想要的）不同，隐式拷贝构造函数执行的成员级初始化通常正是我们想要做的。因此，在大多数情况下，使用隐式拷贝构造函数是完美的。

后面在讨论深拷贝与浅拷贝时，将看到在进行动态内存分配时需要覆盖拷贝构造函数的情况。



### 14.14.5 拷贝构造函数的参数必须是引用

拷贝构造函数的参数必须是左值引用或常量左值引用。由于拷贝构造函数不应修改参数，因此最好使用常量左值引用。



### 14.14.6 按值传递（和按值返回）与拷贝构造函数

当对象通过值传递时，它被复制到参数中。当它和参数是相同的类类型时，通过隐式调用拷贝构造函数来进行复制。类似地，当对象按值返回给调用方时，将隐式调用拷贝构造函数来进行复制。

可以在下面的示例中看到这两种情况：

```C++
#include <iostream>

class Fraction
{
private:
    int m_numerator{ 0 };
    int m_denominator{ 1 };

public:
    // 默认构造函数
    Fraction(int numerator = 0, int denominator = 1)
        : m_numerator{ numerator }, m_denominator{ denominator }
    {
    }

    // 拷贝构造函数
    Fraction(const Fraction& fraction)
        : m_numerator{ fraction.m_numerator }
        , m_denominator{ fraction.m_denominator }
    {
        std::cout << "Copy constructor called\n";
    }

    void print() const
    {
        std::cout << "Fraction(" << m_numerator << ", " << m_denominator << ")\n";
    }
};

void printFraction(Fraction f) // f 按值传递
{
    f.print();
}

Fraction generateFraction(int n, int d)
{
    Fraction f{ n, d };
    return f;
}

int main()
{
    Fraction f{ 5, 3 };

    printFraction(f); // f 按值复制到参数中，使用拷贝构造函数

    Fraction f2{ generateFraction(1, 2) }; // Fraction 按值返回，使用拷贝构造函数

    printFraction(f2); // f2 按值复制到参数中，使用拷贝构造函数

    return 0;
}
```

在上面的示例中，对printFraction(f) 的调用按值传递f。调用拷贝构造函数将f从main复制到函数printFraction的f参数中。

当generateFraction将Fraction返回到main时，将再次隐式调用拷贝构造函数。当将f2传递给printFraction时，第三次调用拷贝构造函数。

如果编译并执行上面的示例，您可能会发现只发生两次对拷贝构造函数的调用。这是一种称为拷贝省略（copy elision）的编译器优化。



### 14.14.7 使用 = default 生成默认拷贝构造函数

如果类没有拷贝构造函数，编译器将隐式为我们生成一个。如果愿意，可以使用=default语法显式请求编译器为我们创建默认的拷贝构造函数：

```C++
#include <iostream>
 
class Fraction
{
private:
    int m_numerator{ 0 };
    int m_denominator{ 1 };
 
public:
    // 默认构造函数
    Fraction(int numerator=0, int denominator=1)
        : m_numerator{numerator}, m_denominator{denominator}
    {
    }

    // 显示请求 默认的拷贝构造函数
    Fraction(const Fraction& fraction) = default;

    void print() const
    {
        std::cout << "Fraction(" << m_numerator << ", " << m_denominator << ")\n";
    }
};

int main()
{
    Fraction f { 5, 3 };
    Fraction fCopy { f };

    f.print();
    fCopy.print();

    return 0;
}
```



### 14.14.8 使用 = delete 以防止复制

有时会遇到这样的情况，即不希望某个类的对象是可复制的。可以通过使用「=delete」语法将拷贝构造函数标记为「删除」来防止这种情况：

```C++
#include <iostream>
 
class Fraction
{
private:
    int m_numerator{ 0 };
    int m_denominator{ 1 };
 
public:
    // 默认构造函数
    Fraction(int numerator=0, int denominator=1)
        : m_numerator{numerator}, m_denominator{denominator}
    {
    }

    // 删除拷贝构造函数，防止类的对象实例能被复制
    Fraction(const Fraction& fraction) = delete;

    void print() const
    {
        std::cout << "Fraction(" << m_numerator << ", " << m_denominator << ")\n";
    }
};

int main()
{
    Fraction f { 5, 3 };
    Fraction fCopy { f }; // 编译失败: 拷贝构造函数被删除了

    return 0;
}
```

在该示例中，当编译器查找构造函数以使用f初始化fCopy时，它将看到拷贝构造函数已被删除。这将导致它发出编译错误。

您还可以通过将拷贝构造函数设置为私有（因为私有函数不能被public调用）来防止public复制类对象。然而，私有副本构造函数仍然可以从类的其他成员调用，因此除非需要，否则不建议使用此解决方案。

有一个众所周知的C++原则，如果类需要用户定义的拷贝构造函数、析构函数或拷贝赋值运算符，那么它可能需要所有这三个。在C++11中，新增了两个，将移动构造函数和移动赋值运算符添加到列表中。

不遵循这个原则可能会导致代码出现故障。在讨论动态内存分配时，将重新讨论这些。



## 14.15 类初始化和拷贝省略

在前面，我们讨论了具有基本类型的对象的6种基本初始化类型：

```C++
int a;         // 无初始值 (默认初始化)
int b = 5;     // 等号后跟初始值 (拷贝初始化)
int c( 6 );    // 初始值在括号中 (直接初始化)

// 列表初始化 (C++11)
int d { 7 };   // 初始值在大括号中 (直接列表初始化)
int e = { 8 }; // 等号后跟大括号中的初始值 (拷贝列表初始化)
int f {};      // 空的大括号 (值初始化)
```

所有这些初始化方式对于具有类类型的对象都有效：

```C++
#include <iostream>

class Foo
{
public:
    
    // 默认构造函数
    Foo()
    {
        std::cout << "Foo()\n";
    }

    // 普通构造函数
    Foo(int x)
    {
        std::cout << "Foo(int) " << x << '\n';
    }

    // 拷贝构造函数
    Foo(const Foo&)
    {
        std::cout << "Foo(const Foo&)\n";
    }
};

int main()
{
    // 调用 Foo() 默认构造函数
    Foo f1;           // 默认构造函数
    Foo f2{};         // 值初始化 (优先使用)
    
    // 调用 foo(int) 普通构造函数
    Foo f3 = 3;       // 拷贝初始化 (非显式构造)
    Foo f4(4);        // 直接初始化
    Foo f5{ 5 };      // 直接列表初始化 (优化使用)
    Foo f6 = { 6 };   // 拷贝列表初始化 (非显式构造)

    // 调用 foo(const Foo&) 拷贝构造函数
    Foo f7 = f3;      // 拷贝初始化
    Foo f8(f3);       // 直接初始化
    Foo f9{ f3 };     // 直接列表初始化 (优化使用)
    Foo f10 = { f3 }; // 拷贝列表初始化

    return 0;
}
```

在现代C++中，拷贝初始化、直接初始化和列表初始化本质上做了相同的事情——它们初始化对象。

对于所有类型的初始化：

1. 初始化类类型时，将检查该类的构造函数集，并使用重载解析来确定最佳匹配的构造函数。这可能涉及参数的隐式转换。
2. 初始化非类类型时，隐式转换规则用于确定隐式转换是否存在。

还值得注意的是，在某些情况下，不允许某些形式的初始化（例如，在构造函数成员初始值设定项列表中，只能使用直接形式的初始化，而不能使用拷贝初始化）。

各种初始化之间有三个关键区别：

1. 列表初始化不允许窄化转换。
2. 拷贝初始化仅考虑非explict构造函数/转换函数。将在下一节进行介绍。
3. 列表初始化将匹配的列表构造函数优先于其他匹配的构造函数。将在后续进行讨论。



### 14.15.1 不必要的副本

考虑这个简单的程序：

```C++
#include <iostream>

class Something
{
    int m_x{};

public:
    Something(int x)
        : m_x{ x }
    {
        std::cout << "Normal constructor\n";
    }

    Something(const Something& s)
        : m_x { s.m_x }
    {
        std::cout << "Copy constructor\n";
    }

    void print() const { std::cout << "Something(" << m_x << ")\n"; }
};

int main()
{
    Something s { Something { 5 } }; // 注意这一行
    s.print();

    return 0;
}
```

在上面的变量s的初始化中，首先构造一个临时Something，用值5初始化（使用 Somethine(int) 构造函数）。然后使用该临时变量来初始化s。由于临时变量和s具有相同的类型（它们都是Something对象），因此通常会在此处调用 Somethine(const Something&) 拷贝构造函数，以将临时变量中的值复制到s中。最终结果是s用值5初始化。

然而，这个程序有些低效，因为不得不进行两个构造函数调用：一个是对 Something(int) 的调用，另一个是对于 Somethine(const Something&) 的调用。请注意，上面的最终需要的print结果与如下代码效果一样：

```c++
Something s { 5 }; // 只调用 Something(int), 不需要拷贝构造函数
```

这个版本产生相同的结果，但更有效，因为它只调用 Something(int) （不需要拷贝构造函数）。



### 14.15.2 拷贝省略

由于编译器可以自由地重写语句来优化它们，因此人们可能想知道编译器是否可以优化掉不必要的副本，并处理 Something s{Something{5}}；。

答案是肯定的，这样做的过程称为拷贝省略。拷贝省略是一种编译器优化技术，允许编译器删除不必要的对象复制。换句话说，在编译器通常调用拷贝构造函数的情况下，编译器可以自由重写代码，以避免调用拷贝构造函数。当编译器优化掉对拷贝构造函数的调用时，可以说拷贝构造函数已被省略。

与其他类型的优化不同，拷贝省略不受“仿佛”规则的约束。也就是说，即使拷贝构造函数有副作用（例如将文本打印到控制台），也允许拷贝省略来省略拷贝构造函数！这就是为什么拷贝构造函数不应该具有复制以外的副作用——如果编译器省略对拷贝构造函数的调用，副作用将不会执行，程序的可观察行为将改变！

可以在上面的例子中看到这一点。如果在C++17编译器上运行该程序，它将产生以下结果：

```C++
Normal constructor
Something(5)
```

编译器省略了拷贝构造函数以避免不必要的复制，因此，打印“Copy constructor”的语句不会执行！由于拷贝省略，程序的可观察行为发生了变化！



### 14.15.3 按值传递和按值返回中的拷贝省略

拷贝构造函数通常在参数按值传递或按值返回时调用。在某些情况下，这些副本可以内自动省略。以下程序演示了其中的一些情况：

```C++
#include <iostream>
class Something
{
public:
	Something() = default;
	Something(const Something&)
	{
		std::cout << "Copy constructor called\n";
	}
};

Something rvo()
{
	return Something{}; // 调用 Something() 和 拷贝构造
}

Something nrvo()
{
	Something s{}; // 调用 Something()
	return s;      // 调用 拷贝构造
}

int main()
{
	std::cout << "Initializing s1\n";
	Something s1 { rvo() }; // 调用 拷贝构造

	std::cout << "Initializing s2\n";
	Something s2 { nrvo() }; // 调用 拷贝构造

    return 0;
}
```

如果没有优化，上述程序将调用拷贝构造函数4次：

1. 当rvo将Something返回到main时。
2. 使用rvo()的返回值初始化s1时，执行一次。
3. nrvo返回s到main时一次。
4. 使用nrvo()的返回值初始化s2时一次。

然而，由于拷贝省略，编译器很可能会省略大多数或所有这些拷贝构造函数调用。Visual Studio 2022省略3种情况（它不省略nrvo()按值返回的情况），GCC省略所有4种情况。

记住编译器何时执行/不执行拷贝省略并不重要。只要知道这是编译器将在可能的情况下执行的优化，如果期望看到您的拷贝构造函数被调用，而它没有被调用，那么拷贝省略可能就是原因。



### 14.15.4 拷贝省略勘误表

在C++17之前，拷贝省略严格来说是编译器可以进行的可选优化。在C++17中，在某些情况下，拷贝省略是强制性的。

在可选的省略情况下，即使对拷贝构造函数的实际调用被省略，也必须有可访问的拷贝构造函数。

在强制省略情况下，可访问的拷贝构造函数不需要存在（换句话说，即使删除了拷贝构造函数，也可能发生强制省略）。



## 14.16 转换构造函数和explicit关键字

在前面，我们介绍了类型转换和隐式类型转化的概念，编译器将根据需要隐式地将一种类型的值转换为另一种类型（如果存在这样的转换方式）的值。

这允许这样做：

```C++
#include <iostream>

void printDouble(double d) // 参数为 double
{
    std::cout << d;
}

int main()
{
    printDouble(5); // 使用 int 来调用

    return 0;
}
```

在上面的示例中，printDouble函数有一个double参数，但传入了一个int类型的值。由于参数的类型和传入值的类型不匹配，编译器将查看是否可以隐式地将int值转换为double值。在这种情况下，使用数值转换规则，int值5将被转换为double值5.0，因为是通过值传递的，所以参数d将用该值进行拷贝初始化。



### 14.16.1 用户定义的转换

现在考虑以下类似的示例：

```C++
#include <iostream>

class Foo
{
private:
    int m_x{};
public:
    Foo(int x)
        : m_x{ x }
    {
    }

    int getX() const { return m_x; }
};

void printFoo(Foo f) // 参数为 Foo
{
    std::cout << f.getX();
}

int main()
{
    printFoo(5); // 输入为 int

    return 0;
}
```

在此版本中，printFoo有一个Foo参数，但传入了一个int类型的数据。由于类型不匹配，编译器将尝试将int值5隐式转换为Foo对象，以便可以调用函数。

上一个示例中的参数和传入数据的类型都是基本类型（因此可以使用内置的数值提升/转换规则进行转换），在当前情况下，参数类型是程序定义的类型。C++没有特定的规则来告诉编译器如何将基本类型的值转换为程序定义的类型。

相反，编译器将查看是否定义了一些函数，可以使用这些函数来执行这种转换。这样的函数称为用户定义的转换。



### 14.16.2 转换构造函数

在上面的示例中，编译器将找到一个函数，该函数允许它将int值5转换为Foo对象。该函数是 Foo(int) 构造函数。

到目前为止，通常使用构造函数来显式构造对象：

```c++
    Foo x { 5 }; // 显式将 int 值 5 转换为 Foo
```

考虑一下它的作用：提供一个int值（5），并获得一个Foo对象作为返回。

在函数调用的上下文中，试图解决相同的问题：

```c++
    printFoo(5); // 隐式将 int 值 5 转换为 Foo
```

提供了一个int值（5），并且希望返回一个Foo对象。Foo(int) 构造函数就是为此而设计的！

因此，在这种情况下，当调用 printFoo(5) 时，使用 Foo(int) 构造函数，并将5作为参数f！

可以用于执行隐式转换的构造函数称为转换构造函数。默认情况下，所有构造函数都是转换构造函数。



### 14.16.3 只能应用一次用户定义的转换

现在考虑以下示例：

```C++
#include <iostream>
#include <string>
#include <string_view>

class Employee
{
private:
    std::string m_name{};

public:
    Employee(std::string_view name)
        : m_name{ name }
    {
    }

    const std::string& getName() const { return m_name; }
};

void printEmployee(Employee e) // 参数为 Employee
{
    std::cout << e.getName();
}

int main()
{
    printEmployee("Joe"); // 提供的是一个字符串字面值

    return 0;
}
```

在这个版本中，将Foo类替换为Employee类。printEmployee有一个Employme参数，传入一个C样式的字符串文本。同时有一个转换构造函数：Employee(std::string_view)。

您可能会惊讶地发现，这个版本不能编译。原因很简单：只能应用一次用户定义的转换来执行隐式转换，而这个示例需要两次。首先，必须将C样式的字符串文本转换为std::string_view（使用std:∶string_view转换构造函数），然后必须将std::string_view转换为Employee（使用 Employer(std:；string_view) 转换构造函数）。

有两种方法可以使此示例工作：

```C++
int main()
{
    using namespace std::literals;
    printEmployee( "Joe"sv); // 现在是 std::string_view 字面值了

    return 0;
}
```

这是可行的，因为现在只需要一个用户定义的转换（从std::string_view到Employee）。

```C++
int main()
{
    printEmployee(Employee{ "Joe" });

    return 0;
}
```

这也可以工作，因为现在只需要一个用户定义的转换（从字符串文本到用于初始化Employee对象的std::string_view）。将显式构造的Employee对象传递给函数不需要进行第二次转换。

后一个示例提供了一种有用的技术：将隐式转换替换为显式定义。在本课后面的部分中，将看到更多的例子。



### 14.16.4 转换构造函数导致的预期外的行为

考虑以下程序：

```C++
#include <iostream>

class Dollars
{
private:
    int m_dollars{};

public:
    Dollars(int d)
        : m_dollars{ d }
    {
    }

    int getDollars() const { return m_dollars; }
};

void print(Dollars d)
{
    std::cout << "$" << d.getDollars();
}

int main()
{
    print(5);

    return 0;
}
```

当调用print(5) 时，Dollars(int) 转换构造函数将5转换为Dollars对象。

尽管这可能是代码编写者的意图，但很难说清楚。调用者完全有可能假设这将打印5，并且不期望编译器以静默和隐式的方式将int值转换为Dollars对象，以便可以满足此函数调用。

虽然这个例子很简单，但在一个更大、更复杂的程序中，编译器执行一些您没有预料到的隐式转换，从而在运行时导致意外行为，这很容易让人感到惊讶。

如果 print(Dollars) 函数只能用Dollars对象调用，而不是用可以隐式转换为Dollars的任何值（特别是像int这样的基本类型），那就更好了。这将减少意外错误的可能性。



### 14.16.5 explicit关键字

为了解决这样的问题，可以使用explicit关键字来告诉编译器，构造函数不应该用作转换构造函数。

使构造函数explicit有两个显著的后果：

1. explicit构造函数不能用于执行拷贝初始化或拷贝列表初始化。
2. explicit构造函数不能用于执行隐式转换（因为它使用拷贝初始化或拷贝列表初始化）。

将上例中的Dollars(int) 构造函数更新为explicit构造函数：

```C++
 #include <iostream>

class Dollars
{
private:
    int m_dollars{};

public:
    explicit Dollars(int d) // 现在是 explicit
        : m_dollars{ d }
    {
    }

    int getDollars() const { return m_dollars; }
};

void print(Dollars d)
{
    std::cout << "$" << d.getDollars();
}

int main()
{
    print(5); // 编译失败，因为 Dollars(int) 是 explicit

    return 0;
}
```

由于编译器不能再使用 Dollars(int) 作为转换构造函数，因此它无法找到将5转换为Dollars的方法。因此，它将生成编译错误。



### 14.16.6 explicit构造函数可用于直接初始化和列表初始化

explicit构造函数仍然可以用于直接和直接列表初始化：

```C++
// 假设为 Dollars(int) 是 explicit
int main()
{
    Dollars d1(5); // ok
    Dollars d2{5}; // ok
}
```

现在，回到前面的示例，在那里创建了 explicit Dollars(int) 构造函数，因此下面生成了一个编译错误：

```c++
    print(5); // 编译失败，因为 Dollars(int) 是 explicit
```

如果确实想用int值5调用print()，但构造函数是explicit的，该怎么办？解决方法很简单：可以自己显式定义Dollars对象，而不是让编译器将5隐式转换为可以传递给print()的Dollars：

```c++
    print(Dollars{5}); // ok: 创建 Dollars 并传给 print() (不再需要转换)
```

这是允许的，因为仍然可以使用显式构造函数来列表初始化对象。由于现在已经显式构造了一个Dollars，因此参数类型与传入数据类型匹配，因此不需要转换！

这不仅可以编译和运行，还可以更好地记录意图，因为它明确了打算用Dollars对象调用该函数的事实。



### 14.16.7 按值返回和explicit构造函数

当从函数中返回值时，如果该值与函数的返回类型不匹配，则将发生隐式转换。就像传递值一样，这种转换不能使用explicit构造函数。

以下程序显示了返回值的一些变化及其结果：

```C++
#include <iostream>

class Foo
{
public:
    explicit Foo() // explicit
    {
    }

    explicit Foo(int x) // explicit
    {
    }
};

Foo getFoo()
{
    // explicit Foo()
    return Foo{ };   // ok
    return { };      // 错误: 不能隐式的列表初始化Foo

    // explicit Foo(int)
    return 5;        // 错误: 不能隐式的将 int 转换为 Foo
    return Foo{ 5 }; // ok
    return { 5 };    // 错误: 不能隐式的将初始化列表转换为 Foo
}

int main()
{
    return 0;
}
```

也许令人惊讶的是，返回{5}被认为是转换。



### 14.16.8 使用explicit的最佳实践

现代最佳实践是使默认情况下接受单个参数的任何构造函数标记为explicit。也包括大多数或全部参数有默认值的有多个参数的构造函数。

这将禁止编译器使用该构造函数进行隐式转换。

如果在特定情况下实际需要这样的转换，则使用列表初始化将隐式转换转换改写为显式定义是很容易的。

以下构造函数不应标记explicit：

1. 复制（和移动）构造函数（因为它们不执行转换）。

以下构造函数通常不应标记explicit：

1. 没有参数的默认构造函数（因为它们仅用于将{}转换为默认对象）。
2. 仅接受多个参数的构造函数。

在某些情况下，将单参数构造函数设置为非explicit是有意义的。当以下所有条件都为真时，这可能很有用：

1. 转换的值在语义上等同于参数值。
2. 转换是性能更好的。

例如，接收C样式字符串，参数类型为std::string_view的构造函数不是explicit的，因为不太可能出现这样的情况，即我们不同意将C样式字符串视为std::string_view。

相反，接收std::string_view，参数类型std::string构造函数需要标记为explicit，因为虽然std:∶string值在语义上等同于std::string_view值，但构造std::string代价较高。