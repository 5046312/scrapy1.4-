# Release notes

## Scrapy 1.5.0 (2017-12-29)

这个版本在代码库中带来了一些小的新特性和改进。一些亮点:

* 在FilesPipeline 和 ImagesPipeline中已经可以支持Google云存储。
* 使用代理服务器爬行变得更加高效，因为与代理的连接现在可以重用。
* 警告、异常和日志消息都得到了改进，使调试更加容易。
* ``scrapy parse`` 命令现在可以设置自定制的请求目标——``--meta``参数。
* 与Python 3.6、PyPy和PyPy3的兼容性得到了改进。通过CI的测试，PyPy和PyPy3现在得到了正式的支持。
* 对HTTP 308、522和524状态码有更好的默认处理。
* 和往常一样，文档也得到了改进。

### 向后不兼容的更改

* Scrapy1.5 减少了对Python 3.3的支持。
* 默认的Scrapy User-Agent现在使用https链接到scrapy.org。**这是技术上向后不兼容的**; 如果您依赖旧的值，则覆盖`USER_AGENT`。
* 日志设置被``custom_settings``覆写的问题被修复了。**这是技术上向后不兼容的** 因为日志工具已经从 ``[scrapy.utils.log]`` 改为 ``[scrapy.crawler]``。如果你想解析Scrapy日志，请更新日志解析器 。
* 在默认情况下，LinkExtractor 现在会忽略 ``m4v`` 扩展，这是行为的变化。
* 522和524的状态码被添加到``RETRY_HTTP_CODES``

### 新特性

- 在``Response.follow``中支持 ``<link>`` 标签 
- 支持 ``ptpython`` REPL 
- 在FilesPipeline 和 ImagesPipeline中已经可以支持Google云存储。
- "scrapy parse" 命令中新的 ``--meta`` 选项允许传递额外的 request.meta 
- 当使用 ``shell.inspect_response``时将填充爬虫变量
- 处理HTTP 308永久重定向
- 添加 522 和 524 到 ``RETRY_HTTP_CODES`` 
- 启动时，日志会记录版本信息
- ``scrapy.mail.MailSender`` 现在可以在Python 3中运行 (需要 Twisted 17.9.0)
- 与代理服务器的连接将被重用 
- 下载的中间件可以添加模板 
- 当解析回调没有定义时，会有明确的 NotImplementedError 信息 
- 爬虫程序可以禁用根日志处理程序的安装 
- 在默认情况下，LinkExtractor 现在会忽略 ``m4v`` 扩展
- 使用设置项 `DOWNLOAD_WARNSIZE`和 `DOWNLOAD_MAXSIZE` 来更好的处理用于响应的日志消息 
- 当一个Url而不是域名，被放到``Spider.allowed_domains``中时会显示警告。


### Bug 修复

- 日志设置被``custom_settings``覆写的问题被修复了。**这是技术上向后不兼容的**，
因为日志工具已经从 ``[scrapy.utils.log]`` 改为 ``[scrapy.crawler]``。如果需要的话，请更新日志解析器 。
- 默认的Scrapy User-Agent现在使用https链接到scrapy.org 。**这是技术上向后不兼容的**; 如果您依赖旧的值，则覆盖`USER_AGENT`。
- 修复PyPy和PyPy3测试错误，并正式支持它们 
- 当 ``DNSCACHE_ENABLED=False``，修复DNS解析器 
- Debian Jessie tox test env 添加 ``cryptography``  
- 如果 Request 回调是可调的，将添加验证检查 
- Port ``extras/qpsclient.py`` to Python 3 
- 在Python 3的环境下使用getfullargspec来停止 DeprecationWarning
- 更新弃用的测试别名 
- 修改 ``SitemapSpider`` 对备用链接的支持

### 文档更新

- 为 ``AUTOTHROTTLE_TARGET_CONCURRENCY`` 设置添加了缺失的要点。
- 更新贡献文档，文档新支持的频道。 
- 在文档中包含对 Scrapy 的引用。
- 修复损坏的链接; 外部链接使用 https:// 
- CloseSpider 拓展部分的文档进行了优化 
- 在 MongoDB 示例中使用了``insert_one()`` 
- 优化了部分拼写、文字错误
- ``CSVFeedSpider.headers`` 文档更清晰了 
-  ``DontCloseSpider`` 异常和 ``spider_idle``文档更清晰了 
- 更新了README中的 "Releases" 部分 
- 修复了 ``DOWNLOAD_FAIL_ON_DATALOSS`` 文档中rst语法 
- 对 startproject 参数描述的小修正 
- 对 Response.body 文档中的数据类型进行了梳理 
- DepthMiddleware 文档中添加了一条有关 ``request.meta['depth']`` 的提示
- CookiesMiddleware 文档中添加了一条有关 ``request.meta['dont_merge_cookies']`` 的提示 
- 更新的项目结构示例 
- 一个更好的 ItemExporters 用法示例
- 爬虫``from_crawler``方法和下载器中间件的文档
