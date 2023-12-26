---
title: 解决Python运行速度与依赖问题，亲测打包神器用Nuitka让你的应用程序起飞！
date: 2023-04-20 14:22:16
top: false
summary: Nuitka是一个基于Python的静态编译器，可以将Python代码转换成C++或者直接编译成机器码。本文将介绍如何使用Nuitka打包Python应用程序，让你的应用程序起飞！
categories: Nuitka
tags:
- Nuitka
- Python
- C++
- PyQT5
- PyInstaller
---

### 一、写在前面

Nuitka是一个基于Python语言开发的编译器，它可以将Python代码编译成本地机器码，从而提高Python代码的运行速度。相较于传统解释器，Nuitka编译器通过消除了字节码执行过程中的一些瓶颈和无用操作，使得Python程序的执行效率大大提升。

使用Nuitka编译器编译Python代码之后，得到的是本地机器码（更具体的说是C/C++代码），不再需要解释器进行执行，因此其运行速度比解释器执行Python代码更快，还可以将代码打包为独立的可执行文件，在没有Python环境的机器上也能够运行Python应用程序。

Nuitka不仅支持Python2和Python3，还提供了丰富的编译选项和调试工具，可以在不同平台和发布配置下生成高效、稳定和兼容性强的可执行文件。总之，Nuitka是一款非常优秀的Python编译器工具，值得开发者们尝试使用。

接下来，我们通过一个简单的示例来看一下Nuitka与PyInstaller的区别，以及如何使用Nuitka打包Python应用程序。

### 二、Nuitka与PyInstaller的区别

Nuitka和PyInstaller都是Python打包工具，但是它们的打包方式不同，Nuitka是将Python代码编译成本地机器码，而PyInstaller是将Python代码打包成可执行文件。

Nuitka和PyInstaller的区别如下：

|                    | Nuitka   | PyInstaller |
|:-------------------|:---------|:------------|
| 打包方式               | 编译成本地机器码 | 打包成可执行文件    |
| 打包速度               | 快        | 慢           |
| 打包后的文件大小           | 小        | 大           |
| 打包后的文件可移植性         | 高        | 低           |
| 打包后的文件执行速度         | 快        | 慢           |
| 打包后的文件占用内存         | 小        | 大           |
| 打包后的文件是否需要Python环境 | 不需要      | 不需要         |

### 三、Nuitka与PyInstaller的安装

Nuitka与PyInstaller的安装非常简单，只需要使用pip命令即可完成安装。

```bash
pip install Nuitka
pip install PyInstaller
```

Nuitka需要配合C++的编译器来进行打包所以还需要下载C++的编译器。

下载vs2019(MSVS)或者MinGW64，反正都是C++的编译器，随便下。

### 四、Nuitka与PyInstaller的使用

#### 4.1 Demo编写

安装PyQt5,通过pip命令即可完成安装。

```bash
pip install PyQt5
```

```python
# -*- coding: utf-8 -*-
import sys
from PyQt5.QtWidgets import QApplication, QWidget, QPushButton, QMessageBox


class Demo(QWidget):
    def __init__(self):
        super().__init__()

        self.initUI()

    def initUI(self):
        self.setGeometry(200, 200, 400, 300)
        self.setWindowTitle('Demo')

        button = QPushButton('Click me', self)
        button.move(150, 120)

        button.clicked.connect(self.onButtonClick)

    def onButtonClick(self):
        QMessageBox.question(self, 'Message', 'Hello, World!', QMessageBox.Ok)


if __name__ == '__main__':
    app = QApplication(sys.argv)
    demo = Demo()
    demo.show()
    sys.exit(app.exec_())
```

首先我们需要导入PyQt5库以及sys模块。然后，我们定义一个名为Demo的类，它继承自QWidget类。在类的构造函数中调用了父类的构造函数和initUI（）方法。

在initUI（）方法中，我们设置了窗口的尺寸和标题，并创建了一个QPushButton对象。最后，将 clicked 信号连接到 onButtonClick()
方法上，该方法在按钮被点击时被执行。

onButtonClick（）方法只是在控制台上打印了一条消息。

最后，我们实例化QApplication类，并创建Demo对象并展示。调用sys.exit（）方法来确保程序能够完美退出。

运行程序，可以看到如下界面：

