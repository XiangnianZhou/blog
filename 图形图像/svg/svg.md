
# 前言
在整理图片相关优化的时候发现自己对 SVG 所知甚少，于是学习了一波，大体总结了一下，整理成文。因为前端开发人员因为细分领域不同，对待SVG的态度差别也很大，对于数据可视化这种对 SVG 有高要求的同学这里的介绍估计没啥干货。对于平常较少接触 SVG 的同学，可以对 SVG 有个初步的认识。

这篇文章主要参考 《SVG 精髓》一书。接下来，开始干活。

# 基础

SVG，即可缩放矢量图形（Scalable Vector Graphics），是一种 XML 应用，可以以一种简洁、可移植的形式表示图形信息。

在矢量图形系统中，图像被描述为一些列的形状。矢量图形阅读器接受在指定的坐标集上绘制形状的指令。而不是接受一系列的已经计算好的像素。

由于 SVG 是 XML 程序，所以图像的有关信息被存储为纯文本，而且具有 XML 的开放性、可移植性以及可交互性。

# 视口

## viewport

SVG 的世界就是一张无限大的画布。文档打算使用的画布区域称作视口（viewport）。

概念说起来很玄乎。看个例子，创建一个简答的SVG：

```xml
<svg width="200" height="200" style="border:solid red 2px">
    <rect width="100" height="100" fill="#4c93cf" stroke="none"/>
</svg>
```

这里的的 `width="200" height="200"` 便是用来设置视口的大小。效果如下： 

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
看看两个的例子，然后，结合下面的一段话，基本就可以理解了。

> SVG 就像是我们的显示器屏幕，viewBox 就是截屏工具的那个选择的框框，最终呈现就是把框框选中的内容再次在显示器全屏显示。

简单的理解就是：
1. 从 SVG 中截取一块区域
2. 显示时，缩放这块区域到视口大小

然后，我们在改一下上面的例子，加深一下理解：

```XML
<svg width="200" height="200" style="border:solid red 2px" viewBox="0 0 800 800" >
    <rect  width="400" height="400" fill="#4c93cf"/>
</svg>
```

基本信息：
- 视口： 200 * 200
- 矩形： 400 * 400
- viewBox： 800 * 800

分析：
1. viewBox 首先截取从(0, 0) 位置截取一个 800 * 800 的区域；
2. 这个区域缩放到和视口大小相等。

此时，
- viewBox 长宽缩小为200， 是原来的 1/4；
- viewBox 中的矩形的长宽也缩小为原来的 1/4， 即长宽都是 100；
- 矩形的面积（100 \* 100）占视口面积（200 \* 200）的 1/4 （如下图）。

