---
layout: post
title: "Objective-C实例对象的内存布局"
excerpt: "在OC中实例对象是如何布局的呢，我们都知道每一个实例对象都拥有一个isa指针。通过苹果开源的部分源码我们可以知道isa指针的定义"
---
在OC中实例对象是如何布局的呢，我们都知道每一个实例对象都拥有一个isa指针。通过苹果开源的部分源码我们可以知道isa指针的定义。

### isa指针的定义

```
# if __arm64__
#   define ISA_MASK        0x0000000ffffffff8ULL
#   define ISA_MAGIC_MASK  0x000003f000000001ULL
#   define ISA_MAGIC_VALUE 0x000001a000000001ULL
#   define ISA_BITFIELD                                                      \
      uintptr_t nonpointer        : 1;                                       \
      uintptr_t has_assoc         : 1;                                       \
      uintptr_t has_cxx_dtor      : 1;                                       \
      uintptr_t shiftcls          : 33; /*MACH_VM_MAX_ADDRESS 0x1000000000*/ \
      uintptr_t magic             : 6;                                       \
      uintptr_t weakly_referenced : 1;                                       \
      uintptr_t deallocating      : 1;                                       \
      uintptr_t has_sidetable_rc  : 1;                                       \
      uintptr_t extra_rc          : 19
#   define RC_ONE   (1ULL<<45)
#   define RC_HALF  (1ULL<<18)

# elif __x86_64__
#   define ISA_MASK        0x00007ffffffffff8ULL
#   define ISA_MAGIC_MASK  0x001f800000000001ULL
#   define ISA_MAGIC_VALUE 0x001d800000000001ULL
#   define ISA_BITFIELD                                                        \
      uintptr_t nonpointer        : 1;                                         \
      uintptr_t has_assoc         : 1;                                         \
      uintptr_t has_cxx_dtor      : 1;                                         \
      uintptr_t shiftcls          : 44; /*MACH_VM_MAX_ADDRESS 0x7fffffe00000*/ \
      uintptr_t magic             : 6;                                         \
      uintptr_t weakly_referenced : 1;                                         \
      uintptr_t deallocating      : 1;                                         \
      uintptr_t has_sidetable_rc  : 1;                                         \
      uintptr_t extra_rc          : 8
#   define RC_ONE   (1ULL<<56)
#   define RC_HALF  (1ULL<<7)

# else
#   error unknown architecture for packed isa
# endif

// SUPPORT_PACKED_ISA
#endif
```

这是arm64 和x86_64下的两种定义。

下面是isa联合体中各个变量的含义：

![1570948142423.jpg](https://iwait.me/assets/imgs/1570948142423.jpg)

### 测试一个实例对象

我定义了一个Person类

```
@interface Person : NSObject{

}
@property(nonatomic,assign)int age;
@property(nonatomic,assign)int grade;
@property(nonatomic,assign)int height;
@property(nonatomic,strong)NSString*name;
@property(nonatomic,strong)NSMutableArray*array;

-(void)printVarAddress;//打印类对象变量地址
```

printVarAddress方法实现：

```
-(void)printVarAddress{
    NSLog(@"%p",&_age);
    NSLog(@"%p",&_grade);
    NSLog(@"%p",&_height);
    NSLog(@"%p",&_name);
    NSLog(@"%p",&_array);
}
```

可以看到打印的地址

![1570955029411.jpg](https://iwait.me/assets/imgs/1570955029411.jpg)

可以看到虽然```height```是```int```类型，但是占了8个字节的存储空间。因为```age```和```grade```已经补齐了八个字节所以为了做到内存对齐```height```自己占了八个字节。

下面是一个对象内存布局示意图，由于太长我只截取了一部分：

![屏幕快照 2019-10-13 下午4.20.08.png](https://iwait.me/assets/imgs/屏幕快照 2019-10-13 下午4.20.08.png)

### 还有一个问题

一个对象的真正占用内存比实际用到的内存大8个字节。

我的验证方法代码：

```
Person *p = [[Person alloc]init];
    size_t size = class_getInstanceSize(p.class);
    NSLog(@"%zu",size);
    size_t realSize =  malloc_size( (__bridge const void *)(p));
    NSLog(@"%zu",realSize);
    NSLog(@"%p",p);
    [p printVarAddress];
```

通过```malloc_size```函数获取到的是的48个字节，通过```class_getInstanceSize```函数获取到的是40个字节。

最后，如有错误请不吝指出。

邮箱：15652628678@163.com
