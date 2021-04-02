Python 实现图片转字符画



本实验用 50 行 Python 代码完成图片转字符画小工具。通过本实验将学习到 Linux 命令行操作，Python 基础，pillow 库的使用，argparse 库的使用。



**1.1 实验知识点**

本节实验中我们将实践以下知识：

1. Linux 命令行操作
2. Python 基础
3. pillow 库的使用
4. argparse 库的使用（[参考教程](http://blog.ixxoo.me/argparse.html)）

**1.2 实验环境**

- Python 3.5
- pillow 5.1.0

PIL 是一个 Python 图像处理库，是本课程使用的重要工具，使用下面的命令来安装 pillow（PIL）库：

$ sudo pip3 install --upgrade pip

$ sudo pip3 install pillow





字符画是一系列字符的组合，我们可以把字符看作是比较大块的像素，一个字符能表现一种颜色（为了简化可以这么理解），字符的种类越多，可以表现的颜色也越多，图片也会更有层次感。

问题来了，我们是要转换一张彩色的图片，这么多的颜色，要怎么对应到单色的字符画上去？这里就要介绍灰度值的概念了。

灰度值：指黑白图像中点的颜色深度，范围一般从0到255，白色为255，黑色为0，故黑白图片也称灰度图像。

另外一个概念是 RGB 色彩：

RGB色彩模式是工业界的一种颜色标准，是通过对红(R)、绿(G)、蓝(B)三个颜色通道的变化以及它们相互之间的叠加来得到各式各样的颜色的，RGB即是代表红、绿、蓝三个通道的颜色，这个标准几乎包括了人类视力所能感知的所有颜色，是目前运用最广的颜色系统之一。- 来自百度百科介绍

我们可以使用灰度值公式将像素的 RGB 值映射到灰度值（注意这个公式并不是一个真实的算法，而是简化的 sRGB IEC61966-2.1 公式，真实的公式更复杂一些，不过在我们的这个应用场景下并没有必要）：



gray ＝ 0.2126 * r + 0.7152 * g + 0.0722 * b



这样就好办了，我们可以创建一个不重复的字符列表，灰度值小（暗）的用列表开头的符号，灰度值大（亮）的用列表末尾的符号。





首先，安装 Python 图像处理库 pillow（PIL）：

## $ sudo pip3 install --upgrade pip

## $ sudo pip3 install pillow



然后在/home/shiyanlou/ 目录下创建 ascii.py 代码文件进行编辑：

## $ cd /home/shiyanlou/

## $ touch ascii.py



使用 vim 或者 gedit 打开代码文件：

$ cd /home/shiyanlou

$ gedit ascii.py



文件打开后依次输入以下的代码内容。

首先导入必要的库，argparse 库是用来管理命令行参数输入的

from PIL import Image

import argparse



我们首先使用 argparse 处理命令行参数，目标是获取输入的图片路径、输出字符画的宽和高以及输出文件的路径：





最后，使用刚刚编写的 ascii.py 来将下载的 ascii_dora.png 转换成字符画，此时执行过程没有指定其他的参数，比如输出文件、输出文件的宽和高，这些参数都将使用默认的参数值：

$ python3 ascii.py ascii_dora.png



然后使用 vim 打开 output.txt 文件：

$ vim output.txt



如果使用 vim 可以按 ESC 键，然后输入 :q 进行退出。