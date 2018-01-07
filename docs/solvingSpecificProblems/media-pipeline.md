下载、处理文件和图片
===========================================

Scrapy 提供可重复使用的`item pipelines`，使用某一特定item下载附件(例如，当抓取时，也想在下载它们的图像到本地)。

这些管道共享一些功能和结构(我们称它们为媒体管道- medie pipeline)，但通常要么使用文件管道（Files Pipeline），要么使用图像管道（Images Pipeline）。

这两种管道都实现了这些特性：

* 避免重新下载最近下载的媒体。
* 指定存储文件的位置(系统目录、Amazon S3 bucket、谷歌云存储)。

图像管道（Images Pipeline）有一些处理图像的额外功能:

* 将所有下载的图像转换为通用格式(JPG)和模式(RGB)
* 生成缩略图
* 检查图像的宽度/高度以确保它们符合最小约束条件

管道还保留了当前正在计划下载的媒体url的内部队列，并将那些到达的包含相同图片的项目连接到那个队列中。 这可以避免多次下载几个项目共享的同一个图片。

使用文件管道 Files Pipeline
========================

当使用  `FilesPipeline`  ，典型的工作流是这样的：

1. 在一个Spider中，可以抓取一个 item，并将所需的url放入``file_urls``字段中。

2. 该 item 从 spider 返回并进入 item pipeline。

3. 当 item 到达 `FilesPipeline` 时，file_urls字段中的url将被安排使用标准的 scrapy 调度器和下载器(这意味着调度器和下载器中间件被重用)，但是因为更高的优先级，将在其他页面被抓取之前处理它们。该 item 在特定的管道阶段仍然“锁定”，直到文件完成下载(或出于某种原因失败)。

4. 当下载文件时，将会填充另一个字段(``files``)。这个字段将包含一个关于已下载文件的信息列表，如下载的路径、原始的抓取url(取自``file_urls``字段)和文件校验。文件字段列表中的文件将保留原始``file_urls``字段的相同顺序。如果某些文件未能下载，将会记录错误，文件将不会出现在``files``字段中。


使用图片管道 Images Pipeline
=========================

`ImagesPipeline` 的用法很像 `FilesPipeline`，
除了使用的默认字段名称不同：使用 ``image_urls`` 存放 item的图像url， ``images`` 字段则存放有关下载图像的信息。

使用 `ImagesPipeline` 处理图像文件的优点是，你可以配置一些额外的功能，如生成缩略图并根据它们的大小对图像进行过滤。

图像管道使用 `Pillow` 进行缩略图并将图像规范化为JPEG / RGB格式，所以需要你安装这个库才能使用它。Python映像库(PIL)在大多数情况下也应该使用，但是它在一些设置中会引起麻烦，所以我们建议使用`Pillow`而不是PIL。


* Pillow: https://github.com/python-pillow/Pillow
* Python Imaging Library: http://www.pythonware.com/products/pil/


开启媒体管道 （Media Pipeline）
============================

要开启媒体管道，你必须首先将它添加到项目的设置 `ITEM_PIPELINES` 中。

Images Pipeline， 使用：

    ITEM_PIPELINES = {'scrapy.pipelines.images.ImagesPipeline': 1}

Files Pipeline， 使用：

    ITEM_PIPELINES = {'scrapy.pipelines.files.FilesPipeline': 1}


> 你也可以同时使用文件和图像管道。

然后，将目标存储目录设置为一个有效值，该值将用于存储下载的图像。否则，即使已经设置了 `ITEM_PIPELINES` 项，管道也将保持禁用状态。

Files Pipeline, 设置 `FILES_STORE` 项：

	FILES_STORE = '/path/to/valid/dir'

Images Pipeline，设置 `IMAGES_STORE` 项：

	IMAGES_STORE = '/path/to/valid/dir'

支持存储
=================

文件系统目前是唯一官方支持的存储，但也支持在`Amazon S3`和谷歌云中存储文件。

