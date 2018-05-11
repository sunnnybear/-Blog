## 分析 BMDragCellCollectionView

![](https://github.com/sunnnybear/Zeno-Blog/blob/master/content/images/BMDrog.gif)
<div style="align: center">
<img src="https://github.com/sunnnybear/Zeno-Blog/blob/master/content/images/BMDrog.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240"/>
</div>

### 为BMDragCellCollectionView添加长按手势

    - (UILongPressGestureRecognizer *)longGesture {
        if (!_longGesture) {
            _longGesture = [[UILongPressGestureRecognizer alloc] initWithTarget:self action:@selector(handlelongGesture:)];
            _longGesture.minimumPressDuration = _minimumPressDuration;
        }
        return _longGesture;
    }

### 长按手势响应事件

1.确定长按触摸点属于哪个___cell___，并确定其___indexpath___

    CGPoint point = [longGesture locationInView:self]; 
    NSIndexPath *indexPath = [self indexPathForItemAtPoint:point];
 
2.通过 ___switch___语句,分情况讨论触___摸状态开始___、___触摸状态改变___、和___其它___三种情况情况

    switch (longGesture.state) {
        case UIGestureRecognizerStateBegan: {}
            break;
        case UIGestureRecognizerStateChanged: {}
            break;
        default: {}
            break;
        }
    

* 触摸开始（刚按住屏幕）  
    判断手势落点位置是否在___cell___上，没有就break  

        _oldIndexPath = indexPath;
            
        // 没有按在cell 上就 break
        if (_oldIndexPath == nil) {
            break;
        }

    取出被长按的___cell___,并获取其中心点  

        UICollectionViewCell *cell = [self cellForItemAtIndexPath:_oldIndexPath];
        self.oldPoint = cell.center;

    把___snapedView___置___nil___,并创建新___snapedView___(___snapedView___是___cell___图像的一个拷贝，当用户拖动___cell___的时候其实拖动的是___snapedView___，真正的___cell___已经被隐藏了)  

        // 先置nil
        if (_snapedView) {
            _snapedView = nil;
        }
        // 是否外部提供拖拽View
        if (self.delegate && [self.delegate respondsToSelector:@selector(dragCellCollectionView: startDragAtIndexPath:)]) {
            _snapedView = [self.delegate dragCellCollectionView:self startDragAtIndexPath:indexPath];
        }
        if (!_snapedView) {
            // 使用系统截图功能，得到cell的快照view
            _snapedView = [cell snapshotViewAfterScreenUpdates:NO];
        }
        // 设置frame
        _snapedView.frame = cell.frame;
        // 添加到 collectionView 不然无法显示
        [self addSubview:_snapedView];  
        //截图后隐藏当前cell
        cell.hidden = YES;

* 触摸状态改变（开始移动）  
    
    获取当前手指位置  

        // 当前手指位置
        _lastPoint = point;
        // 截图视图位置移动
        [UIView animateWithDuration:0.1 animations:^{
            _snapedView.center = _lastPoint;
        }];  

    当手指移动到新位置时，需更新____currentIndexPath___和___self.oldPoint___的值  

         NSIndexPath *index = [self _getChangedIndexPath];
         _currentIndexPath = index;
        self.oldPoint = [self cellForItemAtIndexPath:_currentIndexPath].center;

    更新数据源  

        // 操作
        [self _updateSourceData];
            
        // 移动 会调用willMoveToIndexPath方法更新数据源
        [self moveItemAtIndexPath:_oldIndexPath toIndexPath:_currentIndexPath];  
    设置移动后起始___indexPath___,并更新其所在___cell___  

        // 设置移动后的起始indexPath
        _oldIndexPath = _currentIndexPath;
        [self reloadItemsAtIndexPaths:@[_oldIndexPath]];

* 其它（移动结束）  

    找到当下的___indexPath___

        UICollectionViewCell *cell = [self cellForItemAtIndexPath:_oldIndexPath];  

    结束动画（移除____snapedView___,___显示cell___）  

        [UIView animateWithDuration:0.25 animations:^{
                if (!cell) {
                    _snapedView.center = _oldPoint;
                } else {
                    _snapedView.center = cell.center;
                }
                _snapedView.transform = CGAffineTransformMakeScale(1.0f, 1.0f);
                _snapedView.alpha = 1.0;
            } completion:^(BOOL finished) {
                // 移除截图视图、显示隐藏的cell并开启交互
                [_snapedView removeFromSuperview];
                _snapedView = nil;
                cell.hidden = NO;
        }];

### 改变indexPath（_getChangedIndexPath）

1.获取触摸点的位置

    CGPoint point = [self.longGesture locationInView:self];  

2.遍历拖拽的___cell___的中心点在哪个___cell___里

    [[self visibleCells] enumerateObjectsUsingBlock:^(__kindof UICollectionViewCell * _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
        if (CGRectContainsPoint(obj.frame, point)) {
            index = [self indexPathForCell:obj];
            *stop = YES;
        }
    }];

3.找到且不是当前___cell___就返回此___indexPath___

    if (index) {
        if ((index.item == self.oldIndexPath.item) && (index.row == self.oldIndexPath.row)) {
            return nil;
        }
        return index;
    }

4.如果触摸点没落在任何___cell___中，则找到最应该交换的___cell___

    [[self visibleCells] enumerateObjectsUsingBlock:^(__kindof UICollectionViewCell * _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
        if (CGRectContainsPoint(obj.frame, point)) {
            index = [self indexPathForCell:obj];
            *stop = YES;
        }
        __strong typeof(weakSelf) self = weakSelf;
        CGPoint p1 = self.snapedView.center;
        CGPoint p2 = obj.center;
        // 计算距离
        CGFloat distance = sqrt(pow((p1.x - p2.x), 2) + pow((p1.y - p2.y), 2));
        if (distance < width) {
            width = distance;
            index = [self indexPathForCell:obj];
        }
    }];

### 处理UIcollectionView数据源（_updateSourceData）

1.获取数据源

    // 获取数据源
    NSMutableArray *array = [self.dataSource dataSourceWithDragCellCollectionView:self].mutableCopy;

2.处理数据

    // ==========处理数据
    BOOL dataTypeCheck = ([self numberOfSections] != 1 || ([self  numberOfSections] == 1 && [array[0] isKindOfClass:NSArray.class]));
    if (dataTypeCheck) {
        for (int i = 0; i < array.count; i ++) {
            [array replaceObjectAtIndex:i withObject:[array[i] mutableCopy]];
        }
    }
    if (_currentIndexPath.section == _oldIndexPath.section) {
        NSMutableArray *orignalSection = dataTypeCheck ? (NSMutableArray *)array[_oldIndexPath.section] : (NSMutableArray *)array;
        if (_currentIndexPath.item > _oldIndexPath.item) {
            for (NSUInteger i = _oldIndexPath.item; i < _currentIndexPath.item ; i ++) {
                [orignalSection exchangeObjectAtIndex:i withObjectAtIndex:i + 1];
            }
        } else {
            for (NSUInteger i = _oldIndexPath.item; i > _currentIndexPath.item ; i --) {
                [orignalSection exchangeObjectAtIndex:i withObjectAtIndex:i - 1];
            }
        }
    } else {
        NSMutableArray *orignalSection = array[_oldIndexPath.section];
        NSMutableArray *currentSection = array[_currentIndexPath.section];
        [currentSection insertObject:orignalSection[_oldIndexPath.item] atIndex:_currentIndexPath.item];
        [orignalSection removeObject:orignalSection[_oldIndexPath.item]];
    }

3.通过代理更新视图控制器的数据源  

    // 更新外面的数据源
    [self.delegate dragCellCollectionView:self newDataArrayAfterMove:array];

### 整体思路
1. 添加长按手势  
2. 判断长按的位置是否在cell 上，如果在，找到其indexPath  
3. 用系统快照功能对这个cell进行快照，然后隐藏cell  
4. 拖动cell到合适位置，松手  
5. 这时要判断松手位置是否有cell，如果有的话获取其indexPath，如果没有，找到离松手点最近的cell  
6. 然后要更新数据源，如果处于同一段，就用类似冒泡排序法似的方法，把oldIndexPath.item设置为起始i，把currentIndexPath.item设置为最终i，用一个for循环不停交换相邻两个数据的位置；如果不处于同一段，则移出的那一段移除数据，移进的那一段插入数据  
7. 最后通过代理方法改变试图控制器的数据源  
