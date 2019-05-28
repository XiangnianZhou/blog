前端开发人员因为细分领域不同，对待SVG的态度差别也很大。本文的目标读者是平常较少使用SVG的前端，不是数据可视化这种对SVG有高要求方向的同学。因为我也只知道基础~~

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

这里的的 `width="200" height="200"` 是 SVG 可视区域的大小。效果如下： 

![SVG - viewport](https://raw.githubusercontent.com/XiangnianZhou/blog/master/%E5%9B%BE%E5%BD%A2%E5%9B%BE%E5%83%8F/svg/images/viewport1.png)


## viewbox

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


## preserveAspectRatio



# 形状

# 文字

# 样式

# 在浏览器使用

