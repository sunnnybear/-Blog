## iOS - Block 

### Block的定义

<p align="center"> 
<img src="https://github.com/sunnnybear/Zeno-Blog/blob/master/content/images/block1.png">
</p> 

上图定义了一个int类型的并且含有一个int类型参数的block。

除此之外，Block还有以下的创建方法：

将定义与实现分开

```objc
int (^myBlock)(int);
    
myBlock = ^(int num){
        return num * num;
};
```

通过typedef创建

```objc

typedef int (^MyBlock)(int);

MyBlock block = ^(int num) {
        return num * num;
};

```

被当做方法中的参数

```objc

- (void)method:(int(^)(int))myBlock 

```
更多用法  
![图片取自http://fuckingblocksyntax.com](https://github.com/sunnnybear/Zeno-Blog/blob/master/content/images/block2.png)

如果实在记不住怎么写block，可以直接在Xcode中打出"inlineBlock"，编译器会自动帮你补全。

### Block的类型

Blokc按照在内从中的位置可以分为NSGlobalBlock，NSStackBlock和NSMallocBlock三种类型

NSGlobalBlock
```obcj
// Global 存放在Text区
    int (^globalBlock)(int) = ^(int num) {
        return num * num;
    };
    NSLog(@"%@",globalBlock);
```
特点：不引用任何外部的变量或者只用到全局变量、静态变量，生命周期从创建到应用结束

NSStackBlock
```obcj
// Static 存放在栈区
int i = 9;
NSLog(@"%@",^{NSLog(@"%d",i);});

```
特点：内部用到局部变量、成员属性变量，生命周期系统控制，函数返回即销毁

NSMallocBlock
```obcj
// Malloc 存放在堆区
int multiplier = 8;
int (^mallocBlock)(int) = ^(int num) {
    return multiplier * num;
};
NSLog(@"%@",mallocBlock);

```
特点：内部用到局部变量、成员属性变量，生命周期系统控制，函数返回即销毁

### Block对外界变量的引用
```objc
    {
        int multiplier = 10;
        int (^myBlock)(int) = ^(int num) {
            return multiplier * num;
        };
        multiplier = 100;
        NSLog(@"基本数据类型 - 局部变量 %d",myBlock(2));
    }
    {
        static int multiplier = 10;
        int (^myBlock)(int) = ^(int num) {
            return multiplier * num;
        };
        multiplier = 100;
        NSLog(@"基本数据类型 - 静态变量 %d",myBlock(2));
    }
    {
        NSString* multiplier = @"10";
        int (^myBlock)(int) = ^(int num) {

            return [multiplier intValue] * num;
        };
        multiplier = @"100";
        NSLog(@"指针类型 - 局部变量 %d",myBlock(2));
    }
    {
        static NSString* multiplier = @"10";
        int (^myBlock)(int) = ^(int num) {
            return [multiplier intValue] * num;
        };
        multiplier = @"100";
        NSLog(@"指针类型 - 静态变量 %d",myBlock(2));
    }
    {
        __block int multiplier = 10;
        int (^myBlock)(int) = ^(int num) {
            return multiplier * num;
        };
        multiplier = 100;
        NSLog(@"带有__block关键字的基本类型 - 局部变量 %d",myBlock(2));
    }
```
上边这些Block都引用了外部的变量，那么打印结果会是怎样的呢？

```objc
2018-06-25 09:18:25.510474+0800 block[49832:7227141] 基本数据类型 - 局部变量 20
2018-06-25 09:18:25.510634+0800 block[49832:7227141] 基本数据类型 - 静态变量 200
2018-06-25 09:18:25.510829+0800 block[49832:7227141] 指针类型 - 局部变量 20
2018-06-25 09:18:25.510954+0800 block[49832:7227141] 指针类型 - 静态变量 200
2018-06-25 09:18:25.511464+0800 block[49832:7227141] 带有__block关键字的基本类型 - 局部变量 200
```
如果使用到了基本数据类型的局部变量，那么block会拷贝该变量的值当做常量使用，在Block被创建后再修改局部变量值，Block使用的值也不会更改；

如果用到基本数据类型的静态变量、全局变量、成员属性变量，那么Block会直接访问其地址，所以外部改变，Block内部值也会改变；

如果用到指针类型的局部变量，Block被创建后再修改局部变量值，Block内部只会用到改变之前的值；

如果用到指针类型的全局变量、静态变量、成员变量属性，Block会强引用该对象，所以Block创建后再改变变量值，Block内部使用的值也会更改；

如果想让Block内部使用到最新的基本数据类型的局部变量，可以在语句前加关键字__block。

### Block循环引用问题

每个对象都有一个计数器，当A对象被创建，它的计数器为1。当有别的对象的指针指向它，那么它的计数器就会加1；

当这个指针不在指向它，那么它的计数器就减1；

当这个对象的计数器变为0时，那么说明这个对象不被任何其它对象引用，那么就可以把这个对象销毁了。

在Block中就可能出现A对象持有B对象的指针，B对象持有A对象的指针，AB对象互相引用，谁都无法释放。

例如：
```objc
Person *p1 = [[Person alloc] init];
    [p1 addLevelWithBlock:^(int level) {
        [p1 fight];
        
    }];
```
这时候编译器会发出警告：

```objc
Capturing 'p1' strongly in this block is likely to lead to a retain cycle
```
这样改写，把p1改成弱引用，警告消除。

```objc
__block Person *p1 = [[Person alloc] init];
    [p1 addLevelWithBlock:^(int level) {
        [p1 fight];
        p1 = nil;
    }];
```

下边这样写也会造成警告：

```objc
self.block = ^{
    [self doSomething];
};
```

可以这样改写

```objc
__block ViewController* weakSelf = self;
self.block = ^{
    [weakSelf doSomething];
};
```

警告消除。

### 参考资料：   
[苹果官方文档](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Blocks/Articles/00_Introduction.html#//apple_ref/doc/uid/TP40007502-CH1-SW1)   
[正确使用Block避免Cycle Retain和Crash](http://tanqisen.github.io/blog/2013/04/19/gcd-block-cycle-retain/)  
[谈Objective-C  block的实现](http://blog.devtang.com/2013/07/28/a-look-inside-blocks/)  
[iOS之轻松上手block](http://ios.jobbole.com/84127/)  
[](http://fuckingblocksyntax.com)
