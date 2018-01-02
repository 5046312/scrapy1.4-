# Spiders 爬虫

爬虫都是类(class)，定义了如何抓取特定的网站(或一组网站)，包括如何执行爬行(也就是追踪链接)以及如何从它们的页面中提取结构化数据(例如抓取item)。换句话说，爬虫是你定义为爬行和解析特定网站页面(在某些情况下，是一组网站)的地方。

对于爬虫来说，抓取周期会经历类似这样的步骤：

1. 首先，抓取第一个url，生成初始请求，并指定一个回调函数，调用以从这些请求中下载响应。
要执行的第一个请求是通过调用`start_requests()`方法获得的，该方法(默认情况下)会使用`start_urls`中指定的url地址生成请求，而`parse()`方法作为请求的回调函数。

2. 在回调函数中，解析响应(web页面)，并返回提取到数据的字典、`Item`对象、`Request`对象。这些请求也将包含一个回调(可能是相同的)，然后由Scrapy下载，获得的相应也将由指定的回调进行处理。

3. 在回调函数中,通常使用`选择器`(也可以使用 BeautifulSoup、lxml或其他你喜欢的方式)解析页面内容，然后使用解析后的数据生成 Item。

4. 最后，爬虫返回的 Item 通常会持久化到数据库中(在某些项目管道（Pipeline）中)，或者使用`feed-exports`将其写入到文件中。

尽管这个周期对任何类型的爬虫都适用(或多或少)，有不同种类、不同目的的默认爬虫被绑定到Scrapy中。
我们将在这里讨论这些不同的类型。

# scrapy.Spider

## <font color=#42b983>class</font> scrapy.spiders.Spider

这是最简单的爬虫，也是所有其他蜘蛛必须继承的(包括与Scrapy捆绑在一起的爬虫，以及你自己编写的爬虫)。它没有提供任何特殊的功能。它只提供了一个默认的`start_requests()`方法用来发送`starturl`属性中url的请求，并为每个响应结果调用``parse()``方法。

### 属性

> name

定义此爬虫名称的字符串。  
爬虫的名字是用于Scrapy定位的(并实例化)，所以它必须是独一无二的。
但是，没有什么可以阻止你实例化多个相同的爬虫实例。
这是最重要的爬虫属性，也是必需的。

如果爬虫只在一个域名中爬行，那么通常的做法是在使用域名来命名爬虫。例如，爬行`mywebsite.com`的爬虫通常会命名为`mywebsite`。

!> 在Python 2中，`name`属性必须是 ASCII 格式。

> allowed_domains

一个可选的字符串列表，其中包含该爬虫允许爬行的域名。  
如果启用了`OffsiteMiddleware`，不属于这个列表中指定的域名(或它们的子域)的请求将不会被追踪。
  
如果你的目标Url为 ``https://www.example.com/1.html``，那么就把``'example.com'``添加到该列表中。

> start_urls

当没有特别指定url时，爬虫将从这个Url列表开始爬行。  
因此，第一个下载的页面将列在这里。后续的url将依次从开始的url中包含的数据中生成。

> custom_settings

在运行爬虫时，一个将从项目配置中覆盖的设置项字典。必须定义为一个类属性，因为在实例化之前，设置项会更新。

可用的内置设置项列表：`settings`。

> crawler

这个属性是在初始化类之后由`from_crawler()`类方法设置的，并链接到这个`Crawler`对象的爬虫实例。

爬虫在项目中封装了许多组件，用于它们的单个条目访问(例如扩展、中间件、信号管理器等)。参见`crawler`，了解更多关于它们的信息。

> settings

用于运行此爬虫的配置。这是一个`Settings`实例，请参阅`Settings`章节，了解更多介绍。

> logger

Python日志记录器以爬虫的`name`创建。可以使用它发送日志消息，就像在`logging`中描述的那样。


### 方法

> from_crawler(crawler, \*args, \**kwargs)

