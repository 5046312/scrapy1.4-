# Scrapy 教程

在本教程开始前，我们假设你已经在你的系统里安装了Scrapy。
如果还未安装，可以查看 `安装Scrapy` 章节。

我们将抓取一个名人名言列表的网站。`http://quotes.toscrape.com/`

这个教程将带你完成下面这些需求：
1. 创建一个Scrapy项目。
2. 写一个爬虫 `spider` 来爬行网站、提取数据。
3. 使用命令行来导出抓取到的数据。
4. 修改爬虫进行递归的爬行链接。
5. 使用爬虫参数。

Scrapy是用Python编写的。如果你对Python这门语言比较陌生的话，你可能需要先从了解Python开始，那将会比直接学习Scrapy收获更多！

如果你已经对其他的语言非常熟悉了，并且想快速学习Python的话，我们推荐你查看`Dive Into Python 3`，你也可以看一看`Python Tutorial`来进行学习。

如果你是一个想从Python开始编程的新手，你可以试试电子书`Learn Python The Hard Way`，你也可以认真的读一读`this list of Python resources for non-programmers`.

* Python: https://www.python.org/
* this list of Python resources for non-programmers: https://wiki.python.org/moin/BeginnersGuide/NonProgrammers
* Dive Into Python 3: http://www.diveintopython3.net
* Python Tutorial: https://docs.python.org/3/tutorial
* Learn Python The Hard Way: http://learnpythonthehardway.org/book/

# 新建项目

在开始爬虫项目之前，你需要先创建一个Scrapy项目。进入一个你想存放代码的文件夹，运行:

    scrapy startproject tutorial

将会创建一个 ``tutorial`` 文件夹，内容如下:

    tutorial/
        scrapy.cfg            # 配置文件
        tutorial/             # 项目的Python模块，你将在这里编写代码
            __init__.py
            items.py          # 
            pipelines.py      # 项目中pipeline的定义文件
            settings.py       # 项目的设置文件
            spiders/          # 一会将用来存放你编写蜘蛛代码的文件夹
                __init__.py


# 第一只爬虫

爬虫都是你定义的类，而Scrapy使用它们从网页中爬取数据。它们必须都继承`scrapy.Spider`，并且要定义初始请求要做的工作，定义如何在页面中爬行链接、如何解析页面中的内容来导出数据。

这是我们的第一个爬虫的代码。将它保存在项目中``tutorial/spiders``目录下，命名为``quotes_spider.py``的文件中:

```python
    import scrapy


    class QuotesSpider(scrapy.Spider):
        name = "quotes"

        def start_requests(self):
            urls = [
                'http://quotes.toscrape.com/page/1/',
                'http://quotes.toscrape.com/page/2/',
            ]
            for url in urls:
                yield scrapy.Request(url=url, callback=self.parse)

        def parse(self, response):
            page = response.url.split("/")[-2]
            filename = 'quotes-%s.html' % page
            with open(filename, 'wb') as f:
                f.write(response.body)
            self.log('Saved file %s' % filename)
```


如你所见，爬虫的子类是`scrapy.Spider`，然后定义了一些属性和方法：

* `name`: 是爬虫的标识。项目中必须使用唯一的name，也就是说，你不能为不同的蜘蛛起重复的名字。
* `start_requests()`: 必须返回一个可以迭代的请求（可以返回一个请求列表，或者编写一个生成器函数），这样才会开始爬行。后续的请求也将依次从这些初始请求中生成。
* `parse()`: 用于处理每次请求响应的方法。响应参数是保存页面内容的`TextResponse`的对象，并且有进一步的帮助方法来处理它。

`parse()` 方法经常用于解析响应，将抓取的数据提取为字典形式，然后找到新的url来追踪，再新建一个 `Request` 。

## 如何运行爬虫程序

要让爬虫开始工作，需要到项目根目录，运行:

	scrapy crawl quotes

