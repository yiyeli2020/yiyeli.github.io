---
title: C++二维动态静态数组传参速查

date: 2022-08-05 18:12:12

categories: 2022年8月

tags: [C++, Array]

---
 
动态静态数组传参数都是引用传递


<!-- more -->

[TOC]

参考[^1]

<details>
    <summary>动态静态数组传参</summary>
    
```

//  二维数组
//  Created by yiye on 2022/8/4.
//https://www.cnblogs.com/usa007lhy/p/3286186.html

#include <iostream>
#include <vector>
using namespace std;

//void iniMatrix(int a[3][5], int rows){
void iniMatrix(int a[][5], int rows){
    cout<<"*******初始化二维静态数组******"<<endl;
    for(int i=0;i<rows;i++){
        for(int j=0;j<5;j++){
            a[i][j]=i*10+j;
        }
    }
}

void iniDynamicMatrix(int **a, int rows, int cols){
    cout<<"*******初始化二维动态数组******"<<endl;
    for(int i=0;i<rows;i++){
        for(int j=0;j<cols;j++){
            a[i][j]=i*10+j;
        }
    }
}

void printStaticMatrix(int a[][5],int rows){
    cout<<"*******打印二维静态数组******"<<endl;
    for(int i=0;i<rows;i++){
        for(int j=0;j<5;j++){
            cout<<a[i][j]<<" ";
        }
        cout<<endl;
    }
}
void printDynamicMatrix(int **a,int rows, int cols){
    cout<<"*******打印二维动态数组******"<<endl;
    for(int i=0;i<rows;i++){
        for(int j=0;j<cols;j++){
            cout<<a[i][j]<<" ";
        }
        cout<<endl;
    }
}

void changeArray(int *num, int len){
    for(int i=0;i<len/2;i++){
        num[i] = -1;
    }
}

void printArray(int *num, int len){
    cout<<"**********打印一维数组**********"<<endl;
    for(int i=0;i<len;i++){
        cout<<num[i]<<" ";
    }
    cout<<endl;
}

void printStaticMatrix(int **num, int row, int col){
    cout<<"*******打印二维静态数组******"<<endl;
    for (int i=0; i<row; i++) {
        for (int j=0; j<col; j++) {
            cout<<*((int *)num + col*i+j)<<" "; // 可行方案
//            cout<<((int *)num + col*i)[j]<<" "; //可行方案
//            cout<<num[i][j]<<" ";//不可行方案
        }
        cout<<endl;
    }
}

int main(int argc, const char * argv[]) {
    int a[3][5];
    iniMatrix(a, 3);
    printStaticMatrix(a, 3);
    printStaticMatrix((int **)a, 3, 5);
    cout<<"*******新的二维数组******"<<endl;
    int b[][5] = {1,2,3,4,5,6,7,8,9,10,11,12,13,14,15};
    printStaticMatrix(b, 3);
    int len = 5;
    int num[5]={1,2,3,4,5};
    printArray(num, 5);
    changeArray(num, len);
    printArray(num, 5);
    cout<<"#####静态数组传参数是引用传递#####"<<endl;
    int *num2 = new int[len];
    for(int i=0;i<len;i++){
        num2[i]=i;
    }
    printArray(num2, len);
    changeArray(num2, len);
    printArray(num2, len);
    cout<<"#####动态数组传参数是引用传递#####"<<endl;
    delete[] num2;
    
    int row = 3, col=5;
    int **num3  = new int*[row];
    for (int i=0; i<row; i++) {
        num3[i]=new int[col];
    }
    iniDynamicMatrix(num3, row, col);
    printDynamicMatrix(num3, row, col);
    return 0;
}


```
</details>


[^1]:https://www.cnblogs.com/usa007lhy/p/3286186.html