使用Scrapy创建爬虫的类方法。  
你可能不需要直接重写，因为默认实现为`__init__`方法的代理，使用给定参数`args`和命名参数`kwargs`进行调用。

尽管如此，该方法在新实例中设置`crawler`和`settings`属性，以便稍后在爬虫代码中访问它们。

#### 参数：

- crawler (**Crawler**实例) – crawler to which the spider will be bound
- args (list) – arguments passed to the __init__() method
- kwargs (dict) – keyword arguments passed to the __init__() method


> start_requests()

该方法会返回爬虫第一次请求的可迭代对象。当爬虫执行时就会被调用。
Scrapy只会调用一次这个方法，因此将其实现为生成器是非常安全的。


The default implementation generates ``Request(url, dont_filter=True)`` for each url in `start_urls`.

如果您想要更改用于开始抓取域名的请求，可以使用这个方法覆写。
例如，如果您需要开始使用POST请求登录，您可以这样做:


```python

	class MySpider(scrapy.Spider):
	   name = 'myspider'
	
	   def start_requests(self):
	       return [scrapy.FormRequest("http://www.example.com/login",
	                                  formdata={'user': 'john', 'pass': 'secret'},
	                                  callback=self.logged_in)]
	
	   def logged_in(self, response):
	       # here you would extract links to follow and return Requests for
	       # each of them, with another callback
	       pass
```

> parse(response)

       This is the default callback used by Scrapy to process downloaded
       responses, when their requests don't specify a callback.

       The ``parse`` method is in charge of processing the response and returning
       scraped data and/or more URLs to follow. Other Requests callbacks have
       the same requirements as the :class:`Spider` class.

       This method, as well as any other Request callback, must return an
       iterable of :class:`~scrapy.http.Request` and/or
       dicts or :class:`~scrapy.item.Item` objects.

       :param response: the response to parse
       :type response: :class:`~scrapy.http.Response`

> log(message, [level, component])

       Wrapper that sends a log message through the Spider's :attr:`logger`,
       kept for backwards compatibility. For more information see
       :ref:`topics-logging-from-spiders`.

> closed(reason)

       Called when the spider closes. This method provides a shortcut to
       signals.connect() for the :signal:`spider_closed` signal.

Let's see an example::

```python
    import scrapy


    class MySpider(scrapy.Spider):
        name = 'example.com'
        allowed_domains = ['example.com']
        start_urls = [
            'http://www.example.com/1.html',
            'http://www.example.com/2.html',
            'http://www.example.com/3.html',
        ]

        def parse(self, response):
            self.logger.info('A response from %s just arrived!', response.url)
```

单个回调返回多个请求和Item：

```python
    import scrapy

    class MySpider(scrapy.Spider):
        name = 'example.com'
        allowed_domains = ['example.com']
        start_urls = [
            'http://www.example.com/1.html',
            'http://www.example.com/2.html',
            'http://www.example.com/3.html',
        ]

        def parse(self, response):
            for h3 in response.xpath('//h3').extract():
                yield {"title": h3}

            for url in response.xpath('//a/@href').extract():
                yield scrapy.Request(url, callback=self.parse)
```

Instead of :attr:`~.start_urls` you can use :meth:`~.start_requests` directly;
to give data more structure you can use :ref:`topics-items`::

```python
    import scrapy
    from myproject.items import MyItem

    class MySpider(scrapy.Spider):
        name = 'example.com'
        allowed_domains = ['example.com']

        def start_requests(self):
            yield scrapy.Request('http://www.example.com/1.html', self.parse)
            yield scrapy.Request('http://www.example.com/2.html', self.parse)
            yield scrapy.Request('http://www.example.com/3.html', self.parse)

        def parse(self, response):
            for h3 in response.xpath('//h3').extract():
                yield MyItem(title=h3)

            for url in response.xpath('//a/@href').extract():
                yield scrapy.Request(url, callback=self.parse)
```