这个命令运行了我们刚刚添加的名为 ``quotes`` 的爬虫，它将向 ``quotes.toscrape.com`` 发送请求。你会得到一个类似于这样的输出:

    ... (omitted for brevity)
    2016-12-16 21:24:05 [scrapy.core.engine] INFO: Spider opened
    2016-12-16 21:24:05 [scrapy.extensions.logstats] INFO: Crawled 0 pages (at 0 pages/min), scraped 0 items (at 0 items/min)
    2016-12-16 21:24:05 [scrapy.extensions.telnet] DEBUG: Telnet console listening on 127.0.0.1:6023
    2016-12-16 21:24:05 [scrapy.core.engine] DEBUG: Crawled (404) <GET http://quotes.toscrape.com/robots.txt> (referer: None)
    2016-12-16 21:24:05 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://quotes.toscrape.com/page/1/> (referer: None)
    2016-12-16 21:24:05 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://quotes.toscrape.com/page/2/> (referer: None)
    2016-12-16 21:24:05 [quotes] DEBUG: Saved file quotes-1.html
    2016-12-16 21:24:05 [quotes] DEBUG: Saved file quotes-2.html
    2016-12-16 21:24:05 [scrapy.core.engine] INFO: Closing spider (finished)
    ...

现在再查看当前目录的文件，你应该发现了两个新生成的文件 *quotes-1.html* 和 *quotes-2.html*，里面包含了 ``parse`` 方法通过各个Url所获取的响应body。

> 如果您想知道为什么我们还没有解析HTML，请稍候，我们将很快就开始讨论这个问题。

## 背后发生了什么?

Scrapy 安排调度 ``start_requests``方法返回的 `scrapy.Request` 对象。
在收到每个响应之后，它将实例化`Response`对象，并调用与请求关联的回调方法(在本例中，``parse``方法)将响应作为参数进行传递。

## start_requests方法的快捷方式

与其使用`start_requests`方法来从url生成`scrapy.Request`对象，不如定义类中的`start_urls`属性（url列表）。
`start_requests`将默认使用这个列表来创建初始的请求：

```python
import scrapy


class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = [
        'http://quotes.toscrape.com/page/1/',
        'http://quotes.toscrape.com/page/2/',
    ]

    def parse(self, response):
        page = response.url.split("/")[-2]
        filename = 'quotes-%s.html' % page
        with open(filename, 'wb') as f:
            f.write(response.body)
```

`parse()`方法将处理这些url的每一个请求，尽管我们还没有明确告诉Scrapy要这样做。
这是因为`parse()` 是 Scrapy 的默认回调方法，它在没有被分配回调的情况下默认调用。


## 提取数据

学习如何用 Scrapy 提取数据的最好方法是使用shell的 `Scrapy shell` 来测试选择器:

```
scrapy shell 'http://quotes.toscrape.com/page/1/'
```

一定要记住，当从命令行中运行Scrapy时，url地址一定要包含在引号中，否则包含参数(如 ``&`` 符)的url将不起作用

   在Windows上，要使用双引号：

```
scrapy shell "http://quotes.toscrape.com/page/1/"
```

你将会看到一些类似的东西:

```
[ ... Scrapy log here ... ]
2016-09-19 12:09:27 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://quotes.toscrape.com/page/1/> (referer: None)
[s] Available Scrapy objects:
[s]   scrapy     scrapy module (contains scrapy.Request, scrapy.Selector, etc)
[s]   crawler    <scrapy.crawler.Crawler object at 0x7fa91d888c90>
[s]   item       {}
[s]   request    <GET http://quotes.toscrape.com/page/1/>
[s]   response   <200 http://quotes.toscrape.com/page/1/>
[s]   settings   <scrapy.settings.Settings object at 0x7fa91d888c10>
[s]   spider     <DefaultSpider 'default' at 0x7fa91c8af990>
[s] Useful shortcuts:
[s]   shelp()           Shell help (print this help)
[s]   fetch(req_or_url) Fetch request (or URL) and update local objects
[s]   view(response)    View response in a browser
>>>
```

使用shell，对响应对象使用`CSS`来测试选择元素：

```
>>> response.css('title')
[<Selector xpath='descendant-or-self::title' data='<title>Quotes to Scrape</title>'>]
```

``response.css('title')`` 的运行结果是一种类似列表的对象，称为`SelectorList`，它表示一个`Selector`对象的列表，这些对象包裹着XML / HTML元素，并允许您运行进一步的查询、选择子元素或提取数据。

