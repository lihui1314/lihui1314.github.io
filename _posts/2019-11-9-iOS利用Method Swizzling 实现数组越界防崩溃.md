---
ayout: post
title: "iOS利用Method Swizzling 实现数组越界防崩溃"
excerpt: "黑魔法"
---
### Method Swizzling 代码
```
+(void)lh_methodSwizzWithCls:(Class)cls clsSeletor:(SEL)clsSel swizzedSel:(SEL)swizzedSel{
    Method clsMetod = class_getInstanceMethod(cls, clsSel);
    Method swizzedMethod = class_getInstanceMethod(cls, swizzedSel);
    if (!cls || !clsMetod || !swizzedMethod) return;//判断是否存在要hook的方法（包括父类）
    BOOL addSuccessful = class_addMethod(cls, clsSel, method_getImplementation(swizzedMethod), method_getTypeEncoding(swizzedMethod));//防止子类中没有重写父类方法或者不存在此方法。
    if (addSuccessful) {
        class_replaceMethod(cls, swizzedSel, method_getImplementation(clsMetod), method_getTypeEncoding(clsMetod));//获取父类imp并且替换 swizzedSel 对应的imp.
    }else{
        method_exchangeImplementations(clsMetod, swizzedMethod);
    }
    
}
```

### 在NSArray 和NSMutableArray load方法中实现hook。

在NSArray分类中代码实现如下：

```
+(void)load{
    [super load];
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        NSString* arrayZeroStr = @"__NSArray0"; //真身 NSArray*arr0 = [[NSArray alloc]init];
        NSString* arraySingleStr = @"__NSSingleObjectArrayI";//真身 NSArray*arr1 = @[@"0"];
        NSString* arrayStr = @"__NSArrayI";// NSArray*arr2 = @[@"0",@"1"]; objets>时类的真身
        [NSObject lh_methodSwizzWithCls:NSClassFromString(arrayZeroStr) clsSeletor:@selector(objectAtIndex:) swizzedSel:@selector(lh_safeArrZero_objectAtIndex:)];
        [NSObject lh_methodSwizzWithCls:NSClassFromString(arraySingleStr) clsSeletor:@selector(objectAtIndex:) swizzedSel:@selector(lh_safeArrSingle_objectAtIndex:)];
        [NSObject lh_methodSwizzWithCls:NSClassFromString(arrayStr) clsSeletor:@selector(objectAtIndex:) swizzedSel:@selector(lh_safeArr_objectAtIndex:)];
        
        
    });
}
-(id)lh_safeArrZero_objectAtIndex:(NSInteger)index{
    if (self.count<=index) {
        NSLog(@"数组越界");
        return nil;
    }
    return [self lh_safeArrZero_objectAtIndex:index];
}
-(id)lh_safeArrSingle_objectAtIndex:(NSInteger)index{
    if (self.count<=index) {
        NSLog(@"数组越界");
        return nil;
    }
    return [self lh_safeArrSingle_objectAtIndex:index];
}
-(id)lh_safeArr_objectAtIndex:(NSInteger)index{
    if (self.count<=index) {
        NSLog(@"数组越界");
        return nil;
    }
    return [self lh_safeArr_objectAtIndex:index];
}
```

NSMutableArray分类中代码实现如下：

```
+(void)load{
//    [super load];
    static dispatch_once_t onceToken;//确保只执行一次
    dispatch_once(&onceToken, ^{
        [NSObject lh_methodSwizzWithCls:NSClassFromString(@"__NSArrayM") clsSeletor:@selector(objectAtIndex:) swizzedSel:@selector(lh_safe_MArr_objAtIndex:)];
        [NSObject lh_methodSwizzWithCls:NSClassFromString(@"__NSArrayM") clsSeletor:@selector(objectAtIndexedSubscript:) swizzedSel:@selector(lh_safeMArr_objectAtIndexedSubscript:)];
    });
}
-(id)lh_safe_MArr_objAtIndex:(NSInteger)index{
    if (index>=self.count) {
        NSLog(@"可变数组下标越界");
        return nil;
    }
    return [self lh_safe_MArr_objAtIndex:index];
}

-(id)lh_safeMArr_objectAtIndexedSubscript:(NSInteger)index{
    if (index >= self.count) {
        NSLog(@"可变数组下标越界");
        return nil;
    }
    
    return [self lh_safeMArr_objectAtIndexedSubscript:index];
}

```