# Spider参数

Spider可以通过接受参数来修改其功能。spider参数一般用来定义初始URL或者指定限制爬取网站的部分。 您也可以使用其来配置spider的任何功能。

在运行 `crawl`命令时添加 ``-a`` 可以传递Spider参数:

    scrapy crawl myspider -a category=electronics

Spider在`__init__`中获取参数:

    import scrapy

    class MySpider(scrapy.Spider):
        name = 'myspider'

        def __init__(self, category=None, *args, **kwargs):
            super(MySpider, self).__init__(*args, **kwargs)
            self.start_urls = ['http://www.example.com/categories/%s' % category]
            # ...

The default `__init__` method will take any spider arguments
and copy them to the spider as attributes.
The above example can also be written as follows::

    import scrapy

    class MySpider(scrapy.Spider):
        name = 'myspider'

        def start_requests(self):
            yield scrapy.Request('http://www.example.com/categories/%s' % self.category)

Keep in mind that spider arguments are only strings.
The spider will not do any parsing on its own.
If you were to set the `start_urls` attribute from the command line,
you would have to parse it on your own into a list
using something like
`ast.literal_eval <https://docs.python.org/library/ast.html#ast.literal_eval>`_
or `json.loads <https://docs.python.org/library/json.html#json.loads>`_
and then set it as an attribute.
Otherwise, you would cause iteration over a `start_urls` string
(a very common python pitfall)
resulting in each character being seen as a separate url.

A valid use case is to set the http auth credentials
used by :class:`~scrapy.downloadermiddlewares.httpauth.HttpAuthMiddleware`
or the user agent
used by :class:`~scrapy.downloadermiddlewares.useragent.UserAgentMiddleware`::

    scrapy crawl myspider -a http_user=myuser -a http_pass=mypassword -a user_agent=mybot

Spider参数也可以通过Scrapyd的 ``schedule.json`` API来传递。 参见 Scrapyd documentation.

# Generic Spiders

Scrapy comes with some useful generic spiders that you can use to subclass
your spiders from. Their aim is to provide convenient functionality for a few
common scraping cases, like following all links on a site based on certain
rules, crawling from `Sitemaps`, or parsing an XML/CSV feed.

For the examples used in the following spiders, we'll assume you have a project
with a ``TestItem`` declared in a ``myproject.items`` module::

    import scrapy

    class TestItem(scrapy.Item):
        id = scrapy.Field()
        name = scrapy.Field()
        description = scrapy.Field()



## CrawlSpider

爬取一般网站常用的spider。其定义了一些规则(rule)来提供追踪链接更方便的机制。 也许该spider并不是完全适合您的特定网站或项目，但其对很多情况都使用。 因此您可以以其为起点，根据需求修改部分方法。当然您也可以实现自己的spider。

除了从Spider继承过来的(您必须提供的)属性外，其提供了一个新的属性:
  

#### rules

一个包含一个(或多个) Rule 对象的集合(list)。 每个 Rule 对爬取网站的动作定义了特定表现。 Rule对象在下边会介绍。 如果多个rule匹配了相同的链接，则根据他们在本属性中被定义的顺序，会使用第一个。

该spider也提供了一个可复写(overrideable)的方法:

#### parse_start_url(response)

当start_url的请求返回时，该方法被调用。 该方法分析最初的返回值并必须返回一个 Item 对象或者 一个 Request 对象或者 一个可迭代的包含二者对象。

### Crawling rules

#### Rule(link_extractor, callback=None, cb_kwargs=None, follow=None, process_links=None, process_request=None)

   ``link_extractor`` 是一个 Link Extractor 对象。 其定义了如何从爬取到的页面提取链接。

   ``callback``  是一个callable或string(该spider中同名的函数将会被调用)。 从link_extractor中每获取到链接时将会调用该函数。该回调函数接受一个response作为其第一个参数， 并返回一个包含 `Item` 以及(或) `Request` 对象(或者这两者的子类)的列表(list)。

