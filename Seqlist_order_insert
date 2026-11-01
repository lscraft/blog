有序顺序表插入算法

一：题目描述：
已知一个递增有序的顺序表'L'，设计算法，插入一个整型元素 'x'，插入之后顺序表仍然保持递增有序。
-> 顺序表特点：连续内存存储，随机访问；插入元素需要移动后方元素。

二：算法思路
1. 判断是否已满：如果顺序表有效元素数量已经等于容量，无法插入；
2. 查找插入位置：遍历顺序表，找到第一个大于x的元素下标，该下标就是插入位置 ‘pos’；
3. 元素后移：从最后一个有效元素开始，到pos位置为止，所有元素向后挪一位，腾出位置；
4. 写入元素：把x放到pos位置；
5. 有效长度+1，完成插入。

-> 核心：查找循环条件 `pos <= L->len && x > L->data[pos]`
-> len定义：最后一个有效元素的下标，空表时 len = -1

三：完整源码：
/***********************************************************************************************************************************
 * 
 * 
 * 
 * 已知一个顺序表L，其中的元素递增有序排列，设计一个算法，插入一个元素x（x为int型）后保持该顺序表依然递增有序排列
 * 
 * 
 * 
 * 
 * 
 * 
 *   Copyright(c)			2026-10			18228218705@163.com			All right Reserved
 * *********************************************************************************************************************************/
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#define  Print_t  "%d"


typedef int Datatype_t;


//定义一个结构体来表示顺序表
typedef struct SequenceList{
	int 		 size;		//数据表的容量
	int     	 len;		//有效元素下标（最后一个元素下标，空表为-1）
	Datatype_t   *data;  	//数组存储数据
}Sq_List;

//创建并初始化顺序表
void Sq_Init(Sq_List * L,int size)
{
	L ->size = size;
	L -> len = -1;
	L ->data = (Datatype_t*)calloc(L ->size,sizeof(Datatype_t));
  
  //判断是否申请成功
	if(L->data == NULL)
	{
		perror("calloc memory for data Is failed");
		exit(-1);
	}
}

bool Sq_Insert(Sq_List * L,int x)
{	
	//判断顺序表是否满了
	if(L ->len == L->size -1){
		perror("ERROR:The Sequence Is Full ");
		return false;
	}

	//定义一个变量来找x存放的位置
	int pos = 0;

	//查找插入位置
	while(pos <= L ->len && x > L ->data[pos])
	{
		pos++;
	}

	//所有大于等于x的数字往后移动，给x留位置
	for(int i = L ->len;i >=pos;i--)
	{
		L ->data[i + 1] = L ->data[i];
	}

	//把x放入空位置
	L ->data[pos] = x;
	L -> len++;

	return true;
}

//打印顺序表
void Sq_Print(Sq_List * L)
{
	for(int i = 0; i <= L ->len;i++)
	{
		printf(Print_t, L ->data[i]);
		printf(" ");
	}
	printf("\n");
}

//销毁分配的空间
void Sq_Destroy(Sq_List * L)
{
	if(L->data != NULL)
	{
		free(L ->data);
		L ->data = NULL;
	}
	L ->size = 0;
	L ->len = -1;
}

int main(int argc,char * argv[])
{
	//只分配1个顺序表结构体
	Sq_List * L = (Sq_List *)malloc(sizeof(Sq_List));
	if(L == NULL)
	{
		perror("malloc Sq_List failed");
		exit(-1);
	}

	Sq_Init(L,10);

	//输入初始有序元素，输入个数不能超过9
	int n;
	printf("请输入初始有序元素个数（最多9）：");
	scanf("%d",&n);
	if(n <0 || n >9)
	{
		printf("个数非法\n");
		Sq_Destroy(L);
		free(L);
		return -1;
	}
	printf("请递增输入 %d 个整数：",n);
	for(int i = 0; i < n; i ++)
	{
		scanf(Print_t,&L ->data[i]);
	}
	L->len = n - 1;	//更新最后有效下标

	printf("插入前：");
	Sq_Print(L);

	Sq_Insert(L,5);

	printf("插入5之后：");
	Sq_Print(L);

	//先销毁data，再释放结构体
	Sq_Destroy(L);
	free(L);

	return 0;
}

四：测试样例
输入：
  4
  1 3 4 7
输出：
  插入前：1 3 4 7 
  插入后：1 3 4 5 7 

五：复杂度分析
时间复杂度：最好情况：插在表头，移动全部元素；
           最坏情况：插在表尾，不移动；

六：易错分析
  1. 查找循环条件写成 'pos < L->len'，无法插入到表尾；应当改为：'pos <= L->len'
  2. main 中错误分配 10 个 Sq_List 结构体，然而实际上只需要 1 个；
  3. 输入初始数据后，忘记更新 'L->len'，顺序表逻辑长度错乱；
  4. 'L->data == NULL' 是比较，销毁时赋值应该是 'L->data = NULL'；
  5. 内存泄漏：释放结构体之前没有调用销毁函数释放 data 堆内存；
  6. 没有判断结构体本身 malloc 是否成功，直接访问成员

  
