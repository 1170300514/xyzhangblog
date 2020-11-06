---
title: '[GCD]使用dispatch_once实现单例'
typora-copy-images-to: '[GCD]使用dispatch_once实现单例'
date: 2020-11-06 16:02:49
tags:
	- iOS
	- GCD
summary: 写代码时经常看到使用dispatch_once实现的单例模式，恰好《Effective Objective-C 2.0》书中有提到，记录一下
---



## 使用dispatch_once执行只需运行一次的线程安全代码

单例模式（singleton）对 Objective-C 开发者而言并不陌生，常见的实现方式为：在类编写名为`sharedInstance`的方法，该方法只会返回全类共用的单例实例，而不会在每次调用时都创建新的实例。假设有一个类叫`TestClass`，那么这个共享实例一般会这么写：

```objectivec
+ (id)sharedInstance {
    static TestClass *sharedInstance = nil;
    @synchronized(self) {
        if (!sharedInstance) {
            sharedInstance = [[self alloc] init];
        }
    }
    return sharedInstance;
}
```

单例模式很容易引起激励讨论，Objective-C 的单例尤其如此。线程安全是大家争论的主要问题。为了保证线程安全，上述代码将创建单例实例的代码包裹在**同步块**里。无论是好是坏，反正这种实现方式很常用，这样的代码也随处可见。

不过，GCD 引入了一项特性，能使单例实现起来更加容易。所用的函数是：

```objectivec
void dispatch_once(dispatch_once_t *token, dispatch_block_t block);
```

此函数接受类型为`dispatch_once_t`的特殊参数，称其为**标记**（token），此外还接受 block 参数，对于给定 token 来说，该函数保证相关的 block 必定会执行，且仅执行一次。首次调用该函数时，必然会执行 block 的代码，最重要的一点在于，此操作完全是线程安全的。请注意，对于只需执行一次的块来说，每次调用函数时传入的 token 都必须是完全相同的。因此，开发者通常将标记变量声明在`static`或`global`作用域里。

上述实现单例模式所用的`sharedInstance`方法，可以用此函数来改写：

```objectivec
+ (id)sharedInstance {
    static TestClass *sharedInstance;
    static dispatch_once_t onceToken;   // typedef long dispatch_once_t;
    dispatch_once(&onceToken, ^{
        sharedInstance = [[self alloc] init];
    });
    return sharedInstance;
}
```

使用`disptch_once`可以简化代码并且彻底保证线程安全，开发者根本无须担心加锁或同步。所有问题都由 GCD 在底层处理。由于每次调用时都必须使用完全相同的 token，所以 token 要声明成`static`。把该变量定义在`static`作用域中，可以保证编译器在每次执行`sharedInstance`方法时都会复用这个变量，而不会创建新变量。

此外，`dispatch_once`更高效。它没有使用重量级的同步机制，若是那样做的话，每次运行代码前都要获取锁，相反，此函数采用原子访问（atomic access）来查询标记，以判断其所对应的代码原来是否已经执行过。《Effective Objective-C 2.0》的作者在装有 64 位的 Mac OS X 10.8.2 系统的电脑上简单测试了性能，分别采用`@synchronized`方式及`dispatch_once`方式来实现`sharedInstance`，结果显示，后者的速度几乎是前者的 2 倍。

总结：

1. 经常需要编写**只需执行一次的线程安全代码**（thread-safe single-code execution），通过 GCD 提供的`dispatch_once`函数，很容易就能实现此功能。
2. token 应该声明在`static`或`global`中，这样的话，在把只需一次执行的块传给`dispatch_once`函数时，传进去的标记也是相同的。

## 参考资料

[Effective Objective-C 2.0](https://book.douban.com/subject/21370593/)