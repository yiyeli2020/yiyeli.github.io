---
title: C++基础概念III
date: 2018-08-21 09:58:44
categories: 2018年8月
tags: [C++,Virtual function]

---
 

总结C++基础概念

面向对象的语言有三大特性：继承、封装、多态。虚函数作为多态的实现方式，重要性毋庸置疑。并且作为一个C++程序员，每一次面试都会被问及虚函数的相关问题。下面我们就来讨论讨论虚函数。

 首先，什么是多态，什么又是虚函数呢？先来看看维基百科对多态的解释：

“多态（英语：polymorphism），是指计算机程序运行时，相同的消息可能会送给多个不同的类别之对象，而系统可依据对象所属类别，引发对应类别的方法，而有不同的行为。简单来说，所谓多态意指相同的消息给予不同的对象会引发不同的动作称之。”其实更简单地来说，就是“在用父类指针调用函数时，实际调用的是指针指向的实际类型（子类）的成员函数”。多态性使得程序调用的函数是在运行时动态确定的，而不是在编译时静态确定的。而虚函数则是加了virtual修饰词的类的成员函数。

<!-- more -->



## 1.虚函数与虚函数表的基本知识

每一个拥有virtual function的类实例化对象时，都会额外申请一块内存存储虚函数表存储所有虚函数地址，并在对象某个位置存储一个vptr指针指向该表起始地址。这个指针具体放在什么位置，虚函数表怎么组织，怎么索引各个虚函数，这些都是编译器在编译期间决定的，在不同编译环境下不见得相同。

虚函数在c++中的实现机制就是用虚表和虚指针，但是具体是怎样的呢？从more effecive c++其中一篇文章里面可以知道：是每个类用了一个虚表，每个类的对象用了一个虚指针。

	class A
	{
	public:
	    virtual void f();
	    virtual void g();
	private:
	    int a
	};

	class B : public A
	{
	public:
	    void g();
	private:
	    int b;
	};

	//A，B的实现省略

因为A有virtual void f（），和g（），所以编译器为A类准备了一个虚表vtableA，内容如下：

	A::f 的地址
	A::g 的地址

B因为继承了A，所以编译器也为B准备了一个虚表vtableB，内容如下：

	A::f 的地址
	B::g 的地址

注意：因为B::ｇ是重写了的，所以B的虚表的g放的是B::g的入口地址，但是f是从上面的A继承下来的，所以f的地址是A::f的入口地址。

然后某处有语句 B bB;的时候，编译器分配空间时，除了A的int a，B的成员int b；以外，还分配了一个虚指针vptr，指向B的虚表vtableB，bB的布局如下：

	vptr ： 指向B的虚表vtableB
	int a： 继承A的成员
	int b： B成员

当如下语句的时候：

	A *pa = &bB;

pa的结构就是A的布局（就是说用pa只能访问的到bB对象的前两项，访问不到第三项int b）

那么pa->g()中，编译器知道的是，g是一个声明为virtual的成员函数，而且其入口地址放在表格（无论是vtalbeA表还是vtalbeB表）的第2项，那么编译器编译这条语句的时候就如是转换：

	call *(pa->vptr)[1]（C语言的数组索引从0开始）。

这一项放的是B：：g()的入口地址，则就实现了多态。（注意bB的vptr指向的是B的虚表vtableB）

另外要注意的是，如上的实现并不是唯一的，C++标准只要求用这种机制实现多态，至于虚指针vptr到底放在一个对象布局的哪里，标准没有要求，每个编译器自己决定。以上的结果是根据g++ 4.3.4经过反汇编分析出来的。

## 2.C++虚函数表解析  
C++中的虚函数的作用主要是实现了多态的机制。关于多态，简而言之就是用父类型别的指针指向其子类的实例，然后通过父类的指针调用实际子类的成员函数。这种技术可以让父类的指针有“多种形态”，这是一种泛型技术。所谓泛型技术，说白了就是试图使用不变的代码来实现可变的算法。比如：模板技术，RTTI技术，虚函数技术，要么是试图做到在编译时决议，要么试图做到运行时决议。

虚函数（Virtual Function）是通过一张虚函数表（Virtual Table）来实现的。简称为V-Table。在这个表中，主是要一个类的虚函数的地址表，这张表解决了继承、覆盖的问题，保证其容真实反应实际的函数。这样，在有虚函数的类的实例中这个表被分配在了这个实例的内存中，所以，当我们用父类的指针来操作一个子类的时候，这张虚函数表就显得由为重要了，它就像一个地图一样，指明了实际所应该调用的函数。



