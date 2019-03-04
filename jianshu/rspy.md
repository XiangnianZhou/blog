对于习惯性晚睡但想早点睡的人，对于早上不想被手机闹铃在睡梦中突然叫醒的人，DIY 一个闹铃还是有些必要的。闹铃的功能主要是晚上提醒睡觉，早上提醒起床（哈哈哈。。。）

## 物料

### 硬件

-   树莓派
-   光线传感器模块（自己搞个光敏电阻焊也行）
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

## 提醒睡觉

原理： 晚上 22:30 分之后，用光敏模块检测光线强度（模块输出开关量，调节电位器到一个合适的位置，使其在关灯状态和开灯状态输出不同的信号）， 如果卧室开着灯，每隔 10 分钟就触发蜂鸣器。