* Amazon S3: https://aws.amazon.com/s3/
* Google Cloud Storage: https://cloud.google.com/storage/

文件系统存储
-------------------

存储的文件使用它们URL的 `SHA1 hash` 作为文件名。

比如，对下面的图片URL：

    http://www.example.com/image.jpg

它的 `SHA1 hash` ：

    3afec3b4765f8f0a07b78f98c07b83f013567a0a

将下载并存储在以下文件中：

   <IMAGES_STORE>/full/3afec3b4765f8f0a07b78f98c07b83f013567a0a.jpg

其中：

* ``<IMAGES_STORE>`` 是在`IMAGES_STORE`设置中为图像管道定义的目录。

* ``full`` 是一个子目录，可以将完整的图像从缩略图中分离出来(如果使用的话)。有关更多信息，请参见`缩略图`章节。

Amazon S3 存储
-----------------

`FILES_STORE` and `IMAGES_STORE` can represent an Amazon S3 bucket. Scrapy will automatically upload the files to the bucket.

For example, this is a valid :setting:`IMAGES_STORE` value::

    IMAGES_STORE = 's3://bucket/images'

You can modify the Access Control List (ACL) policy used for the stored files,
which is defined by the :setting:`FILES_STORE_S3_ACL` and
:setting:`IMAGES_STORE_S3_ACL` settings. By default, the ACL is set to
``private``. To make the files publicly available use the ``public-read``
policy::

    IMAGES_STORE_S3_ACL = 'public-read'

For more information, see `canned ACLs`_ in the Amazon S3 Developer Guide.

.. _canned ACLs: https://docs.aws.amazon.com/AmazonS3/latest/dev/acl-overview.html#canned-acl

谷歌云存储
---------------------

`FILES_STORE` and `IMAGES_STORE` can represent a Google Cloud Storage bucket. Scrapy will automatically upload the files to the bucket. (requires `google-cloud-storage` )

* google-cloud-storage: https://cloud.google.com/storage/docs/reference/libraries#client-libraries-install-python

For example, these are valid `IMAGES_STORE` and `GCS_PROJECT_ID` settings::

    IMAGES_STORE = 'gs://bucket/images/'
    GCS_PROJECT_ID = 'project_id'

For information about authentication, see this `documentation`.

* documentation: https://cloud.google.com/docs/authentication/production

用例
=============

为了使用媒体管道，需要先开启它（查看本章上面的内容）。

接下来，如果 spider 返回一个带有url 键(分别是``file_urls``或``image_urls``，对于文件或图像管道)的字典，管道将把结果放在相应的密钥(``files`` or ``images``)下。

如果您更喜欢使用 `Item` 类，那么定义一个具有必要字段的自定义 item，例如在这个用于图像管道的例子：

```
    import scrapy

    class MyItem(scrapy.Item):

        # ... other item fields ...
        image_urls = scrapy.Field()
        images = scrapy.Field()
```

如果你想要为url键或结果键使用另一个字段名，也可以覆盖它。

Files Pipeline，设置`FILES_URLS_FIELD` 和/或 `FILES_RESULT_FIELD` 设置：

    FILES_URLS_FIELD = 'field_name_for_your_files_urls'
    FILES_RESULT_FIELD = 'field_name_for_your_processed_files'

Images Pipeline，设置 `IMAGES_URLS_FIELD` 和/或 `IMAGES_RESULT_FIELD` 设置：

    IMAGES_URLS_FIELD = 'field_name_for_your_images_urls'
    IMAGES_RESULT_FIELD = 'field_name_for_your_processed_images'

如果需要更复杂的东西，并希望覆盖自定义的管道行为，可以参见 `media-pipeline-override`。