这里我们着重看一下这张虚函数表。在C++的标准规格说明书中说到，编译器必需要保证虚函数表的指针存在于对象实例中最前面的位置（这是为了保证正确取到虚函数的偏移量）。 这意味着我们通过对象实例的地址得到这张虚函数表，然后就可以遍历其中函数指针，并调用相应的函数。

假设我们有这样的一个类：




以下是引用片段：

	class Base {
	     public:
	            virtual void f() { cout << "Base::f" << endl; }
	            virtual void g() { cout << "Base::g" << endl; }
	            virtual void h() { cout << "Base::h" << endl; }

	};

   按照上面的说法，我们可以通过Base的实例来得到虚函数表。 下面是实际例程：




以下是引用片段：

	   typedef void(*Fun)(void);

	            Base b;

	            Fun pFun = NULL;

	            cout << "虚函数表地址：" << (int*)(&b) << endl;
	            cout << "虚函数表 — 第一个函数地址：" << (int*)*(int*)(&b) << endl;

	            // Invoke the first virtual function   
	            pFun = (Fun)*((int*)*(int*)(&b));
	            pFun();

以下是引用片段：

	虚函数表地址：0012FED4
	虚函数表 — 第一个函数地址：0044F148
	Base::f

 通过这个示例，我们可以看到，我们可以通过强行把&b转成int *，取得虚函数表的地址，然后，再次取址就可以得到第一个虚函数的地址了，也就是Base::f()，这在上面的程序中得到了验证（把int* 强制转成了函数指针）。通过这个示例，我们就可以知道如果要调用Base::g()和Base::h()，其代码如下：

	 (Fun)*((int*)*(int*)(&b)+0);  // Base::f()
	 (Fun)*((int*)*(int*)(&b)+1);  // Base::g()
	 (Fun)*((int*)*(int*)(&b)+2);  // Base::h()

## 3.动态绑定与静态绑定

静态绑定：编译时绑定，通过对象调用
动态绑定：运行时绑定，通过地址实现

只有采用“指针->函数()”或“引用变量.函数()”的方式调用C++类中的虚函数才会执行动态绑定。对于C++中的非虚函数，因为其不具备动态绑定的特征，所以不管采用什么样的方式调用，都不会执行动态绑定。
即所谓动态绑定，就是基类的指针或引用有可能指向不同的派生类对象，对于非虚函数，执行时实际调用该函数的对象类型即为该指针或引用的静态类型（基类类型）；而对于虚函数，执行时实际调用该函数的对象类型为该指针或引用所指对象的实际类型

## 4.构造函数和析构函数可以是虚函数吗？(C++中虚析构函数的作用)

**构造函数不能是虚函数，析构函数可以是虚函数且推荐最好设置为虚函数。**下面我们来看看为什么。

首先，我们已经知道虚函数的实现则是通过对象内存中的虚函数表指针vptr来实现的。而构造函数是用来实例化一个对象的，通俗来讲就是为对象内存中的值做初始化操作。那么在构造函数完成之前，vptr是没有值的，也就无法通过vptr找到作为虚函数的构造函数所在的代码区，所以构造函数只能作为普通函数存放在类所指定的代码区中。

子类构造时各个初始化步骤的调用顺序
构造顺序：

	1．构造子类构造函数的参数
	2．子类调用基类构造函数
	3．基类设置vptr
	4．基类初始化列表内容进行构造
	5.基类函数体调用
	6.子类设置vptr
	7.子类初始化列表内容进行构造
	8.子类构造函数体调用
	（注意一点，初始化列表内的数据不按书写顺序，而是按类内部的定义顺序）
	析构的顺序恰好相反，所以也不要在析构函数中调用虚函数，那样也是没有意义的。


那么为什么析构函数推荐最好设置为虚函数呢？当我们delete(a)的时候，如果析构函数不是虚函数，那么调用的将会是基类base的析构函数。而当继承的时候，通常派生类会在基类的基础上定义自己的成员，此时我们当时是希望可以调用派生类的析构函数对新定义的成员也进行析构啦。

