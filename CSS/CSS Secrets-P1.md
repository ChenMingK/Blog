> 部分文章内容来源：《CSS SECRETS》<br/>
> 配套 demos：[http://play.csssecrets.io/](http://play.csssecrets.io/)

# 背景与边框
## 边框
使用 background-clip 属性控制背景的绘制区域
- border-box：背景被裁剪到边框盒
- padding-box：背景被裁剪到内边距盒
- content-box：背景被裁减到内容框（默认）

[http://www.w3school.com.cn/cssref/pr\_background-clip.asp](http://www.w3school.com.cn/cssref/pr_background-clip.asp)

### 多重边框

box-shadow 方案：利用第四个参数（扩张半径）让投影面积增大或减小，box-shadow 支持逗号分隔语法，我们可以创建任意数量的投影；

有如下一些注意事项：

1.投影不会影响布局，也不受 box-sizing 属性的影响。

2.这种方法创建出的假 “边框” 不会响应鼠标事件，如果这一点非常重要，你可以给 box-shadow 属性加上 inset 关键字，来使投影绘制在元素的内圈。 

3.box-shadow 是层层叠加的

```css
{
  background: yellowgreen;
  box-shadow: 0 0 0 10px #655, 0 0 0 15px deeppink;
}
```

<img src="https://github.com/ChenMingK/ImagesStore/blob/master/imgs/css-secret1.png" width=200px/>

outline 方案：在某些情况下，你可能只需要两层边框，那就可以先设置一层常规边框，再加行 outline（描边）属性来产生外层的边框。

这种方式的优点是边框样式十分灵活（不像 box-shadow 只能模拟实线边框）；

另外，可以通过 outline-offset 属性来控制它跟元素边缘之间的间距。
```css
{
  background: #333;
  border-radius: 5px;
  outline: 2px dashed white;
  outline-offset: -20px;
}

```

<img src="https://github.com/ChenMingK/ImagesStore/blob/master/imgs/css-secret2.png" width=200px/>

### 蚂蚁行军效果
- 蚂蚁行军效果：[展示](http://dabblet.com/gist/f26dddc71730c3847153)
- 分析：[https://www.jianshu.com/p/04b69172ab9e](https://www.jianshu.com/p/04b69172ab9e)
## 背景
### 灵活的背景定位
如果我们想针对容器某个角对背景图片做偏移定位，有如下一些解决方案
- background-position：该属性允许我们指定背景图片距离任意角的偏移量
- background-origin：有时候我们希望偏移与容器的内边距一致，就可以利用这个属性，其规定了 background-position 的偏移是以什么为基准（边距还是内容）

### 渐变
渐变就是一种代码生成的图像，先设置一个最基础的：
```css
{
  background:  linear-gradient(#fb3, #58a);
}
```
<img src="https://github.com/ChenMingK/ImagesStore/blob/master/imgs/css-secret3.png" width=200px/>

然后我们试着把这两个色标拉近一点：
```css
{
  background:  linear-gradient(#fb3 20%, #58a 80%);
}
```
<img src="https://github.com/ChenMingK/ImagesStore/blob/master/imgs/css-secret4.png" width=200px/>

现在容器顶部的 20% 区域被填充为 #fb3 实色，而底部 20% 区域被填充为 #58a 实色，真正的渐变只出现在容器 60% 的高度区域。

>是否可以这么理解？把色标位置值想成在距离顶部多少位置画一条线，只有这两条线之间的部分才有渐变。且最后一个色标位置之后都是该色标颜色。


如果我们把两个色标重合在一起会发生什么？
```css
{
  background: linear-gradient(#fb3 50%, #58a 0);
  /* 第二个色标的位置值设置为 0，那么它的位置就和前一个色标的位置值相同 */
}
```

<img src="https://github.com/ChenMingK/ImagesStore/blob/master/imgs/css-secret5.png" width=200px/>

可以看到：已经没有任何渐变效果了，只有两块实色，各占据 background-image 一半的面积。

### 水平条纹
因为渐变是一种由代码生成的图像，所以我们能像对待其他任何背景图像那样对待它，而且还可以通过 background-size 来调整其尺寸：

```css
{
  /* 左图-没有平铺的情况 */
  background: linear-gradient(#fb3 50%, #58a 0) no-repeat;
  background-size: 100% 30px;

  /* 右图 */
  background: linear-gradient(#fb3 50%, #58a 0);
  background-size: 100% 30px;
}

```

<img src="https://github.com/ChenMingK/ImagesStore/blob/master/imgs/css-secret6.png" width=500px/>

我们还可以创建多种颜色、不等宽的条纹：
```css
{
  background: linear-gradient(#fb3 33.3%, #58a 0, #58a 66.6%, yellowgreen 0);
  background-size: 100% 45px;
}

```

<img src="https://github.com/ChenMingK/ImagesStore/blob/master/imgs/css-secret7.png" width=200px/>

### 垂直条纹
对于垂直条纹，我们需要在开头加上一个额外的参数来指定渐变的方向（默认 to bottom），然后，还需要把 background-size 的值颠倒一下：
```css
{
  background: linear-gradient(to right, /* 或 90deg*/
                              #fb3 50%, #58a 0);
  background-size: 30px 100%;
}

```
<img src="https://github.com/ChenMingK/ImagesStore/blob/master/imgs/css-secret8.png" width=200px/>

### 斜向条纹
对于斜向条纹，我们可以利用 repeating-linear-gradient() 和 repeating-radial-gradient() 

这两个属性是 linear-gradient() 和 radial-gradient() 的循环式加强版，它们的工作方式与之前的类似，只有一点不同：
**色标是无限循环重复的，直到填满整个背景**。（这里我就不记为啥只用 linear-gradient() 不好实现斜向渐变了，感兴趣的自己尝试）

```css
{
  background: repeating-linear-gradient(45deg, #fb3, #58a 30px);
}
```
它相当于下面这样：
```css
{
  background: linear-gradient(45deg, #fb3, #58a 30px,
                                     #fb3 30px, #58a 60px,
                                     #fb3 60px, #58a 90px,
                                     #fb3 90px, #58a 120px,...)
}

```
<img src="https://github.com/ChenMingK/ImagesStore/blob/master/imgs/css-secret9.png" width=200px/>

### 灵活的同色系条纹
大多数情况下，我们想要的条纹图案并不是由差异极大的几种颜色组成的，这些颜色往往属于同一色系。

我们可以把最深的颜色指定为背景色，同时把半透明白色的条纹叠加在背景色之上来得到浅色条纹：
```css
{
  background: #58a;
  background-image: repeating-linear-gradient(30deg,
                    hsla(0, 0%, 100%, .1),
                    hsla(0, 0%, 100%, .1) 15px,
                    transparent 0, transparent 30px);   
}

```
<img src="https://github.com/ChenMingK/ImagesStore/blob/master/imgs/css-secret10.png" width=200px/>

## 实现常用的图案
只使用一个渐变时，我们能创建的图案并不多，当我们把多个渐变图案组合起来，让它们透过彼此的透明区域显现时，神奇的事情就发生了。

**网格**

我们让水平和垂直的条纹叠加起来，就能得到各式各样的网格：
```css
{
  background: white;
  background-image: linear-gradient(90deg,
                    rgba(200, 0, 0, .5) 50%, transparent 0),
                    linear-gradient(
                    rgba(200, 0, 0, .5) 50%, transparent 0);
  background-size: 30px 30px;
}

```
<img src="https://github.com/ChenMingK/ImagesStore/blob/master/imgs/css-secret11.png" width=200px/>

**图纸效果**

```css
{
  background: #58a;
  background-image: 
    linear-gradient(white 1px, transparent 0),
    linear-gradient(90deg, white 1px, transparent 0);
  background-size: 30px 30px;
}
```

<img src="https://github.com/ChenMingK/ImagesStore/blob/master/imgs/css-secret12.png" width=200px/>

**波点**

径向渐变允许我们创建圆形、椭圆，或是它们的一部分。

```css
{
  background: #655;
  background-image: radial-gradient(tan 30%, transparent 0),
                    radial-gradient(tan 30%, transparent 0);
  background-size: 30px 30px;
  background-position: 0 0, 15px 15px; /* 让背景定位错开 */
}
```
<img src="https://github.com/ChenMingK/ImagesStore/blob/master/imgs/css-secret13.png" width=200px/>

为了达到效果，第二层背景的偏移定位值必须是贴片宽高的一半，这意味着如果要改动贴片的尺寸，需要修改四处。这样代码就很难维护，我们可以写一个 mixin：

```scss
@mixin polka($size, $dot, $base, $accent) {
  background: $base;
  background-image:
    radial-gradient($accent $dot, transparent 0),
    radial-gradient($accent $dot, transparent 0);
  background-size: $size $size;
  background-position: 0 0, $size / 2 $size / 2;
}

// 在创建波点图案时，我们就可以这么调用它
@include polka(30px, 30%, #655, tan);
```


# 形状
## border-radius
四个属性值：border-radius：10px 20px 30px 40px;

表示左上角、右上角、右下角、左下角的圆角大小（**顺时针方向**）

三个属性值：border-radius:10px 30px 60px;

第一个值表示左上角，第二个值表示右上角和左下角（对角），第三个值表示右下角。

两个属性值：border-radius:20px 40px;

第一个值表示左上角和右下角，第二个值表示右上角和左下角。

斜杠二组值：第一组值表示水平半径，第二组值表示垂直半径，每组值也可以同时设置1到4个值，规则与上面相同。

```css
{
  border-radius: 2em 1em 4em / 0.5em 3em;
}
```
等价于：
```css
{
  border-top-left-radius: 2em 0.5em;
  border-top-right-radius: 1em 3em;
  border-bottom-right-radius: 4em 0.5em;
  border-bottom-left-radius: 1em 3em;
}
```

一个沿纵轴劈开的椭圆：
```css
{
  border-radius: 100% 0 0 100% / 50%;
}
```
四分之一椭圆：
```css
{
  border-radius: 100% 0 0 0;
}
```
鸡蛋：
```css
.egg {
  width: 120px;
  height: 160px;
  background: #EC0465;
  border-radius: 60px 60px 60px 60px/100px 100px 60px 60px;
}
```
花瓣：
```css
.flower {
  width: 120px;
  height: 120px;
  background: #EC0465;
  border-radius: 60px 60px 0 60px;
}
```
<img src="https://github.com/ChenMingK/ImagesStore/blob/master/imgs/css-secret14.png" />

## 平行四边形
我们可以通过 skew() 的变形属性来对元素进行斜向拉伸，但是这会导致该元素的内容也斜向变形，有没有办法只让容器的形状倾斜，而保持其内容不变呢？

1.嵌套元素方案

我们可以对内容再应用一次反向的 skew() 变形，从而抵消容器的变形效果，不过这意味着我们需要添加额外的 HTML 元素。

```html
<a href="#" class="button">
    <div>Click me</div>
  </a>

.button { transform: skewX(-45deg); }
.button > div { transform: skewX(45deg); }
```
2.伪元素方案

我们把所有的样式（背景、边框等）应用到伪元素上，然后再对伪元素进行变形。

我们希望伪元素保持良好的灵活性，可以自动继承其宿主元素的尺寸，甚至当宿主元素的尺寸是由其内容来决定时仍然如此。

一个简单的办法就是给宿主元素应用 position：relative 样式，并未伪元素设置 position：absolute，然后把所有偏移量设置为零，以便让它在水平和垂直方向上都被拉伸至宿主元素的尺寸。
```css
.button {
  position: relative;
  /* 其他的文字颜色、内边距等样式 */
}
.button::before {
  content: '';
  position: absolute;
  top: 0; right: 0; bottom: 0; left: 0;
  z-index: -1; /* 防止遮住内容 */
  background: #58a;
  transform: skew(45deg);
}
```
<img src="https://github.com/ChenMingK/ImagesStore/blob/master/imgs/css-secret15.png" width=200px/>

## 菱形图片
裁切路径方案：使用 clip-path 属性，它允许我们使用一系列以逗号分隔的坐标点来指定任意的多边形，百分比值会被解析为元素自身的尺寸。
```css
{
  clip-path: polygon(50% 0, 100% 50%, 50% 100%, 0 50%);
}
```
我们甚至可以利用这个属性来实现动画效果：
```css
img {
  clip-path: polygon(50% 0, 100% 50%, 50% 100%, 0 50%);
  transition: 1s clip-path;
}

img:hover {
  clip-path: polygon(0 0, 100% 0, 100% 100%, 0 100%);
}
```

<img src="https://github.com/ChenMingK/ImagesStore/blob/master/imgs/css-secret16.gif" width=300px/>

相关属性值介绍：[https://www.jb51.net/css/601746.html](https://www.jb51.net/css/601746.html)
