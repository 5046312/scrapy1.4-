## 安装Scrapy

Scrapy可以运行在Python 2.7 和 Python 3.3及更高版本中。


如果你在用`Anaconda` 或 `Miniconda`, 你可以通过 `conda-forge` 来安装包, 已经为 Linux, Windows and OS X升级到最新版本了。

使用 ``conda`` 安装包, 运行

```
conda install -c conda-forge scrapy
```

如果你已经熟悉了Python包的安装方式，你可以使用PyPI来安装包和所有依赖:

```
pip install Scrapy
```

有时候需要注意解决一下Scrapy的依赖项在你系统中的编译问题，所以一定要检查一下 `特定平台安装指南`。

我们强烈建议你用 `虚拟环境` 来安装Scrapy来避免一些包之间的冲突发生。

下面是关于细节和平台的详细说明。

# 值得了解的一些事

Scrapy是用纯Python编写的，它依赖于几个关键的Python包:

* `lxml`, 高效的 XML 和 HTML 解析器
* `parsel`, 在lxml之上编写的HTML / XML数据提取库
* `w3lib`, 用于处理url和web页面编码的多功能helper
* `twisted`, 异步网络框架
* `cryptography` 和 `pyOpenSSL`, 用来处理各种网络安全等级的需要

Scrapy测试过的最旧版本:

* Twisted 14.0
* lxml 3.4
* pyOpenSSL 0.14

Scrapy用这些旧版本的包可以运行，但不保证可以一直正常运行下去，因为并不是针对他们来做的测试。

其中一些包本身依赖于非python包，这可能需要额外的安装步骤，取决于你的运行环境。
请用`特定平台安装指南`来检查

如果包之间的互相依赖有任何问题，请参考他们各自的安装说明:

* `lxml installation`: http://lxml.de/installation.html
* `cryptography installation`: https://cryptography.io/en/latest/installation/

# 使用virtualenv (推荐)

我们建议在所有平台的虚拟环境中安装Scrapy。
可以在全局或用户空间安装Python包。我们不建议直接在系统环境下安装Scrapy。
相反，我们建议在“虚拟环境”(virtualenv)中安装Scrapy。
virtualenv允许您不与已安装的Python系统包冲突(这可能会破坏您的系统工具和脚本)，并且仍然可以正常地使用pip(不包括 ``sudo``)安装包。

要使用虚拟环境，请参阅virtualenv安装说明。
要在全局安装它(在全局安装实际上是有很有用的):

```
$ [sudo] pip install virtualenv
```

查看`user guide`来学习如何创建虚拟环境。

!> 如果您使用Linux或OS X，那么`virtualenvwrapper`是创建`虚拟环境`比较方便的工具。一旦你创建了一个`虚拟环境`，你就可以用`pip`像安装其他Python包一样的安装Scrapy。  
（参见：`特定平台安装指南` 包含可能需要预先安装的非python依赖项的说明）

默认情况下可以使用Python 2或者Python 3，创建Python `虚拟环境`。

* 如果你想用Python3安装Scrapy，就在Python 3 `虚拟环境`中安装Scrapy。
* 如果你想用Python2安装Scrapy，就在Python 2 `虚拟环境`中安装Scrapy。  


* virtualenv: https://virtualenv.pypa.io
* virtualenv installation instructions: https://virtualenv.pypa.io/en/stable/installation/
* virtualenvwrapper: https://virtualenvwrapper.readthedocs.io/en/latest/install.html
* user guide: https://virtualenv.pypa.io/en/stable/userguide/  

# 特定平台安装指南  

## Windows

虽然可以在Windows上使用pip安装Scrapy，但是我们推荐您
安装“Anaconda”或“Miniconda”，并使用该包“conda - forge”频道，因为可以避免大多数安装问题的发生。

一旦你安装了“Anaconda”或“Miniconda”，就可以用命令进行安装了:

```
conda install -c conda-forge scrapy
```

## Ubuntu 12.04 or above

目前，Scrapy已经用了足够多版本的lxml、twisted和pyOpenSSL进行了测试，Scrapy已经与最新的Ubuntu发行版兼容。
但它应该也是支持旧版本的Ubuntu的，比如Ubuntu 12.04，尽管存在TLS连接的潜在问题。


**千万不要** 使用 ``python-scrapy`` Ubuntu提供的Python包, 它们都太老太慢了，已经不适用于最新版Scrapy了！

要在Ubuntu系统(或基于ubuntubased)的系统上安装“Scrapy”，你需要安装这些依赖项:

```
sudo apt-get install python-dev python-pip libxml2-dev libxslt1-dev zlib1g-dev libffi-dev libssl-dev
```

- ``python-dev``, ``zlib1g-dev``, ``libxml2-dev`` 和 ``libxslt1-dev``
  都被 ``lxml`` 依赖
- ``libssl-dev`` 和 ``libffi-dev`` 被 ``cryptography`` 依赖

如果您想在python3上安装Scrapy，您还需要Python 3开发header:

```
sudo apt-get install python3 python3-dev
```

在安装完虚拟环境之后你就可以使用 ``pip`` 来安装Scrapy:

```
pip install scrapy
```

!>同样的非python依赖项也可以用于Debian Wheezy (7.0)及新版环境中安装Scrapy

## Mac OS X

构建Scrapy的依赖关系需要有一个C编译器和development header。
在OS X上，这通常是由苹果的Xcode开发工具提供的。
要安装Xcode命令行工具，打开一个控制台并运行:

```
 xcode-select --install
```

有一个已知的问题< https://github.com/pypa/pip/issues/2468 > 防止“pip”更新系统包。
这是成功安装Scrapy及其附件必须要解决的问题。
这里有一些建议解决方案:

* *(推荐)* **千万不要** 使用系统的Python, 安装一个不和系统部分冲突的全新或更新的版本。
下面就是如何使用`homebrew`包管理器来安装：

  * 按照 http://brew.sh/ 的指示安装`homebrew`

  * 更新 ``PATH`` 变量确保homebrew包应该在系统包之前引用 (如果你使用`zsh`作为默认shell，请将 ``.bashrc`` 对应改为 ``.zshrc`` ):
 
```
echo "export PATH=/usr/local/bin:/usr/local/sbin:$PATH" >> ~/.bashrc
```

  * 重新加载 ``.bashrc`` ，确保已经修改成功::

```
source ~/.bashrc
```

  * 安装python::

```
brew install python
```

  * 最新版本的Python已经将 ``pip``进行了捆绑，所以你不必再分别安装它们了。如果不是这样的，请更新你的python::

```
brew update; brew upgrade python
```

* *(可选)* 在一个隔离的环境中安装Python.
  这种方法是解决上述OS X问题的一种变通方法，但它是管理依赖关系的总体良好实践，并且可以作为第一个方法的补充。

`虚拟环境`是一个可以用来在python中创建虚拟环境的工具。
  我们建议你可以阅读类似的教程
  http://docs.python-guide.org/en/latest/dev/virtualenvs/ 来进行学习.

在完成这些工作之后，你应该能够成功的安装Scrapy了:

```
pip install Scrapy
```
