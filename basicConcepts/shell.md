Scrapy shell
============

Scrapy shell是一个交互终端，供你在未启动spider的情况下快速体验及调试爬取代码。它是用来测试数据提取代码的，但是实际上您可以使用它来测试任何类型的代码，因为它也是一个常规的Python shell。

shell用于测试XPath或CSS表达式，查看它们是如何工作的，还能查看从你试图提取的web页面中提取的数据。它允许你在编写爬行器时交互式地测试你的表达式，而不必每次更改后运行爬行器来测试。

一旦熟悉了Scrapy shell，你就会发现它是开发和调试爬虫的一个非常有用的工具。

配置 shell
=====================

如果你已经安装了 `IPython`， Scrapy shell 将会使用它 (而不是标准的Python控制台)。 `IPython` 控制台功能强大得多，它提供了智能自动完成和彩色输出等功能。

我们强烈建议您安装`IPython`，特别是在Unix系统上(`IPython`擅长的地方)。有关更多信息，请参阅 `IPython安装指南`。

Scrapy 还支持 `bpython`，并将尝试在 `IPython` 不可用的地方使用它。

Through scrapy's settings you can configure it to use any one of
``ipython``, ``bpython`` or the standard ``python`` shell, regardless of which
are installed. This is done by setting the ``SCRAPY_PYTHON_SHELL`` environment
variable; or by defining it in your `scrapy.cfg <topics-config-settings>`:

    [settings]
    shell = bpython

* IPython: http://ipython.org/
* IPython installation guide: http://ipython.org/install.html
* bpython: http://www.bpython-interpreter.org/

启动 shell
================

你可以像这样使用 `shell` 命令来启动 Scrapy shell：

    scrapy shell <url>

``<url>`` 是你想爬取的URL地址。

`shell` 对本地文件同样有效。如果你想玩一下一个网页的本地拷贝，也是很方便的。`shell` 理解本地文件的以下语法：

    # UNIX-style
    scrapy shell ./path/to/file.html
    scrapy shell ../other/path/to/file.html
    scrapy shell /absolute/path/to/file.html

    # File URI
    scrapy shell file:///absolute/path/to/file.html

> 注意：当你使用相对路径时， 要用``./`` (或 ``../`` when relevant)放在前面。``scrapy shell index.html`` 不会像期望的那样工作(这是设计而不是bug)。因为`shell`支持HTTP url而不是文件uri。``index.html``在语法上与类似于``example.com``的相似，`shell`将会把``index.html``当成域名处理，并触发DNS查找错误:

        $ scrapy shell index.html
        [ ... scrapy shell starts ... ]
        [ ... traceback ... ]
        twisted.internet.error.DNSLookupError: DNS lookup failed:
        address 'index.html' not found: [Errno -5] No address associated with hostname.

如果当前文件夹下存在名为``index.html``的文件，`shell` 将不会预先测试。再次强调。


使用 shell
===============

Scrapy终端仅仅是一个普通的Python终端(或 你安装的`IPython` )。其提供了一些额外的快捷方式。

可用的快捷命令
-------------------

 * ``shelp()`` - 打印可用对象及快捷命令的帮助列表

 * ``fetch(url[, redirect=True])`` - fetch a new response from the given
   URL and update all related objects accordingly. You can optionaly ask for
   HTTP 3xx redirections to not be followed by passing ``redirect=False``

 * ``fetch(request)`` - 根据给定的请求(request)或URL获取一个新的response，并更新相关的对象

 * ``view(response)`` - 在本机的浏览器打开给定的response。 其会在response的body中添加一个 < base > tag ，使得外部链接(例如图片及css)能正确显示。 注意，该操作会在本地创建一个临时文件，且该文件不会被自动删除。

* < base > tag: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/base

可用的Scrapy对象
------------------------

Scrapy shell 根据下载的页面会自动创建一些方便使用的对象, 像 `Response` 对象 和 `Selector` 对象 (都有 HTML 和 XML 内容)。

这些对象有:

 * ``crawler`` - the current :class:`~scrapy.crawler.Crawler` object.

 * ``spider`` - the Spider which is known to handle the URL, or a
   :class:`~scrapy.spiders.Spider` object if there is no spider found for
   the current URL

 * ``request`` - a :class:`~scrapy.http.Request` object of the last fetched
   page. You can modify this request using :meth:`~scrapy.http.Request.replace`
   or fetch a new request (without leaving the shell) using the ``fetch``
   shortcut.

 * ``response`` - a :class:`~scrapy.http.Response` object containing the last
   fetched page

 * ``settings`` - the current :ref:`Scrapy settings <topics-settings>`

shell session 示例
========================

这里有一个典型的shell session的例子，我们首先抓取http://scrapy.org页面，然后继续抓取https://reddit.com页面。最后，我们修改了(Reddit)请求方法来POST并重新获取它的错误。我们通过输入Ctrl-D(在Unix系统中)或Ctrl-Z（Windows）结束会话。

请记住，这里提取的数据在你尝试时可能会不一样，因为这些页面不是静态的，在测试时可能会发生变化。本例的唯一目的是让您熟悉Scrapy的工作原理。