> 警告：当编写爬虫规则时，请避免使用 `parse` 作为回调函数。 由于 CrawlSpider 使用 `parse` 方法来实现其逻辑，如果 您覆盖了 `parse` 方法，`crawl spider` 将会运行失败。

   ``cb_kwargs`` 包含传递给回调函数的参数(keyword argument)的字典。

   ``follow`` 是一个布尔(boolean)值，指定了根据该规则从response提取的链接是否需要跟进。 如果 `callback` 为None， `follow` 默认设置为 `True` ，否则默认为 `False` 。

   ``process_links``  是一个callable或string(该spider中同名的函数将会被调用)。 从link_extractor中获取到链接列表时将会调用该函数。该方法主要用来过滤

   ``process_request`` 是一个callable或string(该spider中同名的函数将会被调用)。 该规则提取到每个request时都会调用该函数。该函数必须返回一个request或者None。 (用来过滤request)

### CrawlSpider 例子
接下来给出配合rule使用CrawlSpider的例子:

```python
    import scrapy
    from scrapy.spiders import CrawlSpider, Rule
    from scrapy.linkextractors import LinkExtractor

    class MySpider(CrawlSpider):
        name = 'example.com'
        allowed_domains = ['example.com']
        start_urls = ['http://www.example.com']

        rules = (
            # Extract links matching 'category.php' (but not matching 'subsection.php')
            # and follow links from them (since no callback means follow=True by default).
            Rule(LinkExtractor(allow=('category\.php', ), deny=('subsection\.php', ))),

            # Extract links matching 'item.php' and parse them with the spider's method parse_item
            Rule(LinkExtractor(allow=('item\.php', )), callback='parse_item'),
        )

        def parse_item(self, response):
            self.logger.info('Hi, this is an item page! %s', response.url)
            item = scrapy.Item()
            item['id'] = response.xpath('//td[@id="item_id"]/text()').re(r'ID: (\d+)')
            item['name'] = response.xpath('//td[@id="item_name"]/text()').extract()
            item['description'] = response.xpath('//td[@id="item_description"]/text()').extract()
            return item
```

该spider将从example.com的首页开始爬取，获取category以及item的链接并对后者使用 `parse_item` 方法。 当item获得返回(response)时，将使用XPath处理HTML并生成一些数据填入 `Item` 中。

## XMLFeedSpider

### XMLFeedSpider

XMLFeedSpider被设计用于通过迭代各个节点来分析XML源(XML feed)。 迭代器可以从 `iternodes` ， `xml` ， `html` 选择。 鉴于 `xml` 以及 `html` 迭代器需要先读取所有DOM再分析而引起的性能问题， 一般还是推荐使用 `iternodes` 。 不过使用 `html` 作为迭代器能有效应对错误的XML。

您必须定义下列类属性来设置迭代器以及标签名(tag name):

#### iterator
用于确定使用哪个迭代器的string。可选项有:

- ``'iternodes'`` - 一个高性能的基于正则表达式的迭代器
- ``'html'`` - 使用 `Selector` 的迭代器。 需要注意的是该迭代器使用DOM进行分析，其需要将所有的DOM载入内存， 当数据量大的时候会产生问题。

- ``'xml'`` - 使用 `Selector` 的迭代器。 需要注意的是该迭代器使用DOM进行分析，其需要将所有的DOM载入内存， 当数据量大的时候会产生问题。

默认值为  ``iternodes`` 。

#### itertag

        A string with the name of the node (or element) to iterate in. Example::

            itertag = 'product'

#### namespaces

一个由 ``(prefix, url)`` 元组(tuple)所组成的list。 其定义了在该文档中会被spider处理的可用的namespace。 ``prefix`` 及 ``uri`` 会被自动调用 `register_namespace()` 生成namespace。

