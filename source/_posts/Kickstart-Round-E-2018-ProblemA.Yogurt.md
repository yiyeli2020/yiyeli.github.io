---
title: Kickstart-Round-E-2018-ProblemA.Yogurt
date: 2018-08-26 15:07:00
categories: 2018年8月
tags: [Kickstart,Google,Algorithm,Array]

---
# Kickstart-Round-E-2018-ProblemA.Yogurt


酸奶可以作为开胃菜、主菜或甜点的营养成分，但它必须在过期前食用，而且可能很快过期!此外，不同的酸奶会在不同的日子过期。露西很喜欢酸奶，她刚买了N杯酸奶，但她担心自己可能无法在酸奶过期前全部喝完。从今天开始，第i杯酸奶会过期，在过期当天或之后的任何一天都不能喝。虽然露西很喜欢酸奶，但她每天最多只能喝K杯酸奶。从今天开始，她最多能喝多少杯酸奶?


<!-- more -->
## 问题

Problem
Yogurt can be a nutritious part of an appetizer, main course, or dessert, but it must be consumed before it expires, and it might expire quickly! Moreover, different cups of yogurt might expire on different days.

Lucy loves yogurt, and she has just bought N cups of yogurt, but she is worried that she might not be able to consume all of them before they expire. The i-th cup of yogurt will expire Ai days from today, and a cup of yogurt cannot be consumed on the day it expires, or on any day after that.

As much as Lucy loves yogurt, she can still only consume at most K cups of yogurt each day. What is the largest number of cups of yogurt that she can consume, starting from today?

Input
The first line of the input gives the number of test cases, T. T test cases follow. Each test case starts with one line containing two integers N and K, as described above. Then, there is one more line with N integers Ai, as described above.

Output
For each test case, output one line containing Case #x: y, where x is the test case number (starting from 1) and y is the maximum number of cups of yogurt that Lucy can consume, as described above.

Limits
1 ≤ T ≤ 100.
1 ≤ K ≤ N.
1 ≤ Ai ≤ 109, for all i.
Small dataset
1 ≤ N ≤ 1000.
K = 1.
Large dataset
1 ≤ N ≤ 5000.
Sample

Input 
 	
Output 
	 
	4
	2 1
	1 1
	5 1
	3 2 3 2 3
	2 2
	1 1
	6 2
	1 1 1 7 7 7
	
	Case #1: 1
	Case #2: 3
	Case #3: 2
	Case #4: 5

Note that the last two sample cases would not appear in the Small dataset.

In Sample Case #1, each of the two cups of yogurt will expire in one day. Today, Lucy can consume one of them, but she can only consume at most one cup each day, so she cannot consume both. Tomorrow, Lucy cannot consume the remaining cup of yogurt, because it will have expired.

In Sample Case #3, Lucy can consume up to two cups each day, so she can consume all of the yogurt.

## 思路1（Brute force)

模拟每天发生的过程，先将输入的保质期数组A排序，将最大的保质期天数定义为maxday，然后从今天到maxday进行循环，每天更新最多能喝的杯数和当前的保质期数组。
这里在比赛过程中犯了两个错误，见代码注释
	
	#include <iostream>
	#include<unordered_map>
	#include<cstdio>
	#include<queue>
	#include<vector>
	#include<cmath>
	#include<algorithm>
	#include<stack>
	#include<set>
	using namespace std;
	
	void solve() {// brute force TLE
		int N = 0, K = 0;
		cin >> N >> K;
		vector<int> a; 
		for (int i = 0; i < N; i++) {
		int temp;
		cin >> temp; 
		a.push_back(temp);
		}
		sort(a.begin(), a.end());
		  /*  cout << "a[N-1]=" << a[N - 1] << "  "; 
		cout << "After sort:   "; output(a);*/
		
		int num = 0;
		int maxday = a[N - 1];
		for (int i = 0; i < maxday; i++) {//error i < a[N - 1] 因为a[N-1]会动态变化
		   // cout << "i=" << i << "  ";
		int k = 0;
		for (int j = 0; j < N ; j++) {
		//cout << "j=" << j << endl;
		if ((a[j] > 0)&&(k<K)) {//error (k<K)条件要放到这个if里而不能放到for循环中
		a[j] = 0;
		k++;
		num++;
			}
		}
	   // cout << "  num=" << num<<"  "; output(a);
		for (int mm = 0; mm < N; mm++) { a[mm] -= 1; }
	
		}
		cout << num << endl;
	}
	int main() {
	    freopen("C:\\Users\\yiye\\Downloads\\A-small-attempt0.in", "r", stdin);
	    //freopen("C:\\Users\\yiye\\Downloads\\out.txt", "w", stdout);
	  
	    int T;
	    scanf("%d", &T);
	    for (int i = 1; i <= T; i++) {
	         printf("Case #%d: ", i);
			 solve();
	
	    }
	        //===========================
	        //fclose(stdin);//关闭文件 
	        //fclose(stdout);//关闭文件 
	        return 0;
	}

