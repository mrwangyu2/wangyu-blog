---
title: 在C语言环境下，编写自己的Vector容器
date: 2014-04-28 09:19:08
tags:
- 算法
---

 

 
由 王宇 原创并发布 ：

最近工作中，需要用标准C去实现一些统计数据的功能。开发过程中没有容器非常不方便，所以自己尝试着编写了一个简单的Vector容器。

一、功能说明：

通过一个例子来说明如何使用这个Vector：
#include "containers.h"
#include <stdio.h>
// 测试用的结构体
typedef struct tagTextStruct
{
  int key;
  char* value;
}TestStruct;

// 失败提示函数
static void ABORT( char *file,int line)
{
  fprintf(stderr ,"*****\n\nABORT\nFile %s Line %d\n**********\n\n",file,line);
  abort();
}
#define Abort () ABORT(__FILE__,__LINE__)

// 摧毁资源
static int destructorFunc( void *v)
{     
  // 获得Vector中的一个元素
  TestStruct* ts = (TestStruct*)v;
  if(ts->value != NULL )
  {
    int l = strlen (ts->value);
    // 释放内存
    free(ts->value);
    ts->value = NULL;
  }

  return 0;          
}

// 遍历并打印Vector中的元素
static void PrintVector(Vector *AL)
{
  size_t i;
  TestStruct* ts;
  // 打印Vector中已有元素，和总容量
  printf("Count %ld, Capacity %ld\n",(long)iVector. Size(AL),(long )iVector.GetCapacity(AL));
  // 遍历Vector
  for (i=0; i<iVector.Size (AL);i++) {
    // 获得Vector中的元素
    ts = (TestStruct*)iVector. GetElement(AL, i);

    printf("%d\n" ,ts->key);
    printf("%s\n" ,ts->value);
  }

  printf("\n" );
}


// 测试Vector
static int testVector(Vector** AL)
{
  int errors=0;
  char* s1 = "ts1 value." ;
  char* s2 = "ts2 value" ;
  char* temp1 = NULL ;
  char* temp2 = NULL ;
  TestStruct ts1, ts2;

  *AL = iVector. Create(sizeof (TestStruct),5);
  // 配置摧毁资源的回调函数，这里的destructorFunc是一个指针函数。
  iVector. SetDestructor(*AL, destructorFunc );

  // 初始化元素，并分配堆空间
  ts1.key = 1;
  temp1 = ( char*)malloc (strlen(s1) + 1);
  if(temp1 == NULL)
  {
    Abort();
  }
  strcpy(temp1, s1);
  ts1.value = temp1;

  ts2.key = 2;
  temp2 = ( char*)malloc (strlen(s2) + 1);
  if(temp2 == NULL)
  {
    Abort();
  }
  strcpy(temp2, s2);
  ts2.value = temp2;
  // 将元素加入到Vecotor中
  iVector. Add(*AL,&ts1);
  iVector. Add(*AL,&ts2);
  // 删除一个Vector
  iVector. Erase(*AL, &ts1);

  // 获得容器中的实际大小
  if (1 != iVector.Size (*AL))
    Abort();

  return errors;
}

// 主程序入口
int main (void)
{
  int errors=0;
  Vector* AL = NULL;
  // 测试Vector
  errors += testVector (&AL);
  // 释放Vector
  iVector.Finalize(*AL);

  return errors;
}




二、Vector的结构：


声明Vector类型：

/*----------------------------------------------------------------------------*/
/* Definition of the vector type                                              */
/*----------------------------------------------------------------------------*/
struct _Vector {
  VectorInterface *VTable;       /* The table of functions */
  size_t count;                  /* number of elements in the array */
  unsigned int Flags;            /* Read-only or other flags */
  size_t ElementSize;            /* Size of the elements stored in this array. */
  void *contents;                /* The contents of the collection */
  size_t capacity;               /* allocated space in the contents vector */
  unsigned timestamp;            /* Incremented at each change */
  CompareFunction CompareFn;     /* Element comparison function */
  ErrorFunction RaiseError;      /* Error function */
  const ContainerAllocator *Allocator;
  DestructorFunction DestructorFn;
} ;


声明Vector接口：
/****************************************************************************
 *           Vectors        Interface                                                 *
 ****************************************************************************/