对于构造函数和析构函数与虚函数的关系我们也可以从另一方面来理解。通常是如例子中base *a=new(A)这样创建一个对象时调用构造函数，此时我们是在new函数中直接指定类名字的，当然会直接调用对应类的构造函数，不会出现基类类型实际指向子类的情况出现。而调用析构函数通常是针对对象的操作，如delete(a)，此时我们当然希望可以调用到a实际指向的类型（类A）的析构函数，故需要设置为虚函数。



	class ClxBase
	{
	public:
	    ClxBase() {};
	    virtual ~ClxBase() {};

	    virtual void DoSomething() { cout << "Do something in class ClxBase!" << endl; };
	};

	class ClxDerived : public ClxBase
	{
	public:
	    ClxDerived() {};
	    ~ClxDerived() { cout << "Output from the destructor of class ClxDerived!" << endl; };

	    void DoSomething() { cout << "Do something in class ClxDerived!" << endl; };
	};

	ClxBase *pTest = new ClxDerived;
	pTest->DoSomething();
	delete pTest;

 输出结果是：

	Do something in class ClxDerived!
	Output from the destructor of class ClxDerived!

而如果去掉基类中析构函数的virtual关键字输出则变成


	Do something in class ClxDerived!
	Output from the destructor of class ClxDerived!

也就是说，类ClxDerived的析构函数根本没有被调用！一般情况下类的析构函数里面都是释放内存资源，而析构函数不被调用的话就会造成内存泄漏。我想所有的C++程序员都知道这样的危险性。当然，如果在析构函数中做了其他工作的话，那你的所有努力也都是白费力气。

所以，析构函数推荐最好设置为虚函数是为了当用一个基类的指针删除一个派生类的对象时，派生类的析构函数会被调用。
当然，并不是要把所有类的析构函数都写成虚函数。因为当类里面有虚函数的时候，编译器会给类添加一个虚函数表，里面来存放虚函数指针，这样就会增加类的存储空间。所以，只有当一个类被用来作为基类的时候，才把析构函数写成虚函数。

## 5.含有虚函数的类的大小
### 5.1 空类和单继承的sizeof

	class A
	{
	};

	class B
	{
	public:
	    B() {}
	    ~B() {}
	};

	class C
	{
	public:
	    C() {}
	    virtual ~C() {}
	};

	int main()
	{
	    cout << "sizeof(A)=" << sizeof(A) << endl;
	    cout << "sizeof(B)=" << sizeof(B) << endl;
	    cout << "sizeof(C)=" << sizeof(C) << endl;
	    return 0;
	}
输出：

	sizeof(A)=1
	sizeof(B)=1
	sizeof(C)=4
	请按任意键继续. . .

class A是一个空类型，它的实例不包含任何信息，本来求sizeof应该是0。但当我们声明该类型的实例的时候，它必须在内存中占有一定的空间，否则无法使用这些实例。至于占用多少内存，由编译器决定。Visual Studio 2012中每个空类型的实例占用一个byte的空间。

class B在class A的基础上添加了构造函数和析构函数。由于**构造函数和析构函数的调用与类型的实例无关（调用它们只需要知道函数地址即可）**，在它的实例中不需要增加任何信息。所以sizeof(B)和sizeof(A)一样，在Visual Studio 2012中都是1。



class C在class B的基础上把析构函数标注为虚拟函数。**C++的编译器一旦发现一个类型中有虚拟函数，就会为该类型生成虚函数表，并在该类型的每一个实例中添加一个指向虚函数表的指针**。在32位的机器上，一个指针占4个字节的空间，因此sizeof(C)是4。若是64位则一个指针占8个字节的空间。

### 5.2多继承的sizeof

	#include <iostream>
	using namespace std;

	class Base1 {
	    virtual void fun1() {}
	    virtual void fun11() {}
	public:
	    virtual ~Base1();
	};

	class Base2 {
	    virtual void fun2() {}
	};

	class DerivedFromOne : public Base2
	{
	    virtual void fun2() {}
	    virtual void fun22() {}
	};


	class DerivedFromThree :public DerivedFromOne, public DerivedFromTwo {
	    virtual void fun4(){}
	};
	void main()
	{
	    cout << "sizeof Base1 " << sizeof(Base1) << endl;
	    cout << "sizeof Base1 " << sizeof(Base2) << endl;
	    cout << "sizeof FromOne " << sizeof(DerivedFromOne) << endl;
	    cout << "sizeof FromTwo " << sizeof(DerivedFromTwo) << endl;
	    cout << "sizeof FromThree " << sizeof(DerivedFromThree) << endl;
	    getchar();
	}

