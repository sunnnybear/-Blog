## 分析ScrollView联动动画

### 分析动画效果
1. 整个动画是建立在ScrollView的滚动的偏移量的计算上，也就是ScrollView的contentOffset属性
2. 请注意，当ScrollView滚动时，图片并没有跟着ScrollView移动，而是随着偏移量的改变，图片的透明度发生着变化，具体变化关系如下图所示

<p align="center"> 
<img src="https://github.com/sunnnybear/Zeno-Blog/blob/master/content/images/scrollView联动1">
</p> 
