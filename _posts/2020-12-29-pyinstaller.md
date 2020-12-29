---
layout: post
title: "在macOS上用PyInstaller打包Python程序"
date: 2020-12-29
author: 'dax'
categories: "note"
typora-root-url: ../../bestdax.github.io
---

自己写一个小程序，然后打包成可以独立分发的APP，一直是我的梦想。最近用Python+PyQt5写了一个自动生成算术题的APP，虽然还没有完全完成，但是已经能够满足目前的需求了，所以一弄完，我就想到如何将代码打包。偶然在学PyQt5的书上看到过PyInstaller的片断，决定先从这个开始。虽然预想到会比较困难，但是没想到一连搞了好几天才完全弄好。现在我终于可以说我已经掌握了PyInstaller的打包方法了。把踩过的坑一一记录下来，以防忘记。

## 虚拟环境

之前没有真正写过多少代码，都是把所有的依赖包装在系统的Python里的，但是这个对于项目开发来讲就非常不利了。各个库之间有不同的版本，会有影响。所以在用了PyCharm之后，感觉一下子就顺畅了很多，每个项目会单独建一个虚拟环境。但是之前因为不懂，还是在系统公共的库里安装了一些库，可能还在某个项目里面选择继承公共库，而且对所有项目可用。然后就出现了在环境里面能看到PyQt5，但是运行时说找不到。后来还是在PyCharm解释器界面上无意中看到安装包的地址才明白。但是我不知道怎样去除继续和对所有项目可用。后来是在系统公用的解释器中把所有的库卸载才搞定的。用`pip list`可以查安装了哪些库文件。

另外一个需要注意的是，在终端中，激活了虚拟环境后，有些命令用的是虚拟环境里面的，有些是用的系统公用环境里面的。比如`pyinstaller`就是有的系统公用环境中的，而`PyInstaller`就是用的虚拟环境的。如果不确定可以用`which pyinstaller`来查看具体运行的是哪个里面的。

## PyInstaller

PyInstaller其实就是将整个运行需要用到所有文件封闭在一起，然后移植到其他机器上就无需安装任何的辅助就可以运行了。打包好的程序运行时，会将文件解压到一个临时文件夹，然后在将那个生成的临时文件夹做为一个封闭的环境来运行。在这个里面我有几个事情记录一下。

### Python解释器的版本

版本不能高，在高版本的编译出来的APP会显示`code signature`的错误。后来降级到3.7.2就好了。

### onefile和onedir

之前我的理解是错误的，我以为onedir就是一个文件夹，然后里面有各种各样的文件，这样的话还要压缩了再分发，而安装者还要解压，然后再到文件夹里面运行可执行文件。而onefile就是一个执行文件。从字面上来讲确实是这样的，但是其实onedir和onefile在macOS上看起来几乎差不多，都是一个可以执行的APP。区别在于如果在APP上点击右键显示包内容的话，在macOS文件夹里onefile模式下就只有一个执行文件，而onedir是散装的。这两个模式下，启动速度差很多。我打包的onedir模式的APP150M，而onefile的是50M。不过，onefile模式的速度太慢了。

### 资源文件

资源文件需要告诉PyInstaller在什么地方，要添加到什么地方去。用`PyInstaller --add-data 'src/*.txt:src'`如果有多个的话，可以在后面再加`--add-data`。在代码里面，文件的地址就不能用简单的写法，如前面说的那样，文件会解压在一个临时文件夹里，这样文件的地址也会相应的变化。在网上找的解决方案：

```
import os
import sys


def resource_path(relative_path):
    """ Get absolute path to resource, works for dev and for PyInstaller """
    base_path = getattr(sys, '_MEIPASS2', os.path.dirname(os.path.abspath(__file__)))
    return os.path.join(base_path, relative_path)
```

### hidden-imports

PyQt5如果不添加到hidden-imports里面的话，运行APP的时候会提示找不到PyQt5。`--hidden-import PyQt5`可以解决，另外还要添加一个PyQt5.sip的依赖包。

### 图标

APP的图标当然要有，要不然太不酷了。这个可以选择一个图片，然后到[icoconvert.com](https://icoconvert.com/)上转换一下就可以了。

### 使用PyInstaller

PyInstaller的使用分为两种，命令行和spec文件。我之前浪费的时间主要是在这两种方式的混合使用上，再加上对PyInstaller没有概念。用`PyInstaller script.py`就可以生成一个`dist`和`build`两个文件夹，另外一个`script.spec`文件。在`dist`文件夹里面有一个可以运行的文件，还有一个文件夹。如果脚本很简单，没有hidden-imports，也没有资源文件的话，这样一个APP就可以运行了。我的问题就在于对命令和概念不熟悉的情况下，简单的把生成的spec文件改来改去，网上又很少有这样直接的案例，就浪费了很多时间。

最后我在大致明白了onefile和onedir的区别后，用命令行`pyi-makespec`把后面参数一个一个敲进去，然后在生成的spec文件里面稍加修改就可以了。

`sudo pyi-makespec --add-data 'rsc/*.ttf:rsc' --add-data 'rsc/*.txt:rsc' --add-data 'rsc/*.ico':'rsc' --hidden-import 'PyQt5' --hidden-import 'PyQt5.sip' --onedir --windowed --icon '/Users/dax/PycharmProjects/Quiz/rsc/quiz.ico' main.py`

命令有点长，但是帮我把spec文件里面的格式搞清楚了。

```
# -*- mode: python ; coding: utf-8 -*-

block_cipher = None


a = Analysis(['main.py'],
             pathex=['/Users/dax/PycharmProjects/Quiz'],
             binaries=[],
             datas=[('rsc/*.ttf', 'rsc'), ('rsc/*.txt', 'rsc'), ('rsc/*.ico', 'rsc')],
             hiddenimports=['PyQt5', 'PyQt5.sip'],
             hookspath=[],
             runtime_hooks=[],
             excludes=[],
             win_no_prefer_redirects=False,
             win_private_assemblies=False,
             cipher=block_cipher,
             noarchive=False)
pyz = PYZ(a.pure, a.zipped_data,
             cipher=block_cipher)
exe = EXE(pyz,
          a.scripts,
          [],
          exclude_binaries=True,
          name='Quiz',
          debug=False,
          bootloader_ignore_signals=False,
          strip=False,
          upx=True,
          console=False , icon='/Users/dax/PycharmProjects/Quiz/rsc/quiz.ico')
coll = COLLECT(exe,
               a.binaries,
               a.zipfiles,
               a.datas,
               strip=False,
               upx=True,
               upx_exclude=[],
               name='Quiz')
app = BUNDLE(coll,
             name='Quiz.app',
             icon='/Users/dax/PycharmProjects/Quiz/rsc/quiz.ico',
             bundle_identifier=None)
```

这样就生成了一个onedir的APP，和我平时看到的一模一样的。

## 感悟

1. 没事别升级Python的版本，除非迫不得已；
2. 最好的，最齐全的找答案的地方，其实是帮助文件。如果能够早点仔细看PyInstaller的文档的话，应该能够少走不少弯路。