您可以通过在 `itertag` 属性中制定节点的namespace。

例如:

```python

	class YourSpider(XMLFeedSpider):
	
	    namespaces = [('n', 'http://www.sitemaps.org/schemas/sitemap/0.9')]
	    itertag = 'n:url'
	    # ...
```

除了这些新的属性之外，该spider也有以下可以覆盖(overrideable)的方法:

#### adapt_response(response)

该方法在spider分析response前被调用。您可以在response被分析之前使用该函数来修改内容(body)。 该方法接受一个response并返回一个response(可以相同也可以不同)。

#### parse_node(response, selector)

当节点符合提供的标签名时(``itertag``)该方法被调用。 接收到的response以及相应的 `Selector` 作为参数传递给该方法。 该方法返回一个 `Item` 对象或者 `Request` 对象 或者一个包含二者的可迭代对象(iterable)。

#### process_results(response, results)

当spider返回结果(item或request)时该方法被调用。 设定该方法的目的是在结果返回给框架核心(framework core)之前做最后的处理， 例如设定item的ID。其接受一个结果的列表(list of results)及对应的response。 其结果必须返回一个结果的列表(list of results)(包含Item或者Request对象)。


### XMLFeedSpider example

该spider十分易用。下边是其中一个例子:

```python

    from scrapy.spiders import XMLFeedSpider
    from myproject.items import TestItem

    class MySpider(XMLFeedSpider):
        name = 'example.com'
        allowed_domains = ['example.com']
        start_urls = ['http://www.example.com/feed.xml']
        iterator = 'iternodes'  # This is actually unnecessary, since it's the default value
        itertag = 'item'

        def parse_node(self, response, node):
            self.logger.info('Hi, this is a <%s> node!: %s', self.itertag, ''.join(node.extract()))

            item = TestItem()
            item['id'] = node.xpath('@id').extract()
            item['name'] = node.xpath('name').extract()
            item['description'] = node.xpath('description').extract()
            return item
```

简单来说，我们在这里创建了一个spider，从给定的 ``start_urls`` 中下载feed， 并迭代feed中每个 ``item`` 标签，输出，并在 `Item` 中存储有些随机数据。

## CSVFeedSpider

### CSVFeedSpider

该spider除了其按行遍历而不是节点之外其他和XMLFeedSpider十分类似。 而其在每次迭代时调用的是 `parse_row`。

#### delimiter

       在CSV文件中用于区分字段的分隔符。类型为string。 默认为 `,` (逗号)。

#### quotechar

       A string with the enclosure character for each field in the CSV file
       Defaults to ``'"'`` (quotation mark).

#### headers

在CSV文件中包含的用来提取字段的行的列表。参考下边的例子。

#### parse_row(response, row)

该方法接收一个response对象及一个以提供或检测出来的header为键的字典(代表每行)。 该spider中，您也可以覆盖 `adapt_response` 及 `process_results` 方法来进行预处理(pre-processing)及后(post-processing)处理。

### CSVFeedSpider example

下面的例子和之前的例子很像，但使用了 `CSVFeedSpider`:

```python

    from scrapy.spiders import CSVFeedSpider
    from myproject.items import TestItem

    class MySpider(CSVFeedSpider):
        name = 'example.com'
        allowed_domains = ['example.com']
        start_urls = ['http://www.example.com/feed.csv']
        delimiter = ';'
        quotechar = "'"
        headers = ['id', 'name', 'description']

        def parse_row(self, response, row):
            self.logger.info('Hi, this is a row!: %r', row)

            item = TestItem()
            item['id'] = row['id']
            item['name'] = row['name']
            item['description'] = row['description']
            return item
```

## SitemapSpider

### SitemapSpider

SitemapSpider使您爬取网站时可以通过 Sitemaps 来发现爬取的URL。
其支持嵌套的sitemap，并能从 robots.txt 中获取sitemap的url。

#### sitemap_urls

