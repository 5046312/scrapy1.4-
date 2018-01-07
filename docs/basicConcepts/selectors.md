选择器(Selectors)
=========

当抓取网页时，你做的最常见的任务是从HTML源码中提取数据。现有的一些库可以达到这个目的：

 * `BeautifulSoup`是在程序员间非常流行的网页分析库，它基于HTML代码的结构来构造一个Python对象， 对不良标记的处理也非常合理，但它有一个缺点：慢。

 * `lxml` 是一个基于 ElementTree (不是Python标准库的一部分)的python化的XML解析库(也可以解析HTML)。

Scrapy提取数据有自己的一套机制。它们被称作选择器(seletors)，因为他们通过特定的 XPath 或者 CSS 表达式来“选择” HTML文件中的某个部分。

XPath 是一门用来在XML文件中选择节点的语言，也可以用在HTML上。 CSS 是一门将HTML文档样式化的语言。选择器由它定义，并与特定的HTML元素的样式相关连。

Scrapy选择器构建于 lxml 库之上，这意味着它们在速度和解析准确性上非常相似。

本页面解释了选择器如何工作，并描述了相应的API。不同于 lxml API的臃肿，该API短小而简洁。这是因为 lxml 库除了用来选择标记化文档外，还可以用到许多任务上。

选择器API的完全参考详见 Selector reference

* BeautifulSoup: http://www.crummy.com/software/BeautifulSoup/
* lxml: http://lxml.de/
* ElementTree: https://docs.python.org/2/library/xml.etree.elementtree.html
* cssselect: https://pypi.python.org/pypi/cssselect/
* XPath: https://www.w3.org/TR/xpath
* CSS: https://www.w3.org/TR/selectors


# 使用选择器

## 构造选择器


Scrapy `selector`是以 **文字(text)** 或 `TextResponse 类`构造的 `Selector` 实例。 其根据输入的类型自动选择最优的分析规则(XML vs HTML):

    >>> from scrapy.selector import Selector
    >>> from scrapy.http import HtmlResponse

以文字构造:

    >>> body = '<html><body><span>good</span></body></html>'
    >>> Selector(text=body).xpath('//span/text()').extract()
    [u'good']

以response构造:

    >>> response = HtmlResponse(url='http://example.com', body=body)
    >>> Selector(response=response).xpath('//span/text()').extract()
    [u'good']

为了方便起见，response对象以 .selector 属性提供了一个selector， 您可以随时使用该快捷方法:

    >>> response.selector.xpath('//span/text()').extract()
    [u'good']


## 使用选择器

我们将使用 `Scrapy shell` (提供交互测试)和位于Scrapy文档服务器的一个样例页面，来解释如何使用选择器：

    http://doc.scrapy.org/en/latest/_static/selectors-sample1.html

首先, 我们打开shell:

    scrapy shell http://doc.scrapy.org/en/latest/_static/selectors-sample1.html

接着，当shell载入后，您将获得名为 ``response`` 的shell变量，其为响应的response， 并且在其 ``response.selector`` 属性上绑定了一个selector。

因为我们处理的是HTML，选择器将自动使用HTML语法分析。

