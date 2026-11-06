
## 1、题目描述

实现**带头结点的单链表**基础功能，包含：

- 创建空链表（带头结点）
- 新建结点
- 头插、尾插、按位置中间插入
- 按值删除结点
- 遍历打印链表
- 删除链表中值最小的结点

> 
> 说明：头结点不存储有效数据，'head->next' 指向第一个有效结点；位置 'dest' 从 0 开始计数。

## 2、算法思路

1. **结构体设计**：链表结点包含数据域 'Data' 和后继指针 'next'。
2. **创建链表**：利用'calloc' 分配头结点，自动清零，'head->next = NULL' 代表空链表。
3. **头插法**：新结点的 'next' 指向原来第一个有效结点，再将头结点 'next' 指向新结点。
4. **尾插法**：循环遍历到链表最后一个结点（'temp->next == NULL'），把尾结点 'next' 指向新结点。
5. **按位置插入**：从头部开始遍历，计数 'pos'，走到目标位置的前驱结点后插入新结点；若位置超出链表长度则插入失败。
6. **按值删除**：遍历链表，查找待删数据，找到后修改前驱指针绕过目标结点、释放内存；支持删除第一个匹配值。
7. **删除最小值结点**：一次遍历，同步记录最小值结点和它的前驱结点，遍历完成后通过前驱指针删除最小结点并释放内存。
8. **遍历打印**：从第一个有效结点开始依次输出数据。

## 3、优化后完整源码

```c
/***************************************************************************************************************************************
 * 
 * 
 * 
 * 
 * 
 * 
 * 
 * 
 * 
 * Copyright (c)  	2026-10		18228218705@163.com		All rights Reserved
 * ************************************************************************************************************************************/
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

//链表中的数据元素类型
typedef int DataType_t;

//构造一个结构体来记录链表各项参数
typedef struct LinkedList{
	DataType_t  			Data; 	//结点的数据域
	struct LinkedList	   *next;	//结点的指针域

}Link_L_t;

//创建一个空链表
Link_L_t * Link_L_create(void)
{
	//1.创建一个头结点
	 Link_L_t * head = (Link_L_t*)calloc(1,sizeof(Link_L_t));

	 //创建失败
	 if(head == NULL){
	 	perror("calloc momery for Head is failed");
	 	exit(-1);
	 }

	 //创建成功,对头结点初始化
	 head -> next = NULL;

	 //返回头节点地址
	 return head;
}

//创建新节点，并初始化
Link_L_t * Link_L_NewNode(DataType_t data)
{
	//申请一块结点大小的内存空间
	Link_L_t * Node = (Link_L_t *)calloc(1,sizeof(Link_L_t));
	if(NULL == Node){
		perror("calloc memory for Node is failed");
		return NULL;
	}

	//申请成功
	Node ->data = data;	//给新结点数据域赋值
	Node ->next = NULL;	//新结点后继置空，防止野指针
	return Node;
}

//头插：在头结点后面插入新结点
bool Link_L_HeadAdd( Link_L_t * head,DataType_t data)
{	
	//先判断传入的头指针是否为空，防止解引用崩溃
	if(head == NULL)
	{
		perror("head pointer is NULL");
		return false;
	}

	Link_L_t *Node = Link_L_NewNode(data);
	//如果节点创建失败
	if(Node == NULL)
	{
		perror("Node created is Failed");
		return false;
	}

	//新结点先接管原来头结点后面的链表
	Node -> next = head ->next;
	//头结点指向新结点，完成头插
	head ->next = Node;
	return true;
}

//尾插：在链表最后一个结点后面插入
bool Link_L_TailAdd(Link_L_t * head,DataType_t data)
{	
	//先判断传入的头指针是否为空
	if(head == NULL)
	{
		perror("head pointer is NULL");
		return false;
	}

	Link_L_t * temp = head;	//临时遍历指针，从头结点出发
	Link_L_t *Node = Link_L_NewNode(data);
	//如果节点创建失败
	if(Node == NULL)
	{
		perror("Node created is Failed");
		return false;
	}

	//循环走到链表最后一个结点
	while(temp -> next != NULL)
	{
		temp = temp ->next;
	}
	
	//尾结点的next指向新结点
	temp ->next = Node;
	return true;
}

//中间插入：在dest位置前面插入结点，dest从0开始计数
bool Link_L_MidAdd(Link_L_t * head,DataType_t data,int dest)
{	
	//先判断传入的头指针是否为空
	if(head == NULL)
	{
		perror("head pointer is NULL");
		return false;
	}

	//判断要找的dest是否小于0，位置不能为负数
	if(dest < 0)
	{
		perror("The dest is less than 0");
		return false;
	}
	//记录头节点
	Link_L_t * temp =head; 

	//通过Link_L_NewNode函数创建一个新结点
	Link_L_t *Node = Link_L_NewNode(data);

	//用来记录temp当前走到第几个位置
	int pos = 0;	

	//如果节点创建失败
	if(Node == NULL)
	{
		perror("Node created is Failed");
		return false;
	}

	//循环走到目标位置的前驱结点
	while(temp -> next != NULL && pos < dest)
	{
		temp = temp ->next;
		pos++;
	} 

	//新结点先接过temp后面的链表
	Node -> next= temp ->next;
	//temp指向新结点，完成插入
	temp ->next = Node;
	return true;
}

//删除第一个值等于data的结点
bool Link_L_DelDest(Link_L_t * head,DataType_t data)
{	
	//先判断传入的头指针是否为空
	if(head == NULL)
	{
		perror("head pointer is NULL");
		return false;
	}

	Link_L_t * temp = head;

	//如果链表为空，删除失败
	if(head -> next == NULL)
	{
		perror("The Linked List is NULL");
		return false;
	}

	//链表不为空
	Link_L_t * DelNode;
	while(temp ->next != NULL)
	{
		//判断下一个结点的数据是不是要删除的数据
		if(temp->next->data == data)
		{
			DelNode = temp->next;	//保存待删除结点地址
			temp->next=DelNode->next;	//跳过待删结点，断链
			free(DelNode);	//释放被删除结点的内存，防止内存泄漏
			DelNode = NULL;
			return true;
		}
		temp = temp ->next;
	}
	//循环走完没有找到对应数据
	perror("The Data Is Not Found");
	return false;
}

//遍历：打印链表中所有有效结点的数据
void Link_L_Print(Link_L_t * head)
{
	//先判断传入的头指针是否为空
	if(head == NULL)
	{
		perror("head pointer is NULL");
		return;
	}

	//备份链表第一个有效结点地址，跳过不存数据的头结点
	Link_L_t * temp = head->next;

	printf("[List]: ");
	//依次遍历每一个结点
	while(temp != NULL)
	{
		printf("%d ",temp->data);
		temp = temp ->next;
	}
	printf("\n");
}

//删除最小值结点，只删除遇到的第一个最小值
void Link_L_DelMin(Link_L_t * head)
{
	//先判断传入的头指针是否为空
	if(head == NULL)
	{
		perror("head pointer is NULL");
		return;
	}

	Link_L_t * min_prev; 				  //记录最小值结点直接前驱地址
	Link_L_t * min_node = head ->next;	 //记录最小值结点地址

	Link_L_t * cur = head ->next;     //记录当前遍历结点地址
	Link_L_t * cur_prev = head;	   //记录当前节点的直接前驱

	//判断链表是否为空
	if(head ->next == NULL)
	{
		perror("The LinkedList Is Empty");
		return;
	}

	//遍历链表、找到最小值结点以及它的前驱
	while(cur != NULL)
	{
		//比较链表中结点的数据域大小，如果找到更小的值，更新最小值信息
		if( cur ->data < min_node ->data)
		{
			min_prev = cur_prev;
			min_node = cur;
		}

		//两个指针同步向后移动
		cur_prev = cur;
		cur      = cur ->next;
	}

	//删除最小值结点：前驱结点绕过最小值结点
	min_prev ->next = min_node ->next;

	//释放最小值结点内存
	free(min_node);
	min_node = NULL;
}

//销毁整条链表，释放所有结点内存（包括头结点）
void Link_L_Destroy(Link_L_t * head)
{
	if(head == NULL)
	{
		return;
	}
	Link_L_t * cur = head;
	Link_L_t * del;
	//循环逐个释放结点
	while(cur != NULL)
	{
		del = cur;
		cur = cur->next;
		free(del);
	}
}

int main(int argc,char*argv[])
{
	//创建一条空链表
	Link_L_t * list = Link_L_create();

	//测试样例
	Link_L_TailAdd(list, 10);
	Link_L_TailAdd(list, 30);
	Link_L_TailAdd(list, 20);
	Link_L_Print(list);

	Link_L_HeadAdd(list, 5);
	Link_L_Print(list);

	Link_L_MidAdd(list, 15, 2);
	Link_L_Print(list);

	Link_L_DelDest(list, 30);
	Link_L_Print(list);

	Link_L_DelMin(list);
	Link_L_Print(list);

	//使用完毕销毁链表
	Link_L_Destroy(list);
	list = NULL;
	return 0;
}
```