/* Definition of the functions associated with this type. */
typedef struct tagVector {
  size_t (* Size)(const Vector *AL);
  int (* Clear)(Vector *AL);
  int (* Erase)(Vector *AL,const void *);
  int (* EraseAll)(Vector *AL,const void *);
  int (* Finalize)(Vector *AL);
  int (* Apply)(Vector *AL,int (*Applyfn)( void *element,void * arg),void *arg);
  int (* Equal)(const Vector *first,const Vector *second);
  Vector *(* Copy)(const Vector *AL);
  ErrorFunction (* SetErrorFunction)(Vector *AL,ErrorFunction );
  size_t (* Sizeof)(const Vector *AL);
  Vector *(* Load)(FILE *stream, ReadFunction readFn,void *arg);
  size_t (* GetElementSize)(const Vector *AL);
  int (* Add)(Vector *AL,const void *newval);
  void *(* GetElement)(const Vector *AL,size_t idx);
  int (* EraseAt)(Vector *AL,size_t idx);
  int (* ReplaceAt)(Vector *AL,size_t idx,void *newval);
  int (* IndexOf)(const Vector *AL,const void *data,void *ExtraArgs,size_t *result);

  /* Vector container specific fields */
  size_t (* GetCapacity)(const Vector *AL);
  int (* SetCapacity)(Vector *AL,size_t newCapacity);

  CompareFunction (* SetCompareFunction)(Vector *l,CompareFunction fn);
  int (* Sort)(Vector *AL);
  Vector *(* Create)(size_t elementsize,size_t startsize);
  Vector *(* CreateWithAllocator)(size_t elemsiz,size_t startsiz,const ContainerAllocator *mm);
  int (* CopyElement)(const Vector *AL,size_t idx,void *outbuf);
  void **(* CopyTo)(const Vector *AL);
  int (* Append)(Vector *AL1, Vector *AL2);
  const ContainerAllocator *(* GetAllocator)(const Vector *AL);
  DestructorFunction (* SetDestructor)(Vector *v,DestructorFunction fn);
  int (* SearchWithKey)(Vector *vec,size_t startByte,size_t sizeKey,size_t startIndex,void *item,size_t *result);
  int (* Resize)(Vector *AL,size_t newSize);
} VectorInterface;



内存分配对象：
/************************************************************************** */
/*                                                                          */
/*                          Memory allocation object                        */
/*                                                                          */
/************************************************************************** */
typedef struct tagAllocator {
  void *(* malloc)(size_t);        /* Function to allocate memory */
  void (* free)(void *);           /* Function to release it      */
  void *(* realloc)(void *,size_t);/* Function to resize a block of memory */
  void *(* calloc)(size_t,size_t);
} ContainerAllocator;

三、功能的实现：

  1，创建Vector
static Vector *CreateWithAllocator (size_t elementsize,size_t startsize,const ContainerAllocator *allocator)
{
  Vector *result;
  size_t es;
  // 给Vector结构体分配内存空间。
  /* Allocate space for the array list header structure */
  result = allocator-> malloc(sizeof (*result));// 这个地方的malloc就是stdlib.h中的malloc, 通过指针函数赋予ContainerAllocator结构体。
  // 判断内存分配是否成功。
  if (result == NULL) {
    iError.RaiseError( "iVector.Create",CONTAINER_ERROR_NOMEMORY );
    return NULL ;
  }
  // 初始化内存。
  memset(result,0, sizeof(*result));
  if (startsize == 0)
    startsize = DEFAULT_START_SIZE;

  // 给Vector里面的内容分配内存空间。
  es = startsize * elementsize;              // 元素的个数乘单个元素的大小 ，如果在添加元素时，超出了这个范围会自动扩容。                     
  result->contents = allocator-> malloc(es); // 这个地方的malloc就是stdlib.h中的malloc, 通过指针函数赋予ContainerAllocator结构体。
  // 判断内存分配是否成功。
  if (result->contents == NULL) {
    iError.RaiseError( "iVector.Create",CONTAINER_ERROR_NOMEMORY ); // 这里有一个统一的错误处理接口iError
    allocator-> free(result);  // 这里的free就是stdlib.h中free, 通过指针函数赋予了ContainerAllocator结构体。
    result = NULL;
  }
  else {
    // 初始化内存
    memset(result->contents,0,es);
    // Vector的容量
    result->capacity = startsize;
    // Vector 接口
    result->VTable = &iVector;
    // Vector 中实际拥有的元素数量
    result->ElementSize = elementsize;
    // 元素对比
    result->CompareFn = DefaultVectorCompareFunction;
    result->RaiseError = iError.RaiseError;
    // 内存管理器
    result->Allocator = (ContainerAllocator *)allocator;
  }
  return result;
}

