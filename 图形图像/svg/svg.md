# 基础

在矢量图形系统中，图像被描述为一些列的形状。矢量图形阅读器接受在指定的坐标集上绘制形状的指令。而不是接受一系列的已经计算好的像素。

SVG 最典型的特定是缩放不损失图像的质量。

由于 SVG 是 XML 程序，所以图像的有关信息被存储为纯文本，而且具有 XML 的开放性、可移植性以及可交互性。

# 视口

## viewport

首先，创建一个简单到 SVG。

```
<svg width="200" height="200" style="border:solid red 2px">
    <rect width="100" height="100" fill="#4c93cf" stroke="none"/>
</svg>
```

这里的的 `width="200" height="200"` 是 SVG 可视区域的大小，可以理解为画布大小。

![viewport1](https://github.com/XiangnianZhou/blog/blob/master/%E5%9B%BE%E5%BD%A2%E5%9B%BE%E5%83%8F/svg/images/viewport1.png?raw=true)

## viewbox

接下来我们改装一下上面的代码，让里面的正方形铺满 viewport。

```xml
<svg width="200" height="200" style="border:solid red 2px" viewBox="0 0 100 100">
    <rect  width="100" height="100" fill="#4c93cf"/>
</svg>
```

效果如下：
![viewbox1](https://github.com/XiangnianZhou/blog/blob/master/%E5%9B%BE%E5%BD%A2%E5%9B%BE%E5%83%8F/svg/images/viewbox1.png?raw=true)

相较于上面的代码，这里仅仅多了 viewBox 属性，不过这个 viewBox **不是很容易解释，但是很容易理解**。
看看上面的例子，然后，结合下面的一段话，基本就可以理解了。

> viewport 就像是我们的显示器屏幕，viewBox 就是截屏工具的那个选择的框框，最终呈现就是把框框选中的内容再次在显示器全屏显示。

简单的理解就是：

1. 从 viewport 中截取一块区域
2. 缩放这块区域到 viewport 大小

然后，我们在改一下上面的例子，加深一下理解：

```
<svg width="200" height="200" style="border:solid red 2px" viewBox="0 0 800 800" >
    <rect  width="400" height="400" fill="#4c93cf"/>
</svg>
```

基本信息：

-   viewport： 200 \* 200
-   矩形： 400 \* 400
-   viewBox： 800 \* 800

分析：

1. viewBox 首先截取从(0, 0) 位置截取一个 800 \* 800 的区域；
2. 这个区域缩放到和 viewport 大小相等。

此时，

-   viewBox 长宽缩小为 200， 是原来的 1/4；
-   viewBox 中的矩形的长宽也缩小为原来的 1/4， 即长宽都是 100；
-   矩形的面积（100 \* 100）占 SVG 的面积（200 \* 200）的 1/4 （如下图）。

![viewbox2](https://github.com/XiangnianZhou/blog/blob/master/%E5%9B%BE%E5%BD%A2%E5%9B%BE%E5%83%8F/svg/images/viewbox2.png?raw=true)

## preserveAspectRatio

上面的例子中，viewport 和 viewBox 的比例是相等时 viewbox 默认会铺满 viewport，如果比例不相等又会如何呢？很明显将面临 viewBox 缩放的问题。对于这个问题，这里可以类比 CSS 的`background-size` 和 `background-position` 属性（暂且不讨论其他相关属性）：

1. `background-size: cover|contain`:

-   `cover`：图片宽高比不变、铺满整个容器的宽高，而图片多出的部分则会被截掉；
-   `contain`: 图片自身的宽高比不变，缩放至图片自身能完全显示出来，所以容器会有留白区域；

2. `background-position: xpos ypos`
   可能 CSS 更灵活，这里只需类比这种形式，其中，xpos 和 ypos 的有效值有 `top`, `center` 和 `bottom`。

当容器和背景长宽比不想同时，这两个属性分别指定了背景如何进行伸缩以及如何处理伸缩后对齐问题。

SVG 中，使用 `preserveAspectRatio` 属性来指定上面二者的行为，语法如下：

```
preserveAspectRatio="<align> [<meetOrSlice>]"
```

其中，align 类似 `background-position` 用来指定对于留有的空白的情况，如何对齐。有效值类似，**xMidYMid**，即 x 和 y 的三种对齐方式：min, mid 和 max。
这里需要注意的是，两个值是**连在一起**，且要采用**驼峰**命名法，比如： xMidYMin， xMaxYMax。

meetOrSlice，有效值为：

-   meet， 保持 viewbox 宽高比，尽可能放大 viewbox，使其 viewport 内是可见的。这时，viewport 可能会出现留白。
-   slice，保持宽高比，viewbox 尽可能小，但要使 viewbox 全部覆盖 viewport。这个时候，可能会使部分 viewbox 超出 viewport，超出部分会被裁掉。

改扯是的扯完了，看代码：

```
<svg width="200" height="200" style="border:solid red 2px" viewBox="0 0 100 200" preserveAspectRatio="xMaxYMid meet">
    <rect  width="100" height="100" fill="#4c93cf"/>
</svg>
```

基本信息：

-   viewport： 200 \* 200
-   矩形： 100 \* 100
-   viewBox： 100 \* 200

分析：

-   viewbox 是 包含矩形在内的占 1/2 viewport 面积的区域；
-   meet，会显示全部的 viewbox，不出现裁剪
-   此时，viewbox 的高和 viewport 的高相同，所以，y 方向的对齐方式无效
-   viewbox 的是 viewport 的 1/2，设置水平方向右对齐：xMax

![preserveAspectRatio](https://github.com/XiangnianZhou/blog/blob/master/%E5%9B%BE%E5%BD%A2%E5%9B%BE%E5%83%8F/svg/images/preserveAspectRatio.png?raw=true)
