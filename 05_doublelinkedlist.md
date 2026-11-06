## 1、题目描述

```
实现带头结点双向链表基础接口，包含创建链表、新建结点、头插、尾插、按下标插入、删除指定数值结点、打印链表、销毁链表。
双向链表每个结点包含数据域、前驱指针prev、后继指针next，可以双向遍历。
```

## 2、算法思路

```
1. 采用"带头结点"设计，头结点不存有效数据，统一空链表和非空链表的插入逻辑，减少特殊判断。
2. 新建结点：调用calloc申请内存，数据赋值，prev、next初始化为NULL。
3. 头插：在头结点的后一个位置插入，先连新结点的后继，再修改原首结点的前驱，最后修改头结点后继与新结点前驱。
4. 尾插：循环遍历找到最后一个有效结点，把新结点接到尾部。
5. 指定下标插入：下标从0开始，遍历走到目标位置前一个结点，再修改双向指针；下标越界直接返回失败。
6. 删除指定值结点：遍历查找目标结点，找到后把前后结点互相链接，释放当前结点内存；找不到提示错误。
7. 打印：从第一个有效结点向后遍历输出。
8. 销毁链表：循环逐个释放所有有效结点，最后释放头结点，避免内存泄漏。
9. 所有插入删除都要判断结点是否为尾结点，避免访问NULL->prev造成段错误。
```

## 3、完整源码

