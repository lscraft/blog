## 1、题目描述

```
实现带头结点的单向循环链表，完成链表基础操作接口：

- 创建空单向循环链表
- 新建结点
- 头插、尾插、按位置中间插入结点
- 按指定数值删除结点
- 遍历打印链表
- 删除链表中数值最小的结点
- 销毁整条链表，释放内存

要求：

1. 头结点不存储有效数据；
2. 空链表状态：头结点的next指向自己；
3. 链表尾部结点next不再置NULL，指向链表第一个有效结点；
4. 增加参数合法性判断，处理空链表、内存申请失败、位置越界等异常场景。

单向循环链表和普通单链表最大区别：不存在以NULL作为遍历结束标记，遍历终止条件是回到起始结点，这个特点也是最容易出错的地方。
```
## 2、算法思路

```
1. 头结点：head作为头结点，不存有效数据；链表为空时 `head->next = head`；
2. 头插：新结点插在头结点之后；需要先找到链表尾结点，修改尾结点的next指向新的首结点，维持循环闭环；
3. 尾插：遍历找到尾结点（尾结点next == 首结点），新结点接在尾部，新结点next指向首结点；
4. 按位置插入：从头结点向后遍历，走到目标位置前一个结点，完成结点插入；需要判断位置是否越界；
5. 按值删除：循环遍历链表查找目标数据，找到目标结点后修改前驱指针，释放结点内存；特殊处理删除首结点的情况，同步更新后继；
6. 遍历打印：从第一个有效结点开始循环输出，回到首结点时停止打印；
7. 删除最小值结点：遍历链表同时记录最小值结点和它的前驱结点，找到后修改前驱指针，释放最小结点；如果最小值结点是首结点，同步更新后继；
8. 链表销毁：循环逐个释放所有有效结点，最后释放头结点，防止内存泄漏。
```

## 3、完整源码（注释风格和你原代码保持一致）

