## 带有视差效果的无线轮播图

<p align="center"> 
<img src="https://github.com/sunnnybear/Zeno-Blog/blob/master/content/images/infinalloop.gif">
</p> 

[动画原作者YouXianMing](https://github.com/YouXianMing/Animations)

### 分析动画效果
- 首先这个滚动视图是UICollectionView
- “无限”的效果其实是添加很多重复的组，然后把初始的Scroll的初始indexPath定位在中间的某个点
- 视差效果，就是建立起scrollView的indexPath，与cell中图片的frame的线性关系

### 关键代码部分

1.通过NSTimer实现自动滚动

```objc
- (void)setupTimer {
    
    NSTimer *timer = [NSTimer scheduledTimerWithTimeInterval:self.scrollTimeInterval target:self selector:@selector(action)
                                                    userInfo:nil repeats:YES];
    
    [[NSRunLoop mainRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
    _currentTimer = timer;
}

// 记得在结束时候要销毁NSTimer
- (void)invalidateTimer {
    
    [_currentTimer invalidate];
    _currentTimer = nil;
}

// 定时器方法
- (void)action {
    // 判断数据源是否为空
    if (self.models.count == 0) {
        
        return;
    }
    // 获取到能从屏幕上看到的最后一个cell的indexPath
    NSIndexPath *currentIndexPath = [[self.collectionView indexPathsForVisibleItems] lastObject];
    
    // 新的indexPath.row
    NSInteger newRow     = (currentIndexPath.row + 1) % self.models.count;
    // 新的indexPath.sction
    NSInteger newSection = currentIndexPath.section + (newRow == 0 ? 1 : 0);
    
    NSIndexPath *newIndexPath = [NSIndexPath indexPathForRow:newRow inSection:newSection];
    
    // 滚动到新的下标的位置
    [self.collectionView scrollToItemAtIndexPath:newIndexPath
                                atScrollPosition:(self.flowLayout.scrollDirection == UICollectionViewScrollDirectionHorizontal ? UICollectionViewScrollPositionLeft : UICollectionViewScrollPositionTop)
                                        animated:YES];
}

```

2.根据scrollView的偏移量建立cell图片的视差效果
scrollView 代理方法

```objc
- (void)scrollViewDidScroll:(UIScrollView *)scrollView {
    
    [self.collectionView.visibleCells enumerateObjectsUsingBlock:^(__kindof CustomInfiniteLoopCell *obj, NSUInteger idx, BOOL * _Nonnull stop) {
        // 获得每个cell相对于屏幕上的原点
        // 调用 cell的 contentOffSet方法
        CGPoint point = [obj convertPoint:obj.bounds.origin fromView:self];
        [obj contentOffset:point];
    }];

}

```

loopCell.m

```objc
@interface LoopViewCell () {
    
    Math *_math;
}

@property (nonatomic, strong) PlaceholderImageView *imageView;

@end

@implementation LoopViewCell

- (void)setupCollectionViewCell {
    
    self.layer.masksToBounds = YES;
    // 创建一条线性函数 Y = kX + b
    _math = [Math mathOnceLinearEquationWithPointA:MATHPointMake(0, Width / 2.f) PointB:MATHPointMake(Width/2, 0)];
}

- (void)buildSubView {
    
    self.imageView                     = [[PlaceholderImageView alloc] initWithFrame:self.bounds];
    self.imageView.placeholderImage    = [UIImage imageNamed:@"详情默认图"];
    [self addSubview:self.imageView];
}

- (void)loadContent {
        
    self.imageView.urlString = [self.dataModel imageUrlString];
}

- (void)contentOffset:(CGPoint)offset {
    
    [self resetImageViewCenterPoint];
}

- (void)resetImageViewCenterPoint {
    // 得到自己的原点相对于屏幕的位置
    CGPoint point = [self frameOriginFromView:self.window];
    // 根据这个点的变化来改变 imageView的中心点
    self.imageView.centerX = _math.k * point.x + _math.b;
}

@end
```
<p align="center"> 
<img src="https://github.com/sunnnybear/Zeno-Blog/blob/master/content/images/infinaloop.png">
</p> 

