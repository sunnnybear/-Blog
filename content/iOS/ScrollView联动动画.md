## 分析ScrollView联动动画

### 分析动画效果
1. 整个动画是建立在ScrollView的滚动的偏移量的计算上，也就是ScrollView的contentOffset属性
2. 请注意，当ScrollView滚动时，图片并没有跟着ScrollView移动，而是随着偏移量的改变，图片的透明度发生着变化，具体变化关系如下图所示

<p align="center"> 
<img src="https://github.com/sunnnybear/Zeno-Blog/blob/master/content/images/scrollView联动1.png">
</p> 

当ScrollView的偏移量的x值为0的时候，第一张图片的透明度为1；当ScrollView开始滚动，第一张图片的偏移量渐渐变小，当x为width时（width等于屏幕宽度），透明度变为零；与此同时，第二张图片的偏移量则由0变为了1，这也就完成了一次图片切换。