# 前言
在整理[图片相关优化](https://juejin.im/post/5d09c7966fb9a07ebf4b729e)的时候发现自己对 SVG 所知甚少，于是学习了一波，大体总结了一下，整理成文。因为前端开发人员因为细分领域不同，对待 SVG 的态度差别也很大，对于数据可视化这种对 SVG 有高要求的同学这里的介绍估计没啥干货。对于平常较少接触 SVG 的同学，可以对 SVG 有个初步的认识。

这篇文章主要参考 《SVG 精髓》一书。接下来，开始干活。

# 基础

SVG，即可缩放矢量图形（Scalable Vector Graphics），是一种 XML 应用，可以以一种简洁、可移植的形式表示图形信息。

在矢量图形系统中，图像被描述为一系列的形状。矢量图形阅读器接受在指定的坐标集上绘制形状的指令。而不是接受一系列的已经计算好的像素。

由于 SVG 是 XML 程序，所以图像的有关信息被存储为纯文本，而且具有 XML 的开放性、可移植性以及可交互性。

# 视口

## viewport

SVG 的世界就是一张无限大的画布。文档打算使用的画布区域称作视口（viewport）。

概念说起来很玄乎。看个例子，创建一个简答的 SVG：

```xml
<svg width="200" height="200" style="border:solid red 2px">
    <rect width="100" height="100" fill="#4c93cf" stroke="none"/>
</svg>
```

这里的 `width="200" height="200"` 便是用来设置视口的大小；style 设置样式， rect 元素用来画一个矩形。效果如下：

![SVG - viewport](https://raw.githubusercontent.com/XiangnianZhou/blog/master/%E5%9B%BE%E5%BD%A2%E5%9B%BE%E5%83%8F/svg/images/viewport1.png)

## viewbox

语法：

```
viewBox="x, y, width, height"
```

接下来我们改装一下上面的代码，让里面的正方形铺满视口。

```xml
<svg width="200" height="200" style="border:solid red 2px" viewBox="0 0 100 100">
    <rect  width="100" height="100" fill="#4c93cf"/>
</svg>
```

效果如下：
![SVG - viewbox1](https://raw.githubusercontent.com/XiangnianZhou/blog/master/%E5%9B%BE%E5%BD%A2%E5%9B%BE%E5%83%8F/svg/images/viewbox1.png)

相较于上面的代码，这里仅仅多了 viewBox 属性。不过这个 viewBox 属性**解释起来有点麻烦，但是很容易理解**。
看两个例子，然后，结合下面的一段话，基本就可以理解了。

> SVG 就像是我们的显示器屏幕，viewBox 就是截屏工具的那个选择的框框，最终呈现就是把框框选中的内容再次在显示器全屏显示。

简单的理解就是：

1. 从 SVG 中截取一块区域
2. 显示时，缩放这块区域到视口大小

然后，我们再改一下上面的例子，加深一下理解：

```XML
<svg width="200" height="200" style="border:solid red 2px" viewBox="0 0 800 800" >
    <rect  width="400" height="400" fill="#4c93cf"/>
</svg>
```

基本信息：

-   视口： 200 \* 200
-   矩形： 400 \* 400
-   viewBox： 800 \* 800

分析：

1. viewBox 首先截取从(0, 0) 位置截取一个 800 \* 800 的区域；
2. 这个区域缩放到和视口大小相等。

此时，

-   viewBox 长宽缩小为 200， 是原来的 1/4；
-   viewBox 中的矩形的长宽也缩小为原来的 1/4， 即长宽都是 100；
-   矩形的面积（100 \* 100）占视口面积（200 \* 200）的 1/4 （如下图）。

![SVG - viewbox2](https://raw.githubusercontent.com/XiangnianZhou/blog/master/%E5%9B%BE%E5%BD%A2%E5%9B%BE%E5%83%8F/svg/images/viewbox2.png)

上面的例子里，viewport 和 viewBox 的长宽比是相等的，最后的效果很舒服。如果二者长宽比不相等，viewbox 要在 viewport 中显示是要瓢的，我们如何指定 viewbox “瓢”的规则呢？ 答案是 `preserveAspectRatio` 。

## preserveAspectRatio

在解释这个概念之前，我们先看看 CSS 中 background 相关的两个属性（暂且不讨论其他相关属性）：

1. `background-size: cover|contain`:

-   `cover`：图片宽高比不变、铺满整个容器的宽高，而图片多出的部分则会被截掉；
-   `contain`: 图片自身的宽高比不变，缩放至图片自身能完全显示出来，所以容器会有留白区域；

2. `background-position: xpos ypos`
   CSS 的取值更灵活，现在只需回忆 xpos 和 ypos 的这几个有效值， `top`, `center` 和 `bottom` 以及 `left`, `center` 和 `right`。

这两个属性分别指定了背景如何伸缩和如何对齐。在 SVG 中，使用 `preserveAspectRatio` 属性来指定上这两种的行为，语法如下：

```XML
preserveAspectRatio="<align> [<meetOrSlice>]"
```

其中，align 类似 `background-position` 用来指定对齐方式。有效值类似，**xMidYMid**，即指定关于 x 轴和 y 轴的三种对齐方式：min, mid 和 max。这里需要注意的是，两个值是**连在一起**，且要采用**驼峰**命名法，比如： xMidYMin， 表示：viewBox 的最小 x 值对齐 viewport 的左边部， viewBox 的最小 y 值对齐 viewport 的顶边。

meetOrSlice：

-   meet， 保持 viewbox 宽高比，尽可能放大 viewbox，**使其 viewport 内是可见的**。这时，**viewport 可能会出现留白**。
-   slice，保持宽高比，viewbox 尽可能小，但要使 **viewbox 全部覆盖 viewport**。这个时候，可能会使部分 viewbox 超出 viewport，**超出部分会被裁掉**。

因为可以类比 CSS 属性，这里就不提供例子了（主要是我懒。。）

viewbox 带来的变化，其实源自其对坐标系的改变。接下来我们看看 SVG 的坐标系。

# 坐标系

先介绍两个概念：
**初始视口坐标系**，是一个建立在视口上的坐标系。原点(0,0)在视口的左上角，x 轴正向指向右，y 轴正向指向下，初始坐标系中的一个单位等于视口中的一个"像素"。

**用户坐标系**，是建立在 SVG 画布上的坐标系。这个坐标系一开始和视口坐标系完全一样。使用 viewBox 属性，初始用户坐标系统可以变成与视窗坐标系不一样的坐标系，如上面关于 viewBox 属性的第一个例子中，用户坐标系的一个单位等于视口坐标系的两个单位。

通过 viewBox 创建了一个用户坐标系之后，后代元素的定位不再以初始坐标系定位，而是以新建的用户坐标系定位。

## 坐标系统变换

开始具体看这节内容之前先记住一句话： **SVG 是坐标系统变换而不是元素变换**。

SVG 变换包括缩放，移动，倾斜和旋转等，类似 CSS 的 transform，SVG 中使用 **transform 属性**。不同的是

-   CSS 中的变换是相对于元素自身的中心点，而 SVG 中是**相对于坐标系原点**的；
-   CSS 对元素的变换； SVG 是对坐标系的变换：根据变换元素，先复制当前用户坐标系，然后**对用户坐标系副本进行变换**；
-   SVG 变换中不指定单位，默认的单位等于当前用户坐标系单位；
-   CSS 变换还包含 3D 效果，SVG1.1 不支持 3D 效果。

注：貌似 SVG2 版本中可通过类似 transform-origin 属性的形式设置变换的原点；以及支持 CSS3 规范的 CSS 2D 和 CSS 3D 变换。

### translate 位移

语法：

```
transform="translate(<x> [<y>])"
```

其中 y 方向偏移为非必选，默认为 0。

HTML 元素的偏移是相对自己的中心点，SVG 元素 的偏移是相对 SVG 的左上角。
虽然在最后的表现上是一样的。但在渲染机制上有着很大的不同：

-   CSS 中，会先画好元素，然后再发生偏移。
-   SVG 中，先偏移坐标系，然后再渲染元素。

### scale 缩放

语法：

```
transform="scale(<x> [<y>])"
```

其中：
x，表示沿 x 轴的缩放值，所有 x 坐标乘以给定的 x；
y，表示沿 y 轴的缩放值，所有 y 坐标乘以给定的 y；
y 是可选的，如果没有默认与 x 值相同。

HTML 元素的缩放与 SVG 元素的缩放比较。
HTML 中的缩放：

-   相对元素中心点；
-   缩放元素本身；
-   元素定位不发生变化。

SVG 中的缩放：

-   相对 SVG 左上角；
-   缩放坐标系；
-   元素被重新定位。

### skew 斜切

语法：

```
transform="skewX(<a>)"

transform="skewY(<a>)"
```

其中，

-   skewX 声明一个沿 x 轴的倾斜；
-   skewY 声明一个沿 y 轴的倾斜。

同其他的 transform 变换，skew 的操作对象也是坐标轴，操作完成后会被重新定位。

### rotate 旋转

语法：

```
transform="rotate(<a> [<x> <y>])"
```

其中，

-   a 表示旋转角度，默认是单位是度（deg）；
-   x 和 y 可选，代表旋转中心点；
-   如果没有设置 x，y 的值，默认为当前用户坐标系的原点。

rotate 与上面介绍的另外几个操作不同的是，可以指定旋转中心点，看似和 CSS 中的旋转规则相同，其实：

```
transform="rotate（a x y)"
```

等同于：

```XML
transform="translate(x y) rotate（a) translate(-x -y)"
```

### 变换序列

一个图形对象上可以做多个变换。我们只需将多个变换通过空格或逗号分隔，依次放入 transform 属性即可。

```XML
<rect height="15" width="20" transform="translate(30, 20) scale(2)" fill=" gray" />
```

# 形状

在 SVG 建立坐标系统之后就可以画图了， 下面介绍 SVG 中的形状。

## 矩形

语法：

```XML
<rect x="x" y="y" rx="rx" ry="ry" width="width" height="height" />
```

其中：

-   x，y 指定矩形左上角的位置
-   width 和 height 为矩形的宽高
-   rx 和 ry，表示圆角半径。默认为 0

栗子：

```XML
<svg width="200" height="200">
    <rect width="150" height="80" fill="red" rx="15" ry="15"/>
</svg>
```

![圆角矩形](https://raw.githubusercontent.com/XiangnianZhou/blog/master/%E5%9B%BE%E5%BD%A2%E5%9B%BE%E5%83%8F/svg/images/rect.png)

## 圆形

语法：

```XML
<circle cx="cx" cy="cy" r="r" />
```

其中：

-   cx，cy 指定圆心的位置
-   r 指定半径

## 椭圆

语法：

```XML
<ellipse cx="cx" cy="cy" rx="rx" ry="ry" />
```

其中：

-   rx, ry 分别指定椭圆的 x 半径和 y 半径
-   cx，cy 指定椭圆中心的位置

## 线

语法：

```XML
<line x1="x1" y1="y1" x2="x2" y2="y2" />
```

其中：

-   x1，y1，指定一个点
-   x2，y2， 指定另一个点
-   两点确定一条线

## 折线

语法：

```XML
<polyline points="x1 y1, x2 y2, x3 y3, x4 y4, … , xn yn" />
```

其中：

-   points：点集合
-   点集中的每个数字都可以通过空格，逗号或换行隔开，但是更推荐上面的写法，这样能比较清晰的看出来每个点的坐标

## 多边形

语法：

```XML
<polygon points="x1 y1, x2 y2, x3 y3, x4 y4, … , xn yn" />
```

语法与 polyline 相同，不同的是绘制最后回到第一个点，形成一个闭合图形。

## 路径

上面介绍的所有基本形状都是 `<path>` 元素的简写形式。
`<path>` 元素通过指定一系列相互连接的线、圆弧和曲线绘制任意形状的轮廓。
所有描述的轮廓的数据都放在 `<path>` 元素的 d（data 的缩写）属性中，d 的数据包括命令以及命令对应的坐标信息。

上面关于 `<path>` 的介绍， 让人感觉很玄幻，简单理解**d 属性的值是“命令+参数”的序列**。

-   命令，一个单个字符，用来描述行为
-   参数，一般是指定命令对应的坐标信息

先看一个简单的例子来瞅瞅，好对 d 属性有个最初的印象：

```XML
<svg width="200" height="1200">
    <path d="M20 20 L180 20 L180 80 l-160 0 Z" stroke-width="8" fill="transparent"  stroke="red"/>
</svg>
```

效果如下：

![path 画一个矩形](https://raw.githubusercontent.com/XiangnianZhou/blog/master/%E5%9B%BE%E5%BD%A2%E5%9B%BE%E5%83%8F/svg/images/path-rect.png)

`d="M20 20 L180 20 L180 80 l-160 0 l0 -60"` 中，M，L 和 l 即为**命令**，命令后面便是这个命令作用的坐标。
每个路径都必须以 M 命令（move to）开始。和 Canvas 一样，moveto 命令用来把“笔”移动到一个位置，然后开始“画画”。

在 moveto 命令后面是一个 L（lineto）命令，用来绘制一条线。通过前三条命令画出来了两条线。接下来的 l 命令，还是 lineto 命令，与 L 命令不同的是，l 命令的坐标是相对坐标（相对当前画笔位置）。
其实，每种命令都有两种表示方式：

-   大写字母，表示绝对定位；
-   小写字母，表示相对定位。

最后一个 Z（closepath）命令，用来闭合路径，与第一个点连成线。

画完第一个框框，并且了解路径的基本的套路后，开始了解更多的命令：

### 直线命令

除了上面介绍的 lineto 命令，还有两个简写形式的命令，用来画水平方向和竖直方向的直线。

| 简写形式 | 等价的冗长形式 | 效　果                                      |
| -------- | -------------- | ------------------------------------------- |
| H 20     | L 20 current_y | 绘制一条到绝对位置 (20, current_y) 的线     |
| h 20     | l 20 0         | 绘制一条到 (current_x + 20,current_y) 的线  |
| V 20     | L current_x 20 | 绘制一条到绝对位置 (current_x,20) 的线      |
| v 20     | l 0 20         | 绘制一条到 (current_x, current_y + 20) 的线 |

还有更简洁的形式：
L H 和 V（及其小写）后面都可以跟多个坐标值。跟 `<polyline>` 元素一样，会依次连接起来这些坐标点。

### 曲线命令

绘制平滑曲线的命令分为贝塞尔曲线和弧形（椭圆弧）两种。

#### 椭圆弧

绘制直线段相对简单，因为路径上的两个点就就唯一确定一条直线段。但如果是椭圆弧的话，由于在两个点之间可以绘制无限条弧线，因此我们必须给出额外信息，以在它们之间绘制一条曲线路径。

1. 给出圆弧上的两个点，并且给出椭圆的 x 半径和 y 半径，可以确定出两个椭圆。如下图 a；
2. 此时，两点之间存在 4 个圆弧，两个大弧（角度大于或等于 180 度）两个小弧（角度小于 180 度），通过指定大小弧可以确定出两条圆弧。
3. 由两个大弧和两个小弧的绘制方向不同，可以确定出一条圆弧。

![椭圆弧命令](https://raw.githubusercontent.com/XiangnianZhou/blog/master/%E5%9B%BE%E5%BD%A2%E5%9B%BE%E5%83%8F/svg/images/ellipse-path.jpg)

不过，上面的描述中默认椭圆的 x 轴 和 y 轴都是坐标轴平行的，如果有旋转角，我们还需要知道这个旋转角度才能唯一确定出一条圆弧。

圆弧命令以字母 A （绝对坐标的缩写）或者 a （相对坐标的缩写）开始，后面紧跟以下 7
个参数。

-   点所在椭圆的 x 半径和 y 半径。
-   椭圆的 x 轴旋转角度 x-axis-rotation 。
-   large-arc-flag ，如果需要圆弧的角度小于 180 度，其为 0；如果需要圆弧的角度大于或等于 180 度，则为 1。
-   sweep-flag ，如果需要弧以逆时针绘制则为 0，以顺时针绘制则为 1。
-   终点的 x 坐标和 y 坐标（起点由最后一个绘制的点或者最后一个 moveto 命令确定）。

命令形式：

```XML
A rx ry x-axis-rotation large-arc-flag sweep-flag x y
a rx ry x-axis-rotation large-arc-flag sweep-flag dx dy
```

试着画一下这四个圆弧：

```XML
<svg width="400" height="300">
    <path d="M 125,75 A100,50 0 1,0 225,125" fill="none" stroke="red" stroke-width="3"/>
    <path d="M 125,75 A100,50 0 0,1 225,125" fill="none" stroke="blue"  stroke-width="3"/>
<path d="M 125,75 A100,50 0 1,1 225,125" fill="none" stroke="black" stroke-width="3"/>
    <path d="M 125,75 A100,50 0 0,0 225,125" fill="none" stroke="green"  stroke-width="3"/>
</svg>
```

![画四个圆弧](https://raw.githubusercontent.com/XiangnianZhou/blog/master/%E5%9B%BE%E5%BD%A2%E5%9B%BE%E5%83%8F/svg/images/ellipse-demo.png)

### 贝塞尔曲线

#### 二次贝塞尔曲线

最简单的贝塞尔曲线是二次曲线。不看数学原理的话，简单理解：**我们能通过三个点（起点、终点和控制点）确定一条曲线**。

在 SVG 中，通过在 `<path>` d 属性中使用 Q 或者 q 命令指定一个二次曲线。
绘制曲线起点在 (30, 75)，终点在 (300, 120)，控制点在 (240, 30)，代码如下：

```XML
<svg width="400" height="300">
  <path d="M30 75 Q240 30, 300 120" fill="none" stroke="red" stroke-width="3"/>
</svg>
```

![二次贝塞尔](https://raw.githubusercontent.com/XiangnianZhou/blog/master/%E5%9B%BE%E5%BD%A2%E5%9B%BE%E5%83%8F/svg/images/path-q.png)

二次曲线命令后面可以跟多组坐标，下一组坐标会以上一组坐标的终点为起点绘制曲线。不过这种方法得到的曲线不是平滑曲线，日常中很多场景是希望得到一个平滑曲线。对于这种需求，SVG 提供了流畅的二次曲线命令命令（用 T 或 t 表示）。
T 命令会自动计算控制点的位置，方法是“使新的控制点与上一条命令中的控制点相对于
当前点中心对称”(PS 的钢笔工具很容易模拟这个过程)。因为控制点是自动计算得到的，所以 T 命令只要一组坐标值。

```
<svg width="200" height="200">
  <path d="M30 100 Q 80 30, 100 100 T 200 80"  fill="none" stroke="red" stroke-width="3"/>
</svg>
```

![平滑二次曲线](https://raw.githubusercontent.com/XiangnianZhou/blog/master/%E5%9B%BE%E5%BD%A2%E5%9B%BE%E5%83%8F/svg/images/path-q2.png)

#### 三次贝塞尔曲线

二次曲线和三次曲线之间的区别是三次曲线有两个控制点。这里不讨论其几何原理（原理也不太复杂），三次曲线能呈现更复杂的形状。

![三次贝塞尔曲线](https://raw.githubusercontent.com/XiangnianZhou/blog/master/%E5%9B%BE%E5%BD%A2%E5%9B%BE%E5%83%8F/svg/images/path-c-more.jpg)

三次曲线：

-   使用命令 C 或 c
-   同二次曲线一样，命令后面可以有多组坐标
-   同二次曲线一样，也可以平滑连接多个曲线，使用 S（或 s）命令

在 CSS 中的 `cubic-bezier` 就是用来确定一个三次贝塞尔曲线（Cubic Bézier curves）的。不过 CSS 中确定了起始点(0,0)和终点(1,1)，所以 `cubic-bezier` 只需两个控制点就可以确定一条三次曲线。

接下来模拟一条 CSS 中的三次曲线 `cubic-bezier(0,1, 1,0)`：

```XML
<svg width="200" height="200" viewbox="0 0 100 100">
  <path d="M0 100 C 0,0  100,100 100,0"  fill="none" stroke="red"/>
</svg>
```

![对比](https://raw.githubusercontent.com/XiangnianZhou/blog/master/%E5%9B%BE%E5%BD%A2%E5%9B%BE%E5%83%8F/svg/images/path-c-demo.png)

# 文字

## text 元素

在一个 SVG 文档中，`<text>` 元素内部可以放任何的文字。
如：

```XML
<text>balabala</text>
```

和 CSS 差不多，SVG 中的文本也有很多字体和文本有关的属性，甚至属性名都是完全相同的。如：
font-family、font-style、font-weight、font-variant、font-stretch、font-size、font-size-adjust、kerning、letter-spacing、word-spacing 和 text-decoration。

### 文本位置

语法：

```XML
<text x="x" y="y">balabala</text>
```

其中：

-   x：将文本向右移动 x
-   y：将文本想下移动 y （指定文本基线位置）
-   x 和 y 值是可以带单位的
-   文本移动是相对 SVG 的左上角

另外 x 和 y 还可以接受一个数列：

```XML
<text x="x1 x2 x3 x4 x5 … xn" y="y1 y2 y3 y4 y5 … yn">balabala</text>
```

分别定义每个符号的位置：第一个符号(x1,y1)，第二个符号(x2,y2)...，没有指定的继续使用最后组坐标。

### 文本对齐

```
<text text-anchor="start|middle|end">balabala</text>
```

基于文本坐标位置的对齐方式。

### 文本偏移

```
<text dx="dx" dy="dy">balabala</text>
```

相对文本位置的偏移量。

dx 和 dy 也可以接受一个数列，表示对多个符号的偏移量。

### 文本旋转

```
<text rotate="r">balabala</text>
```

表示对每个字符进行旋转 r 度，其中 r，没有单位，默认单位为 deg。

当然，也可以给 rotate 一个数列，给多个符号指定不同的旋转角度。

注意：如果想对整个文本元素进行旋转，需使用上面提到的转换坐标系的方式`transform="rotate(r)"`。

### 纵向文本

-   `writing-mode：tb` （tb：top to bottom），从上到下。实现文本的纵向排列。
-   `glyph-orientation-vertical` 属性来设置文本（不是整个文本元素）旋转角度。

## tspan 元素

用来标记大块文本的**子部分**，它必须是一个 `<text>` 元素的或别的 `<tspan>` 元素的子元素。

使用场景和 HTML 中的 span 标签很相似，可以用来在一段文本中单独处理一小段文本内容，比如加粗，倾斜和上下标等。

## textPath 元素

利用 xlint:href 属性把字符对齐到路径，让字体可以顺着路径排列。
这是个很骚的操作，必须得有例子：

```XML
<svg width="800" height="800">
    <path id="path" d="M 50 110 A 40 30 0 1 0 230 110 M 250 110" fill="none" stroke="none" />
    <text>
        <textPath xlink:href="#path" font-size="22">
            风 吹 水 面 层 层 浪 ~
        </textPath>
    </text>
</svg>
```

![textPath的例子](https://raw.githubusercontent.com/XiangnianZhou/blog/master/%E5%9B%BE%E5%BD%A2%E5%9B%BE%E5%83%8F/svg/images/textpath-demo.png)

# 样式

## 填充和边框

上面的例子上已经在多次使用了，即是 fill 和 stroke：

-   fill，设置内部颜色
-   stroke，设置线条颜色

色值和 CCS 中相同，可以使用颜色名，也可以使用 rgb，rgba 和 16 进制色值等。

stroke 相关的一些内容需要额外讲一下：

-   stroke-width： 描边的宽度。以路径为中心，向两边扩散绘制。
-   stroke-linecap： 控制边框起点和终点的形状
-   stroke-linejoin：控制两条描边线段之间的连接方式
-   stroke-dasharray： 将虚线类型应用在描边上

---

SVG 提供了四种样式书写的方式：

-   内联样式
-   内部样式表
-   外部样式表
-   表现属性

## 内联样式

```XML
<circle cx="20" cy="20" r="10"
    style="stroke: black; stroke-width: 1.5; fill: blue;
    fill-opacity: 0.6"/>
```

## 内部样式表

内部样式表可以写一些常用的样式，也是使用 `<style>` 标签，不过与 HTML 有些不同：

-   `<style>` 需要定义在 `<defs>` 内；
-   样式表要写在 `<!CDATA[ ]]>` 中。

```XML
<defs>
    <style type="text/css"><![CDATA[
        circle {
            fill: #ffc;
            stroke: blue;
            stroke-width: 2;
            stroke-dasharray: 5 3;
        }
    ]]></style>
</defs>
```

## 外部样式表

同 HTML 一样，SVG 也可以使用外部 CSS 样式表。

```XML
<?xml-stylesheet href="ext_style.css" type="text/css"?>
```

## 表现属性

上面的例子中已经多次使用了表现属性来指定样式，类似：

```xml
<circle cx="10" cy="10" r="5"
fill="red" stroke="black" stroke-width="2"/>
```

表现属性位于优先级列表的最底部，任何来自内联样式，内部样式表或者外部样式表的样式声明都会覆盖表现属性。

注：指定样式的首选是 style 属性或者样式表。

# 分组和引用

除了表示图形的一些元素外，SVG 中还有一波表示文档结构的元素。

## g 元素

`<g>` 元素会**将其所有子元素作为一个组合**，通常组合还会有一个唯一的 id 作为名称。每个组合还可以拥有自己的 `<title>` 和 `<desc>` 来供基于文本的 XML 应用程序识别，或者为视障用户提供更好的可访问性。

## use 元素

`<use>` 元素用于**重用（再次显示）分组或独立图形**。

```XML
<svg viewBox="0 0 1024 1024" width="400" height="100" preserveAspectRatio="xMinYMin meet">
    <g id="shape">
        <path d="M567.1 512l318.5-319.3c5-5 1.5-13.7-5.6-13.7h-90.5c-2.1 0-4.2 0.8-5.6 2.3l-273.3 274-90.2-90.5c12.5-22.1 19.7-47.6 19.7-74.8 0-83.9-68.1-152-152-152s-152 68.1-152 152 68.1 152 152 152c27.7 0 53.6-7.4 75.9-20.3l90 90.3-90.1 90.3C341.6 589.4 315.7 582 288 582c-83.9 0-152 68.1-152 152s68.1 152 152 152 152-68.1 152-152c0-27.2-7.2-52.7-19.7-74.8l90.2-90.5 273.3 274c1.5 1.5 3.5 2.3 5.6 2.3H880c7.1 0 10.7-8.6 5.6-13.7L567.1 512zM288 370c-44.1 0-80-35.9-80-80s35.9-80 80-80 80 35.9 80 80-35.9 80-80 80z m0 444c-44.1 0-80-35.9-80-80s35.9-80 80-80 80 35.9 80 80-35.9 80-80 80z"></path>
    </g>

    <!-- 对上面的分组引用 三 次 -->
    <use xlink:href="#shape" x="1024" fill="#666" y="0" />
    <use xlink:href="#shape" x="2048" fill="#999" y="0" />
    <use xlink:href="#shape" x="3072" fill="#bbb" y="0" />
</svg>
```

效果如下：
![use的例子](https://raw.githubusercontent.com/XiangnianZhou/blog/master/%E5%9B%BE%E5%BD%A2%E5%9B%BE%E5%83%8F/svg/images/use-1.png)

`<use>` 的引用是支持跨 svg 的，甚至是支持跨文件的。

## defs 元素

`<defs>` 主要是用来**定义（但不显示）想要复用的元素**的。

```XML
<svg viewBox="0 0 1024 1024" width="400" height="100" preserveAspectRatio="xMinYMin meet">
    <defs>
        <g id="shape">
            <path d="M567.1 512l318.5-319.3c5-5 1.5-13.7-5.6-13.7h-90.5c-2.1 0-4.2 0.8-5.6 2.3l-273.3 274-90.2-90.5c12.5-22.1 19.7-47.6 19.7-74.8 0-83.9-68.1-152-152-152s-152 68.1-152 152 68.1 152 152 152c27.7 0 53.6-7.4 75.9-20.3l90 90.3-90.1 90.3C341.6 589.4 315.7 582 288 582c-83.9 0-152 68.1-152 152s68.1 152 152 152 152-68.1 152-152c0-27.2-7.2-52.7-19.7-74.8l90.2-90.5 273.3 274c1.5 1.5 3.5 2.3 5.6 2.3H880c7.1 0 10.7-8.6 5.6-13.7L567.1 512zM288 370c-44.1 0-80-35.9-80-80s35.9-80 80-80 80 35.9 80 80-35.9 80-80 80z m0 444c-44.1 0-80-35.9-80-80s35.9-80 80-80 80 35.9 80 80-35.9 80-80 80z"></path>
        </g>
    </defs>

    <!-- 引用 四 次 -->
    <use xlink:href="#shape" x="0" fill="#333" y="0" />
    <use xlink:href="#shape" x="1024" fill="#666" y="0" />
    <use xlink:href="#shape" x="2048" fill="#999" y="0" />
    <use xlink:href="#shape" x="3072" fill="#bbb" y="0" />
</svg>
```

## symbol 元素

`<symbol>` 元素提供了另一种组合元素的方式。和 `<g>` 元素不同，`<symbol>` 元素永远不会显示。
`<symbol>` 可以创建自己的视窗，所以可以指定 `viewBox` 和 `preserveAspectRatio` 属性，**通过给 `<use>` 元素添加 width 和 height 属性就可以让 `<symbol>` 适配视口大小**。SVG Sprite 技术就是利用这个特点，将多个单独的图标放到一个个的 `<symbol>` 中。

```html
<!DOCTYPE html>
<html>
    <body>
        <svg>
            <symbol
                id="shape"
                viewBox="0 0 1024 1024"
                preserveAspectRatio="xMinYMin meet"
            >
                <path
                    d="M567.1 512l318.5-319.3c5-5 1.5-13.7-5.6-13.7h-90.5c-2.1 0-4.2 0.8-5.6 2.3l-273.3 274-90.2-90.5c12.5-22.1 19.7-47.6 19.7-74.8 0-83.9-68.1-152-152-152s-152 68.1-152 152 68.1 152 152 152c27.7 0 53.6-7.4 75.9-20.3l90 90.3-90.1 90.3C341.6 589.4 315.7 582 288 582c-83.9 0-152 68.1-152 152s68.1 152 152 152 152-68.1 152-152c0-27.2-7.2-52.7-19.7-74.8l90.2-90.5 273.3 274c1.5 1.5 3.5 2.3 5.6 2.3H880c7.1 0 10.7-8.6 5.6-13.7L567.1 512zM288 370c-44.1 0-80-35.9-80-80s35.9-80 80-80 80 35.9 80 80-35.9 80-80 80z m0 444c-44.1 0-80-35.9-80-80s35.9-80 80-80 80 35.9 80 80-35.9 80-80 80z"
                ></path>
            </symbol>
        </svg>

        <div style="font-size:40px;">
            石头
            <svg width="60" height="60" style="vertical-align:middle;">
                <use xlink:href="#shape" x="5" y="5" width="50" height="50" />
            </svg>
            布
        </div>
    </body>
</html>
```

疗效：
![symbol的例子](https://raw.githubusercontent.com/XiangnianZhou/blog/master/%E5%9B%BE%E5%BD%A2%E5%9B%BE%E5%83%8F/svg/images/use-3.png)

# 在浏览器使用 SVG

## 作为图像

将图像包含在 `<img>` 元素内。

```HTML
<img src="./scissor.svg" />
```

或者 CSS 中：

```HTML
<div style="height:200px; width:200px;background:url(./scissor.svg)"></div>
```

仅仅作为图像引入：

-   svg 不能与主页面通信；
-   主页面的样式对 SVG 无效；
-   主页面脚本无法感知和修改 SVG 的结构；
-   大多数 Web 浏览器都不会加载 SVG 自己引用的文件，包括其他图像文件、外部脚本，甚至是 Web 字体文件。

在`<img>`元素内包含 SVG 时，注意宽高的计算方式：

> HTML `<img>` 元素定义了一个空间，表示浏览器应该将图像绘制到这个空间中。要使用的图像文件指定在 src（source）属性内。

> 图像的高度和宽度可以使用属性或者 CSS 属性（优先考虑）设置。

> 如果不指定 `<img>` 元素的尺寸，就会使用图像文件固有的尺寸。如果只指定高度（宽度），宽度（高度）就会按比例缩放，以使高宽比（宽高比）与图像文件固有的尺寸匹配。

复习完了 `<img>` 元素的知识，接下来讨论一下 SVG 的尺寸问题：

> 对于栅格图像来说，它固有的尺寸就是它的像素尺寸。对于 SVG 来说，则更为复杂。如果文件中的根元素 `<svg>` 带有明确的 height 和 width 属性，则它们会被用作文件的固有尺寸。如果只指定 height 或者 width 而不是两个都指定，并且 `<svg>` 带有 viewBox 属性，那么将用 viewBox 计算宽高比，图像也会被缩放以匹配指定的尺寸。如果 `<svg>` 带有 viewBox 属性而没有尺寸，则 viewBox 的 height 和 width 将被视为像素长度。

> 如果 `<img>`元素和 `<svg>` 根元素都没有任何有关图像尺寸的信息，浏览器应该为嵌入内容应用默认 HTML 尺寸，通常是 150 像素高、300 像素宽，但是最好不要依赖默认尺寸。

在 CSS 作为图像使用 SVG 时，宽高的计算方式与此相同。

## 将 SVG 作为应用程序

哇~~ 这个狠了，啥叫将 SVG 作为应用程序？？
就是怼到 `<object>` 或 `<embed>` 里。

```HTML
<object data="cat.svg" type="image/svg+xml"
    title="Cat Object" alt="Stick Figure of a Cat">
    <!-- 文本或者栅格图像用作备用选项 -->
    <p>No SVG support! Here's a substitute:</p>
    <img src="cat.png" title="Cat Fallback"
    alt="A raster rendering of a Stick Figure of a Cat"/>
</object>
```

与图像不同的是，嵌入的 SVG 可以包含外部文件，同时脚本可以在该对象和父页面之间进行通信。

## HTML5 中内联 SVG

在上文的 symbol 元素部分，我们已经见过这种用法了。

默认情况下，定位 SVG 时采用内联显示模式（这意味着它和前后的文本会被插入到同一行），并且其尺寸会基于 `<svg>` 元素的 height 和 width 属性决定。
可使用 CSS 的 height 和 width CSS 属性改变尺寸，使用 display、margin、padding 和许多其他 CSS 定位属性改变其定位。
主文档指定的样式会被 SVG 继承。

# 最后

洋洋洒洒写了一大坨，感觉写的也不咋滴。
