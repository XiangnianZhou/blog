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

## 转换坐标系统

svg 元素可以通过缩放，移动，倾斜和旋转来变换。类似 css 的 transform。
svg 的 transform 属性。

不同：
css 中是相对于元素自身的中心点的，而 svg 是相对于坐标系原点的。

注：未来的 SVG2 版本中可通过类似 transform-origin 的形式设置变换的原点。

### translate 位移

语法：

```
transform="translate(<x> [<y>])"
```

其中 y 方向偏移为非必选，如果没有默认为 0。

HTML 元素的偏移是相对自己的中心点，**svg 元素** 的偏移是相对 SVG 的左上角。
虽然，在随后的表现上是一样的。
但，在渲染机制上有着很大的不同：
css 中，会先画好元素，然后再发生偏移。
svg 中，先偏移坐标系，然后再渲染元素。

### scale 缩放

语法：

```
transform="scale(<x> [<y>])"
```

其中：
x， 表示沿 x 轴的缩放值，用来水平描述延长或拉伸元素；
y，表示沿 y 轴的缩放值，用来垂直延长或缩放元素；
y 是可选的，如果没有默认与 x 值相同。

HTML 中的缩放：

-   相对元素中心点
-   缩放元素本身
-   元素定位不发生变化

SVG 中的缩放：

-   相对 SVG 左上角
-   缩放坐标系
-   元素被重新定位

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

-   a 表示旋转角度，<del>无单位，默认是单位是度（deg）；</del>
-   x 和 y 可选，代表旋转中心点，<del>无单位</del>
-   如果没有设置 xy 的值，默认为当前用户坐标系的原点。

rotate 与上面介绍的另外几个操作不同的是，可以指定旋转中心点，看似和 CSS 中的旋转规则相同，其实：

```
transform="rotate（a x y)"
```

等同于：

```
transform="translate(x y) rotate（a) translate(-x -y)"
```

# 文本

## 基本概念

首先需要料及一些关于字体的一些概念：

-   字符：一个 Unicode 码点。
-   符号：字符的视觉呈现。
-   字体：一坨字符对应的符号集合。

我的解释可能不太准确，下面贴出来比较正经的解释。引自《SVG 精髓》及英文原版书。

中文版:

> -   **字符**: 在 XML 文档中，字符是指带有一个数字值的一个或多个字节，数字值与 Unicode 标准对应。例如，字母 g 是 Unicode 值为 103 的字符。
> -   **符号**: 符号（glyph）是指字符的视觉呈现。每个字符都可以用很多不同的符号来呈现。图 9-1 展示了用两种不同的符号呈现的单词“glyphs”。注意开头的字母 g，同一个字符，但符号却明显不同。
>     图是酱婶的： ![两种符号](index_files/f955246e-5ca2-4bb3-ac1f-eae8e1431c99.png)
>     一个符号可能由多个字符构成。一些字体为特定的字母组合（如 fl 和 ff）准备了单独的符号，以使它们更好看，这种特性叫作“连字”（ligature）。有时候，一个字符也可能由几个符号组合而成，比如打印程序可能会组合符号 e 和重音符号（ ́）来打印字符 é（Unicode 值为 233）。
>     **字体**: 字体是指代表某个字符集合的一组符号。

英文版：

> **Character**: A character, as far as an XML document is concerned, is a byte or bytes with anumeric value according to the Unicode standard. For example, what we call the letter g is the character with Unicode value 103.
> **Glyph**: A glyph is the visible representation of a character or characters. A single character can have many different glyphs to represent it. Figure 9-1 shows the word glyphs written with two different sets of glyphs—look particularly at the initial g—it’s the same character, but the glyphs are markedly different.
> 图见中文版 ↑↑
> Multiple characters can reduce to a single glyph; some fonts have separate glyphs for the letter combinations fl and ff to make their spacing look better (these are called ligatures). Other times, a single character can be composed of multiple glyphs; a print program might create the character é (which has Unicode value 233) by combining the e glyph with a nonspacing accent mark (´).
> **Font**: A font is a collection of glyphs representing a certain set of characters.

了解了基本概念后，继续深入理解几个关于字体的概念：

-   基线（Baseline）：字体中的所有符号以基线对齐;
-   上坡度（ascent）： 基线到字体中最高字符顶部的距离称为上坡度;
-   下坡度（descent）：基线到最深字符底部的距离称为下坡度;
-   em-box: 字符的总高度为上坡度和下坡度之和，也称为 em 高度。em-box 是指宽度为 em 高度的方块;
-   大写字母高度（cap-height）是指基线上的大写字母的高度;
-   x 高度，是指基线到小写字母 x 顶部的高度。x 高度通常能比 em 高度更好地衡量一个字体的尺寸和可读性。
    ![符号的度量](index_files/422d7afe-0a0e-4479-ab57-9d16d5b93a64.png)

## text 元素

在一个 SVG 文档中，`<text>`元素内部可以放任何的文字。
如：

```
<text>balabala</text>
```

和 CSS 差不多，SVG 中的文本也有很多字体和文本有关的属性，甚至属性名都是完全相同的。如：
font-family、font-style、font-weight、font-variant、font-stretch、font-size、font-size-adjust、kerning、letter-spacing、word-spacing 和 text-decoration。

### 文本位置

语法：

```
<text x="x" y="y">balabala</text>
```

其中：

-   x：将文本向右移动 x
-   y：将文本想下移动 y （指定文本基线位置）
-   x 和 y 值是可以带单位的
-   文本移动是相对 SVG 的左上角

另外 x 和 y 还可以接受一个数列：

```
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

注意：如果相对整个文本元素进行旋转，使用转换坐标系的方式`transform="rotate(r)"`。

### 纵向文本

`writing-mode：tb`: top to bottom，从上到下。实现文本的纵向排列。
`glyph-orientation-vertical`： 属性来设置文本（不是整个文本元素）旋转角度。
据我测试，后一个属性的兼容性不是太好。

## tspan 元素

用来标记大块文本的子部分，它必须是一个 `<text>` 元素的或别的 `<tspan>` 元素的子元素。

使用场景和 HTML 中的 span 标签很相似，可以用来在一段文本中单独处理一小段文本内容，比如加粗，倾斜和上下标等。

## textPath 元素

利用 xlint:href 属性把字符对齐到路径，让字体可以顺着路径排列。
这是个很骚的操作，必须得有例子：

```
<svg width="800" height="800">
    <path id="path" d="M 50 110 A 40 30 0 1 0 230 110 M 250 110" fill="none" stroke="none" />
    <text>
        <textPath xlink:href="#path" font-size="22">
            风 吹 水 面 层 层 浪 ~
        </textPath>
    </text>
</svg>
```

![textPath的例子](index_files/6fd7a734-898a-47b4-8fa9-eabe58877cf4.png)
