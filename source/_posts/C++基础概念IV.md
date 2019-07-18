---
title: C++基础概念IV
date: 2018-8-31 10:01:30
categories: 2018年8月
tags: [C++,sizeof]

---
 
重点探讨类的大小，使用sizeof

1.类的大小为类的非静态成员数据的类型大小之和，也就是说静态成员数据不作考虑。

2.普通成员函数与sizeof无关。

3.虚函数由于要维护在虚函数表，所以要占据一个指针大小，也就是4字节。

4.类的总大小也遵守类似class字节对齐的，调整规则。


<!-- more -->


## 1.空类

	#include<iostream>
	using namespace std;
	class A {

	};
	class B {
	public:
	    B() { };
	    ~B() {};
	};

	int main() {
	    cout << "sizeof A=" << sizeof(A) << endl;
	    cout << "sizeof B=" << sizeof(B) << endl;
	}

空类的大小都为1
   因为一个空类也要实例化，所谓类的实例化就是在内存中分配一块地址，每个实例在内存中都有独一无二的地址。同样空类也会被实例化，所以编译器会给空类隐含的添加一个字节，这样空类实例化之后就有了独一无二的地址了。所以空类的sizeof为1。

   而析构函数，跟构造函数这些成员函数，是跟sizeof无关的，也不难理解因为我们的sizeof是针对实例，而普通成员函数，是针对类体的，一个类的成员函数，多个实例也共用相同的函数指针，所以自然不能归为实例的大小

## 2.含有静态变量和虚析构函数的类

	class Base  
	{  
	public:  
	    Base();                  
	    virtual ~Base();         //每个实例都有虚函数表  
	    void set_num(int num)    //普通成员函数，为各实例公有，不归入sizeof统计  
	    {  
	        a=num;  
	    }  
	private:  
	    int  a;                  //占4字节  
	    char *p;                 //4字节指针  
	};  

	class Derive:public Base  
	{  
	public:  
	    Derive():Base(){};       
	    ~Derive(){};  
	private:  
	    static int st;         //非实例独占  
	    int  d;                     //占4字节  
	    char *p;                    //4字节指针  

	};  

	int main()   
	{   
	    cout<<sizeof(Base)<<endl;  
	    cout<<sizeof(Derive)<<endl;  
	    return 0;  
	}

Base类里的int  a;char *p;占8个字节。

而虚析构函数virtual ~Base();的指针占4子字节。

其他成员函数不归入sizeof统计。

Derive类首先要具有Base类的部分，也就是占12字节。

int  d;char *p;占8字节

static int st;不归入sizeof统计

所以一共是20字节。

## 3. 字节对齐原则

	class Derive:public Base  
	{  
	public:  
	    Derive():Base(){};  
	    ~Derive(){};  
	private:  
	    static int st;  
	    int  d;  
	    char *p;  
	    char c;  

	};  

一个char c;增加了4字节（char本身占1个字节），说明类的大小也遵守类似class字节对齐的补齐规则。
一般来说，普通继承的空间计算结果应当是sizeof(基类)+sizeof(派生类)，然后考虑对齐，内存空间必须是类中数据成员所占用最大空间的整数倍。不过这是一般情况，具体怎么算要看编译器，比如将上述代码中的两个数据成员交换位置，结果则可能不同。

## 4.含虚函数的子类普通继承含虚函数的父类

	class Test
	{
	public:
	    int a;
	    virtual void get(){}
	};
	class Test2 : public Test
	{
	public:
	    int a;
	    virtual void set(){}
	};

虽然子类和父类都包含虚函数， 但它们存放于同一个虚表中，因此只需要一个指针，在普通继承下，不管是子类还是父类包含虚函数，子类的**sizeof=sizeof(基类非静态数据成员)+sizeof(派生类非静态数据成员)+sizeof(指针)**。


## 5.子类虚继承父类(C++中虚拟继承的概念)


### 5.1.C++中虚拟继承的概念
为了解决从不同途径继承来的同名的数据成员在内存中有不同的拷贝造成数据不一致问题，将共同基类设置为虚基类。这时从不同的路径继承过来的同名数据成员在内存中就只有一个拷贝，同一个函数名也只有一个映射。这样不仅就解决了二义性问题，也节省了内存，避免了数据不一致的问题。

	class 派生类名：virtual 继承方式  基类名
virtual是关键字，声明该基类为派生类的虚基类。
在多继承情况下，虚基类关键字的作用范围和继承方式关键字相同，只对紧跟其后的基类起作用。
声明了虚基类之后，虚基类在进一步派生过程中始终和派生类一起，维护同一个基类子对象的拷贝。

执行顺序

	首先执行虚基类的构造函数，多个虚基类的构造函数按照被继承的顺序构造；

	执行基类的构造函数，多个基类的构造函数按照被继承的顺序构造；

	执行成员对象的构造函数，多个成员对象的构造函数按照申明的顺序构造；

	执行派生类自己的构造函数；

	析构以与构造相反的顺序执行；
### 5.2.子类虚继承父类
虚继承比较特别，一般的计算公式应当是**sizeof(子类)=sizeof(基类)+sizeof(虚表指针)+sizeof(子类非静态数据成员)
**

	class A
	{
	public:
		int a;
	};
	class B : virtual public A
	{
	public:
		int b;
	};

sizeof结果都是12，基类A大小为4，子类B数据成员为4，因为是虚继承，可能需要一个指向虚基类的指针，再占用4字节，得到12。

然而，对于包含虚函数的子类虚继承不含虚函数的父类时，结果发生了变化

	class A
	{
	public:
	    int a;
	};
	class B : virtual public A
	{
	public:
	    int a;
	    virtual void set(){}
	};
