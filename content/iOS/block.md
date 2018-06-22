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
        int (^mallocBlock)(int) = ^(int num) {
            return multiplier * num;
        };
        multiplier = 100;
        NSLog(@"%d",mallocBlock(2));
    }
    {
        static int multiplier = 10;
        int (^mallocBlock)(int) = ^(int num) {
            return multiplier * num;
        };
        multiplier = 100;
        NSLog(@"%d",mallocBlock(2));
    }
    {
        __block int multiplier = 10;
        int (^mallocBlock)(int) = ^(int num) {
            return multiplier * num;
        };
        multiplier = 100;
        NSLog(@"%d",mallocBlock(2));
    }
```
上边这三个Block都引用了外部的变量，那么打印结果会是怎样的呢？

```objc
2018-06-22 17:25:18.650663+0800 block[33826:3936670] 20
2018-06-22 17:25:18.650868+0800 block[33826:3936670] 200
2018-06-22 17:25:18.651261+0800 block[33826:3936670] 200
```

