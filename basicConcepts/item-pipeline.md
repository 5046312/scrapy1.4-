Item Pipeline
=============

当Item在Spider中被收集之后，会被传递到Item Pipeline，它使用一些组件按照一定的顺序对Item进行处理。

每个item pipeline组件(有时称之为“Item Pipeline”)是实现了简单方法的Python类。他们接收到Item并通过它执行一些行为，同时也决定此Item是否继续通过pipeline，或是被丢弃而不再进行处理。
Each item pipeline component (sometimes referred as just "Item Pipeline") is a
Python class that implements a simple method. They receive an item and perform
an action over it, also deciding if the item should continue through the
pipeline or be dropped and no longer processed.

以下是item pipeline的一些典型应用：

* 清理HTML数据
* 验证爬取的数据(检查item包含某些字段)
* 查重(并丢弃)
* 将爬取结果保存到数据库


编写你自己的item pipeline
==============================

每个item pipeline组件是Python类，同时必须实现以下方法:

## process_item(self, item, spider)

每个item pipeline组件都需要调用该方法， `process_item()`有多种返回结果: 返回数据的字典，返回一个 `Item`(或任何继承类) 对象, 返回 `Twisted Deferred` 或 抛出 `DropItem` 异常。被丢弃的item将不会被之后的pipeline组件所处理。

参数：
* item: `Item` 对象 或 字典。
* spider: `Spider` 对象 - 爬取 item 的爬虫

此外,他们也可以用以下方法实现:

## open_spider(self, spider)

当spider被开启时，这个方法被调用。

参数：

* spider: `Spider` 对象 - 爬取 item 的爬虫

## close_spider(self, spider)

当spider被关闭时，这个方法被调用

参数：

* spider: `Spider` 对象 - 爬取 item 的爬虫

## from_crawler(cls, crawler)

   If present, this classmethod is called to create a pipeline instance
   from a :class:`~scrapy.crawler.Crawler`. It must return a new instance
   of the pipeline. Crawler object provides access to all Scrapy core
   components like settings and signals; it is a way for pipeline to
   access them and hook its functionality into Scrapy.

   :param crawler: crawler that uses this pipeline
   :type crawler: :class:`~scrapy.crawler.Crawler` object


* Twisted Deferred: https://twistedmatrix.com/documents/current/core/howto/defer.html

Item pipeline 例子
=====================

验证价格，丢弃没有价格的item
--------------------------------------------------

让我们来看一下以下这个假设的pipeline，它为那些不含税(``price_excludes_vat`` 属性)的item调整了 price 属性，同时丢弃了那些没有价格的item:

```python

from scrapy.exceptions import DropItem

class PricePipeline(object):

    vat_factor = 1.15

    def process_item(self, item, spider):
        if item['price']:
            if item['price_excludes_vat']:
                item['price'] = item['price'] * self.vat_factor
            return item
        else:
            raise DropItem("Missing price in %s" % item)

```

将item写入JSON文件
--------------------------

以下pipeline将所有(从所有spider中)爬取到的item，存储到一个独立地 ``items.jl`` 文件，每行包含一个序列化为JSON格式的item:

```python

   import json

   class JsonWriterPipeline(object):

       def open_spider(self, spider):
           self.file = open('items.jl', 'w')

       def close_spider(self, spider):
           self.file.close()

       def process_item(self, item, spider):
           line = json.dumps(dict(item)) + "\n"
           self.file.write(line)
           return item
```

> 注意： JsonWriterPipeline的目的只是为了介绍怎样编写item pipeline，如果你想要将所有爬取的item都保存到同一个JSON文件， 你需要查看 `Feed exports` 。

Write items to MongoDB
----------------------

在本例中，我们将使用 pymongo 将item写入MongoDB中。MongoDB地址和数据库名都在Scrapy设置中指定，
MongoDB collection is named after item class.

The main point of this example is to show how to use `from_crawler()`
method and how to clean up the resources properly.::

```
    import pymongo

    class MongoPipeline(object):

        collection_name = 'scrapy_items'

        def __init__(self, mongo_uri, mongo_db):
            self.mongo_uri = mongo_uri
            self.mongo_db = mongo_db

        @classmethod
        def from_crawler(cls, crawler):
            return cls(
                mongo_uri=crawler.settings.get('MONGO_URI'),
                mongo_db=crawler.settings.get('MONGO_DATABASE', 'items')
            )

        def open_spider(self, spider):
            self.client = pymongo.MongoClient(self.mongo_uri)
            self.db = self.client[self.mongo_db]

        def close_spider(self, spider):
            self.client.close()

        def process_item(self, item, spider):
            self.db[self.collection_name].insert_one(dict(item))
            return item

```

* MongoDB: https://www.mongodb.org/
* pymongo: https://api.mongodb.org/python/current/


Take screenshot of item
-----------------------

This example demonstrates how to return Deferred_ from `process_item()` method.
It uses Splash to render screenshot of item url. Pipeline
makes request to locally running instance of Splash. After request is downloaded and Deferred callback fires, it saves item to a file and adds filename to an item.

```

    import scrapy
    import hashlib
    from urllib.parse import quote


    class ScreenshotPipeline(object):
        """Pipeline that uses Splash to render screenshot of
        every Scrapy item."""

        SPLASH_URL = "http://localhost:8050/render.png?url={}"

        def process_item(self, item, spider):
            encoded_item_url = quote(item["url"])
            screenshot_url = self.SPLASH_URL.format(encoded_item_url)
            request = scrapy.Request(screenshot_url)
            dfd = spider.crawler.engine.download(request, spider)
            dfd.addBoth(self.return_item, item)
            return dfd

        def return_item(self, response, item):
            if response.status != 200:
                # Error happened, return item.
                return item

            # Save screenshot to file, filename will be hash of url.
            url = item["url"]
            url_hash = hashlib.md5(url.encode("utf8")).hexdigest()
            filename = "{}.png".format(url_hash)
            with open(filename, "wb") as f:
                f.write(response.body)

            # Store filename in item.
            item["screenshot_filename"] = filename
            return item
```

* Splash: https://splash.readthedocs.io/en/stable/
* Deferred: https://twistedmatrix.com/documents/current/core/howto/defer.html

去重
-----------------

一个用于查找重复的过滤器，丢弃那些已经被处理过的item。假设我们的item有一个唯一的id，但是我们spider返回的多个item中包含有相同的id:

```

    from scrapy.exceptions import DropItem

    class DuplicatesPipeline(object):

        def __init__(self):
            self.ids_seen = set()

        def process_item(self, item, spider):
            if item['id'] in self.ids_seen:
                raise DropItem("Duplicate item found: %s" % item)
            else:
                self.ids_seen.add(item['id'])
                return item
```

启用 Item Pipeline 组件
=====================================

为了启用一个Item Pipeline组件，你必须将它的类添加到 `ITEM_PIPELINES` 配置，就像下面这个例子:

```
   ITEM_PIPELINES = {
       'myproject.pipelines.PricePipeline': 300,
       'myproject.pipelines.JsonWriterPipeline': 800,
   }
```

分配给每个类的整型值，确定了他们运行的顺序: item 则按数字从低到高的顺序通过pipeline,通常将这些数字定义在0-1000范围内。