包含您要爬取的url的sitemap的url列表(list)。 您也可以指定为一个 robots.txt ，spider会从中分析并提取url。

#### sitemap_rules

一个包含 ``(regex, callback)`` 元组的列表(list):

* ``regex`` 是一个用于匹配从sitemap提供的url的正则表达式。 ``regex`` 可以是一个字符串或者编译的正则对象(compiled regex object)。

* callback指定了匹配正则表达式的url的处理函数。 ``callback`` 可以是一个字符串(spider中方法的名字)或者是callable。

例如:

    sitemap_rules = [('/product/', 'parse_product')]

规则按顺序进行匹配，之后第一个匹配才会被应用。

如果您忽略该属性，sitemap中发现的所有url将会被 ``parse`` 函数处理。

#### sitemap_follow

一个用于匹配要跟进的sitemap的正则表达式的列表(list)。其仅仅被应用在 使用 `Sitemap index files` 来指向其他sitemap文件的站点。

默认情况下所有的sitemap都会被跟进。

#### sitemap_alternate_links

指定当一个 `url` 有可选的链接时，是否跟进。 有些非英文网站会在一个 `url` 块内提供其他语言的网站链接。

例如:

```xml
    <url>
        <loc>http://example.com/</loc>
        <xhtml:link rel="alternate" hreflang="de" href="http://example.com/de"/>
    </url>
```

当 ``sitemap_alternate_links`` 设置时，两个URL都会被获取。 当 ``sitemap_alternate_links`` 关闭时，只有 ``http://example.com/`` 会被获取。

默认 ``sitemap_alternate_links`` 关闭。


### SitemapSpider 例子

简单的例子: 使用 ``parse`` 处理通过sitemap发现的所有url:

```python

    from scrapy.spiders import SitemapSpider

    class MySpider(SitemapSpider):
        sitemap_urls = ['http://www.example.com/sitemap.xml']

        def parse(self, response):
            pass # ... scrape item here ...
```

用特定的函数处理某些url，其他的使用另外的callback:

```python

    from scrapy.spiders import SitemapSpider

    class MySpider(SitemapSpider):
        sitemap_urls = ['http://www.example.com/sitemap.xml']
        sitemap_rules = [
            ('/product/', 'parse_product'),
            ('/category/', 'parse_category'),
        ]

        def parse_product(self, response):
            pass # ... scrape product ...

        def parse_category(self, response):
            pass # ... scrape category ...
```
跟进 robots.txt 文件定义的sitemap并只跟进包含有 ``/sitemap_shop`` 的url:

```python

    from scrapy.spiders import SitemapSpider

    class MySpider(SitemapSpider):
        sitemap_urls = ['http://www.example.com/robots.txt']
        sitemap_rules = [
            ('/shop/', 'parse_shop'),
        ]
        sitemap_follow = ['/sitemap_shops']

        def parse_shop(self, response):
            pass # ... scrape shop here ...
```

使用其他url组合SitemapSpider:

```python
    from scrapy.spiders import SitemapSpider

    class MySpider(SitemapSpider):
        sitemap_urls = ['http://www.example.com/robots.txt']
        sitemap_rules = [
            ('/shop/', 'parse_shop'),
        ]

        other_urls = ['http://www.example.com/about']

        def start_requests(self):
            requests = list(super(MySpider, self).start_requests())
            requests += [scrapy.Request(x, self.parse_other) for x in self.other_urls]
            return requests

        def parse_shop(self, response):
            pass # ... scrape shop here ...

        def parse_other(self, response):
            pass # ... scrape other here ...

```

* Sitemaps: http://www.sitemaps.org
* Sitemap index files: http://www.sitemaps.org/protocol.html#index
* robots.txt: http://www.robotstxt.org/
* TLD: https://en.wikipedia.org/wiki/Top-level_domain
* Scrapyd documentation: https://scrapyd.readthedocs.io/en/latest/