![Nuitka_2023-04-20_14-39-02](https://wangxs020202.gitee.io/pbad/new/Nuitka_2023-04-20_14-39-02.gif)

#### 4.2 Nuitka打包

接下来，我们使用Nuitka将上面的Python代码编译成本地机器码，然后再运行看看效果。

首先，我们需要使用Nuitka命令将Python代码编译成本地机器码，命令如下：

```bash
nuitka --standalone --windows-disable-console --mingw64 --nofollow-imports --show-memory --show-progress --plugin-enable=pyqt5 --output-dir=o 你的.py
```

```txt
--standalone：生成独立的可执行文件
--windows-disable-console：禁用控制台窗口
--mingw64：使用MinGW64编译器
--nofollow-imports：不跟踪导入的模块
--show-memory：显示内存占用情况
--show-progress：显示打包进度
--plugin-enable=qt-plugins：启用Qt插件
--include-qt-plugins=sensible,styles：包含Qt插件
--output-dir=o：指定输出目录
```

Nuitka提供了丰富的编译选项，可以根据不同的需求进行选择，下面是Nuitka的编译选项列表。

| 编译选项                      | 说明           |
|:--------------------------|:-------------|
| --standalone              | 生成独立的可执行文件   |
| --mingw64                 | 使用MinGW64编译器 |
| --mingw32                 | 使用MinGW32编译器 |
| --msvc                    | 使用MSVC编译器    |
| --follow-imports          | 跟踪导入的模块      |
| --nofollow-imports        | 不跟踪导入的模块     |
| --windows-disable-console | 禁用控制台窗口      |
| --windows-icon-from-ico   | 使用ICO图标      |
| --windows-icon-from-exe   | 使用EXE图标      |
| --output-dir              | 指定输出目录       |
| --show-progress           | 显示打包进度       |
| --show-scons              | 显示Scons的输出   |
| --show-modules            | 显示导入的模块      |
| --show-memory             | 显示内存占用情况     |

编译完成后，会在指定的输出目录下生成一个名为你的.exe的可执行文件，双击运行，可以看到如下界面：

![Nuitka_2023-04-20_14-39-02](https://wangxs020202.gitee.io/pbad/new/Nuitka_2023-04-20_15-23-29.gif)

#### 4.2 PyInstaller打包

下面我们使用PyInstaller将上面的Python代码打包，然后再运行看看效果。

首先，我们需要使用PyInstaller命令将Python代码打包成可执行文件，命令如下：

```bash
pyinstaller -F -w 你的.py
```

```txt
-F：打包成一个可执行文件
-w：禁用控制台窗口
```

PyInstaller提供了丰富的打包选项，可以根据不同的需求进行选择，下面是PyInstaller的打包选项列表。

| 打包选项 | 说明         |
|:-----|:-----------|
| -F   | 打包成一个可执行文件 |
| -D   | 打包成一个文件夹   |
| -w   | 禁用控制台窗口    |
| -i   | 使用ICO图标    |
| -c   | 打包时不删除临时文件 |
| -n   | 指定输出文件名    |
| -p   | 指定第三方模块的路径 |
| -d   | 指定依赖文件的路径  |
| -s   | 打包时不删除临时文件 |
| -v   | 打包时显示详细信息  |
| -h   | 显示帮助信息     |

打包完成后，会在当前目录下生成一个名为dist的文件夹，里面包含一个名为你的.exe的可执行文件，双击运行，可以看到如下界面：

![Nuitka_2023-04-20_16-10-09](https://wangxs020202.gitee.io/pbad/new/Nuitka_2023-04-20_16-10-09.gif)

### 五、Nuitka与PyInstaller的对比

#### 5.1 可执行文件大小

Nuitka打包后的可执行文件大小为 4.95M，PyInstaller打包后的可执行文件大小为 34.6M。

![Nuitka_2023-04-20_16-15-37](https://wangxs020202.gitee.io/pbad/new/Nuitka_2023-04-20_16-15-37.png)

#### 5.2 打包速度

Nuitka首次打包速度会有点长，时间取决于你所用的库的多少，但是第二次打包速度会很快，因为Nuitka会缓存之前的打包结果，下次打包时会直接使用缓存结果，所以第二次打包速度会很快。

PyInstaller打包速度较快，时间取决于你所用的库的多少，因为我们所引用的包就只有一个PyQt5，所以打包速度也可以接受。

#### 5.3 打包后的可执行文件运行速度

Nuitka打包后的可执行文件运行速度较快，PyInstaller打包后的可执行文件运行速度与前者对比就略显不足了。

#### 5.4 打包后的可执行文件占用内存

根据众多网友的测试得出的结果，Nuitka打包后的可执行文件占用内存较少，PyInstaller打包后的可执行文件占用内存较多。但是，作者通过demo的测试发现两者区别并不是很大，大概都在 10-15M 左右。

![Nuitka_2023-04-20_16-47-58](https://wangxs020202.gitee.io/pbad/new/Nuitka_2023-04-20_16-47-58.png)

#### 5.5 打包后的可执行文件的兼容性

Nuitka打包后的可执行文件的兼容性较好，PyInstaller打包后的可执行文件的兼容性较差。主要是在跨平台方面，Nuitka打包后的可执行文件可以在Windows、Linux、MacOS等平台上运行，而PyInstaller只能在打包平台上运行。

### 六、总结

本文主要介绍了Nuitka和PyInstaller两种Python代码打包工具，介绍了它们的安装、使用方法，以及它们的对比。在实际使用中，作者发现Nuitka打包后的可执行文件的兼容性较好，运行速度较快，占用内存较少，而PyInstaller打包后的可执行文件的兼容性较差，运行速度较慢，占用内存较多。

相较于传统解释器，使用Nuitka编译器可以消除字节码执行过程中的一些瓶颈和无用操作，使得Python程序的执行效率大大提升。此外，通过编译优化，生成的可执行文件可以直接在目标平台上运行，不需要安装Python环境。同时，Nuitka提供了丰富的编译选项和调试工具，可以在不同平台和发布配置下生成高效、稳定和兼容性强的可执行文件。

总之，采用Nuitka编译器可以提高Python程序的性能和可靠性，并且拥有更好的移植性。对于Python开发者而言，学习和应用这一工具可以进一步提高Python开发的效率和质量。

