对于习惯性晚睡但想早点睡的人，对于早上不想被手机闹铃在睡梦中突然叫醒的人，DIY 一个闹铃还是有些必要的。闹铃的功能主要是晚上提醒睡觉，早上提醒起床（哈哈哈。。。）

## 物料

### 硬件

-   树莓派
-   光敏电阻传感器模块
-   蜂鸣器
-   红外接收模块
-   红外发射模块
-   遥控灯

### 软件

-   Linux on Raspberry Pi
-   Nodejs
-   Lirc

## 准备

### 树莓派（Raspbian ）上安装 Nodejs：

可以先装一个 nvm，使用`nvm install` 安装；下面介绍一种去[官网](https://nodejs.org/en/download/)下载最新版的 nodejs 版本的方法：
因为和一般的 PC（x86 和 x64 架构）不同，树莓派的 CPU 是 ARM 架构，所以，要下载对应的 Nodejs 二进制文件。
下图提供了，不同的树莓派版本对应的不同的下载版本；
![树莓派下载Nodejs](https://raw.githubusercontent.com/XiangnianZhou/blog/master/jianshu/images/3396239-bae8e9232483284f.png)

找到对应的版本之后，可以直接使用浏览器下载，也可以复制下载链接使用`wget` 指令下载；然后对下载下来的 node-xx.tar.gz 执行如下命令：

```
tar -xvf node-xx.tar.gz
cd node-xx
sudo cp -R * /usr/local/
```

此时 Nodejs 已经可以使用了；

### 初始化项目

创建 pakage.json

```
npm init
```

安装依赖库：
用到了第三方库 `onoff`， 它可以让 Nodejs 在访问树莓派的 GPIO。更多玩法请看[这里](https://github.com/fivdi/onoff)。

```
npm install onoff --save
```

### 硬件连接和调试

因为在某宝很容易买到树莓派或者 Arduino 的模块，而且也很便宜，推荐直接购买模块。
本次的项目中用到的光敏电阻模块，红外接收模块，红外发射模块和蜂鸣器模块都是比较简单的模块，只有三个引脚，分别对应 VCC，GND 和输入/输出引脚。链接方法不做过多解释。

准备工作做完后，来开始撸代码了。

#### 蜂鸣器

我这里蜂鸣器连接的是 `GPIO22`，蜂鸣器为**高电平触发的有源蜂鸣器**，所以给 `GPIO22` 置高电平就可以，代码如下：

```javascript
const Gpio = require('onoff').Gpio;
const beep = new Gpio(22, 'out');

const beepOn = () => {
    beep.writeSync(1);
    setTimeout(() => {
        beep.writeSync(0);
    }, 350);
};

beepOn();
```

保存并运行脚本 `node index.js`, 听见蜂鸣器“哔~”的一声。
以上便是驱动蜂鸣器的基本代码。

#### 光敏电阻传感器

我采用的是一个开关量输出的树莓派模块，可以通过调节模块上的电位器来设置阈值，当外界光线亮度超过阈值时，模块输出低电平，否则输出高电平。
模块的输出管脚接树莓派的 `GPIO27`，程序如下：

```javascript
const lightInput = new Gpio(27, 'in');
console.log(lightInput.readSync());
```

运行脚本发现光线充足时，打印 1，用手挡住传感器打印 0。
到此，我们完成了判断外界光线强度基本代码。

#### 红外遥控信号的解码与编码

这个问题听起来很高大上，但是如果去百度搜索 `树莓派红外遥控`相关内容，会发现大量介绍如何使用树莓派进行编解码的文章。其原理是，通过**lirc 软件**，在 Linux 上实现红外发送与接收。
下面列出我参考的几篇文章：

-   [一篇 github 上的文章](https://gist.github.com/prasanthj/c15a5298eb682bde34961c322c95378b)
-   [一篇关于 NEC 协议的文章](http://www.cnblogs.com/yulongchen/archive/2013/04/12/3017409.html)
-   [使用树莓派红外控制空调和风扇](https://linux.cn/article-3782-1.html)
-   [树莓派学习手记](https://segmentfault.com/a/1190000014135418)

硬件连接：
红外接收模块的，I/O 引脚接树莓派 `GPIO17`, 红外接收模块的信号引脚接树莓派的 `GPIO18`。

安装：

```
$ sudo apt-get update
$ sudo apt-get install lirc
```

打开 `/etc/modules` 文件：

```
sudo leafpad /etc/modules
```

然后添加如下代码，并保存退出：

```
lirc_dev
lirc_rpi gpio_in_pin=18 gpio_out_pin=17
```

打开 `/etc/lirc/hardware.conf` 文件：

```
sudo leafpad  /etc/lirc/hardware.conf
```

添加如下代码，并保存退出：

```
LIRCD_ARGS="--uinput --listen"
LOAD_MODULES=true
DRIVER="default"
DEVICE="/dev/lirc0"
MODULES="lirc_rpi"
```

编辑 `/etc/lirc/lirc_options.conf` 文件：

```
driver    = default
device    = /dev/lirc0
```

正常情况现在就可以重启了。

LIRC 软件的常用命令：

```
# 启动
sudo /etc/init.d/lircd start

# 停止
sudo /etc/init.d/lircd stop

# 重启
sudo /etc/init.d/lircd restart

# 查看状态
sudo /etc/init.d/lircd status
```

测试红外接收：

```
sudo /etc/init.d/lircd stop

mode2 -d /dev/lirc0
```

输出一坨类似下面的东东：

```
space 16777215
pulse 8999
space 4457
pulse 680
space 1627
......
```

其实，这就是遥控器发出的信号了， 结合这里的 [ NEC 协议](http://www.cnblogs.com/yulongchen/archive/2013/04/12/3017409.html)，我们就能猜到，这其实就是红外信号脉冲(pulse)和空白(space)的持续时间。

录入红外信号：

```
sudo /etc/init.d/lircd stop

sudo irrecord -d /dev/lirc0
```

一般来讲，按提示一步一步进行就行。不过这里需要注意的是，这种方式，每个按键的名字，必须使用 lirc 的命名空间内规定的名字。

使用下面指令查看命名空间：

```
irrecord -l --list-namespace
```

因为我看见遥控器上有【亮度+】 和 【亮度-】 这些按键，从命名空间里找一个合适的名字也很麻烦，所以要使用 `--disable-namespace` 参数，表示不使用 lirc 的命名空间。

```
irrecord -d /dev/lirc0 --disable-namespace
```

如果遥控是有“状态”的，可以使用 row 模式录入信号：

```
# 参数-f --force 表示 Force raw mode
irrecord -f -d /dev/lirc0 --disable-namespace
```

我发现，这种模式对于简单的遥控的录入也很友好。
我录入好的文件，[在这里](https://github.com/XiangnianZhou/rpi-alarm/blob/master/attach/light.row.lircd.conf)

测试一下录入的信号能不能用：

```
# 复制刚才录入的配置文件到/etc/lirc下
sudo cp ~/light.row.lircd.conf /etc/lirc/lircd.conf

# 启动lirc
sudo /etc/init.d/lircd start

# 发送
# irsend SEND_ONCE <device-name> <key-name>
irsend SEND_ONCE light on
```

将红外发射管对着要控制的灯的接收器， 执行上面 `irsend` 命令，发现灯会被打开，到此红外遥控信号的录入与发送都已经完成。

### Nodejs 部分

beep.js 驱动蜂鸣器的脚本，原理很简单，给高电平它就会响，给低电平就不响了，我想要的是 “哔-哔-哔---哔-哔-哔---” 的效果，于是每个循环单元，需要给 GPIO 三次置高电平。代码如下：

```JavaScript
const Gpio = require('onoff').Gpio;
const beepGpio = new Gpio(22, 'out');

function beepOn(minutes) {
    const ms = minutes * 60 * 1000;
    const beep = () => beepGpio.writeSync(1);
    const space = () => beepGpio.writeSync(0);

    // 每一个beepUnit，是一个 哔-哔-哔--
    const beepUnit = () => {
        beep();
        setTimeout(space, 150);
        setTimeout(beep, 180);
        setTimeout(space, 320);
        setTimeout(beep, 350);
        setTimeout(space, 480);
    };

    beepUnit();
    const beepTimer = setInterval(beepUnit, 1000);
    setTimeout(() => {
        clearInterval(beepTimer);
        space();
    }, ms);
}
```

测试一下：

```JavaScript
const beep = require('./beep');
beep(0.1);
```

执行以上代码，出现了六次“哔-哔-哔--”。

## 提醒睡觉

原理： 晚上 22:30 分之后，用光敏模块检测光线强度（模块输出开关量，调节电位器到一个合适的位置，使其在关灯状态和开灯状态输出不同的信号）， 如果卧室开着灯，每隔 10 分钟就触发蜂鸣器。