首先，我们启动 shell:

    scrapy shell 'http://scrapy.org' --nolog

接着该终端(使用Scrapy下载器(downloader))获取URL内容并打印可用的对象及快捷命令(注意到以 ``[s]` 开头的行):

    [s] Available Scrapy objects:
    [s]   scrapy     scrapy module (contains scrapy.Request, scrapy.Selector, etc)
    [s]   crawler    <scrapy.crawler.Crawler object at 0x7f07395dd690>
    [s]   item       {}
    [s]   request    <GET http://scrapy.org>
    [s]   response   <200 https://scrapy.org/>
    [s]   settings   <scrapy.settings.Settings object at 0x7f07395dd710>
    [s]   spider     <DefaultSpider 'default' at 0x7f0735891690>
    [s] Useful shortcuts:
    [s]   fetch(url[, redirect=True]) Fetch URL and update local objects (by default, redirects are followed)
    [s]   fetch(req)                  Fetch a scrapy.Request and update local objects
    [s]   shelp()           Shell help (print this help)
    [s]   view(response)    View response in a browser

    >>>


之后，你就可以操作这些对象了:

    >>> response.xpath('//title/text()').extract_first()
    'Scrapy | A Fast and Powerful Scraping and Web Crawling Framework'

    >>> fetch("http://reddit.com")

    >>> response.xpath('//title/text()').extract()
    ['reddit: the front page of the internet']

    >>> request = request.replace(method="POST")

    >>> fetch(request)

    >>> response.status
    404

    >>> from pprint import pprint

    >>> pprint(response.headers)
    {'Accept-Ranges': ['bytes'],
     'Cache-Control': ['max-age=0, must-revalidate'],
     'Content-Type': ['text/html; charset=UTF-8'],
     'Date': ['Thu, 08 Dec 2016 16:21:19 GMT'],
     'Server': ['snooserv'],
     'Set-Cookie': ['loid=KqNLou0V9SKMX4qb4n; Domain=reddit.com; Max-Age=63071999; Path=/; expires=Sat, 08-Dec-2018 16:21:19 GMT; secure',
                    'loidcreated=2016-12-08T16%3A21%3A19.445Z; Domain=reddit.com; Max-Age=63071999; Path=/; expires=Sat, 08-Dec-2018 16:21:19 GMT; secure',
                    'loid=vi0ZVe4NkxNWdlH7r7; Domain=reddit.com; Max-Age=63071999; Path=/; expires=Sat, 08-Dec-2018 16:21:19 GMT; secure',
                    'loidcreated=2016-12-08T16%3A21%3A19.459Z; Domain=reddit.com; Max-Age=63071999; Path=/; expires=Sat, 08-Dec-2018 16:21:19 GMT; secure'],
     'Vary': ['accept-encoding'],
     'Via': ['1.1 varnish'],
     'X-Cache': ['MISS'],
     'X-Cache-Hits': ['0'],
     'X-Content-Type-Options': ['nosniff'],
     'X-Frame-Options': ['SAMEORIGIN'],
     'X-Moose': ['majestic'],
     'X-Served-By': ['cache-cdg8730-CDG'],
     'X-Timer': ['S1481214079.394283,VS0,VE159'],
     'X-Ua-Compatible': ['IE=edge'],
     'X-Xss-Protection': ['1; mode=block']}
    >>>



从爬虫中调用Shell来检查相应
====================================================

有时，如果你只是检查所期望的响应是否到达那里，您需要检查在爬虫的某一点上正在处理的响应。

这可以通过 ``scrapy.shell.inspect_response`` 函数来实现。

以下是如何在spider中调用该函数的例子： 

```
    import scrapy


    class MySpider(scrapy.Spider):
        name = "myspider"
        start_urls = [
            "http://example.com",
            "http://example.org",
            "http://example.net",
        ]

        def parse(self, response):
            # We want to inspect one specific response.
            if ".org" in response.url:
                from scrapy.shell import inspect_response
                inspect_response(response, self)

            # Rest of parsing code.
```

当运行spider时，将得到类似的输出:

```
    2014-01-23 17:48:31-0400 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://example.com> (referer: None)
    2014-01-23 17:48:31-0400 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://example.org> (referer: None)
    [s] Available Scrapy objects:
    [s]   crawler    <scrapy.crawler.Crawler object at 0x1e16b50>
    ...

    >>> response.url
    'http://example.org'
```

然后，可以检查提取代码是否在工作::

    >>> response.xpath('//h1[@class="fn"]')
    []

呃，看来是没有。你可以在浏览器里查看response的结果，判断是否是您期望的结果:

    >>> view(response)
    True

最后你可以点击Ctrl-D(Windows下Ctrl-Z)来退出shell终端，恢复爬取:

```
    >>> ^D
    2014-01-23 17:50:03-0400 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://example.net> (referer: None)
    ...
```

注意: 由于该终端屏蔽了Scrapy引擎，你在这个Shell中不能使用 ``fetch`` 快捷命令。 当离开Shell时，spider会从其停下的地方恢复爬取，正如上面显示的那样。
