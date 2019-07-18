---
title: 剑指offer思路总结
date: 2018-08-23 08:45:12
categories: 2018年8月
tags: [Coding Interviews,Algorithm]


---
 

剑指offer已经刷过两遍，总结一下思路方便回忆

<!-- more -->


## 13.在O(1)时间删除链表结点

	分情况讨论：
	先在链表中顺序搜索
	要删除的节点i不是尾结点：
	{
		先将i的下一个结点j的内容复制到i
		然后把i的指针指向结点j的下一个结点；
		此时再删除结点j
	}
	链表只有一个结点{
		则删除该结点
		然后将其置为NULL
		将头结点置为空
	}
	链表有多个结点而且要删除的结点是尾结点{
		从头结点顺序遍历，只要找到该结点i的前序结点就可以，然后执行删除操作
	}

## 14.调整数组顺序使奇数位于偶数前面

### Brute Force
顺序扫描碰到偶数就将其放到末尾

### 初级解法

双指针指向头尾部分，头偶尾奇则交换两个数字

### 可扩展解法

仍使用头尾指针，但用函数指针的方法将判断奇偶的函数解耦出来，整体逻辑框架不需要改动

## 15.链表中倒数第K个节点

快慢指针，快指针比慢指针领先k-1步出发，快指针到尾节点时慢指针指向的是所求

### 举一反三
#### 1.求链表中间结点
快慢指针，快指针每次走两步，慢指针每次走一步，快指针到末尾时慢指针指向中间结点

#### 2.判断单向链表是否成环
同前面，若快指针追上慢指针则成环

## 16.反转链表
需要定义三个指针，指向当前遍历的结点，前一个结点和后一个结点,先要找到反转链表后的头结点即原始链表的尾节点，需要满足的条件是后结点为空

	////Non-Recursive
	ListNode* ReverseList(ListNode* pHead)
	{
	    ListNode* pReversedHead = nullptr;
	    ListNode* pNode = pHead;
	    ListNode* pPrev = nullptr;
	    while (pNode != nullptr)
	    {
	        ListNode *pNext = pNode->m_pNext;
	        if (pNext == NULL) {//find the last node
	            pReversedHead = pNode;
	        }
	        pNode->m_pNext = pPrev;//Revserse

	        pPrev = pNode;// forward
	        pNode = pNext;
	    }

	    return pReversedHead;
	}
	ListNode* ReverseList(ListNode* pHead)
	{
	    if (pHead == NULL || pHead->m_pNext == nullptr) return pHead;
	    ListNode *PReverseHead = pHead;
	    pHead=ReverseList(pHead->m_pNext);
	    PReverseHead->m_pNext->m_pNext = PReverseHead;//进入尾节点
	    PReverseHead->m_pNext = NULL;
	    return pHead;
	}

## 28.字符串的排列