那么，通过查看 `HTML code` 该页面的源码，我们构建一个XPath来选择title标签内的文字:

    >>> response.selector.xpath('//title/text()')
    [<Selector (text) xpath=//title/text()>]

由于在response中使用XPath、CSS查询十分普遍，因此，Scrapy提供了两个实用的快捷方式: ``response.xpath()`` 及 ``response.css()``:

    >>> response.xpath('//title/text()')
    [<Selector (text) xpath=//title/text()>]
    >>> response.css('title::text')
    [<Selector (text) xpath=//title/text()>]

如你所见， ``.xpath()`` 及 ``.css()`` 方法返回一个类 SelectorList 的实例, 它是一个新选择器的列表。这个API可以用来快速的提取嵌套数据:

    >>> response.css('img').xpath('@src').extract()
    [u'image1_thumb.jpg',
     u'image2_thumb.jpg',
     u'image3_thumb.jpg',
     u'image4_thumb.jpg',
     u'image5_thumb.jpg']

为了提取真实的原文数据，你需要调用 ``.extract()`` 方法如下::

    >>> response.xpath('//title/text()').extract()
    [u'Example website']

如果你只想提取第一个匹配到的元素，可以使用``.extract_first()``方法：

    >>> response.xpath('//div[@id="images"]/a/text()').extract_first()
    u'Name: My image 1 '

如果没有找到任何元素则返回 ``None``:

    >>> response.xpath('//div[@id="not-exists"]/text()').extract_first() is None
    True

也可以传递一个参数，来代替默认``None``的返回值：

    >>> response.xpath('//div[@id="not-exists"]/text()').extract_first(default='not-found')
    'not-found'

注意CSS选择器可以使用CSS3伪元素来选择text文本或属性节点：

    >>> response.css('title::text').extract()
    [u'Example website']

现在我们将得到根URL(base URL)和一些图片链接:

    >>> response.xpath('//base/@href').extract()
    [u'http://example.com/']

    >>> response.css('base::attr(href)').extract()
    [u'http://example.com/']

    >>> response.xpath('//a[contains(@href, "image")]/@href').extract()
    [u'image1.html',
     u'image2.html',
     u'image3.html',
     u'image4.html',
     u'image5.html']

    >>> response.css('a[href*=image]::attr(href)').extract()
    [u'image1.html',
     u'image2.html',
     u'image3.html',
     u'image4.html',
     u'image5.html']

    >>> response.xpath('//a[contains(@href, "image")]/img/@src').extract()
    [u'image1_thumb.jpg',
     u'image2_thumb.jpg',
     u'image3_thumb.jpg',
     u'image4_thumb.jpg',
     u'image5_thumb.jpg']

    >>> response.css('a[href*=image] img::attr(src)').extract()
    [u'image1_thumb.jpg',
     u'image2_thumb.jpg',
     u'image3_thumb.jpg',
     u'image4_thumb.jpg',
     u'image5_thumb.jpg']

嵌套选择器
-----------------

选择器方法( ``.xpath()`` or ``.css()`` )返回相同类型的选择器列表，因此你也可以对这些选择器调用选择器方法。下面是一个例子:

    >>> links = response.xpath('//a[contains(@href, "image")]')
    >>> links.extract()
    [u'<a href="image1.html">Name: My image 1 <br><img src="image1_thumb.jpg"></a>',
     u'<a href="image2.html">Name: My image 2 <br><img src="image2_thumb.jpg"></a>',
     u'<a href="image3.html">Name: My image 3 <br><img src="image3_thumb.jpg"></a>',
     u'<a href="image4.html">Name: My image 4 <br><img src="image4_thumb.jpg"></a>',
     u'<a href="image5.html">Name: My image 5 <br><img src="image5_thumb.jpg"></a>']

    >>> for index, link in enumerate(links):
    ...     args = (index, link.xpath('@href').extract(), link.xpath('img/@src').extract())
    ...     print 'Link number %d points to url %s and image %s' % args

    Link number 0 points to url [u'image1.html'] and image [u'image1_thumb.jpg']
    Link number 1 points to url [u'image2.html'] and image [u'image2_thumb.jpg']
    Link number 2 points to url [u'image3.html'] and image [u'image3_thumb.jpg']
    Link number 3 points to url [u'image4.html'] and image [u'image4_thumb.jpg']
    Link number 4 points to url [u'image5.html'] and image [u'image5_thumb.jpg']

在选择器中使用正则匹配
----------------------------------------

``Selector`` 也有一个 ``.re()`` 方法，用来通过正则表达式来提取数据。然而，不同于使用 ``.xpath()`` 或者 ``.css()`` 方法, ``.re()`` 方法会返回unicode字符串的列表。所以你无法构造嵌套式的 ``.re()`` 调用。

下面是一个例子，从上面的 HTML code 中提取图像名字:

    >>> response.xpath('//a[contains(@href, "image")]/text()').re(r'Name:\s*(.*)')
    [u'My image 1',
     u'My image 2',
     u'My image 3',
     u'My image 4',
     u'My image 5']

There's an additional helper reciprocating ``.extract_first()`` for ``.re()``,
named ``.re_first()``. Use it to extract just the first matching string::

    >>> response.xpath('//a[contains(@href, "image")]/text()').re_first(r'Name:\s*(.*)')
    u'My image 1'


使用相对XPath
----------------------------

Keep in mind that if you are nesting selectors and use an XPath that starts
with ``/``, that XPath will be absolute to the document and not relative to the ``Selector`` you're calling it from.

For example, suppose you want to extract all ``<p>`` elements inside ``<div>``
elements. First, you would get all ``<div>`` elements::

    >>> divs = response.xpath('//div')

At first, you may be tempted to use the following approach, which is wrong, as
it actually extracts all ``<p>`` elements from the document, not only those
inside ``<div>`` elements::

    >>> for p in divs.xpath('//p'):  # this is wrong - gets all <p> from the whole document
    ...     print p.extract()

This is the proper way to do it (note the dot prefixing the ``.//p`` XPath)::

    >>> for p in divs.xpath('.//p'):  # extracts all <p> inside
    ...     print p.extract()

Another common case would be to extract all direct ``<p>`` children::

    >>> for p in divs.xpath('p'):
    ...     print p.extract()

For more details about relative XPaths see the `Location Paths`_ section in the
XPath specification.

* Location Paths: https://www.w3.org/TR/xpath#location-paths



XPath表达式中的变量
------------------------------

XPath allows you to reference variables in your XPath expressions, using
the ``$somevariable`` syntax. This is somewhat similar to parameterized
queries or prepared statements in the SQL world where you replace
some arguments in your queries with placeholders like ``?``,
which are then substituted with values passed with the query.

Here's an example to match an element based on its "id" attribute value,
without hard-coding it (that was shown previously)::

    >>> # `$val` used in the expression, a `val` argument needs to be passed
    >>> response.xpath('//div[@id=$val]/a/text()', val='images').extract_first()
    u'Name: My image 1 '

Here's another example, to find the "id" attribute of a ``<div>`` tag containing
five ``<a>`` children (here we pass the value ``5`` as an integer)::

    >>> response.xpath('//div[count(a)=$cnt]/@id', cnt=5).extract_first()
    u'images'

All variable references must have a binding value when calling ``.xpath()``
(otherwise you'll get a ``ValueError: XPath error:`` exception).
This is done by passing as many named arguments as necessary.

`parsel`_, the library powering Scrapy selectors, has more details and examples
on `XPath variables`_.

.. _parsel: https://parsel.readthedocs.io/
.. _XPath variables: https://parsel.readthedocs.io/en/latest/usage.html#variables-in-xpath-expressions

使用EXSLT扩展
----------------------

因建于 lxml 之上, Scrapy选择器也支持一些 EXSLT 扩展，可以在XPath表达式中使用这些预先制定的命名空间：



| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |


| prefix | namespace | usage |

======  =====================================    =======================
re      http://exslt.org/regular-expressions    `regular expressions`
set     http://exslt.org/sets                   `set manipulation`
======  =====================================    =======================

##### 正则表达式

The ``test()`` function, for example, can prove quite useful when XPath's
``starts-with()`` or ``contains()`` are not sufficient.

Example selecting links in list item with a "class" attribute ending with a digit::

    >>> from scrapy import Selector
    >>> doc = """
    ... <div>
    ...     <ul>
    ...         <li class="item-0"><a href="link1.html">first item</a></li>
    ...         <li class="item-1"><a href="link2.html">second item</a></li>
    ...         <li class="item-inactive"><a href="link3.html">third item</a></li>
    ...         <li class="item-1"><a href="link4.html">fourth item</a></li>
    ...         <li class="item-0"><a href="link5.html">fifth item</a></li>
    ...     </ul>
    ... </div>
    ... """
    >>> sel = Selector(text=doc, type="html")
    >>> sel.xpath('//li//@href').extract()
    [u'link1.html', u'link2.html', u'link3.html', u'link4.html', u'link5.html']
    >>> sel.xpath('//li[re:test(@class, "item-\d$")]//@href').extract()
    [u'link1.html', u'link2.html', u'link4.html', u'link5.html']
    >>>

.. warning:: C library ``libxslt`` doesn't natively support EXSLT regular
    expressions so `lxml`_'s implementation uses hooks to Python's ``re`` module.
    Thus, using regexp functions in your XPath expressions may add a small
    performance penalty.

##### Set operations

These can be handy for excluding parts of a document tree before
extracting text elements for example.

Example extracting microdata (sample content taken from http://schema.org/Product)
with groups of itemscopes and corresponding itemprops::

    >>> doc = """
    ... <div itemscope itemtype="http://schema.org/Product">
    ...   <span itemprop="name">Kenmore White 17" Microwave</span>
    ...   <img src="kenmore-microwave-17in.jpg" alt='Kenmore 17" Microwave' />
    ...   <div itemprop="aggregateRating"
    ...     itemscope itemtype="http://schema.org/AggregateRating">
    ...    Rated <span itemprop="ratingValue">3.5</span>/5
    ...    based on <span itemprop="reviewCount">11</span> customer reviews
    ...   </div>
    ...
    ...   <div itemprop="offers" itemscope itemtype="http://schema.org/Offer">
    ...     <span itemprop="price">$55.00</span>
    ...     <link itemprop="availability" href="http://schema.org/InStock" />In stock
    ...   </div>
    ...
    ...   Product description:
    ...   <span itemprop="description">0.7 cubic feet countertop microwave.
    ...   Has six preset cooking categories and convenience features like
    ...   Add-A-Minute and Child Lock.</span>
    ...
    ...   Customer reviews:
    ...
    ...   <div itemprop="review" itemscope itemtype="http://schema.org/Review">
    ...     <span itemprop="name">Not a happy camper</span> -
    ...     by <span itemprop="author">Ellie</span>,
    ...     <meta itemprop="datePublished" content="2011-04-01">April 1, 2011
    ...     <div itemprop="reviewRating" itemscope itemtype="http://schema.org/Rating">
    ...       <meta itemprop="worstRating" content = "1">
    ...       <span itemprop="ratingValue">1</span>/
    ...       <span itemprop="bestRating">5</span>stars
    ...     </div>
    ...     <span itemprop="description">The lamp burned out and now I have to replace
    ...     it. </span>
    ...   </div>
    ...
    ...   <div itemprop="review" itemscope itemtype="http://schema.org/Review">
    ...     <span itemprop="name">Value purchase</span> -
    ...     by <span itemprop="author">Lucas</span>,
    ...     <meta itemprop="datePublished" content="2011-03-25">March 25, 2011
    ...     <div itemprop="reviewRating" itemscope itemtype="http://schema.org/Rating">
    ...       <meta itemprop="worstRating" content = "1"/>
    ...       <span itemprop="ratingValue">4</span>/
    ...       <span itemprop="bestRating">5</span>stars
    ...     </div>
    ...     <span itemprop="description">Great microwave for the price. It is small and
    ...     fits in my apartment.</span>
    ...   </div>
    ...   ...
    ... </div>
    ... """
    >>> sel = Selector(text=doc, type="html")
    >>> for scope in sel.xpath('//div[@itemscope]'):
    ...     print "current scope:", scope.xpath('@itemtype').extract()
    ...     props = scope.xpath('''
    ...                 set:difference(./descendant::*/@itemprop,
    ...                                .//*[@itemscope]/*/@itemprop)''')
    ...     print "    properties:", props.extract()
    ...     print

    current scope: [u'http://schema.org/Product']
        properties: [u'name', u'aggregateRating', u'offers', u'description', u'review', u'review']

    current scope: [u'http://schema.org/AggregateRating']
        properties: [u'ratingValue', u'reviewCount']

    current scope: [u'http://schema.org/Offer']
        properties: [u'price', u'availability']

    current scope: [u'http://schema.org/Review']
        properties: [u'name', u'author', u'datePublished', u'reviewRating', u'description']

    current scope: [u'http://schema.org/Rating']
        properties: [u'worstRating', u'ratingValue', u'bestRating']

    current scope: [u'http://schema.org/Review']
        properties: [u'name', u'author', u'datePublished', u'reviewRating', u'description']

    current scope: [u'http://schema.org/Rating']
        properties: [u'worstRating', u'ratingValue', u'bestRating']

    >>>

Here we first iterate over ``itemscope`` elements, and for each one,
we look for all ``itemprops`` elements and exclude those that are themselves
inside another ``itemscope``.

.. _EXSLT: http://exslt.org/
.. _regular expressions: http://exslt.org/regexp/index.html
.. _set manipulation: http://exslt.org/set/index.html


Some XPath tips
---------------

Here are some tips that you may find useful when using XPath
with Scrapy selectors, based on `this post from ScrapingHub's blog`_.
If you are not much familiar with XPath yet,
you may want to take a look first at this `XPath tutorial`_.


.. _`XPath tutorial`: http://www.zvon.org/comp/r/tut-XPath_1.html
.. _`this post from ScrapingHub's blog`: https://blog.scrapinghub.com/2014/07/17/xpath-tips-from-the-web-scraping-trenches/


Using text nodes in a condition
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When you need to use the text content as argument to an `XPath string function`_,
avoid using ``.//text()`` and use just ``.`` instead.

This is because the expression ``.//text()`` yields a collection of text elements -- a *node-set*.
And when a node-set is converted to a string, which happens when it is passed as argument to
a string function like ``contains()`` or ``starts-with()``, it results in the text for the first element only.

Example::

    >>> from scrapy import Selector
    >>> sel = Selector(text='<a href="#">Click here to go to the <strong>Next Page</strong></a>')

Converting a *node-set* to string::

    >>> sel.xpath('//a//text()').extract() # take a peek at the node-set
    [u'Click here to go to the ', u'Next Page']
    >>> sel.xpath("string(//a[1]//text())").extract() # convert it to string
    [u'Click here to go to the ']

A *node* converted to a string, however, puts together the text of itself plus of all its descendants::

    >>> sel.xpath("//a[1]").extract() # select the first node
    [u'<a href="#">Click here to go to the <strong>Next Page</strong></a>']
    >>> sel.xpath("string(//a[1])").extract() # convert it to string
    [u'Click here to go to the Next Page']

So, using the ``.//text()`` node-set won't select anything in this case::

    >>> sel.xpath("//a[contains(.//text(), 'Next Page')]").extract()
    []

But using the ``.`` to mean the node, works::

    >>> sel.xpath("//a[contains(., 'Next Page')]").extract()
    [u'<a href="#">Click here to go to the <strong>Next Page</strong></a>']

.. _`XPath string function`: https://www.w3.org/TR/xpath/#section-String-Functions

Beware of the difference between //node[1] and (//node)[1]
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``//node[1]`` selects all the nodes occurring first under their respective parents.

``(//node)[1]`` selects all the nodes in the document, and then gets only the first of them.

Example::

    >>> from scrapy import Selector
    >>> sel = Selector(text="""
    ....:     <ul class="list">
    ....:         <li>1</li>
    ....:         <li>2</li>
    ....:         <li>3</li>
    ....:     </ul>
    ....:     <ul class="list">
    ....:         <li>4</li>
    ....:         <li>5</li>
    ....:         <li>6</li>
    ....:     </ul>""")
    >>> xp = lambda x: sel.xpath(x).extract()

This gets all first ``<li>``  elements under whatever it is its parent::

    >>> xp("//li[1]")
    [u'<li>1</li>', u'<li>4</li>']

And this gets the first ``<li>``  element in the whole document::

    >>> xp("(//li)[1]")
    [u'<li>1</li>']

This gets all first ``<li>``  elements under an ``<ul>``  parent::

    >>> xp("//ul/li[1]")
    [u'<li>1</li>', u'<li>4</li>']

And this gets the first ``<li>``  element under an ``<ul>``  parent in the whole document::

    >>> xp("(//ul/li)[1]")
    [u'<li>1</li>']

When querying by class, consider using CSS
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Because an element can contain multiple CSS classes, the XPath way to select elements
by class is the rather verbose::

    *[contains(concat(' ', normalize-space(@class), ' '), ' someclass ')]

If you use ``@class='someclass'`` you may end up missing elements that have
other classes, and if you just use ``contains(@class, 'someclass')`` to make up
for that you may end up with more elements that you want, if they have a different
class name that shares the string ``someclass``.

As it turns out, Scrapy selectors allow you to chain selectors, so most of the time
you can just select by class using CSS and then switch to XPath when needed::

    >>> from scrapy import Selector
    >>> sel = Selector(text='<div class="hero shout"><time datetime="2014-07-23 19:00">Special date</time></div>')
    >>> sel.css('.shout').xpath('./time/@datetime').extract()
    [u'2014-07-23 19:00']

This is cleaner than using the verbose XPath trick shown above. Just remember
to use the ``.`` in the XPath expressions that will follow.


.. _topics-selectors-ref:

Built-in Selectors reference
============================

.. module:: scrapy.selector
   :synopsis: Selector class

Selector objects
----------------

.. class:: Selector(response=None, text=None, type=None)

  An instance of :class:`Selector` is a wrapper over response to select
  certain parts of its content.

  ``response`` is an :class:`~scrapy.http.HtmlResponse` or an
  :class:`~scrapy.http.XmlResponse` object that will be used for selecting and
  extracting data.

  ``text`` is a unicode string or utf-8 encoded text for cases when a
  ``response`` isn't available. Using ``text`` and ``response`` together is
  undefined behavior.

  ``type`` defines the selector type, it can be ``"html"``, ``"xml"`` or ``None`` (default).

    If ``type`` is ``None``, the selector automatically chooses the best type
    based on ``response`` type (see below), or defaults to ``"html"`` in case it
    is used together with ``text``.

    If ``type`` is ``None`` and a ``response`` is passed, the selector type is
    inferred from the response type as follows:

        * ``"html"`` for :class:`~scrapy.http.HtmlResponse` type
        * ``"xml"`` for :class:`~scrapy.http.XmlResponse` type
        * ``"html"`` for anything else

   Otherwise, if ``type`` is set, the selector type will be forced and no
   detection will occur.

  .. method:: xpath(query)

      Find nodes matching the xpath ``query`` and return the result as a
      :class:`SelectorList` instance with all elements flattened. List
      elements implement :class:`Selector` interface too.

      ``query`` is a string containing the XPATH query to apply.

      .. note::

          For convenience, this method can be called as ``response.xpath()``

  .. method:: css(query)

      Apply the given CSS selector and return a :class:`SelectorList` instance.

      ``query`` is a string containing the CSS selector to apply.

      In the background, CSS queries are translated into XPath queries using
      `cssselect`_ library and run ``.xpath()`` method.

      .. note::

          For convenience this method can be called as ``response.css()``

  .. method:: extract()

     Serialize and return the matched nodes as a list of unicode strings.
     Percent encoded content is unquoted.

  .. method:: re(regex)

     Apply the given regex and return a list of unicode strings with the
     matches.

     ``regex`` can be either a compiled regular expression or a string which
     will be compiled to a regular expression using ``re.compile(regex)``

    .. note::

        Note that ``re()`` and ``re_first()`` both decode HTML entities (except ``&lt;`` and ``&amp;``).

  .. method:: register_namespace(prefix, uri)

     Register the given namespace to be used in this :class:`Selector`.
     Without registering namespaces you can't select or extract data from
     non-standard namespaces. See examples below.

  .. method:: remove_namespaces()

     Remove all namespaces, allowing to traverse the document using
     namespace-less xpaths. See example below.

  .. method:: __nonzero__()

     Returns ``True`` if there is any real content selected or ``False``
     otherwise.  In other words, the boolean value of a :class:`Selector` is
     given by the contents it selects.


SelectorList objects
--------------------

.. class:: SelectorList

   The :class:`SelectorList` class is a subclass of the builtin ``list``
   class, which provides a few additional methods.

   .. method:: xpath(query)

       Call the ``.xpath()`` method for each element in this list and return
       their results flattened as another :class:`SelectorList`.

       ``query`` is the same argument as the one in :meth:`Selector.xpath`

   .. method:: css(query)

       Call the ``.css()`` method for each element in this list and return
       their results flattened as another :class:`SelectorList`.

       ``query`` is the same argument as the one in :meth:`Selector.css`

   .. method:: extract()

       Call the ``.extract()`` method for each element in this list and return
       their results flattened, as a list of unicode strings.

   .. method:: re()

       Call the ``.re()`` method for each element in this list and return
       their results flattened, as a list of unicode strings.


Selector examples on HTML response
----------------------------------

Here's a couple of :class:`Selector` examples to illustrate several concepts.
In all cases, we assume there is already a :class:`Selector` instantiated with
a :class:`~scrapy.http.HtmlResponse` object like this::

      sel = Selector(html_response)

1. Select all ``<h1>`` elements from an HTML response body, returning a list of
   :class:`Selector` objects (ie. a :class:`SelectorList` object)::

      sel.xpath("//h1")

2. Extract the text of all ``<h1>`` elements from an HTML response body,
   returning a list of unicode strings::

      sel.xpath("//h1").extract()         # this includes the h1 tag
      sel.xpath("//h1/text()").extract()  # this excludes the h1 tag

3. Iterate over all ``<p>`` tags and print their class attribute::

      for node in sel.xpath("//p"):
          print node.xpath("@class").extract()

Selector examples on XML response
---------------------------------

Here's a couple of examples to illustrate several concepts. In both cases we
assume there is already a :class:`Selector` instantiated with an
:class:`~scrapy.http.XmlResponse` object like this::

      sel = Selector(xml_response)

1. Select all ``<product>`` elements from an XML response body, returning a list
   of :class:`Selector` objects (ie. a :class:`SelectorList` object)::

      sel.xpath("//product")

2. Extract all prices from a `Google Base XML feed`_ which requires registering
   a namespace::

      sel.register_namespace("g", "http://base.google.com/ns/1.0")
      sel.xpath("//g:price").extract()

.. _removing-namespaces:

Removing namespaces
-------------------

When dealing with scraping projects, it is often quite convenient to get rid of
namespaces altogether and just work with element names, to write more
simple/convenient XPaths. You can use the
:meth:`Selector.remove_namespaces` method for that.

Let's show an example that illustrates this with GitHub blog atom feed.

.. highlight:: sh

First, we open the shell with the url we want to scrape::

    $ scrapy shell https://github.com/blog.atom

.. highlight:: python

Once in the shell we can try selecting all ``<link>`` objects and see that it
doesn't work (because the Atom XML namespace is obfuscating those nodes)::

    >>> response.xpath("//link")
    []

But once we call the :meth:`Selector.remove_namespaces` method, all
nodes can be accessed directly by their names::

    >>> response.selector.remove_namespaces()
    >>> response.xpath("//link")
    [<Selector xpath='//link' data=u'<link xmlns="http://www.w3.org/2005/Atom'>,
     <Selector xpath='//link' data=u'<link xmlns="http://www.w3.org/2005/Atom'>,
     ...

If you wonder why the namespace removal procedure isn't always called by default
instead of having to call it manually, this is because of two reasons, which, in order
of relevance, are:

1. Removing namespaces requires to iterate and modify all nodes in the
   document, which is a reasonably expensive operation to perform for all
   documents crawled by Scrapy

2. There could be some cases where using namespaces is actually required, in
   case some element names clash between namespaces. These cases are very rare
   though.

.. _Google Base XML feed: https://support.google.com/merchants/answer/160589?hl=en&ref_topic=2473799