在Codeblocks下结果仍为12，VS2010下却为16。子类增加了一个虚函数，Codeblocks下结果不变，在VS下结果却发生了变化。

可以这样认为，在VS下，计算公式可能如下：**sizeof(子类)=sizeof(基类)+sizeof(指示虚基类的指针)+sizeof(子类非静态数据成员)+sizeof(虚表指针)**。在CB下可能对内存进行了优化（个人推测）

对于子类和父类中都含虚函数的虚继承，结果如下：

	class Test
	{
	public:
	    int a;
	    virtual void get(){}
	};
	class Test2 : virtual Test
	{
	public:
	    int a;
	    virtual void set(){}
	};

在Codeblocks下结果为16，VS2010下为20。
这是因为，使用了虚继承，子类和父类的虚函数存放在不同的虚表中，因此子类和父类都需要一个指向虚表的指针。

## 6.多重继承

	#include<iostream>
	using namespace std;
	class A
	{
	public:
	    int a;
	};
	class B : public A
	{
	public:
	    int b;
	};
	class C : public A
	{
	public:
	    int c;
	};
	class D : public B, public C
	{
	public:
	    int d;
	};

	int main() {
	    cout << "sizeof A=" << sizeof(A) << endl;
	    cout << "sizeof B=" << sizeof(B) << endl;
	    cout << "sizeof C=" << sizeof(C) << endl;
	    cout << "sizeof D=" << sizeof(D) << endl;
	}

就是基类的相加。**sizeof(D)=sizeof(B)+sizeof(C)+sizeof(子类非静态数据成员)**

	sizeof A=4
	sizeof B=8
	sizeof C=8
	sizeof D=20
	请按任意键继续. . .

## 7.多重虚继承

虚继承存在的意义就是为了减少内存开销和二义性，实现对象共享。

	class A
	{
	public:
		int a;
	};
	class B :  virtual public A
	{
	public:
		int b;
	};
	class C : virtual public A
	{
	public:
		int c;
	};
	class D : public B, public C
	{
	public:
		int d;
	};

sizeof(D)在CB和VS下都是24。D中包含a,b,c,d四个数据成员，还包含两个指向虚基类A的指针，这种情况下，VS和CB都没有将两个指针合为一个。这种情况也可以这样考虑，sizeof(D)=sizeof(B)+sizeof(C)+sizeof(子类非静态数据成员)，但由于是虚继承，虚基类A中数据成员a只需要保留一份，而我们算了两次，所以还需要减去A的数据成员，所以公式应当是sizeof(D)=sizeof(B)+sizeof(C)-sizeof(A的非静态数据成员)+sizeof(子类非静态数据成员)。


## 8.字节对齐


１:数据成员对齐规则：结构(struct)(或联合(union))的数据成员，第一个数据成员放在offset为0的地方，以后每个数据成员存储的起始位置要从该成员大小或者成员的子成员大小（只要该成员有子成员，比如说是数组，结构体等）的整数倍开始(比如int在32位机为４字节,则要从４的整数倍地址开始存储。



２:结构体作为成员:如果一个结构里有某些结构体成员,则结构体成员要从其内部最大元素大小的整数倍地址开始存储.(struct a里存有struct b,b里有char,int ,double等元素,那b应该从8的整数倍开始存储.)


３:收尾工作:结构体的总大小,也就是sizeof的结果,.必须是其内部最大成员的整数倍.不足的要补齐.

4:在代码前加一句#pragma pack(1),告诉编译器,所有的对齐都按照1的整数倍对齐,换句话说就是没有对齐规则.如果#pragma pack (n)中指定的n大于结构体中最大成员的size，则其不起作用，结构体仍然按照size最大的成员进行对界

## 9.struct结构体变量，union共用体变量，enum枚举变量的大小

struct结构体变量大小等于结构体中的各个成员变量所占内存大小总和，union共用体变量大小等于共用体结构中占用内存最大的成员的内存大小。

首先先明白概念：
数据类型，指固定内存大小的别名，如int类型为4个字节内存。
变量，一段连续存储空间的别名。这段连续存储空间的大小，即变量的大小，由定义该变量的数据类型决定，即该数据类型代表的固定内存大小。数据类型，是变量的模板。

应用到枚举上：
枚举类型，指一个被命名的整型常数的集合。即枚举类型，本质上是一组常数的集合体，只是这些常数有各自的命名。枚举类型，是一种用户自定义数据类型。
枚举变量，由枚举类型定义的变量。枚举变量的大小，即枚举类型所占内存的大小。由于枚举变量的赋值，一次只能存放枚举结构中的某个常数。所以枚举变量的大小，实质是常数所占内存空间的大小（常数为int类型，当前主流的编译器中一般是32位机器和64位机器中int型都是4个字节），枚举类型所占内存大小也是这样。
另外，可以用编译器内置的指示符sizeof，计算出枚举变量（或枚举类型）的大小进行验证。

	#include <iostream>
	using namespace std;
	typedef enum
	{
	    Char,
	    Short,
	    Int,
	    Double,
	    Float,
	}TEST_TYPE;
	int main() {
	    TEST_TYPE val;
	    cout << sizeof(val) << endl;

	    //验证例程
	    enum number {
	        one = 1,
	        two = 2,
	        three = 3,
	        four = 4
	    };
	    enum number num1;
	    cout << "sizeof 枚举结构 enum number =" << sizeof(enum number) << endl;
	    cout << "sizeof 定义后的枚举变量 variable num1=" << sizeof(num1) << endl;
	    num1 = three;
	    cout << "size of  赋值后的枚举变量 =" << sizeof(num1) << endl;
	    return 0;
	}

	输出为
	4
	sizeof 枚举结构 enum number =4
	sizeof 定义后的枚举变量 variable num1=4
	size of  赋值后的枚举变量 =4
	请按任意键继续. . .
