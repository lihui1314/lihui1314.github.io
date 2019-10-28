---
layout: post
title: "GCD实现支持设置repeat和repeat次数的定时器，能自动识别seletor参数类型并传出所需参数"
excerpt: "GCD实现支持设置repeat和repeat次数的定时器，能自动识别seletor参数类型并传出所需参数"
---
###GCD实现支持repeat和repeat次数的定时器，能自动识别seletor参数类型并传出所需参数
###核心代码如下：

```
-(void)lh_setGCDtimerWithTimer:(dispatch_source_t)timer interval:(uint64_t)interval repeat:(BOOL)repeat repeatCountLimit:(NSInteger)count sel:(SEL)sel{
    if (timer == nil) {
        return;
    }
    __block NSInteger i  = 0;
    dispatch_source_set_timer(timer, DISPATCH_TIME_NOW, interval*NSEC_PER_SEC, 0.1* NSEC_PER_SEC);
    __weak typeof(timer)wektimer = timer;
    __weak typeof(self)wekSelf = self;
    dispatch_source_set_event_handler(timer, ^{
        if (count > 0) {
            if (count - i == 0) {
                dispatch_cancel(wektimer);
            }
        }
        NSMethodSignature *methodSignature = [[wekSelf class] instanceMethodSignatureForSelector:sel];
        if (methodSignature.numberOfArguments == 2) {
            IMP imp = [wekSelf methodForSelector:sel];
            void (*func)(id, SEL) = (void *)imp;
            func(wekSelf,sel);
        }else if (methodSignature.numberOfArguments == 3){
            const char* type = [methodSignature getArgumentTypeAtIndex:2];
            if (type[0] == 'q' || type[0] == 'i') {
                IMP imp = [wekSelf methodForSelector:sel];
                void (*func)(id, SEL,NSInteger) = (void *)imp;
                func(wekSelf,sel,i);
            }
            if (type[0] == '@') {
                IMP imp = [wekSelf methodForSelector:sel];
                void (*func)(id, SEL,id) = (void *)imp;
                func(wekSelf,sel,@{@"i":@(i),@"count":@(count)});
            }
            
        }else if (methodSignature.numberOfArguments >3){
            const char* typeF = [methodSignature getArgumentTypeAtIndex:2];
            const char* typeN = [methodSignature getArgumentTypeAtIndex:3];
            if ((typeF[0] == 'q' || typeF[0] == 'i' )&& (typeN[0] == 'q' || typeN[0] == 'i' ) ) {
                IMP imp = [wekSelf methodForSelector:sel];
                void (*func)(id, SEL,NSInteger,NSInteger) = (void *)imp;
                func(wekSelf,sel,i,count);
            }else if (typeF[0] == '@' ){
                IMP imp = [wekSelf methodForSelector:sel];
                void (*func)(id, SEL,id) = (void *)imp;
                func(wekSelf,sel,@{@"i":@(i),@"count":@(count)});
            }else if ((typeF[0] == 'i' || typeF[0] == 'q') &&((typeN[0]=='i')&&(typeN[0]!='q'))){
                IMP imp = [wekSelf methodForSelector:sel];
                void (*func)(id, SEL,NSInteger) = (void *)imp;
                func(wekSelf,sel,i);
            }
            
        }
        if (!repeat) {
            dispatch_resume(wektimer);
        }
        i ++;
    });
    dispatch_resume(timer);
}
```

如有错误，请不吝指出。

邮箱：15652628678@163.com
