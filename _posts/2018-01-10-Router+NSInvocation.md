---
layout:     post
title:      "项目中使用NSInvocation实现模块解耦"
subtitle:   ""
date:       2018-01-10
author:     "wuhh"
header-img: "img/blog-header/SetupGithubPages-header.jpeg"
catalog: true
tags:
    - iOS
---

# 前言
项目发展到一定阶段，为了更好的开发和维护，一般都会对项目的模块进行拆分，解耦，达到组件化的目的。要实现组件化，网上很非常多的资料。本文就对目前项目中使用的组件化进行了整理，也算记录自己的学习成果吧，同时为大家提供一种思路。由于个人能力有限，如有错误之处，欢迎大家指正，QQ:1453500137。（太懒了，博客的评论系统没有搞）
[Demo](https://github.com/wuhh1991/WHHInvokeRouterDemo)在这里。

# 整体设计
首先项目中有一个***WHHInvokeRouter***来负责模块与模块之间的调用请求，***WHHInvokeRouter***提供两个方法，根据URL字符串来调用，和根据target与selector调用。`WHHInvokeRouter.h`文件如下：

```objc
@interface WHHInvokeRouter : NSObject

+ (nonnull instancetype)sharedInstance;

+ (nullable id)dispatchInvokes:(nonnull NSString *)target action:(nullable NSString *)action error:(NSError * _Nullable __autoreleasing * _Nullable)error, ...;
+ (nullable id)dispatchInvokesWithUrl:(nonnull NSString *)url;

@end
```

然后模块需要实现一个***WHHInvokeRouter***的分类（category），将开放给外部调用的接口在扩展中声明。例如登录模块，`WHHInvokeRouter+Login.h`中开放了一个登录接口 ***- (void)loginWithUsername:(NSString \*)usernamme Password:(NSInteger)password;*** 

```objc
@interface WHHInvokeRouter (Login)

- (void)loginWithUsername:(NSString *)usernamme Password:(NSInteger)password;
- (NSString *)testDict:(NSDictionary *)dic AndArray:(NSArray *)array andBlock:(void(^)(int status))block;
@end
```
登录模块的所有者是清楚自己提供的接口的 Target 和 selector，通过Router提供的方法，将模块调用请求转发到对应的方法。
`WHHInvokeRouter+Login.m`中代码如下：

```objc
@implementation WHHInvokeRouter (Login)

- (void)loginWithUsername:(NSString *)usernamme Password:(NSInteger)password
{
    [WHHInvokeRouter dispatchInvokes:@"login" action:@"loginWithUsername:Password:" error:nil,usernamme,password];
}

- (NSString *)testDict:(NSDictionary *)dic AndArray:(NSArray *)array andBlock:(void(^)(int status))block
{
   return [WHHInvokeRouter dispatchInvokes:@"login" action:@"testDict:AndArray:andBlock:" error:nil,dic,array,block];
}
@end
```


其他模块如果想要调用登录的话，只需要调用方法就可以了。比如：***[[WHHInvokeRouter sharedInstance] loginWithUsername:@"wuhh" Password:123];*** 

```objc
[[WHHInvokeRouter sharedInstance] loginWithUsername:@"wuhh" Password:123];
```

# Router模块间调用的具体实现

系统给我们提供了 ***- (id)performSelector:(SEL)aSelector withObject:(id)object;*** 方法，但是这个方法最多只接受两个参数，如果参数多余两个的话，就只能用字典等方式来传递了，有可能出现漏传参数的情况。所以我们想到了使用 ***NSInvocation*** 来实现。组织文字不是我的长项，所以还是让代码来说话吧。

方法： ***+ (nullable id)dispatchInvokes:(nonnull NSString \*)target action:(nullable NSString \*)action error:(NSError \* _Nullable __autoreleasing \* _Nullable)error, ...;***     的具体实现如下：

```objc
+ (nullable id)dispatchInvokes:(nonnull NSString *)target action:(nullable NSString *)action error:(NSError * _Nullable __autoreleasing * _Nullable)error, ...
{
    
    NSString *targetName = [NSString stringWithFormat:@"_interface_%@", [target stringByTrimmingCharactersInSet:[NSCharacterSet whitespaceCharacterSet]]];
    Class targetClass = NSClassFromString(targetName);
    
    id targetObject = [[targetClass alloc] init];
    if (!targetObject) {
        NSLog(@"没有找到对应模块，跳转404页面");
        return nil;
    }
    
    SEL actionSelector = NSSelectorFromString([action stringByTrimmingCharactersInSet:[NSCharacterSet whitespaceCharacterSet]]);
    if (![targetObject respondsToSelector:actionSelector]) {
        NSLog(@"对应模块没有相应的接口，跳转错误页面");
        return nil;
    }
    
    NSMethodSignature *methodSignature = [targetClass instanceMethodSignatureForSelector:actionSelector];
    NSInvocation *invocation = [NSInvocation invocationWithMethodSignature:methodSignature];
    invocation.target = targetObject;
    invocation.selector = actionSelector;
    
    // 参数处理
    
    va_list argList;
    va_start(argList, error);
    for (int i = 2; i < [methodSignature numberOfArguments]; i++) {
        const char *argType = [methodSignature getArgumentTypeAtIndex:i];
        
#define WHH_SET_ARGUMENT(_encodeType,_valueType)      \
    if (strncmp(argType, _encodeType, 1) == 0) {                \
        _valueType value = va_arg(argList, _valueType);       \
        [invocation setArgument:&value atIndex:i];  \
        continue;                                      \
    }
        
        WHH_SET_ARGUMENT("c", int);
        WHH_SET_ARGUMENT("C", int);
        WHH_SET_ARGUMENT("s", int);
        WHH_SET_ARGUMENT("S", int);
        WHH_SET_ARGUMENT("i", int);
        WHH_SET_ARGUMENT("I", unsigned int);
        WHH_SET_ARGUMENT("l", long);
        WHH_SET_ARGUMENT("L", unsigned long);
        WHH_SET_ARGUMENT("q", long long);
        WHH_SET_ARGUMENT("Q", unsigned long long);
        WHH_SET_ARGUMENT("f", double);
        WHH_SET_ARGUMENT("d", double);
        WHH_SET_ARGUMENT("B", int);
        WHH_SET_ARGUMENT("@", id);
    }
    va_end(argList);

    [invocation invoke];
    
    id result = nil;
    if (methodSignature.methodReturnLength > 0) { // >0 说明有返回值
        const char *returnType = methodSignature.methodReturnType;
        if( !strcmp(returnType, @encode(id)) ){
            [invocation getReturnValue:&result];
        }
        else
        {

            NSUInteger length = [methodSignature methodReturnLength]; //如果返回值为普通类型NSInteger  BOOL
            
            void *buffer = (void *)malloc(length); //根据长度申请内存
            
            [invocation getReturnValue:buffer];  //为变量赋值
            
            if( !strcmp(returnType, @encode(BOOL)) ) {
                result = [NSNumber numberWithBool:*((BOOL*)buffer)];
            }
            else if( !strcmp(returnType, @encode(NSInteger)) ){
                result = [NSNumber numberWithInteger:*((NSInteger*)buffer)];
            }
            result = [NSValue valueWithBytes:buffer objCType:returnType];
        }
    }
    
    return result;
}

```

利用 ***va_list*** 接收可变个数参数，根据 [OC Type Encodings](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html) 设置参数。另外推荐模块方法返回值类型为void，模块与模块之间通过block来返回数据。另外对于NSInvocation不熟悉的，可以参考这篇[文章](http://blog.csdn.net/onlyou930/article/details/7449102)。

这个Demo只支持模块的方法中参数为常用类型，对于Class、SEL等类型的参数，建议使用字符串。

# 参考

* [OC Type Encodings](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html)
* [NSInvocation简单使用](http://blog.csdn.net/onlyou930/article/details/7449102)