![SVG - viewbox2](https://raw.githubusercontent.com/XiangnianZhou/blog/master/%E5%9B%BE%E5%BD%A2%E5%9B%BE%E5%83%8F/svg/images/viewbox2.png)


上面的例子里，viewport 和 viewBox 的长宽比是相等的，最后的效果很舒服。如果二者长宽比不相等，viewbox 要在 viewport 中显示是要瓢的，我们如何指定 viewbox “瓢”的规则呢？ 答案是 `preserveAspectRatio` 。

## preserveAspectRatio

在解释这个概念之前，我们先看看 CSS 中 background 相关的两个属性（暂且不讨论其他相关属性）：
1. `background-size: cover|contain`:
- `cover`：图片宽高比不变、铺满整个容器的宽高，而图片多出的部分则会被截掉；
- `contain`: 图片自身的宽高比不变，缩放至图片自身能完全显示出来，所以容器会有留白区域；

2. `background-position: xpos ypos`
CSS 的取值更灵活，现在只需回忆 xpos 和 ypos 的这几个有效值， `top`, `center` 和 `bottom` 以及 `left`, `center` 和 `right`。

这两个属性分别指定了背景如何伸缩和如何对齐。在 SVG 中，使用 `preserveAspectRatio` 属性来指定上这两种的行为，语法如下：

```XML
preserveAspectRatio="<align> [<meetOrSlice>]"
```

其中，align 类似 `background-position` 用来指定对齐方式。有效值类似，**xMidYMid**，即指定关于x轴和y轴的三种对齐方式：min, mid 和 max。这里需要注意的是，两个值是**连在一起**，且要采用**驼峰**命名法，比如： xMidYMin， 表示：viewBox的最小x值对齐viewport的左边部， viewBox的最小y值对齐viewport的顶边。

meetOrSlice：
- meet， 保持 viewbox 宽高比，尽可能放大 viewbox，**使其 viewport 内是可见的**。这时，**viewport 可能会出现留白**。
- slice，保持宽高比，viewbox 尽可能小，但要使 **viewbox 全部覆盖 viewport**。这个时候，可能会使部分 viewbox 超出 viewport，**超出部分会被裁掉**。

因为可以类比 CSS 属性，这里就不提供例子了（主要是我懒。。）

viewbox 带来的变化，其实源自其对坐标系的改变。接下来我们看看 SVG 的坐标系。

# 坐标系

先介绍两个概念：
**初始视口坐标系**是一个建立在视口上的坐标系。原点(0,0)在视口的左上角，x轴正向指向右，y轴正向指向下，初始坐标系中的一个单位等于视口中的一个"像素"。

**用户坐标系**是建立在 SVG 画布上的坐标系。这个坐标系一开始和视口坐标系完全一样。使用 viewBox 属性，初始用户坐标系统可以变成与视窗坐标系不一样的坐标系，如上面关于 viewBox 属性的第一个例子中，用户坐标系的一个单位等于视口坐标系的两个单位。

通过 viewBox 创建了一个用户坐标系之后，后代元素的定位不再以厨师坐标系定位，而是以新建的用户坐标系。

## 坐标系统变换
开始具体看这节内容之前先记住一句话： **SVG 是坐标系统变换而不是元素变换**。

SVG 元素的变换包括缩放，移动，倾斜和旋转等，类似 CSS 的 transform，SVG 中使用 **transform 属性**。不同的是
- CSS 中的变换是相对于元素自身的中心点，而 SVG 中是**相对于坐标系原点**的；
- CSS 对元素的变换； SVG 是对坐标系的变换：根据变换元素，先复制当前用户坐标系，然后**对用户坐标系副本进行变换**；
- SVG 变换中不指定单位，默认的单位等于当前用户坐标系单位；
- CSS 变换还包含 3D 效果，SVG1.1 不支持 3D 效果。

注：貌似 SVG2 版本中可通过类似 transform-origin 属性的形式设置变换的原点；以及支持 CSS3 规范的 CSS 2D 和 CSS 3D 变换。


### translate 位移
语法：
```
transform="translate(<x> [<y>])"
```
其中 y 方向偏移为非必选，默认为0。

HTML元素的偏移是相对自己的中心点，SVG 元素 的偏移是相对 SVG 的左上角。
虽然，在最后的表现上是一样的。但，在渲染机制上有着很大的不同：
- CSS 中，会先画好元素，然后再发生偏移。
- SVG 中，先偏移坐标系，然后再渲染元素。

### scale 缩放

语法：
```
transform="scale(<x> [<y>])"
```

其中：
x，表示沿x轴的缩放值，所有 x 坐标乘以给定的 x；
y，表示沿y轴的缩放值，所有 y 坐标乘以给定的 y；
y 是可选的，如果没有默认与x值相同。

HTML 元素的缩放与 SVG 元素的缩放比较。
HTML中的缩放：
- 相对元素中心点；
- 缩放元素本身；
- 元素定位不发生变化。

SVG 中的缩放：
- 相对SVG左上角；
- 缩放坐标系；
- 元素被重新定位。

### skew 斜切

语法：
```
transform="skewX(<a>)"

transform="skewY(<a>)"
```
其中，
- skewX 声明一个沿x轴的倾斜；
- skewY 声明一个沿y轴的倾斜。

同其他的 transform 变换，skew 的操作对象也是坐标轴，操作完成后会被重新定位。

### rotate 旋转
语法：

```
transform="rotate(<a> [<x> <y>])"
```

其中，
- a 表示旋转角度，默认是单位是度（deg）；
- x和y可选，代表旋转中心点；
- 如果没有设置x，y的值，默认为当前用户坐标系的原点。

rotate 与上面介绍的另外几个操作不同的是，可以指定旋转中心点，看似和 CSS 中的旋转规则相同，其实：

```
transform="rotate（a x y)"
```
等同于：

```
transform="translate(x y) rotate（a) translate(-x -y)"
```

### 变换序列
一个图形对象上可以做多个变换。我们只需将多个变换通过空格或逗号分隔，依次放入 transform 属性即可。

```
<rect height="15" width="20" transform="translate(30, 20) scale(2)" fill=" gray" />
```

# 形状

# 文字

# 样式

# 在浏览器使用

