title: 一个Ｃ语言的泛型哈希表实现
date: 2017-10-30
tags: C hashtable
categories: C
---

# 一个Ｃ语言的泛型哈希表实现
[这里是实现代码](https://github.com/PaulusChen/Kylin/blob/Dev/Kylin/KHashTable.h)

其实不得不说和C相比我还是更喜欢C++一些，很多人似乎认为C的性能要比C++更好，然而C作为C++的子集,C++中如果完全使用C语言语法，似乎可以保证和Ｃ有可以忽略不计的性能差异(函数改名对符号表大小的影响)，C++中可以通过一些技巧把数据结构声明为POD我认为这也充分的体现了C++的设计哲学，在不添加额外负担的前提下,尽可能对语言特性提供更多的支持。我的理解就是如果一些特性你不去使用，它就不该拖慢程序的性能。然而在语法上C++提供了对于一些特性的编译器级别实现，比如多态，我认为很多情况下应该不比一个二流程序员手动实现的版本性能更差，然而C++却在这个基础上提供了一些编译器行为和检查，在编译器就完成了很多工作，减少了实现的工作量和出错误的故障点。所以我比较粉C++。


然而，Ｃ＋＋复杂的语法似乎又成了另一个让人头疼的问题，盘根错节的语法间关系让人生畏。


言归正传，由于最近的项目使用Ｃ语言实现，所以有了上面的这个hash表，实现它的原因就是想尝试对Ｃ语言的哈希表提供类型参数，从而通过编译器的检查机制发现一些可以避免的错误，比如回调函数和对象的类型搞错造成的问题，减少了很多void指针的出现，似乎让我放心很多。先看看使用方法法


``` C
// 被测的存储对象类型
typedef struct {
    uint64_t span;
    double data1;
    uint64_t data2;
    KHListNode node;
    uint32_t Key;
} KHTTest;

// Hash表使用的比较函数
static inline int nodeCmp(const uint64_t *keyA,const uint64_t *keyB) {
    return *keyA - *keyB;
}


// 这里声明了一个表类型KHT,表明了它要存储KHTTest类型的数据结构，数据中使用data2作为key,使用node作为嵌入式数据结构的句柄(为了增加空间局部性提高缓存命中率) 最后指明了使用nodeCmp作为比较函数
HashTemplateDeclar(KHT,KHTTest,data2,node,nodeCmp) {
    return *key;
}

// 这里是需要放到C文件中的hash表实现部分, 由于rehash的实现代码量较大，担心放到insert这么高频使用的函数实现中会造成源代码膨胀
HashTemplateImpl(KHT)

// gtest 的测试case
TEST(KHASH_TABLE,AddAndBasic) {
    KHTable htable;
    KHashTable(KHT,Init)(&htable,1024,0); // 这个是调用KHT的初始化函数，给了hash表的初始参考大小
    KHTTest test1;
    test1.data1 = 19.22;
    test1.data2 = 0x12345678;
    KHashTable(KHT,Insert)(&htable,&test1); // 向hash表插入元素

    KHTTest test2;
    test2.data1 = 20.11;
    test2.data2 = 0x87654321;
    KHashTable(KHT,Insert)(&htable,&test2);

    KHTTest test3;
    test3.data1 = 0.11;
    test3.data2 = 0x43218765;
    KHashTable(KHT,Insert)(&htable,&test3);

    uint64_t key = 0x87654321;
    KHTTest *p = KHashTable(KHT,Search)(&htable,&key);
    ASSERT_DOUBLE_EQ(p->data1,20.11);

    KHashTable(KHT,Delete)(&htable,&test2);
    key = 0x87654321;
    p = KHashTable(KHT,Search)(&htable,&key);
    ASSERT_EQ(NULL,p);

    KHashTable(KHT,Destory)(&htable,NULL,NULL);
}

```

这个hash表实现的主要优点是
1. 头文件依赖，函数调用大多数采用inline，减少了函数调用成本，而且就连回调的比较函数都可以实现为inline的
2. 使用了内核中的嵌入式数据结构，数据结构的指针至于数据结构内部，在hash表间迭代搜索的时候增加了缓存命中率
3. 为数据结构的操作提供了类型信息，让编译器更容易检查到错误。

缺点就是借口有些复杂，两个括号这个语法容易让人蒙圈，一些IDE的补全对齐可能受到影响