// 具体实现，此函数将会赋予指针函数接口：Vector *(* Create)(size_t elementsize,size_t startsize);
static Vector *Create (size_t elementsize,size_t startsize)
{
  return CreateWithAllocator(elementsize,startsize,CurrentAllocator);
}

2， 向Vector中添加元素：
// 当超出规划大小时(vector->capacity), 将自动增长。
static int grow(Vector *AL)
{
  size_t newcapacity;
  void **oldcontents;
  int r = 1;
  // 自增长为原先的四分之一
  newcapacity = AL->capacity + 1+AL->capacity/4;
  oldcontents = ( void **)AL->contents;
  // 变更内存空间
  AL->contents = AL->Allocator->realloc (AL->contents,newcapacity*AL->ElementSize);
  if (AL->contents == NULL ) {
    NoMemory(AL,"Resize" );                                     // 输出错误。
    AL->contents = oldcontents;
    r = CONTAINER_ERROR_NOMEMORY;
  }
  else {
    // 变更容量
    AL->capacity = newcapacity;
    AL->timestamp++;
  }
  return r;
}

// 具体实现，此函数将会赋予指针函数接口： int (* Add)(Vector *AL,const void *newval);
/*------------------------------------------------------------------------
Procedure:     Add ID:1
Purpose:       Adds an item at the end of the Vector
Input:         The Vector and the item to be added
Output:        The number of items in the Vector or negative if
error
Errors:
------------------------------------------------------------------------*/
static int Add(Vector *AL, const void *newval)
{
  char *p;
  // 判断输入参数是否合法
  if (AL == NULL ) {
    return NullPtrError ("Add");
  }

  if (newval == NULL ) {
    AL->RaiseError( "iVector.Add",CONTAINER_ERROR_BADARG );
    return CONTAINER_ERROR_BADARG ;
  }
  // 扩容
  if (AL->count >= AL->capacity) {
    int r = grow (AL);
    if (r <= 0)
      return r;
  }
  // 将添加的元素分布到内存当中
  p = ( char *)AL->contents;
  p += AL-> count*AL->ElementSize;
  memcpy(p,newval,AL->ElementSize);
  AL->timestamp++;

  ++AL-> count;
  return 1;
}


3，删除Vector中的元素：
// 按照index 删除Vector中的元素
static int EraseAt(Vector *AL,size_t idx)
{
  char *p;
  // 判断输入参数是否合法
  if (AL == NULL) {
    return NullPtrError ("EraseAt");
  }
  if (idx >= AL-> count) {
    AL->RaiseError( "iVector.Erase",CONTAINER_ERROR_INDEX ,AL,idx);
    return CONTAINER_ERROR_INDEX ;
  }
  // 确定元素在内存中的位置
  p = ( char *)AL->contents;
  p += AL->ElementSize * idx;
  // 如果用户定义了催化资源函数，则调用
  if (AL->DestructorFn) {
    AL->DestructorFn(p);
  }
  // 在内存中，移除元素
  if (idx < (AL-> count-1)) {
    memmove(p,p+AL->ElementSize,(AL->count -idx-1)*AL->ElementSize);
  }
  AL-> count--;
  AL->timestamp++;
  return 1;
}


static int EraseInternal(Vector *AL, const void *elem,int all)
{
  size_t i;
  char *p;
  CompareInfo ci;
  int result = CONTAINER_ERROR_NOTFOUND;
  // 判断输入参数是否合法
  if (AL == NULL) {
    return NullPtrError ("Erase");
  }
  if (elem == NULL) {
    AL->RaiseError( "iVector.Erase",CONTAINER_ERROR_BADARG );
    return CONTAINER_ERROR_BADARG ;
  }
  if (AL->Flags & CONTAINER_READONLY)
    return ErrorReadOnly(AL,"Erase" );

restart:
  p = AL->contents;
  ci.ContainerLeft = AL;
  ci.ContainerRight = NULL;
  ci.ExtraArgs = NULL;
  // 遍历Vecotr容器
  for (i=0; i<AL-> count;i++) {
    // 判断删除的内容, 这里的CompareFn 是一个指针函数，用户可以自定义它，并且授予Vector
    if (!AL->CompareFn(p,elem,&ci)) {
      if (i > 0) {
        // 获得元素的内存位置
        p = GetElement(AL,--i);
        result = EraseAt(AL,i+1);
      } else {
        // 删除第一个元素
        result = EraseAt(AL,0);
        // all = 0 则指删除一个，不在遍历Vector
        if (result < 0 || all == 0) return result;
        if (all) goto restart;
      }
      if (all == 0) return result;
      result = 1;
    }
    p += AL->ElementSize;
  }
  return result;

}
// 具体实现，此函数将会赋予指针函数接口： int (* Erase)(Vector *AL,const void *);
static int Erase(Vector *AL, const void *elem)
{
  // 参数0 ，表示只删除一个元素
  return EraseInternal(AL,elem,0);
}