## 4、测试样例

```
1. 尾插：10、30、20
2. 头插：5
3. 在位置2插入15
4. 删除值30
5. 删除最小值结点
```
### 预期输出
```
[List]: 10 30 20 
[List]: 5 10 30 20 
[List]: 5 10 15 30 20 
[List]: 5 10 15 20 
[List]: 10 15 20 
```
## 5、易错分析（你原始代码的BUG逐条整理）

```
1、 **函数缺少返回值声明**`Link_L_MidAdd`、`Link_L_DelDest` 原来没有写返回类型 `bool`，属于函数定义不完整。

	修正：`bool Link_L_MidAdd(...)`
```
```
2、 **遍历打印函数错误**


printf("data = %d\n",temp->next);


- 打印的是指针地址，不是结点数据 `temp->data`
- 循环逻辑混乱，容易野指针访问

 	修正：`temp = head->next`，`printf("%d", temp->data)`
```
```
3、 **删除函数没有free内存，内存泄漏**

	原始： `Link_L_DelDest` 找到结点后，只修改指针，**没有free释放被删除结点**。

	修正：`free(DelNode);`
```
```
4、 **中间插入判断越界逻辑错误**


if(temp == NULL){
    perror("position %d is over than bounds\n",dest);


`temp` 是前驱指针，遍历条件 `temp->next && pos < dest` 下 `temp` 永远不会为NULL，这个判断永远进不去，无法拦截超长位置。

	修正：去掉这个错误判断；超出链表长度时，自动插到链表尾部
```