要从上面的标题中提取文本，你可以这样做:

```
>>> response.css('title::text').extract()
['Quotes to Scrape']
```

这里要注意两点: 
* 我们已经在CSS查询中添加了``::text``，这意味着我们在``<title>``的元素中只选择文本元素。如果我们不指定``::text``，我们将获得完整的标题元素，包括它的标签。

```
>>> response.css('title').extract()
['<title>Quotes to Scrape</title>']
```

*  ``extract()`` 的返回结果是一个列表, 因为我们一会儿要处理`SelectorList`实例，如果你只想知道第一项的结果，你可以这样做:

```
>>> response.css('title::text').extract_first()
'Quotes to Scrape'
```

也可以这样代替:

```
>>> response.css('title::text')[0].extract()
'Quotes to Scrape'
```

而且使用``extract_first()``可以在找不到任何元素的时候返回``None``，避免了``IndexError``。

这里有一个小经验：对于大多数的抓取代码，更希望它能够对由于在页面上找不到的东西而产生的错误保持弹性，这样即使某些部分不能被抓取到，但至少可以得到 **一些** 数据。

除了 `extract()` 和 `extract_first()` 方法，也可以使用 `Selector.re()` 用*正则表达式*来选择元素：

```
>>> response.css('title::text').re(r'Quotes.*')
['Quotes to Scrape']
>>> response.css('title::text').re(r'Q\w+')
['Quotes']
>>> response.css('title::text').re(r'(\w+) to (\w+)')
['Quotes', 'Scrape']
```

为了找到合适的CSS选择器，可能需要在web浏览器中的shell找到可用的响应页面。
可以使用浏览器开发工具或扩展，如Firebug (查看 `firebug` 和 `firefox` 章节)。

`Selector Gadget` 是一个很好的工具，可以快速地找到CSS选择器，以选择在许多浏览器中使用的可视化元素。

* regular expressions: https://docs.python.org/3/library/re.html
* Selector Gadget: http://selectorgadget.com/


### XPath：简短的介绍

除了`CSS`，Scrapy 还支持使用 `XPath` 表达式:

```
>>> response.xpath('//title')
[<Selector xpath='//title' data='<title>Quotes to Scrape</title>'>]
>>> response.xpath('//title/text()').extract_first()
'Quotes to Scrape'
```

XPath表达式非常强大，并且是Scrapy选择器的基础。实际上，CSS选择器在底层也是被转换为XPath的。
如果仔细阅读shell中选择器对象的文本说明，你就会有所发现。

虽然XPath表达式可能不像CSS选择器那样受欢迎，但它提供了更多的功能，因为除了导航结构，它还可以查看内容。
使用XPath，可以选择诸如:*选择包含文本"下一页"的链接*。这使得XPath非常适用于抓取任务，我们鼓励你学习XPath，即使你已经知道如何构造CSS选择器，它也会使抓取变得更加容易。

关于Xpath我们在这里就不赘述了, 你可以去:`using XPath with Scrapy Selectors here`学习更多关于Xpath的内容。 
 
我们也推荐 `this tutorial to learn XPath through examples <http://zvon.org/comp/r/tut-XPath_1.html>` 和 `this tutorial to learn "how to think in XPath" <http://plasmasturm.org/log/xpath101/>`。


* XPath: https://www.w3.org/TR/xpath
* CSS: https://www.w3.org/TR/selectors

### 提取名言和作者

相信你已经了解了一些关于选择器和提取的知识，让我们通过编写从网页中提取名言的代码来完成我们的爬虫吧。

http://quotes.toscrape.com 中的每条名言都是类似这种Html结构:

```html
<div class="quote">
    <span class="text">“The world as we have created it is a process of our
    thinking. It cannot be changed without changing our thinking.”</span>
    <span>
        by <small class="author">Albert Einstein</small>
        <a href="/author/Albert-Einstein">(about)</a>
    </span>
    <div class="tags">
        Tags:
        <a class="tag" href="/tag/change/page/1/">change</a>
        <a class="tag" href="/tag/deep-thoughts/page/1/">deep-thoughts</a>
        <a class="tag" href="/tag/thinking/page/1/">thinking</a>
        <a class="tag" href="/tag/world/page/1/">world</a>
    </div>
</div>
```
让我们打开Scrapy shell，看看如何提取我们想要的数据:

