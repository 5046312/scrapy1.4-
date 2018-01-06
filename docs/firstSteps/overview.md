# Scrapy 初见

Scrapy 是一款专为抓取网页和提取可用于数据挖掘、信息处理或历史档案等应用场景结构化数据的应用框架。
尽管Scrapy当初被设计成“网络爬虫”，但是它也能用于使用API来提取数据，或是仅作为一个通用的网络爬虫来使用。

# 运行爬虫Demo

为了快速向你们展示Scrapy，我们将用一个最简单的方式来运行一个爬虫Demo。
下面就是从 http://quotes.toscrape.com 页面上爬取名人名言的爬虫代码。

```python
import scrapy
    class QuotesSpider(scrapy.Spider):
        name = "quotes"
        start_urls = [
            'http://quotes.toscrape.com/tag/humor/',
        ]
    
    def parse(self, response):
        for quote in response.css('div.quote'):
            yield {
                'text': quote.css('span.text::text').extract_first(),
                'author': quote.xpath('span/small/text()').extract_first(),
            }

        next_page = response.css('li.next a::attr("href")').extract_first()
        if next_page is not None:
            yield response.follow(next_page, self.parse)
```

把代码复制到text文件中，然后起一个类似 ``quotes_spider.py`` 的名字，然后使用
`runspider` 命令来运行爬虫:

```
    scrapy runspider quotes_spider.py -o quotes.json
```

当运行完成之后，你会在文件目录下找到一个 包含 名言、作者的 ``quotes.json`` 文件
就像这样 (格式化)::
```json
[{
    "author": "Jane Austen",
    "text": "\u201cThe person, be it gentleman or lady, who has not pleasure in a good novel, must be intolerably stupid.\u201d"
},
{
    "author": "Groucho Marx",
    "text": "\u201cOutside of a dog, a book is man's best friend. Inside of a dog it's too dark to read.\u201d"
},
{
    "author": "Steve Martin",
    "text": "\u201cA day without sunshine is like, you know, night.\u201d"
},
...]
```

# 刚刚发生了什么?

当你运行命令 ``scrapy runspider quotes_spider.py``, Scrapy 会寻找它内部定义的蜘蛛，然后通过爬虫引擎来运行它。

爬虫首先是按照 ``start_urls`` 属性（上面的Demo中只有"humor"分类的url）中定义的url地址开始请求，
然后调用默认回调方法 ``parse``, 传递了一个response对象作为第二个参数。

在回调方法 ``parse`` 中, 我们将使用CSS选择器循环提取的名言文本和作者以字典的形式返回(yield)，
寻找前往下一页的链接，使用同一个``parse``回调方法再一次进行请求。

你可能已经发现了关于Scrapy的一个主要特性：
> 每一个请求都可以有计划、异步进行的。

这意味着Scrapy不需要等待请求完成，便可以发送其他的请求（或做其他的事）。
这也意味着即使有一些请求失败或出错了，其他的请求依旧可以继续运行。

虽然这个特性能让你设计蜘蛛的爬行速度超级快（同时发送多个容错率很高的请求），但我们更推荐你可以通过设置来让爬行速度慢下来以表尊重。
比如，你可以在每次请求之间设置一些时间间隔、限制每个域名？或者IP的并发量，甚至可以使用自动限速的拓展来自动进行设置！

> 注意：这里生成的是Json格式的文件，你可以很容易的导出XML、CSV等其他格式的文件，或者直接储存到后台服务器中去。你也可以编写一个item pipeline将数据保存到数据库中。

# 然后呢?

你已经大概了解了Scrapy是如何从网页中导出和储存item了，但这只是皮毛。。。
Scrapy提供了很多让爬虫更简便有效的强大的特性，例如：
1. 可以使用内置的拓展CSS和XPath选择器（用正则提取）就可以从html/xml源文件中选择和提取数据
2. 很多可以使用正则的提取方法。
3. 一个交互shell控制台(IPython aware)，可以让你在编写或调试CSS和XPath提取器抓取数据时非常轻松。
4. 内置支持以多种格式(JSON、CSV、XML)生成feed导出，并将它们存储在多个后端(FTP、S3、本地系统)。
5. 健壮的编码支持和自动检测，用于处理外来的、非标准的和破碎的编码声明。
6. 强大的可扩展性，允许使用自定义的API(中间件、扩展和管道pipelines)来拓展更多的功能。
7. 更多内置拓展和中间件来进行方便你的工作：
    1. cookies 和 session 处理
    2. HTTP特性，像压缩，身份识别，缓存
    3. user-agent 伪装
    4. robots.txt
    5. 限制爬行深度
    6. 更多……
8. 一个运行在Scrapy进程中的Telnet控制台，来更方便的调试你的爬虫。
9. 另外还有一些其他的好东西，比如可以从Sitemaps和XML/CSV抓取数据的可复用蜘蛛，一个用于自动下载图像(或其他媒体文件)的媒体pipeline，一个缓存DNS解析器，当然还有更多!  

# 下一步

你的下一步就要开始安装Scrapy了，接下来你可以通过教程学习到如何创建一个完整的Scrapy项目，也可以参与讨论快速进步。感谢你的支持！  