如果你有多个从 ImagePipeline 继承的图像管道，并且希望在不同的管道中有不同的设置，那么可以在管道类的大写名称之前设置设置键。如果你的管道被称为 MyPipeline，并且希望有自定义的IMAGES_URLS_FIELD，那么你可以定义设置项：MYPIPELINE_IMAGES_URLS_FIELD，这样就可以使用自定义设置。


附加功能
===================

文件过期
---------------

图像管道会避免下载最近下载过的文件。

若要调整此保留延迟时间，请使用 `FILES_EXPIRES` 设置项 (或使用图像管道则设置`IMAGES_EXPIRES`)，该设置指定了延迟的天数：

    # 120 days of delay for files expiration
    FILES_EXPIRES = 120

    # 30 days of delay for images expiration
    IMAGES_EXPIRES = 30

这两个设置的默认值为90天。

如果项目中有FilesPipeline的子类管道，并且想要有不同的设置，你可以设置以大写的类名前面的设置键。例如，给定管道类名为MyPipeline，可以这样设置设置键：

    MYPIPELINE_FILES_EXPIRES = 180

这样管道类 MyPipeline 过期时间设置为180。


生成图片的缩略图
-------------------------------

图像管道可以自动创建下载图像的缩略图。

为了使用这个特性，必须设置 `IMAGES_THUMBS` 为一个字典，其中键是缩略图名称，值是它们的尺寸。

例如：

   IMAGES_THUMBS = {
       'small': (50, 50),
       'big': (270, 270),
   }

当使用此功能时，图像管道将使用这种格式创建每个指定大小的缩略图：

    <IMAGES_STORE>/thumbs/<size_name>/<image_id>.jpg

其中：

* ``<size_name>`` 是在设置项 `IMAGES_THUMBS` 中指定的字典的key (``small``, ``big`` 等)

* ``<image_id>`` 是图片url的 `SHA1 hash`

* SHA1 hash: https://en.wikipedia.org/wiki/SHA_hash_functions

图像文件存储的例子中使用了 ``small`` 和 ``big`` 作为缩略图名：

   <IMAGES_STORE>/full/63bbfea82b8880ed33cdb762aa11fab722a90a24.jpg
   <IMAGES_STORE>/thumbs/small/63bbfea82b8880ed33cdb762aa11fab722a90a24.jpg
   <IMAGES_STORE>/thumbs/big/63bbfea82b8880ed33cdb762aa11fab722a90a24.jpg

第一个是从网站下载的完整图像。

过滤掉尺寸小的图片
--------------------------

当使用图像管道时，可以删除尺寸过小的图片，通过 `IMAGES_MIN_HEIGHT` 和 `IMAGES_MIN_WIDTH` 设置，指定最小允许的高、宽尺寸。

例如：

   IMAGES_MIN_HEIGHT = 110
   IMAGES_MIN_WIDTH = 110


> 尺寸限制不会影响缩略图的生成。

可以只设置一个大小限制或两者都设置。
当设置它们时，只有满足两个最小尺寸的图像才能被保存。

上面的例子中，尺寸(105×105)或(105×200)或(200×105)的图像都将被删除，因为至少有一个维度小于了约束限制。

默认情况下，没有大小限制，因此会处理所有图像。

允许重定向
---------------------

默认媒体管道忽略重定向，即HTTP重定向到媒体文件URL请求将意味着媒体(media)下载失败。

要处理媒体重定向，请将该设置设置为``True``:

    MEDIA_ALLOW_REDIRECTS = True



扩展媒体管道
=============================

.. module:: scrapy.pipelines.files
   :synopsis: Files Pipeline

在这里你可以看到在自定义文件管道中覆盖的方法：