```c
/********************************************************************************************************************************
 * 
 * 
 * 
 * 设计单向循环链表的接口
 * 
 * 
 * 
 * 
 * Copyright(c)		        2026-11		        18228218705@163.com		   	  				All right reserved
 * 
 * 
 * ******************************************************************************************************************************/
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

//链表中的数据元素类型
typedef int DataType_t;

//构造一个结构体来记录链表各项参数
typedef struct CirLinkNode{
	DataType_t  			data; 	//结点的数据域
	struct CirLinkNode	   *next;	//结点的指针域

}CirLink_L_t;

//创建一个空单向循环链表
CirLink_L_t * CirLink_L_create(void)
{
	//1.创建一个头结点
	CirLink_L_t * head = (CirLink_L_t*)calloc(1,sizeof(CirLink_L_t));

	//创建失败
	if(head == NULL){
		perror("calloc memory for Head is failed");
		exit(-1);
	}

	//创建成功,对头结点初始化,指针域指向自身
	head -> next = head;

	//返回头节点地址
	return head;
}

//创建新节点，并初始化
CirLink_L_t * CirLink_L_NewNode(DataType_t data)
{
	//申请内存空间存放新结点
	CirLink_L_t * Node = (CirLink_L_t *)calloc(1,sizeof(CirLink_L_t));
	if(NULL == Node){
		perror("calloc memory for Node is failed");
		return NULL;
	}

	//申请成功
	Node ->data = data;
	//新结点指针域先置空
	Node ->next = NULL;
	return Node;
}

//头插：在头结点后面插入结点
bool CirLink_L_HeadAdd( CirLink_L_t * head,DataType_t data)
{	
	//判断传入的头结点是否为空
	if(head == NULL)
	{
		perror("head is NULL");
		return false;
	}

	//调用函数生成新结点
	CirLink_L_t *Node = CirLink_L_NewNode(data);
	//如果节点创建失败
	if(Node == NULL)
	{
		perror("Node created is Failed");
		return false;
	}

	//链表为空的情况下
	if(head -> next == head)
	{
		head -> next = Node;
		Node -> next = Node;
		return true;
	}
	
	//链表非空，先找到尾结点
	CirLink_L_t * temp = head;

	//循环条件：找到尾结点，尾结点next等于首结点
	while(temp ->next != head ->next)
	{
		temp = temp ->next;
	}

	//新结点的后继指向原来的首结点
	Node -> next = head ->next;

	//尾结点指向新结点，维持闭环
	temp ->next = Node;

	//头结点指向新结点，新结点成为新首结点
	head ->next =Node;

	return true;
}

//尾插：在链表尾部插入结点
bool CirLink_L_tailAdd(CirLink_L_t * head,DataType_t data)
{	
	//判断传入的头结点是否为空
	if(head == NULL)
	{
		perror("head is NULL");
		return false;
	}

	//调用函数生成新结点
	CirLink_L_t *Node = CirLink_L_NewNode(data);
	//如果节点创建失败
	if(Node == NULL)
	{
		perror("Node created is Failed");
		return false;
	}

	//链表为空的情况下
	if(head -> next == head)
	{
		head -> next = Node;
		Node -> next = Node;
		return true;
	}

	//链表非空，寻找尾结点
	CirLink_L_t * temp = head;
	while(temp -> next != head ->next)
	{
		temp = temp ->next;
	}
	
	//尾结点后继指向新结点
	temp ->next = Node;

	//新结点的后继指向首结点，形成循环
	Node ->next = head ->next;
	return true;
}

//中间插入：在dest位置前面插入结点，位置从0开始计数
bool CirLink_L_MidAdd(CirLink_L_t * head,DataType_t data,int dest)
{	
	//判断要找的dest是否小于0，头结点是否非法
	if(head == NULL || dest < 0)
	{
		perror("invalid param");
		return false;
	}

	//调用函数创建一个新结点
	CirLink_L_t *Node = CirLink_L_NewNode(data);

	//用来记录当前走到第几个位置
	int pos = 0;	

	//如果节点创建失败
	if(Node == NULL)
	{
		perror("Node created is Failed");
		return false;
	}

	//链表为空的情况下
	if(head -> next == head)
	{
		//空链表只能在0号位置插入
		if(dest != 0)
		{
			free(Node);
			perror("position out of bounds");
			return false;
		}
		head -> next = Node;
		Node -> next = Node;
		return true;
	}
	
	//链表不为空，从头结点开始向后遍历
	CirLink_L_t * temp = head;
	while(temp -> next != head->next && pos < dest)
	{
		temp = temp ->next;
		pos++;
	} 

	//循环结束后pos不等于dest，代表位置越界
	if(pos != dest){
		free(Node);
		perror("position is over than bounds");
		return false;
	}

	//完成插入操作
	Node -> next= temp ->next;
	temp ->next = Node;
	return true;
}

//删除指定数字：删除第一个匹配data的结点
bool CirLink_L_DelDest(CirLink_L_t * head,DataType_t data)
{	
	//判断传入头结点是否为空
	if(head == NULL)
	{
		perror("head is NULL");
		return false;
	}

	//如果链表为空，删除失败
	if(head -> next == head)
	{
		perror("The Linked List is NULL");
		return false;
	}

	CirLink_L_t * temp = head;
	CirLink_L_t * DelNode;

	//循环链表终止条件：回到首结点
	while(temp ->next != head ->next)
	{
		if(temp->next->data == data)
		{
			DelNode = temp->next;
			temp->next = DelNode->next;
			free(DelNode);
			return true;
		}
		temp = temp->next;
	}
	//单独判断首结点是否是目标结点
	if(temp->next->data == data)
	{
		DelNode = temp->next;
		temp->next = DelNode->next;
		head->next = DelNode->next;
		free(DelNode);
		return true;
	}

	perror("The Data Is Not Found");
	return false;
}

//遍历：打印链表所有结点数据
bool CirLink_L_Print(CirLink_L_t * head)
{
	//判断传入头结点是否为空
	if(head == NULL)
	{
		perror("head is NULL");
		return false;
	}

	//备份链表首结点地址
	CirLink_L_t * temp = head->next;

	if(head ->next == head)
	{
		printf("Current linkedlist Is Empty\n");
		return false;
	}

	//do while适合循环链表，先打印再后移
	do
	{
		printf("data = %d\n", temp->data);
		temp = temp ->next;
	}while(temp != head ->next);

	return true;
}

//删除最小值结点
void CirLink_L_DelMin(CirLink_L_t * head)
{
	//判断头结点为空或者链表为空
	if(head == NULL || head->next == head)
	{
		perror("The LinkedList Is Empty");
		return;
	}

	CirLink_L_t * min_prev; 				  //记录最小值结点直接前驱地址
	CirLink_L_t * min_node = head ->next;	 //记录最小值结点地址

	CirLink_L_t * p = head ->next;           //遍历用指针，从首元结点开始
	CirLink_L_t * p_prev = head;	           //当前节点的直接前驱

	min_prev = p_prev;

	//遍历链表、找到最小值结点
	do
	{
		//比较链表中结点的数据域大小
		if( p->data < min_node ->data)
		{
			min_prev = p_prev;
			min_node = p;
		}
		//指针向后移动
		p_prev = p;
		p = p ->next;

	}while(p != head ->next);

	//删除最小值结点
	min_prev ->next = min_node ->next;

	//如果最小值结点是首元结点，更新后继
	if(head->next == min_node)
	{
		head->next = min_node->next;
	}

	//释放最小值结点内存
	free(min_node);
	min_node = NULL;
}

//销毁整条链表，释放全部结点内存
void CirLink_L_Destroy(CirLink_L_t **head)
{
	//头结点为空直接返回
	if(*head == NULL)
	{
		return;
	}
	//链表为空，直接释放头结点
	if((*head)->next == *head)
	{
		free(*head);
		*head = NULL;
		return;
	}

	CirLink_L_t *p = (*head)->next;
	CirLink_L_t *del = NULL;

	//循环释放每一个有效结点
	while(p != *head)
	{
		del = p;
		p = p->next;
		free(del);
	}
	//释放头结点
	free(*head);
	*head = NULL;
}

int main(int argc,char*argv[])
{
	//创建链表
	CirLink_L_t *plist = CirLink_L_create();

	//测试：尾插10、20
	CirLink_L_tailAdd(plist,10);
	CirLink_L_tailAdd(plist,20);
	//测试：头插5
	CirLink_L_HeadAdd(plist,5);
	//测试：位置2插入15
	CirLink_L_MidAdd(plist,15,2);
	printf("插入后链表\n");
	CirLink_L_Print(plist);

	//测试：删除数值10
	CirLink_L_DelDest(plist,10);
	printf("删除10后\n");
	CirLink_L_Print(plist);

	//测试：删除最小值结点
	CirLink_L_DelMin(plist);
	printf("删除最小值后\n");
	CirLink_L_Print(plist);

	//销毁链表
	CirLink_L_Destroy(&plist);
	return 0;
}
```