4，确定Vector的大小：
// 具体实现，此函数将会赋予指针函数接口： size_t (* Size)(const Vector *AL);
static size_t Size(const Vector *AL)
{
  // 判断输入参数是否合法
  if (AL == NULL) {
    NullPtrError("Size" );
    return 0;
  }
  // 返回Vector的实际大小
  return AL-> count;
}

5，获得Vector中的元素：
// 具体实现，此函数将会赋予指针函数接口：    void *(* GetElement)(const Vector *AL,size_t idx);
static void *GetElement( const Vector *AL,size_t idx)
{
  char *p;
  // 判断输入参数是否合法
  if (AL == NULL) {
    NullPtrError("GetElement" );
    return NULL ;
  }

  if (idx >=AL-> count ) {
    return  AL->RaiseError("iVector.GetElement" ,CONTAINER_ERROR_INDEX,AL,idx);
  }
  // 获得内容的首地址
  p = AL->contents;
  // 计算元素地址
  p += idx*AL->ElementSize;
  return p;
}

6，释放Vector资源：
// 具体实现，此函数将会赋予指针函数接口： int (* Clear)(Vector *AL);
/*------------------------------------------------------------------------
  Procedure:     Clear ID:1
  Purpose:       Frees all memory from an array list object without
  freeing the object itself
  Input:         The array list to be cleared
  Output:        The number of objects that were in the array list
  Errors:        The array list must be writable
  ------------------------------------------------------------------------*/
static int Clear(Vector *AL)
{
  // 判断输入参数是否合法
  if (AL == NULL) {
    return NullPtrError ("Clear");
  }
  // 判断用户是否配置了摧毁函数，这里的DestructorFn 是一个指针函数。在例子中，就是函数destructorFunc()
  if (AL->DestructorFn) {
    size_t i;
    unsigned char *p = (unsigned char *)AL->contents;
    // 遍历Vector 并且使用DestructorFn 释放每一个元素的资源
    for(i=0; i<AL->count ;i++) {
      AL->DestructorFn(p);
      p += AL->ElementSize;
    }
  }

  AL-> count = 0;
  AL->timestamp = 0;
  AL->Flags = 0;

  return 1;
}

// 具体实现，此函数将会赋予指针函数接口：    int (* Finalize)(Vector *AL);
/*------------------------------------------------------------------------
Procedure:     Finalize ID:1
Purpose:       Releases all memory associated with an Vector,
including the Vector structure itself
Input:         The Vector to be destroyed
Output:        The number of items that the Vector contained or
zero if error
Errors:        Input must be writable
------------------------------------------------------------------------*/
static int Finalize(Vector *AL)
{
  unsigned Flags;
  int result;
  // 判断输入参数是否合法
  if (AL == NULL)
    return CONTAINER_ERROR_BADARG ;
  // 释放元素所拥有的资源
  result = Clear(AL);
  if (result < 0)
    return result;
  // 释放Vector自身分配的内存空间
  if (AL->VTable != &iVector)
    AL->Allocator-> free(AL->VTable);      //这里的free就是stdlib.h中free, 通过指针函数赋予了ContainerAllocator结构体。
  AL->Allocator-> free(AL->contents);
  AL->Allocator-> free(AL);
  return result;
}

四、总结

由于篇幅的原因，这里仅仅介绍了这个Vector的基本功能，即 Create、Add、Erase、Size、GetElement、Finalize 。其他功能，例如：Appent RelaceAt Search IndexOf Sort Resize 等等。就不在这里介绍了， 原因是这篇博客的目的仅仅是为学习和研究C语言的朋友抛砖引玉，我在学习过程中总结了以下几方面相对比较重要的内容，提供给大家参考：


1、充分理解C语言程序，在内存中的分布。堆、栈、数据区等等。
2、内存的分配和释放。
3、指针和指针函数的灵活运用。
4、结构体和指针函数的配合使用，形成接口对象。