1）一个类中若有虚函数，（不论是自己的虚函数，还是继承而来的），那么类中就有一个成员变量：虚函数指针，这个指针指向一个虚函数表，虚函数表的第一项是类的typeinfo信息，之后的项为此类的所有虚函数的地址。

2）假设经过成员对齐后的类的大小为size个字节。那么类的sizeof大小可以这么计算：size + 4*（虚函数指针的个数n）。代码中，DerivedFromTwo继承自2个分支，所以有2个虚函数指针，所以sizeof大小为0 + 4* 2 = 8。

3）带有虚函数的类的sizeof大小，实际上和虚函数的个数不相关，相关的是虚函数指针。

可以在VS中通过命令查看虚函数表，见7.如何在VS2015中查看虚函数表和类内存布局


## 6.有覆盖和无覆盖时的虚函数表

### 一般继承（无虚函数覆盖）
子类没有重载任何父类的函数

	class A
	{
	public:
	    virtual void f();
	    virtual void g();
		virtual void h();
	private:
	    int a
	};

	class B : public A
	{
	public:
	    void f1();
	    void g1();
		void h1();
	private:
	    int b;
	};

	//A，B的实现省略

对于实例B b
其虚函数表

	A::f 的地址
	A::g 的地址
	A::h 的地址
	B::f1 的地址
	B::g1 的地址
	B::h1 的地址


我们可以看到下面几点：

     1）虚函数按照其声明顺序放于表中。

     2）父类的虚函数在子类的虚函数前面。


### 一般继承（有虚函数覆盖）

	class A
	{
	public:
	    virtual void f();
	    virtual void g();
		virtual void h();
	private:
	    int a
	};

	class B : public A
	{
	public:
	    void f();
	    void g1();
		void h1();
	private:
	    int b;
	};

	//A，B的实现省略

对于实例B b
其虚函数表

	A::f 的地址
	A::g 的地址
	A::h 的地址
	B::f1 的地址
	B::g1 的地址
	B::h1 的地址


我们可以看到下面几点：

     1）虚函数按照其声明顺序放于表中。

     2）父类的虚函数在子类的虚函数前面。

继承后覆盖了父类的一个函数：f()。

	B::f 的地址
	A::g 的地址
	A::h 的地址
	B::g1 的地址
	B::h1 的地址
   我们从表中可以看到下面几点，

 1）覆盖的f()函数被放到了虚表中原来父类虚函数的位置。

  2）没有被覆盖的函数依旧。

	A *b = new B();

    b->f();
由b所指的内存中的虚函数表的f()的位置已经被B::f()函数地址所取代，于是在实际调用发生时，是B::f()被调用了。这就实现了多态。

### 多重继承（无虚函数覆盖）

	class Base1 {
	public:
	            virtual void f() { cout << "Base1::f" << endl; }
	            virtual void g() { cout << "Base1::g" << endl; }
	            virtual void h() { cout << "Base1::h" << endl; }

	};

	class Base2 {
	public:
	            virtual void f() { cout << "Base2::f" << endl; }
	            virtual void g() { cout << "Base2::g" << endl; }
	            virtual void h() { cout << "Base2::h" << endl; }
	};

	class Base3 {
	public:
	            virtual void f() { cout << "Base3::f" << endl; }
	            virtual void g() { cout << "Base3::g" << endl; }
	            virtual void h() { cout << "Base3::h" << endl; }
	};


	class Derive : public Base1, public Base2, public Base3 {
	public:
	            virtual void f1() { cout << "Derive::f" << endl; }
	            virtual void g1() { cout << "Derive::g1" << endl; }
	};

产生了三个父类的虚函数表

	Base1::f 的地址
	Base1::g 的地址
	Base1::h 的地址
	Derive::f1 的地址
	Derive::g1 的地址

和

	Base2::f 的地址
	Base2::g 的地址
	Base2::h 的地址

以及

	Base2::f 的地址
	Base2::g 的地址
	Base2::h 的地址

可见

     1）  每个父类都有自己的虚表。

     2）  子类的成员函数被放到了第一个父类的表中。（所谓的第一个父类是按照声明顺序来判断的）

