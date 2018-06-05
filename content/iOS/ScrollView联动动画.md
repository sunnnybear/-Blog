## 分析ScrollView联动动画
<p align="center"> 
<img src="https://github.com/sunnnybear/Zeno-Blog/blob/master/content/images/ScrollViewGif.gif">
</p> 

[动画原作者YouXianMing](https://github.com/YouXianMing/Animations)

### 分析动画效果
- 整个动画是建立在ScrollView的滚动的偏移量的计算上，也就是ScrollView的contentOffset属性

- 请注意，当ScrollView滚动时，图片并没有跟着ScrollView移动，而是随着偏移量的改变，图片的透明度发生着变化，具体变化关系如下图所示，当ScrollView的偏移量的x值为0的时候，第一张图片的透明度为1；当ScrollView开始滚动，第一张图片的偏移量渐渐变小，当偏移量的x为width时（width等于屏幕宽度），透明度变为零；与此同时，第二张图片的偏移量则由0变为了1，这也就完成了一次图片切换。

<p align="center"> 
<img src="https://github.com/sunnnybear/Zeno-Blog/blob/master/content/images/scrollView联动1.png">
</p> 

- 底部的TitleView是由三个button组成，button的点击事件为改变ScrollView的偏移量。这里由于button封装在TitleView内部，所以通过代理方法来实现在视图控制器中改变ScrollView的偏移量

- TitleView上的红色线条的移动，也是根据ScrollView的偏移量计算得出，红色线条的x值等于ScrollView的偏移量除以图片的数量

### 关键代码实现
1.代码的关键部分就是需要一个类来计算偏移量与某一张图片透明度的线性关系  

ScrollComputingValue.h文件
```objc
#import <Foundation/Foundation.h>
#import <UIKit/UIKit.h>

@interface ScrollComputingValue : NSObject

@property (nonatomic)           CGFloat  startValue;    // Y值为0时的偏移量
@property (nonatomic)           CGFloat  midValue;      // Y值为1时的偏移量
@property (nonatomic)           CGFloat  endValue;      // Y值为0时的偏移量

/**
 *  输入scrollView的偏移量
 */
@property (nonatomic)           CGFloat  inputValue;

/**
 *  输出图片的透明度
 */
@property (nonatomic, readonly) CGFloat  outputValue;

/**
 *  接收完 起始，中，终 偏移量后，开始计算
 */
- (void)makeTheSetupEffective;

@end
```

ScrollComputingValue.m文件

```objc
#import "ScrollComputingValue.h"
#import "Math.h"

@interface ScrollComputingValue ()

@property (nonatomic, strong) Math    *frontLine;   // y从0到1的线
@property (nonatomic, strong) Math    *backLine;    // y从01到0的线

@property (nonatomic)         CGFloat  outputValue; // 最终输出的透明度值
@end

@implementation ScrollComputingValue

- (void)makeTheSetupEffective {
    
    // Math 是一个类，传入两个点就可以得到这条线 的 k值 和 b值 （Y = kX + b）
    self.frontLine = [Math mathOnceLinearEquationWithPointA:MATHPointMake(self.startValue, 0) PointB:MATHPointMake(self.midValue, 1)];
    self.backLine  = [Math mathOnceLinearEquationWithPointA:MATHPointMake(self.midValue, 1) PointB:MATHPointMake(self.endValue, 0)];
}

@synthesize inputValue = _inputValue;

- (void)setInputValue:(CGFloat)inputValue {

    _inputValue = inputValue;
    
    if (inputValue <= _startValue) {
        
        _outputValue = 0;
        
    } else if (inputValue <= _midValue) {
        
        _outputValue = _frontLine.k * inputValue + _frontLine.b;
        
    } else if (inputValue <= _endValue) {
        
        _outputValue = _backLine.k * inputValue + _backLine.b;
        
    } else {
        
        _outputValue = 0;
    }
}

@end

```

Math.h

```objc
#import <Foundation/Foundation.h>
#import <UIKit/UIKit.h>

struct MATHPoint {
    
    CGFloat x;
    CGFloat y;
    
}; typedef struct MATHPoint MATHPoint;

static inline MATHPoint MATHPointMake(CGFloat x, CGFloat y) {
    
    MATHPoint p; p.x = x; p.y = y; return p;
}

@interface Math : NSObject

@property (nonatomic) CGFloat  k;
@property (nonatomic) CGFloat  b;

/**
 *  通过点a点b计算一条直线的表达式
 *
 *  @param pointA Point A.
 *  @param pointB Point B.
 *
 *  @return Math object.
 */
+ (instancetype)mathOnceLinearEquationWithPointA:(MATHPoint)pointA PointB:(MATHPoint)pointB;
+ 
@end
```

Math.m

```objc
#import "Math.h"

@implementation Math

+ (instancetype)mathOnceLinearEquationWithPointA:(MATHPoint)pointA PointB:(MATHPoint)pointB {
    
    Math *equation = [[[self class] alloc] init];
    
    CGFloat x1 = pointA.x; CGFloat y1 = pointA.y;
    CGFloat x2 = pointB.x; CGFloat y2 = pointB.y;
    
    equation.k = calculateSlope(x1, y1, x2, y2);
    equation.b = calculateConstant(x1, y1, x2, y2);
    
    return equation;
}

// 计算斜率 （k值）
CGFloat calculateSlope(CGFloat x1, CGFloat y1, CGFloat x2, CGFloat y2) {
    
    if (x2 == x1) {
        
        return 0;
    }
    
    return (y2 - y1) / (x2 - x1);
}

// 计算常量 （b值）
CGFloat calculateConstant(CGFloat x1, CGFloat y1, CGFloat x2, CGFloat y2) {
    
    if (x2 == x1) {
        
        return 0;
    }
    
    return (y1*(x2 - x1) - x1*(y2 - y1)) / (x2 - x1);
}

@end

```

2.底部TitleView的实现

TitleView实际就是自定义的UIButton，仔细观察，TitleView文字的颜色是会随着偏移量的变化而变化的

ScrollTitleView.h
```objc
#import <UIKit/UIKit.h>

@interface ScrollTitleView : UIButton

/**
 *  Title
 */
@property (nonatomic, strong) NSString  *title;

/**
 *  通过这个输入值来改变字体颜色
 */
@property (nonatomic) CGFloat  inputValue;

/**
 *  Build sub view.
 */
- (void)buildSubViews;

@end

```

ScrollTitleView.m
```objc
#import "ScrollTitleView.h"
#import "UIFont+Fonts.h"
#import "UIView+GlowView.h"

@interface ScrollTitleView ()

@property (nonatomic, strong) UILabel *frontLabel;
@property (nonatomic, strong) UILabel *backLabel;

@end

@implementation ScrollTitleView

- (void)buildSubViews {
    
    self.backLabel                        = [[UILabel alloc] initWithFrame:self.bounds];
    self.backLabel.text                   = self.title;
    self.backLabel.font                   = [UIFont HeitiSCWithFontSize:16.f];
    self.backLabel.textAlignment          = NSTextAlignmentCenter;
    self.backLabel.textColor              = [[UIColor whiteColor] colorWithAlphaComponent:0.5f];
    self.backLabel.userInteractionEnabled = NO;
    [self addSubview:self.backLabel];
    
    self.frontLabel                        = [[UILabel alloc] initWithFrame:self.bounds];
    self.frontLabel.text                   = self.title;
    self.frontLabel.font                   = [UIFont HeitiSCWithFontSize:16.f];
    self.frontLabel.textAlignment          = NSTextAlignmentCenter;
    self.frontLabel.textColor              = [UIColor redColor];
    self.frontLabel.userInteractionEnabled = NO;
    [self addSubview:self.frontLabel];
}

@synthesize inputValue = _inputValue;

// 字体颜色的变化的实现，其实是创建两个frame相同的Label，然后改变他们的alpha值
- (void)setInputValue:(CGFloat)inputValue {
    
    _inputValue           = inputValue;
    self.frontLabel.alpha = inputValue;
    self.backLabel.alpha  = 1 - inputValue;
}

@end

```

3.得到这个计算类后，我们就可以通过输入偏移量来获得对应图片的透明度值和TitleView的Title的变色效果

创建ScrollComputingValue对象、图片对象和TitleView对象，并把ScrollComputingValue对象存入数组
```objc
 for (int i = 0; i < self.titles.count; i++) {
        
        // Setup pictures.
        UIImageView *imageView        = [[UIImageView alloc] initWithFrame:CGRectMake(0, 0, self.contentView.width, self.contentView.height)];
        imageView.image               = [UIImage imageNamed:_pictures[i]];
        imageView.contentMode         = UIViewContentModeScaleAspectFill;
        imageView.tag                 = kPictureTag + i;
        imageView.layer.masksToBounds = YES;
        [self.contentView addSubview:imageView];
        
        // Setup TitleViews.
        ScrollTitleView *titleView = [[ScrollTitleView alloc] initWithFrame:CGRectMake(i * Width / 3.f, 0, Width / 3.f, TabbarHeight)];
        titleView.title            = self.titles[i];
        titleView.tag              = kTitleViewTag + i;
        [titleView buildSubViews];
        [titleView addTarget:self action:@selector(scrollTitleViewEvents:) forControlEvents:UIControlEventTouchUpInside];
        [self.titlesContentView addSubview:titleView];
        
        // Init values.
        if (i == 0) {
            
            titleView.inputValue = 1.f;
            imageView.alpha      = 1.f;
            
        } else {
        
            titleView.inputValue = 0.f;
            imageView.alpha      = 0.f;
        }
        
        // Setup ScrollComputingValues.
        ScrollComputingValue *value = [ScrollComputingValue new];
        value.startValue            = -Width + i * Width;
        value.midValue              = 0      + i * Width;
        value.endValue              = +Width + i * Width;
        [value makeTheSetupEffective];
        [self.computingValuesArray addObject:value];
    }
```

在ScrollView的代理中计算图片的透明度 并 设置TitleView的透明度
```objc
 - (void)scrollViewDidScroll:(UIScrollView *)scrollView {
    
    CGFloat offsetX = scrollView.contentOffset.x;
    _redView.x      = offsetX / 3.f;
    
    for (int i = 0; i < _titles.count; i++) {
        
        ScrollTitleView      *titleView = [_titlesContentView viewWithTag:kTitleViewTag + i];
        UIImageView          *imageView = [self.contentView viewWithTag:kPictureTag + i];

        ScrollComputingValue *value     = _computingValuesArray[i];

        titleView.inputValue = value.outputValue;
        value.inputValue     = offsetX;
        imageView.alpha      = value.outputValue;
    }
}

```

TitleView的点击事件，改变偏移量值
```objc
- (void)scrollTitleViewEvents:(ScrollTitleView *)titleView {
 
    NSInteger index = titleView.tag - kTitleViewTag;
    [UIView animateWithDuration:0.35f animations:^{
        
        self.scrollView.contentOffset = CGPointMake(index * Width, 0);
    }];
}
```