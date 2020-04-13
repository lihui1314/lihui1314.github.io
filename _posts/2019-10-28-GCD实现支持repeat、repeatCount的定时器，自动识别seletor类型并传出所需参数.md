---
layout: post
title: "GCD实现支持设置repeat和repeat次数的定时器，能自动识别seletor参数类型并传出所需参数"
excerpt: "GCD实现支持设置repeat和repeat次数的定时器，能自动识别seletor参数类型并传出所需参数"
---
### GCD实现支持repeat和repeat次数的定时器，能自动识别seletor参数类型并传出所需参数

### 提供接口

```
//默认开启时间为 interval
-(void)lh_setGCDTimerWithTimer:(dispatch_source_t)timer
                      interVal:(uint64_t)interval
                        repeat:(BOOL)repat
                           sel:(SEL)sel;

-(void)lh_setGCDtimerWithTimer:(dispatch_source_t)timer
                      interval:(uint64_t)interval
                        repeat:(BOOL)repeat
              repeatCountLimit:(NSInteger)count
                           sel:(SEL)sel
                      startNow:(BOOL)startNow;

-(void)lh_setGCDtimerWithTimer:(dispatch_source_t)timer
                     startTime:(uint64_t)startTime
                      interval:(uint64_t)interval
                        repeat:(BOOL)repeat
              repeatCountLimit:(NSInteger)count
                           sel:(SEL)sel;
```



### 核心代码如下：

```
/**
 GCD定时器

 @param timer timer
 @param interval interval
 @param startTime 启动定时器时间 为当前时间之后多少秒。
 @param repeat 是否crepeat
 @param countLimit  重复次数限制
 @param sel sel
 @param startNow 是否立即开始，如果不是的话，开始时间为时间间隔
 */
-(void)lh_innerSetGCDtimerWithTimer:(dispatch_source_t)timer startTime:(uint64_t)startTime interval:(uint64_t)interval repeat:(BOOL)repeat repeatCountLimit:(NSInteger)countLimit sel:(SEL)sel startNow:(BOOL)startNow{
    
    uint64_t startT = startTime;
    if (startTime <= 0) {
        startT = interval;
    }
    
    BOOL needRecord = YES;
    if (countLimit == 0 && repeat ==YES) {
        needRecord = NO;
    }
    
    if (timer == nil) {
        return;
    }
    if (repeat == NO) {
        countLimit = 1;
    } else {
        countLimit = (countLimit < 1) ? MAXFLOAT : countLimit;
    }
    int j = 0;
    if (!startNow) {
        j = 1;
    }
    __block NSInteger i  = j;
    dispatch_time_t start = startNow ? DISPATCH_TIME_NOW : dispatch_time(DISPATCH_TIME_NOW, startT *NSEC_PER_SEC);
    dispatch_source_set_timer(timer,start, interval*NSEC_PER_SEC, 0.1* NSEC_PER_SEC);
    __weak typeof(timer)wektimer = timer;
    __weak typeof(self)wekSelf = self;
    NSMethodSignature *methodSignature = [[wekSelf class] instanceMethodSignatureForSelector:sel];
    NSInteger numberOfArguments = methodSignature.numberOfArguments;
    IMP imp = [wekSelf methodForSelector:sel];
    if (!imp) {
        return;
    }
    dispatch_source_set_event_handler(timer, ^{
        IMP imp = [wekSelf methodForSelector:sel];
        if(!imp) return;
        if (countLimit > 0) {
            if (countLimit - i == 0) {
                dispatch_cancel(wektimer);
            }
        }
      
        if (numberOfArguments == 2) {

            void (*func)(id, SEL) = (void *)imp;
            func(wekSelf,sel);
        }else if (numberOfArguments == 3){
            const char* type = [methodSignature getArgumentTypeAtIndex:2];
            if (type[0] == 'q' || type[0] == 'i') {
                void (*func)(id, SEL,NSInteger) = (void *)imp;
                func(wekSelf,sel,i);
            }
            if (type[0] == '@') {
                void (*func)(id, SEL,id) = (void *)imp;
                func(wekSelf,sel,@{@"i":@(i),@"countLimit":@(countLimit)});
            }
            
        }else if (numberOfArguments >3){
            const char* typeF = [methodSignature getArgumentTypeAtIndex:2];
            const char* typeN = [methodSignature getArgumentTypeAtIndex:3];
            if ((typeF[0] == 'q' || typeF[0] == 'i' )&& (typeN[0] == 'q' || typeN[0] == 'i' ) ) {
                void (*func)(id, SEL,NSInteger,NSInteger) = (void *)imp;
                func(wekSelf,sel,i,countLimit);
            }else if (typeF[0] == '@' ){
                void (*func)(id, SEL,id) = (void *)imp;
                func(wekSelf,sel,@{@"i":@(i),@"countLimit":@(countLimit)});
            }else if ((typeF[0] == 'i' || typeF[0] == 'q') &&((typeN[0]=='i')&&(typeN[0]!='q'))){
                void (*func)(id, SEL,NSInteger) = (void *)imp;
                func(wekSelf,sel,i);
            }
            
        }
        if (needRecord) {
           i++;
        }
    });
    dispatch_resume(timer);
}
```

如有错误，请不吝指出。

邮箱：15652628678@163.com