```
$ scrapy shell 'http://quotes.toscrape.com'
```

我们获取一个含有名言html元素的列表:

```
>>> response.css("div.quote")
```

上面查询返回的每个选择器都允许我们对它们的子元素进行进一步查询。
让我们将第一个选择器分配一个变量，这样我们就可以直接在特定的名言上使用CSS选择器:

```
>>> quote = response.css("div.quote")[0]
```

现在，让我们用我们刚创建的``quote``对象导出其中的 ``标题``, ``作者`` 和 ``标签``：

```
>>> title = quote.css("span.text::text").extract_first()
>>> title
'“The world as we have created it is a process of our thinking. It cannot be changed without changing our thinking.”'
>>> author = quote.css("small.author::text").extract_first()
>>> author
'Albert Einstein'
```

由于标签是字符串的列表，所以我们可以使用``extract()``方法来获取所有的字符串:

```
>>> tags = quote.css("div.tags a.tag::text").extract()
>>> tags
['change', 'deep-thoughts', 'thinking', 'world']
```

我们已经找到了如何提取每个元素的方法，现在我们可以遍历所有名言元素，并将它们组合成Python字典:

```
>>> for quote in response.css("div.quote"):
...     text = quote.css("span.text::text").extract_first()
...     author = quote.css("small.author::text").extract_first()
...     tags = quote.css("div.tags a.tag::text").extract()
...     print(dict(text=text, author=author, tags=tags))
{'tags': ['change', 'deep-thoughts', 'thinking', 'world'], 'author': 'Albert Einstein', 'text': '“The world as we have created it is a process of our thinking. It cannot be changed without changing our thinking.”'}
{'tags': ['abilities', 'choices'], 'author': 'J.K. Rowling', 'text': '“It is our choices, Harry, that show what we truly are, far more than our abilities.”'}
    ... a few more of these, omitted for brevity
>>>
```

## 使用爬虫导出数据

让我们回到蜘蛛。直到现在，它还没有提取任何数据，只是将整个HTML页面保存到一个本地文件中。让我们将提取逻辑集成到爬虫中。

Scrapy爬虫通常生成许多包含从页面中提取数据的字典。要做到这一点,在回调中我们使用``yield``关键字：

```python
import scrapy


class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = [
        'http://quotes.toscrape.com/page/1/',
        'http://quotes.toscrape.com/page/2/',
    ]

    def parse(self, response):
        for quote in response.css('div.quote'):
            yield {
                'text': quote.css('span.text::text').extract_first(),
                'author': quote.css('small.author::text').extract_first(),
                'tags': quote.css('div.tags a.tag::text').extract(),
            }
```

如果运行爬虫，日志中将会输出提取的数据::

```
2016-09-19 18:57:19 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/page/1/>
{'tags': ['life', 'love'], 'author': 'André Gide', 'text': '“It is better to be hated for what you are than to be loved for what you are not.”'}
2016-09-19 18:57:19 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/page/1/>
{'tags': ['edison', 'failure', 'inspirational', 'paraphrased'], 'author': 'Thomas A. Edison', 'text': "“I have not failed. I've just found 10,000 ways that won't work.”"}
```

# 储存抓取的数据

存储这些抓取到的数据最简单方法是使用“Feed 导出”，使用下面的命令：

```
scrapy crawl quotes -o quotes.json
```

这将生成一个``quotes.json``文件，包含所有抓取到的item（序列化为json格式）。

出于历史原因，Scrapy将会把内容追加到一个给定的文件中，而不是覆盖原本的内容。
如果在第二次运行前，不删除（或清空）文件中的内容，运行此命令两次，最终将得到一个破坏的JSON文件。
你也可以使用其他格式，例如 `JSON Lines`:

```
scrapy crawl quotes -o quotes.jl
```