## 4、测试样例

```
1. 调用创建接口，生成空单向循环链表
2. 尾插：10、20
3. 头插：5
4. 在下标2的位置插入15
5. 打印当前链表
6. 删除值等于10的结点
7. 删除链表中数值最小的结点
8. 打印当前链表
9. 销毁链表，释放所有内存
```
### 预期输出

```
插入后链表
	data = 5
	data = 10
	data = 15
	data = 20
删除10后
	data = 15
	data = 20
删除最小值后
	data = 15
	data = 20
```

## 5、易错分析

```
1、循环链表遍历终止条件误用 `!= NULL`

单向循环链表结点的next永远不会等于NULL，用NULL作为循环结束条件会直接死循环。
删除函数、找尾结点、找最小值的代码里都出现这个问题，循环链表判断结束要判断是否回到首结点。
```

```
2、没有对传入的head指针判空

调用接口的时候，如果外部传入NULL给head，直接使用`head->next`会触发段错误，程序崩溃。

```

```
3、中间插入越界判断逻辑错误

原代码循环条件写`temp -> next && pos < dest`，然后判断`if(temp == NULL)`，循环链表temp永远不会变成NULL，越界判断永远无法触发，会出现越界插入破坏链表结构。

```

```
4、按值删除存在内存泄漏，没有处理首结点删除场景

1. 找到待删除结点之后，没有free释放结点内存；
2. 如果被删除的结点正好是首结点，没有更新`head->next`，哨兵的后继指针会变成野指针，链表直接损坏。
```

```
5、缺少链表销毁函数

没有destroy接口，反复创建链表不释放，会持续内存泄漏。
```