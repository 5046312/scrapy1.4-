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

Spiders can receive arguments that modify their behaviour. Some common uses for
spider arguments are to define the start URLs or to restrict the crawl to
certain sections of the site, but they can be used to configure any
functionality of the spider.

Spider arguments are passed through the :command:`crawl` command using the
``-a`` option. For example::

    scrapy crawl myspider -a category=electronics

Spiders can access arguments in their `__init__` methods::

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

Spider arguments can also be passed through the Scrapyd ``schedule.json`` API.
See `Scrapyd documentation`_.

.. _builtin-spiders:

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

.. class:: CrawlSpider

   This is the most commonly used spider for crawling regular websites, as it
   provides a convenient mechanism for following links by defining a set of rules.
   It may not be the best suited for your particular web sites or project, but
   it's generic enough for several cases, so you can start from it and override it
   as needed for more custom functionality, or just implement your own spider.

   Apart from the attributes inherited from Spider (that you must
   specify), this class supports a new attribute:

   .. attribute:: rules

       Which is a list of one (or more) :class:`Rule` objects.  Each :class:`Rule`
       defines a certain behaviour for crawling the site. Rules objects are
       described below. If multiple rules match the same link, the first one
       will be used, according to the order they're defined in this attribute.

   This spider also exposes an overrideable method:

   .. method:: parse_start_url(response)

      This method is called for the start_urls responses. It allows to parse
      the initial responses and must return either an
      :class:`~scrapy.item.Item` object, a :class:`~scrapy.http.Request`
      object, or an iterable containing any of them.

### Crawling rules

.. class:: Rule(link_extractor, callback=None, cb_kwargs=None, follow=None, process_links=None, process_request=None)

   ``link_extractor`` is a :ref:`Link Extractor <topics-link-extractors>` object which
   defines how links will be extracted from each crawled page.

   ``callback`` is a callable or a string (in which case a method from the spider
   object with that name will be used) to be called for each link extracted with
   the specified link_extractor. This callback receives a response as its first
   argument and must return a list containing :class:`~scrapy.item.Item` and/or
   :class:`~scrapy.http.Request` objects (or any subclass of them).

   .. warning:: When writing crawl spider rules, avoid using ``parse`` as
       callback, since the :class:`CrawlSpider` uses the ``parse`` method
       itself to implement its logic. So if you override the ``parse`` method,
       the crawl spider will no longer work.

   ``cb_kwargs`` is a dict containing the keyword arguments to be passed to the
   callback function.

   ``follow`` is a boolean which specifies if links should be followed from each
   response extracted with this rule. If ``callback`` is None ``follow`` defaults
   to ``True``, otherwise it defaults to ``False``.

   ``process_links`` is a callable, or a string (in which case a method from the
   spider object with that name will be used) which will be called for each list
   of links extracted from each response using the specified ``link_extractor``.
   This is mainly used for filtering purposes.

   ``process_request`` is a callable, or a string (in which case a method from
   the spider object with that name will be used) which will be called with
   every request extracted by this rule, and must return a request or None (to
   filter out the request).

### CrawlSpider example

Let's now take a look at an example CrawlSpider with rules::

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


This spider would start crawling example.com's home page, collecting category
links, and item links, parsing the latter with the ``parse_item`` method. For
each item response, some data will be extracted from the HTML using XPath, and
an :class:`~scrapy.item.Item` will be filled with it.

## XMLFeedSpider

.. class:: XMLFeedSpider

    XMLFeedSpider is designed for parsing XML feeds by iterating through them by a
    certain node name.  The iterator can be chosen from: ``iternodes``, ``xml``,
    and ``html``.  It's recommended to use the ``iternodes`` iterator for
    performance reasons, since the ``xml`` and ``html`` iterators generate the
    whole DOM at once in order to parse it.  However, using ``html`` as the
    iterator may be useful when parsing XML with bad markup.

    To set the iterator and the tag name, you must define the following class
    attributes:

    .. attribute:: iterator

        A string which defines the iterator to use. It can be either:

           - ``'iternodes'`` - a fast iterator based on regular expressions

           - ``'html'`` - an iterator which uses :class:`~scrapy.selector.Selector`.
             Keep in mind this uses DOM parsing and must load all DOM in memory
             which could be a problem for big feeds

           - ``'xml'`` - an iterator which uses :class:`~scrapy.selector.Selector`.
             Keep in mind this uses DOM parsing and must load all DOM in memory
             which could be a problem for big feeds

        It defaults to: ``'iternodes'``.

    .. attribute:: itertag

        A string with the name of the node (or element) to iterate in. Example::

            itertag = 'product'

    .. attribute:: namespaces

        A list of ``(prefix, uri)`` tuples which define the namespaces
        available in that document that will be processed with this spider. The
        ``prefix`` and ``uri`` will be used to automatically register
        namespaces using the
        :meth:`~scrapy.selector.Selector.register_namespace` method.

        You can then specify nodes with namespaces in the :attr:`itertag`
        attribute.

        Example::

            class YourSpider(XMLFeedSpider):

                namespaces = [('n', 'http://www.sitemaps.org/schemas/sitemap/0.9')]
                itertag = 'n:url'
                # ...

    Apart from these new attributes, this spider has the following overrideable
    methods too:

    .. method:: adapt_response(response)

        A method that receives the response as soon as it arrives from the spider
        middleware, before the spider starts parsing it. It can be used to modify
        the response body before parsing it. This method receives a response and
        also returns a response (it could be the same or another one).

    .. method:: parse_node(response, selector)

        This method is called for the nodes matching the provided tag name
        (``itertag``).  Receives the response and an
        :class:`~scrapy.selector.Selector` for each node.  Overriding this
        method is mandatory. Otherwise, you spider won't work.  This method
        must return either a :class:`~scrapy.item.Item` object, a
        :class:`~scrapy.http.Request` object, or an iterable containing any of
        them.

    .. method:: process_results(response, results)

        This method is called for each result (item or request) returned by the
        spider, and it's intended to perform any last time processing required
        before returning the results to the framework core, for example setting the
        item IDs. It receives a list of results and the response which originated
        those results. It must return a list of results (Items or Requests).