```c
/****************************************************************************************************************************
 * 
 * 
 * 
 * 
 * 
 * 
 * 设计双向链表的接口
 * 
 * 
 * 
 * 
 * 
 * Copyright (c)			2026-11				18228218705@163.com			All  right  Reserved
 * 
 * ******************************************************************************************************************************/
#include <stdio.h>
#include <stdlib.h>

//双向链表节点数据类型
typedef int Datatype_t;

//构建双向链表的结点
typedef struct DoubleLinkedList
{
	Datatype_t 				  data;		//结点数据域
	struct DoubleLinkedList  *next;		//结点直接后继的指针域
	struct DoubleLinkedList  *prev;		//结点直接前驱的指针域
}DouLList_t;

//创建空双向链表，带头结点
DouLList_t * DouLList_create(void)
{
	//创建一个头结点，并对头节点申请内存
	DouLList_t * head = (DouLList_t *)calloc(1,sizeof(DouLList_t));
	if( NULL == head)
	{
		perror("Calloc memory for Head Is Failed");
		exit(-1);
	}

	head ->next = NULL;	 
	head ->prev = NULL;

	return head;
}

//创建新的结点，给结点存入指定数据
DouLList_t * DouLList_NewNode(Datatype_t val)
{
	//给新结点申请一块内存空间
	DouLList_t * new_node = (DouLList_t *)calloc(1, sizeof(DouLList_t));
	if(NULL == new_node)
	{
		perror("Calloc memory for NewNode Is Failed");
		exit(-1);
	}
	//给结点数据域赋值
	new_node->data = val;
	//新结点初始前后指针置空
	new_node->next = NULL;
	new_node->prev = NULL;

	return new_node;
}

//头插，在头结点后面插入新结点
int DouLList_HeadInsert(DouLList_t *head, Datatype_t val)
{
	//判断传入的头结点是否为空
	if(NULL == head)
	{
		printf("链表头结点非法！\n");
		return -1;
	}
	//调用函数生成新结点
	DouLList_t *new_node = DouLList_NewNode(val);

	//新结点的后继指向原来头结点后面第一个结点
	new_node->next = head->next;
	//如果链表不是空链表，原来第一个结点的前驱指向新结点
	if(head->next != NULL)
	{
		head->next->prev = new_node;
	}
	//头结点后继更新为新结点
	head->next = new_node;
	//新结点前驱指向头结点
	new_node->prev = head;

	return 0;
}

//尾插，在链表末尾插入新结点
int DouLList_TailInsert(DouLList_t *head, Datatype_t val)
{
	//判断传入的头结点是否为空
	if(NULL == head)
	{
		printf("链表头结点非法！\n");
		return -1;
	}
	//调用函数生成新结点
	DouLList_t *new_node = DouLList_NewNode(val);
	//定义遍历指针，从头部开始找最后一个结点
	DouLList_t *p = head;

	//循环找到最后一个结点
	while(p->next != NULL)
	{
		p = p->next;
	}
	//最后结点的后继指向新结点
	p->next = new_node;
	//新结点的前驱指向原来的尾结点
	new_node->prev = p;

	return 0;
}

//指定下标插入，下标从0开始，下标0代表第一个有效结点
int DouLList_IndexInsert(DouLList_t *head, int index, Datatype_t val)
{
	//判断传入的头结点是否为空
	if(NULL == head)
	{
		printf("链表头结点非法！\n");
		return -1;
	}
	//判断下标是否小于0
	if(index < 0)
	{
		printf("插入下标不能为负数！\n");
		return -1;
	}
	//调用函数生成新结点
	DouLList_t *new_node = DouLList_NewNode(val);
	//定义遍历指针，从头结点开始向后遍历
	DouLList_t *p = head;
	//记录当前走到的下标
	int i = 0;

	//循环走到待插入位置的前一个结点
	while(p->next != NULL && i < index)
	{
		p = p->next;
		i++;
	}
	//如果遍历结束i还小于index，说明下标越界
	if(i < index)
	{
		printf("插入下标超出链表范围！\n");
		free(new_node);
		return -1;
	}
	//修改双向指针完成插入
	new_node->next = p->next;
	if(p->next != NULL)
	{
		p->next->prev = new_node;
	}
	p->next = new_node;
	new_node->prev = p;

	return 0;
}

//删除第一个匹配指定数值的结点
int DouLList_DelVal(DouLList_t *head, Datatype_t val)
{
	//判断传入的头结点是否为空
	if(NULL == head)
	{
		printf("链表头结点非法！\n");
		return -1;
	}
	//定义遍历指针，从第一个有效结点开始查找
	DouLList_t *p = head->next;

	//循环查找数据等于val的结点
	while(p != NULL && p->data != val)
	{
		p = p->next;
	}
	//p为空代表没有找到对应数值结点
	if(NULL == p)
	{
		printf("没有找到值为%d的结点！\n", val);
		return -1;
	}
	//修改前后结点指针，跳过当前要删除结点
	p->prev->next = p->next;
	if(p->next != NULL)
	{
		p->next->prev = p->prev;
	}
	//释放目标结点内存
	free(p);
	p = NULL;

	return 0;
}

//打印双向链表所有有效结点的数据
void DouLList_Print(DouLList_t *head)
{
	//判断传入的头结点是否为空
	if(NULL == head)
	{
		printf("链表头结点非法！\n");
		return;
	}
	//定义遍历指针，从第一个有效结点开始
	DouLList_t *p = head->next;
	printf("链表元素：");
	//循环遍历直到链表末尾
	while(p != NULL)
	{
		printf("%d ", p->data);
		p = p->next;
	}
	printf("\n");

	return;
}

//销毁整个双向链表，释放所有结点内存，包括头结点
void DouLList_Destroy(DouLList_t *head)
{
	//判断传入的头结点是否为空
	if(NULL == head)
	{
		printf("链表头结点非法！\n");
		return;
	}
	//定义遍历指针保存当前结点
	DouLList_t *p = head->next;
	//临时指针用来保存下一个结点地址
	DouLList_t *tmp = NULL;

	//循环逐个释放有效结点
	while(p != NULL)
	{
		tmp = p->next;
		free(p);
		p = tmp;
	}
	//最后释放头结点
	free(head);
	head = NULL;
	return;
}

//测试主函数
int main(int argc, char const *argv[])
{
	//创建带头结点双向链表
	DouLList_t *list = DouLList_create();

	DouLList_TailInsert(list, 10);
	DouLList_TailInsert(list, 20);
	DouLList_HeadInsert(list, 5);
	DouLList_IndexInsert(list, 2, 15);
	DouLList_Print(list);

	DouLList_DelVal(list, 20);
	DouLList_Print(list);

	//销毁链表，释放全部内存
	DouLList_Destroy(list);
	list = NULL;

	return 0;
}
```

## 4、测试样例

```
1. 尾插10、尾插20
2. 头插5
3. 下标2位置插入15
4. 打印链表
5. 删除数值20
6. 再次打印链表
7. 销毁链表释放内存

```

预期输出：

```
链表元素：5 10 15 20 
链表元素：5 10 15 
```

## 5、易错点分析

```
1. 访问`p->prev`的时候，必须判断p是不是首结点；访问`p->next->prev`的时候，必须判断p->next不为NULL，否则会空指针访问直接段错误。
2. 下标插入时如果下标越界，已经申请好的new_node必须free，不然直接内存泄漏。
3. 双向链表相比单链表多维护prev指针
4. 头结点本身不存有效数据，遍历打印的时候一定要从head->next开始，不能直接从head开始。
5. 销毁链表时，一定要先用临时指针保存下一个结点地址，再free当前结点；最后不要忘记释放头结点。
```