## 思路2（Brute force)改进版本

在每天的模拟过程中用lower_bound找到第一个非零的位置，然后将改天中连续K个杯子状态置为已喝，这样可以将时间复杂度从O(n*n)变为O(nlog(n))

	
	#include <iostream>
	#include<unordered_map>
	#include<cstdio>
	#include<queue>
	#include<vector>
	#include<cmath>
	#include<algorithm>
	#include<stack> 
	#include <functional> 
	#include<set>
	using namespace std;
	void solve() {// brute force TLE
	    int N = 0, K = 0;
	    cin >> N >> K;
	    vector<int> a;
	    for (int i = 0; i < N; i++) {
	        int temp;
	        cin >> temp;
	        a.push_back(temp);
	    }
	    sort(a.begin(), a.end());
	    
	    int num = 0;
	        int maxday = a[N - 1];
	        for (int i = 0; i < maxday; i++) {//error i < a[N - 1] 因为a[N-1]会动态变化
	            //cout << "i=" << i << "  ";    
	            int k = 0;
	            vector<int>::iterator nonezeroindex = lower_bound(a.begin(), a.end(), 1);
	            for (int j = nonezeroindex-a.begin(); j <nonezeroindex - a.begin()+ K && j<N ; j++) {
	                a[j] = 0;
	                num++;            
	            }
	            //cout << "  nonezeroindex="<<(nonezeroindex-a.begin())<<"  num=" << num<<"  "; output(a);
	            for (int mm = nonezeroindex - a.begin(); mm < N; mm++) { a[mm] -= 1; }
	            
	        }
	    cout << num << endl;
	}
	int main() {
	    //freopen("C:\\Users\\yiye\\Downloads\\A-small-attempt0.in", "r", stdin);
	    //freopen("C:\\Users\\yiye\\Downloads\\out.txt", "w", stdout);
	    freopen("out.txt", "r", stdin);
	    int T;
	    scanf("%d", &T);
	    for (int i = 1; i <= T; i++) {
	        printf("Case #%d: ", i);
	        solve();
	    }
	        //===========================
	        //fclose(stdin);//关闭文件 
	        //fclose(stdout);//关闭文件 
	        return 0;
	    } 

## 思路3：

这是E轮中第一名选手ijn的解法，核心代码用五行实现了我十行的功能，浓缩的都是精华啊

	#include<bits/stdc++.h>
	using namespace std;
	int t,n,m,a[5005],i,j,k,ans;
	int main()
	{
		freopen("a.in","r",stdin);
		freopen("a.out","w",stdout);
		scanf("%d",&t);
		for(int xxx=1;xxx<=t;xxx++)
		{
			printf("Case #%d: ",xxx);
			scanf("%d%d",&n,&m);//m为K
			for(i=1;i<=n;i++)scanf("%d",a+i);
			sort(a+1,a+n+1);
			for(i=k=1,ans=0;i<=n;k++)
			{
				j=min(i+m,n+1);
				ans+=j-i;
				i=j;
				for(;i<=n&&a[i]<=k;i++);
			}
			printf("%d\n",ans);
		}
		return 0;
	}

将其	改写成易于理解的形式	

	void solve() {
	    int N, K;
	    cin >> N >> K;
	    vector<int> a(N+1,0);
	    for (int i = 1; i <= N; i++) cin >> a[i];
	    sort(a.begin(), a.end());//实现排序功能
	    int  num=0;
	    for (int k = 1, i = 1; i <= N; k++) {//对数组进行遍历，ans为最多能喝的杯数，k为已经过去的天数
	        int j = min(i + K, N + 1);//寻找可以喝的酸奶序列的终点
	        num += j - i;//更新
	        i = j;
	        for (; i <= N && a[i] <= k; i++);//跳过过期的杯子
	    }
	    cout << num << endl;
	}