`JSON Lines`格式很有用，因为它是流式的，可以轻松地附加新记录。在运行两次时，不会遇到相同的JSON问题。
另外，由于每个记录都是单独的行，您可以处理成大文件，而不需要将所有内容都放在内存中，有像JQ这样的工具来帮助在命令行中执行这些操作。

在一些小型项目中(比如本教程中的项目)，这就足够了。但是，如果想要用抓取的Item来执行更复杂的操作，可以编写一个`Item Pipeline`。
当项目创建时，在``tutorial/pipelines.py``中生成了Pipeline的占位符文件。如果只是想存储这些抓取来的item，则不需要设置任何Pipeline。

* JSON Lines: http://jsonlines.org
* JQ: https://stedolan.github.io/jq


# 追踪链接

如果你不只是想从 http://quotes.toscrape.com 抓取前两页，而是获得网站中所有页面中的名言，
那么你就需要学习如何从全部页面中提取数据了，接下来让我们一起看看究竟该如何才能追踪到全部的链接吧！

第一件事就是提取想要跟随页面的链接。检查页面中的html信息，我们可以看到有一个链接是跳转到下一页的:

```html
<ul class="pager">
    <li class="next">
        <a href="/page/2/">Next <span aria-hidden="true">&rarr;</span></a>
    </li>
</ul>
```

我们可以试着在shell中把它提取出来:

```
>>> response.css('li.next a').extract_first()
'<a href="/page/2/">Next <span aria-hidden="true">→</span></a>'
```

这样我们就定位了这个元素，但我们想要得到``href``属性。
为此，Scrapy支持一个CSS扩展，可以让你用下面的方法来选择属性内容:

```
>>> response.css('li.next a::attr(href)').extract_first()
'/page/2/'
```

现在让我们看看我们的爬虫，它会递归地跟踪到下一页的链接，从中提取数据：

```python
import scrapy


class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = [
        'http://quotes.toscrape.com/page/1/',
    ]

    def parse(self, response):
        for quote in response.css('div.quote'):
            yield {
                'text': quote.css('span.text::text').extract_first(),
                'author': quote.css('small.author::text').extract_first(),
                'tags': quote.css('div.tags a.tag::text').extract(),
            }

        next_page = response.css('li.next a::attr(href)').extract_first()
        if next_page is not None:
            next_page = response.urljoin(next_page)
            yield scrapy.Request(next_page, callback=self.parse)

```

现在，在提取数据之后，``parse()``方法会查找前往下一个页面的链接，使用``urljoin``方法（因为链接可以是相对的）创建一个绝对URL，并返回一个前往下一页的新请求，注册为回调来提取下一个页面的数据，并继续爬行所有的页面。

你在这里看到的是Scrapy跟踪链接的机制：当你在一个回调方法中产生一个请求时，Scrapy将会调度这个请求，并在请求结束时注册一个回调方法。

利用这个机制，你可以构建复杂的爬虫程序，根据你定义的规则跟踪链接，并根据访问的页面提取不同类型的数据。

在我们的例子中创建了一个循环，可以跟踪链接到下一个页面的全部url，直到找不到为止 -- 可以非常方便地抓取博客、论坛和其他具有分页功能的站点。

## 发送请求的快捷方式

你可以使用 `response.follow()`，作为创建请求对象的快捷方式：

```python
import scrapy


class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = [
        'http://quotes.toscrape.com/page/1/',
    ]

    def parse(self, response):
        for quote in response.css('div.quote'):
            yield {
                'text': quote.css('span.text::text').extract_first(),
                'author': quote.css('span small::text').extract_first(),
                'tags': quote.css('div.tags a.tag::text').extract(),
            }

        next_page = response.css('li.next a::attr(href)').extract_first()
        if next_page is not None:
            yield response.follow(next_page, callback=self.parse)
```
与 ``scrapy.Request``不同， ``response.follow`` 支持相对url，所以不需要再调用``urljoin``方法，值得注意的是 ``response.follow`` 只能返回Request实例。你依然必须要返回这个Request。

你也可以给 ``response.follow`` 传递一个选择器，这个选择器应该提取必要的属性:

    for href in response.css('li.next a::attr(href)'):
        yield response.follow(href, callback=self.parse)

