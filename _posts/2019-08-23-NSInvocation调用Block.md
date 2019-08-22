---
layout: post
title: "NSInvocation调用Block"
excerpt: "Block的本质是OC对象，可以通过NSInvocation调用Block"
---
##### 先说一下```NSInvocation ```：我们可以通过NSInvocation调用某个对象的方法。

简单代码示例如下：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    NSMethodSignature*signature = [self methodSignatureForSelector:@selector(tesWithAge:name:)];
    NSInvocation*invocation = [NSInvocation invocationWithMethodSignature:signature];
    invocation.target = self;
    NSString*name = @"LH";
    NSInteger age = 18;
    [invocation setSelector:@selector(tesWithAge:name:)];
    [invocation setArgument:&age atIndex:2];
    [invocation setArgument:&name atIndex:3];
    __autoreleasing NSString* returnValue =nil;
    [invocation invoke];
    [invocation getReturnValue:&returnValue];
    NSLog(@"%@",returnValue);
 
    // Do any additional setup after loading the view.
}

-(NSString*)tesWithAge:(NSInteger)age name:(NSString*)name{
    NSLog(@"sdsdds");
    return [NSString stringWithFormat:@"姓名:%@,年龄:%ld",name,(long)age];
}
```

我们可以通过设置target、方法签名、selector,实现调用其他对象的方法。如果方法带参数可以通过设置argument，来传递参数值。有一点需要注意，设置argument的下标是从2开始的。默认0是self,1是_cmd。

如果有返回值，可以通过```getReturnValue:```方法获取到返回值，不过有一点需要注意。如果返回的是OC对象类型，需要加```__autoreleasing``` 防止过度释放或者通过```__bridge```来转化为OC对象。原因是，在arc模式下，getReturnValue：仅仅是从invocation的返回值拷贝到指定的内存地址，如果返回值是一个NSObject对象的话，是没有处理做到内存管理。

##### NSInvocation调用Block

上面实现了关于NSInvocation对其他对象方法的调用，那么如何实现对block的调用呢，首先我们block到底是什么，block的本质就是oc对象。

在Aspects源码中是这样实现的：

```
typedef struct _AspectBlock {
	__unused Class isa;
	AspectBlockFlags flags;
	__unused int reserved;
	void (__unused *invoke)(struct _AspectBlock *block, ...);
	struct {
		unsigned long int reserved;
		unsigned long int size;
		// requires AspectBlockFlagsHasCopyDisposeHelpers
		void (*copy)(void *dst, const void *src);
		void (*dispose)(const void *);
		// requires AspectBlockFlagsHasSignature
		const char *signature;
		const char *layout;
	} *descriptor;
	// imported variables
} *AspectBlockRef;
```

首先定义了一个结构体用来接收block对象。

然后获取到block的签名：

```
static NSMethodSignature *aspect_blockMethodSignature(id block, NSError **error) {
    AspectBlockRef layout = (__bridge void *)block;
	if (!(layout->flags & AspectBlockFlagsHasSignature)) {
        NSString *description = [NSString stringWithFormat:@"The block %@ doesn't contain a type signature.", block];
        AspectError(AspectErrorMissingBlockSignature, description);
        return nil;
    }
	void *desc = layout->descriptor;
	desc += 2 * sizeof(unsigned long int);
	if (layout->flags & AspectBlockFlagsHasCopyDisposeHelpers) {
		desc += 2 * sizeof(void *);
    }
	if (!desc) {
        NSString *description = [NSString stringWithFormat:@"The block %@ doesn't has a type signature.", block];
        AspectError(AspectErrorMissingBlockSignature, description);
        return nil;
    }
	const char *signature = (*(const char **)desc);
   
	return [NSMethodSignature signatureWithObjCTypes:signature];
}
```

中间有一段代码是这样的:

```
void *desc = layout->descriptor;
	desc += 2 * sizeof(unsigned long int);
	if (layout->flags & AspectBlockFlagsHasCopyDisposeHelpers) {
		desc += 2 * sizeof(void *);
    }
```

示例Block: 此bloc会被拷贝到堆区

```
   __block int ss = 10;
    LHblock block = ^(){
        ss++;
    };
```

其实当block被copy到堆区的时候，机构体中增加了copy 和dispose函数作为结构体的成员变量，所以再取signature的时候做了如下操作```	desc += 2 * sizeof(void *);```。即指针多偏移了```2 * sizeof(void *)```。

获取block签名后进行调用：

```
- (BOOL)invokeWithInfo:(id<AspectInfo>)info {
    NSInvocation *blockInvocation = [NSInvocation invocationWithMethodSignature:self.blockSignature];
    NSInvocation *originalInvocation = info.originalInvocation;
    NSUInteger numberOfArguments = self.blockSignature.numberOfArguments;

    // Be extra paranoid. We already check that on hook registration.
    if (numberOfArguments > originalInvocation.methodSignature.numberOfArguments) {
        AspectLogError(@"Block has too many arguments. Not calling %@", info);
        return NO;
    }
    
    // The `self` of the block will be the AspectInfo. Optional.
    if (numberOfArguments > 1) {
        [blockInvocation setArgument:&info atIndex:1];
    }
    
	void *argBuf = NULL;
    for (NSUInteger idx = 2; idx < numberOfArguments; idx++) {
        const char *type = [originalInvocation.methodSignature getArgumentTypeAtIndex:idx];
		NSUInteger argSize;
		NSGetSizeAndAlignment(type, &argSize, NULL);
        
		if (!(argBuf = reallocf(argBuf, argSize))) {
            AspectLogError(@"Failed to allocate memory for block invocation.");
			return NO;
		}
        
		[originalInvocation getArgument:argBuf atIndex:idx];
		[blockInvocation setArgument:argBuf atIndex:idx];
    }
    
    [blockInvocation invokeWithTarget:self.block];
    if (argBuf != NULL) {
        free(argBuf);
    }
    return YES;
}
```

注意点 由于block是没有selector的 所以不需要设置。还有就是给block设置arguments参数时下标是从1开始的，原因是不存在_cmd参数。

如图：

![blocksig.png](https://iwait.me/assets/imgs/blocksig.png)






