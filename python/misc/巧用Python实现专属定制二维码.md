本课程通过调用MyQR接口来实现生成个人所需二维码，并可以设置二维码的大小、是否在现有图片的基础上生成、是否生成动态二维码。

本课程主要面向Python初学者。

#### 知识点

- Python基础
- MyQR库

#### 效果截图

##### 普通二维码

![qrcode](/Users/huyida/OneDrive/USERPRO/Python/mrcode/qrcode.png)

##### 图片的艺术二维码

![artistic](/Users/huyida/OneDrive/USERPRO/Python/mrcode/artistic1.png)

##### 动画二维码

![artistic](/Users/huyida/OneDrive/USERPRO/Python/mrcode/artistic.gif)

## 创建环境

打开终端

### 下载`MyQR`

```sh
$ pip install MyQR
```

接下来，我们会自己制作普通二维码、带有图片的艺术二维码和动态二维码。

在命令行中输入 python3，进入 python 环境：

```
$ python
```

## 生成二维码

### 普通二维码

在 python环境中输入以下代码：

```python
from MyQR import myqr
myqr.run('https://www.huyidada.com')
```

大功告成，那么来看一看自己制作的第一张二维码图片吧!

效果图：

![qrcode](/Users/huyida/OneDrive/USERPRO/Python/mrcode/qrcode.png)

### 图片二维码

光是二维码，是否太单调了呢？没关系，我们能加上我们想要的图片，使二维码更具辨识度！ 我们准备了公众号的Logo：

公众号Logo图片：

![logo](/Users/huyida/OneDrive/USERPRO/Python/mrcode/logo.png)

当然，Sources文件夹里有更多的图片，你也可以选择你个人喜爱的一张来制作艺术二维码！

让我们将这张图加入到我们的二维码中，加入过程需要在参数里指定公众号Logo图片的地址，我们也要设置新图片的保存名，以免和上一张二维码图片冲突。

```python
myqr.run(
    words='http://www.huyidada.com',
    picture='logo.png',
    save_name='artistic.png',
)
```

公众号Logo二维码：

![artistic](/Users/huyida/OneDrive/USERPRO/Python/mrcode/artistic.png)

黑白的，似乎不是那么好看，彩色的如何呢？ 实现彩色也非常简单，在参数里将 `colorized` 参数值设为 `True`。

```python
myqr.run(words='http://www.huyidada.com',
         picture='logo.png',
         colorized=True,
         save_name='artistic1.png')
```

彩色公众号Logo二维码：

![artistic](/Users/huyida/OneDrive/USERPRO/Python/mrcode/artistic1.png)

### 动画二维码

其实生成动态二维码，并没有想象的那么复杂。 在这里，我们使用美丽的新垣结衣GIF！

新垣结衣GIF:

![1-3.3-1](https://doc.shiyanlou.com/document-uid731737labid7103timestamp1530003207033.png)

在生成动态二维码的过程中，值得注意的一点是，我们生成保存的文件也必须是` .gif` 格式哟。 让我们赶快开始！

```python
myqr.run(words='http://www.huyidada.com',
         picture='code.gif',
         colorized=True,
         save_name='artistic.gif')
```

新鲜出炉的动图，新垣结衣动态二维码：

![artistic](/Users/huyida/OneDrive/USERPRO/Python/mrcode/artistic.gif)

效果很不错呢，拿起手机试着扫扫看。

## 总结

二维码的内容，就到此结束了。二维码在日常生活中的使用场景很多，大家可以结合实际生活来使用。