对于 ``<a>`` 标签，有一个快捷方法： ``response.follow`` 可以自动利用其中的href属性，所以代码还可以再简洁一些：

    for a in response.css('li.next a'):
        yield response.follow(a, callback=self.parse)

注意：

    ``response.follow(response.css('li.next a'))`` 是无效的，因为``response.css`` 返回一个包含全部结果的类似列表的选择器对象，并不是单独的一个选择器。使用``for``循环来解决才是正确的方法，或者 ``response.follow(response.css('li.next a')[0])`` 获取第一个选择器。

## 更多的例子和模式

这是另一个显示回调和跟随链接的爬虫，这一次是用来抓取作者信息的：

```python
    import scrapy


    class AuthorSpider(scrapy.Spider):
        name = 'author'

        start_urls = ['http://quotes.toscrape.com/']

        def parse(self, response):
            # follow links to author pages
            for href in response.css('.author + a::attr(href)'):
                yield response.follow(href, self.parse_author)

            # follow pagination links
            for href in response.css('li.next a::attr(href)'):
                yield response.follow(href, self.parse)

        def parse_author(self, response):
            def extract_with_css(query):
                return response.css(query).extract_first().strip()

            yield {
                'name': extract_with_css('h3.author-title::text'),
                'birthdate': extract_with_css('.author-born-date::text'),
                'bio': extract_with_css('.author-description::text'),
            }
```
爬虫将从主页开始，它将分别调用``parse_author``回调，来追踪所有链接到作者页面全部url，以及``parse``回调中的分页链接。

这里我们将回调传递给``response.follow``作为位置参数，使代码更简洁；它也同样适用于``scrapy.Request``。

``parse_author``回调定义了一个helper函数，用于从CSS查询中提取和清理数据，并以Python字典的形式返回作者的数据。

另一件有趣的事情是，即使有许多来自同一作者的引用，我们也不需要多次访问同一个作者页面。默认情况下，Scrapy会过滤掉已经访问过的url的重复请求，避免因为编程错误而过多地攻击服务器。这可以通过设置项 `DUPEFILTER_CLASS` 来配置。

如果你能够很好地理解如何追踪链接和Scrapy的回调机制将非常有帮助。

As yet another example spider that leverages the mechanism of following links,
check out the `CrawlSpider` class for a generic spider that implements a small rules engine that you can use to write your crawlers on top of it.

另外，一个常见的模式是使用多个页面的数据构建一个项目，使用一个技巧将额外的数据传递给回调函数。


使用爬虫参数
======================

你可以给爬虫传递一个命令行参数 ``-a``:

    scrapy crawl quotes -o quotes-humor.json -a tag=humor

这些参数会传递到爬虫的``__init__``方法中，成为默认属性。

在这个例子中，``tag``参数的值将赋值给 ``self.tag``。
你可以使用这种方式让你的爬虫仅获取带有特定标记的名言，并根据该参数构建URL:

```python
    import scrapy


    class QuotesSpider(scrapy.Spider):
        name = "quotes"

        def start_requests(self):
            url = 'http://quotes.toscrape.com/'
            tag = getattr(self, 'tag', None)
            if tag is not None:
                url = url + 'tag/' + tag
            yield scrapy.Request(url, self.parse)

        def parse(self, response):
            for quote in response.css('div.quote'):
                yield {
                    'text': quote.css('span.text::text').extract_first(),
                    'author': quote.css('small.author::text').extract_first(),
                }

            next_page = response.css('li.next a::attr(href)').extract_first()
            if next_page is not None:
                yield response.follow(next_page, self.parse)
```

如果你传递 ``tag=humor`` 参数给爬虫，你会发现它只会访问含有 ``humor`` 标签的url，例如 ``http://quotes.toscrape.com/tag/humor``。

你可以在`<spiderargs>`了解更多有关爬虫参数的内容。

下一步
==========

本教程只介绍了Scrapy的基础部分，但是这里还有很多其他的功能没有提到。
查看`overview`章节中的 `whatelse` 部分，会对重要部分有简要的概述。

你可以继续从`basics`部分了解有关更多命令行工具、爬虫、选择器和其他教程中美提到的内容。如果你想先使用一些示例项目，可以查看`examples`的内容。  