.. class:: FilesPipeline

   .. method:: FilesPipeline.get_media_requests(item, info)

      As seen on the workflow, the pipeline will get the URLs of the images to
      download from the item. In order to do this, you can override the
      :meth:`~get_media_requests` method and return a Request for each
      file URL::

         def get_media_requests(self, item, info):
             for file_url in item['file_urls']:
                 yield scrapy.Request(file_url)

      Those requests will be processed by the pipeline and, when they have finished
      downloading, the results will be sent to the
      :meth:`~item_completed` method, as a list of 2-element tuples.
      Each tuple will contain ``(success, file_info_or_error)`` where:

      * ``success`` is a boolean which is ``True`` if the image was downloaded
        successfully or ``False`` if it failed for some reason

      * ``file_info_or_error`` is a dict containing the following keys (if success
        is ``True``) or a `Twisted Failure`_ if there was a problem.

        * ``url`` - the url where the file was downloaded from. This is the url of
          the request returned from the :meth:`~get_media_requests`
          method.

        * ``path`` - the path (relative to :setting:`FILES_STORE`) where the file
          was stored

        * ``checksum`` - a `MD5 hash`_ of the image contents

      The list of tuples received by :meth:`~item_completed` is
      guaranteed to retain the same order of the requests returned from the
      :meth:`~get_media_requests` method.

      Here's a typical value of the ``results`` argument::

          [(True,
            {'checksum': '2b00042f7481c7b056c4b410d28f33cf',
             'path': 'full/0a79c461a4062ac383dc4fade7bc09f1384a3910.jpg',
             'url': 'http://www.example.com/files/product1.pdf'}),
           (False,
            Failure(...))]

      By default the :meth:`get_media_requests` method returns ``None`` which
      means there are no files to download for the item.

   .. method:: FilesPipeline.item_completed(results, item, info)

      The :meth:`FilesPipeline.item_completed` method called when all file
      requests for a single item have completed (either finished downloading, or
      failed for some reason).

      The :meth:`~item_completed` method must return the
      output that will be sent to subsequent item pipeline stages, so you must
      return (or drop) the item, as you would in any pipeline.

      Here is an example of the :meth:`~item_completed` method where we
      store the downloaded file paths (passed in results) in the ``file_paths``
      item field, and we drop the item if it doesn't contain any files::

          from scrapy.exceptions import DropItem

          def item_completed(self, results, item, info):
              file_paths = [x['path'] for ok, x in results if ok]
              if not file_paths:
                  raise DropItem("Item contains no files")
              item['file_paths'] = file_paths
              return item

      By default, the :meth:`item_completed` method returns the item.


.. module:: scrapy.pipelines.images
   :synopsis: Images Pipeline

See here the methods that you can override in your custom Images Pipeline:

.. class:: ImagesPipeline

    The :class:`ImagesPipeline` is an extension of the :class:`FilesPipeline`,
    customizing the field names and adding custom behavior for images.

   .. method:: ImagesPipeline.get_media_requests(item, info)

      Works the same way as :meth:`FilesPipeline.get_media_requests` method,
      but using a different field name for image urls.

      Must return a Request for each image URL.

   .. method:: ImagesPipeline.item_completed(results, item, info)

      The :meth:`ImagesPipeline.item_completed` method is called when all image
      requests for a single item have completed (either finished downloading, or
      failed for some reason).

      Works the same way as :meth:`FilesPipeline.item_completed` method,
      but using a different field names for storing image downloading results.

      By default, the :meth:`item_completed` method returns the item.


自定义图片管道的例子
==============================

这里有一个图像管道的完整示例，其方法在上面进行了测试：

    import scrapy
    from scrapy.pipelines.images import ImagesPipeline
    from scrapy.exceptions import DropItem

    class MyImagesPipeline(ImagesPipeline):

        def get_media_requests(self, item, info):
            for image_url in item['image_urls']:
                yield scrapy.Request(image_url)

        def item_completed(self, results, item, info):
            image_paths = [x['path'] for ok, x in results if ok]
            if not image_paths:
                raise DropItem("Item contains no images")
            item['image_paths'] = image_paths
            return item

* Twisted Failure: https://twistedmatrix.com/documents/current/api/twisted.python.failure.Failure.html
* MD5 hash: https://en.wikipedia.org/wiki/MD5
