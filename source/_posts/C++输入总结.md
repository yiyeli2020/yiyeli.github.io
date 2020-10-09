---
title: C++输入总结

date: 2018-08-29 10:33:12

categories: 2018年8月

tags: [C++,stdin]

---
 

总结常用的C++输入


<!-- more -->

总结下常用的C++字符，字符串，数字输入方法和问题：

### 1.while里面用cin >> val如何结束
### 2.如何在while（cin >> val）跳出循环后能继续执行程序中其他的输入操作
### 3.getline和cin.getline()的区别及使用方法
```
    #include<iostream>
	#include<vector>
	#include<map>
	#include<cctype>
	#include<string>
	using namespace std;

	int main()
	{
	    /*
	    Q1:
	    while里面用cin >> val是什么意思，用这个当条件的话，通过检测其流的状态来判断结束；

	    （1）若流是有效的，即流未遇到错误，那么检测成功；

	    （2）若遇到文件结束符，或遇到一个无效的输入时（例如输入的值不是一个整数），istream对象的状态会变为无效，条件就为假；

	    怎样才是文件结束符呢？

	    不同的操作系统有不同的约定，在windows系统中，输入文件结束符的方法是先按Ctrl+Z，然后再按Enter；在UNIX系统中，包括Mac OS X系统中，文件结束输入为Ctrl+D;
	     */
	    //freopen("in.txt", "r", stdin);
	    int val;
	    vector<int>nums;  
	    while (cin >> val ) {//在键盘输入时要ctrl+z再按回车,文件输入则不用
	        nums.push_back(val);

	    }
	    for (auto n : nums) cout << n << "  ";
	    cout << endl;
	    /*
	    Q2:
	    如何在while（cin >> val）跳出循环后能继续执行程序中其他的输入操作:
	    输入ctrl+z使程序跳出了while(cin>>val)的循环，但是它并不能继续执行循环外的输入操作也就是直接退出了程序。

	    那这是为什么呢？

	    要理清这件事首先我们要对了解的处理机制，cin遇到ctrl+z便认为输入结束，也就是不再接受键盘的输入（但是它会读取缓冲区已经存在的数据），

	    但是ctrl+z还是会留在缓冲区内

	    也就是说当我们输入换行+（ctrl+z）+换行后。程序跳出while循环，但是ctrl+z还在缓冲区内，当程序继续执行时，
	    当cin再去读的时候，发现缓冲区存在ctrl+z（上次跳出循环遗留下的），于是它就走了，也就是什么都没读到。所以用户也无法输入。
	    解决方法很简单

	    就是及时清除缓冲区

	    也就是在程序跳出循环后利用函数

	    cin.clear();
	    cin.ignore();
	    */
	    cin.clear();
	    cin.ignore();
	    string str;
	    getline(cin, str);
	    cout << str << endl;
	    /*
	    用于string类的。使用需包含头文件#include<string>。getline(cin,string s)，接收一个字符串，可以接收空格、回车等

	    与cin.getline()的区别：1.cin.getline()接收输入字符串的是数组，getline（）是string类型。

	    2.cin.getline()可以接收空格，但不能接收回车；getline()可以接收空格和回车

	    3.cin.getline()会在数组结尾是'\0'，getline()不会

	    */
	    char s[20];
	    cin.getline(s,20);
	    cout << s << endl;
	    /*实际是cin.getline(接收字符串到m，接收个数n，结束字符)。接收一个字符串，可以接收空格等，最后一个字符为‘\0’。结束符可以通过设置第三个参数自己设置，默认是回车。m不能为string类型。

	    注意：实际接收到的要比n少一个，因为最后一个字符为'\0'。
	    */


	    char c;
	    cin.get(c);
	    cout << c << endl;
	    return 0;

	}
```
## 4.将一组未指定个数的输入数字转换为数组
输入样例：
5 24 39 60 15 28 27 40 50 90
需要#include< string >和#include< vector >
```
    int val=0;
    freopen("in.txt", "r", stdin);
    vector<int> nums;
    //while (cin >> val) {
    //    nums.push_back(val);
    //}
    string in;
    getline(cin,in);
    int i = 0;
    while (i < in.size()) {
        //cout << "i=" << i << "   val=" << val << endl;
        if (in[i] == ' ') {
            val = 0;
            i++;
        }
        while (in[i] != ' ' && i < in.size()) {
                val = val * 10 + (in[i] - '0');
                i++;
            }
            nums.push_back(val);
        //cout << "i=" << i << "   val=" << val << endl;
    }
```