### XMLFeedSpider example

These spiders are pretty easy to use, let's have a look at one example::

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

Basically what we did up there was to create a spider that downloads a feed from
the given ``start_urls``, and then iterates through each of its ``item`` tags,
prints them out, and stores some random data in an :class:`~scrapy.item.Item`.

## CSVFeedSpider

.. class:: CSVFeedSpider

   This spider is very similar to the XMLFeedSpider, except that it iterates
   over rows, instead of nodes. The method that gets called in each iteration
   is :meth:`parse_row`.

   .. attribute:: delimiter

       A string with the separator character for each field in the CSV file
       Defaults to ``','`` (comma).

   .. attribute:: quotechar

       A string with the enclosure character for each field in the CSV file
       Defaults to ``'"'`` (quotation mark).

   .. attribute:: headers

       A list of the column names in the CSV file.

   .. method:: parse_row(response, row)

       Receives a response and a dict (representing each row) with a key for each
       provided (or detected) header of the CSV file.  This spider also gives the
       opportunity to override ``adapt_response`` and ``process_results`` methods
       for pre- and post-processing purposes.

### CSVFeedSpider example

Let's see an example similar to the previous one, but using a
:class:`CSVFeedSpider`::

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


## SitemapSpider

.. class:: SitemapSpider

    SitemapSpider allows you to crawl a site by discovering the URLs using
    `Sitemaps`_.

    It supports nested sitemaps and discovering sitemap urls from
    `robots.txt`_.

    .. attribute:: sitemap_urls

        A list of urls pointing to the sitemaps whose urls you want to crawl.

        You can also point to a `robots.txt`_ and it will be parsed to extract
        sitemap urls from it.

    .. attribute:: sitemap_rules

        A list of tuples ``(regex, callback)`` where:

        * ``regex`` is a regular expression to match urls extracted from sitemaps.
          ``regex`` can be either a str or a compiled regex object.

        * callback is the callback to use for processing the urls that match
          the regular expression. ``callback`` can be a string (indicating the
          name of a spider method) or a callable.

        For example::

            sitemap_rules = [('/product/', 'parse_product')]

        Rules are applied in order, and only the first one that matches will be
        used.

        If you omit this attribute, all urls found in sitemaps will be
        processed with the ``parse`` callback.

    .. attribute:: sitemap_follow

        A list of regexes of sitemap that should be followed. This is is only
        for sites that use `Sitemap index files`_ that point to other sitemap
        files.

        By default, all sitemaps are followed.

    .. attribute:: sitemap_alternate_links

        Specifies if alternate links for one ``url`` should be followed. These
        are links for the same website in another language passed within
        the same ``url`` block.

        For example::

            <url>
                <loc>http://example.com/</loc>
                <xhtml:link rel="alternate" hreflang="de" href="http://example.com/de"/>
            </url>

        With ``sitemap_alternate_links`` set, this would retrieve both URLs. With
        ``sitemap_alternate_links`` disabled, only ``http://example.com/`` would be
        retrieved.

        Default is ``sitemap_alternate_links`` disabled.


### SitemapSpider examples

Simplest example: process all urls discovered through sitemaps using the
``parse`` callback::

    from scrapy.spiders import SitemapSpider

    class MySpider(SitemapSpider):
        sitemap_urls = ['http://www.example.com/sitemap.xml']

        def parse(self, response):
            pass # ... scrape item here ...

Process some urls with certain callback and other urls with a different
callback::

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

Follow sitemaps defined in the `robots.txt`_ file and only follow sitemaps
whose url contains ``/sitemap_shop``::

    from scrapy.spiders import SitemapSpider

    class MySpider(SitemapSpider):
        sitemap_urls = ['http://www.example.com/robots.txt']
        sitemap_rules = [
            ('/shop/', 'parse_shop'),
        ]
        sitemap_follow = ['/sitemap_shops']

        def parse_shop(self, response):
            pass # ... scrape shop here ...

Combine SitemapSpider with other sources of urls::

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

* Sitemaps: http://www.sitemaps.org
* Sitemap index files: http://www.sitemaps.org/protocol.html#index
* robots.txt: http://www.robotstxt.org/
* TLD: https://en.wikipedia.org/wiki/Top-level_domain
* Scrapyd documentation: https://scrapyd.readthedocs.io/en/latest/