### 多重继承（有虚函数覆盖）
	class Base1 {
	public:
	            virtual void f() { cout << "Base1::f" << endl; }
	            virtual void g() { cout << "Base1::g" << endl; }
	            virtual void h() { cout << "Base1::h" << endl; }

	};

	class Base2 {
	public:
	            virtual void f() { cout << "Base2::f" << endl; }
	            virtual void g() { cout << "Base2::g" << endl; }
	            virtual void h() { cout << "Base2::h" << endl; }
	};

	class Base3 {
	public:
	            virtual void f() { cout << "Base3::f" << endl; }
	            virtual void g() { cout << "Base3::g" << endl; }
	            virtual void h() { cout << "Base3::h" << endl; }
	};


	class Derive : public Base1, public Base2, public Base3 {
	public:
	            virtual void f() { cout << "Derive::f" << endl; }
	            virtual void g1() { cout << "Derive::g1" << endl; }
	};
在子类中覆盖了父类的f()函数

产生了三个父类的虚函数表

	Derive::f 的地址
	Base1::g 的地址
	Base1::h 的地址
	Derive::g1 的地址

和

	Derive::f 的地址
	Base2::g 的地址
	Base2::h 的地址

以及

	Derive::f 的地址
	Base2::g 的地址
	Base2::h 的地址

可见三个父类虚函数表中的f()的位置被替换成了子类的函数指针。这样，我们就可以任一静态类型的父类来指向子类，并调用子类的f()了。

以下是引用片段：

			Derive d;
			Base1 *b1 = &d;
            Base2 *b2 = &d;
            Base3 *b3 = &d;
            b1->f(); //Derive::f()
            b2->f(); //Derive::f()
            b3->f(); //Derive::f()

            b1->g(); //Base1::g()
            b2->g(); //Base2::g()
            b3->g(); //Base3::g()            

多重继承的输出部分例程

	#include <iostream>
	using namespace std;   
	typedef void(*Fun)(void);

	int main()  
	{
	            Fun pFun = NULL;

	            Derive d;
	            int** pVtab = (int**)&d;

	            //Base1's vtable
	            //pFun = (Fun)*((int*)*(int*)((int*)&d+0)+0);
	            pFun = (Fun)pVtab[0][0];
	            pFun();

	            //pFun = (Fun)*((int*)*(int*)((int*)&d+0)+1);
	            pFun = (Fun)pVtab[0][1];
	            pFun();

	            //pFun = (Fun)*((int*)*(int*)((int*)&d+0)+2);
	            pFun = (Fun)pVtab[0][2];
	            pFun();

	            //Derive's vtable
	            //pFun = (Fun)*((int*)*(int*)((int*)&d+0)+3);
	            pFun = (Fun)pVtab[0][3];
	            pFun();

	            //The tail of the vtable
	            pFun = (Fun)pVtab[0][4];
	            cout<<pFun<<endl;


	            //Base2's vtable
	            //pFun = (Fun)*((int*)*(int*)((int*)&d+1)+0);
	            pFun = (Fun)pVtab[1][0];
	            pFun();

	            //pFun = (Fun)*((int*)*(int*)((int*)&d+1)+1);
	            pFun = (Fun)pVtab[1][1];
	            pFun();

	            pFun = (Fun)pVtab[1][2];
	            pFun();  

	            //The tail of the vtable
	            pFun = (Fun)pVtab[1][3];
	            cout<<pFun<<endl;



	            //Base3's vtable
	            //pFun = (Fun)*((int*)*(int*)((int*)&d+1)+0);
	            pFun = (Fun)pVtab[2][0];
	            pFun();

	            //pFun = (Fun)*((int*)*(int*)((int*)&d+1)+1);
	            pFun = (Fun)pVtab[2][1];
	            pFun();

	            pFun = (Fun)pVtab[2][2];
	            pFun();  

	            //The tail of the vtable
	            pFun = (Fun)pVtab[2][3];
	            cout<<pFun<<endl;

	            return 0;
	}


## 7.如何在VS2015中查看虚函数表和类内存布局

在Developer Command Prompt For VS2015中使用cl命令的reportSingleClassLayoutXXX选项打印指定类的内存布局和虚函数表

格式为

	cl [filename].cpp /d1reportSingleClassLayout[className]

例如文件名为Test.cpp，类名为Derived
则输入

	cl Test.cpp /d1reportSingleClassLayoutDerived  
