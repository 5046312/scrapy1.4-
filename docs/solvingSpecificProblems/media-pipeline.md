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

Using the `ImagesPipeline` is a lot like using the `FilesPipeline`,
except the default field names used are different: you use ``image_urls`` for
the image URLs of an item and it will populate an ``images`` field for the information
about the downloaded images.

The advantage of using the `ImagesPipeline` for image files is that you
can configure some extra functions like generating thumbnails and filtering
the images based on their size.

The Images Pipeline uses `Pillow` for thumbnailing and normalizing images to
JPEG/RGB format, so you need to install this library in order to use it.
`Python Imaging Library` (PIL) should also work in most cases, but it is known
to cause troubles in some setups, so we recommend to use `Pillow` instead of
PIL.

* Pillow: https://github.com/python-pillow/Pillow
* Python Imaging Library: http://www.pythonware.com/products/pil/


开启媒体管道 （Media Pipeline）
============================

To enable your media pipeline you must first add it to your project
:setting:`ITEM_PIPELINES` setting.

For Images Pipeline, use::

    ITEM_PIPELINES = {'scrapy.pipelines.images.ImagesPipeline': 1}

For Files Pipeline, use::

    ITEM_PIPELINES = {'scrapy.pipelines.files.FilesPipeline': 1}


> You can also use both the Files and Images Pipeline at the same time.


Then, configure the target storage setting to a valid value that will be used
for storing the downloaded images. Otherwise the pipeline will remain disabled,
even if you include it in the :setting:`ITEM_PIPELINES` setting.

For the Files Pipeline, set the :setting:`FILES_STORE` setting::

   FILES_STORE = '/path/to/valid/dir'

For the Images Pipeline, set the :setting:`IMAGES_STORE` setting::

   IMAGES_STORE = '/path/to/valid/dir'

支持存储
=================

File system is currently the only officially supported storage, but there are
also support for storing files in `Amazon S3`_ and `Google Cloud Storage`_.

.. _Amazon S3: https://aws.amazon.com/s3/
.. _Google Cloud Storage: https://cloud.google.com/storage/

文件系统存储
-------------------

The files are stored using a `SHA1 hash`_ of their URLs for the file names.

For example, the following image URL::

    http://www.example.com/image.jpg

Whose `SHA1 hash` is::

    3afec3b4765f8f0a07b78f98c07b83f013567a0a

Will be downloaded and stored in the following file::

   <IMAGES_STORE>/full/3afec3b4765f8f0a07b78f98c07b83f013567a0a.jpg

Where:

* ``<IMAGES_STORE>`` is the directory defined in :setting:`IMAGES_STORE` setting
  for the Images Pipeline.

* ``full`` is a sub-directory to separate full images from thumbnails (if
  used). For more info see :ref:`topics-images-thumbnails`.

Amazon S3 存储
-----------------

.. setting:: FILES_STORE_S3_ACL
.. setting:: IMAGES_STORE_S3_ACL

:setting:`FILES_STORE` and :setting:`IMAGES_STORE` can represent an Amazon S3
bucket. Scrapy will automatically upload the files to the bucket.

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

.. setting:: GCS_PROJECT_ID

:setting:`FILES_STORE` and :setting:`IMAGES_STORE` can represent a Google Cloud Storage
bucket. Scrapy will automatically upload the files to the bucket. (requires `google-cloud-storage`_ )

.. _google-cloud-storage: https://cloud.google.com/storage/docs/reference/libraries#client-libraries-install-python

For example, these are valid :setting:`IMAGES_STORE` and :setting:`GCS_PROJECT_ID` settings::

    IMAGES_STORE = 'gs://bucket/images/'
    GCS_PROJECT_ID = 'project_id'

For information about authentication, see this `documentation`_.

.. _documentation: https://cloud.google.com/docs/authentication/production

用例
=============

.. setting:: FILES_URLS_FIELD
.. setting:: FILES_RESULT_FIELD
.. setting:: IMAGES_URLS_FIELD
.. setting:: IMAGES_RESULT_FIELD

In order to use a media pipeline first, :ref:`enable it
<topics-media-pipeline-enabling>`.

Then, if a spider returns a dict with the URLs key (``file_urls`` or
``image_urls``, for the Files or Images Pipeline respectively), the pipeline will
put the results under respective key (``files`` or ``images``).

If you prefer to use :class:`~.Item`, then define a custom item with the
necessary fields, like in this example for Images Pipeline::

    import scrapy

    class MyItem(scrapy.Item):

        # ... other item fields ...
        image_urls = scrapy.Field()
        images = scrapy.Field()

If you want to use another field name for the URLs key or for the results key,
it is also possible to override it.

For the Files Pipeline, set :setting:`FILES_URLS_FIELD` and/or
:setting:`FILES_RESULT_FIELD` settings::

    FILES_URLS_FIELD = 'field_name_for_your_files_urls'
    FILES_RESULT_FIELD = 'field_name_for_your_processed_files'

For the Images Pipeline, set :setting:`IMAGES_URLS_FIELD` and/or
:setting:`IMAGES_RESULT_FIELD` settings::

    IMAGES_URLS_FIELD = 'field_name_for_your_images_urls'
    IMAGES_RESULT_FIELD = 'field_name_for_your_processed_images'

If you need something more complex and want to override the custom pipeline
behaviour, see :ref:`topics-media-pipeline-override`.

If you have multiple image pipelines inheriting from ImagePipeline and you want
to have different settings in different pipelines you can set setting keys
preceded with uppercase name of your pipeline class. E.g. if your pipeline is
called MyPipeline and you want to have custom IMAGES_URLS_FIELD you define
setting MYPIPELINE_IMAGES_URLS_FIELD and your custom settings will be used.


附加功能
===================

File expiration
---------------

.. setting:: IMAGES_EXPIRES
.. setting:: FILES_EXPIRES

The Image Pipeline avoids downloading files that were downloaded recently. To
adjust this retention delay use the :setting:`FILES_EXPIRES` setting (or
:setting:`IMAGES_EXPIRES`, in case of Images Pipeline), which
specifies the delay in number of days::

    # 120 days of delay for files expiration
    FILES_EXPIRES = 120

    # 30 days of delay for images expiration
    IMAGES_EXPIRES = 30

The default value for both settings is 90 days.

If you have pipeline that subclasses FilesPipeline and you'd like to have
different setting for it you can set setting keys preceded by uppercase
class name. E.g. given pipeline class called MyPipeline you can set setting key:

    MYPIPELINE_FILES_EXPIRES = 180

and pipeline class MyPipeline will have expiration time set to 180.

.. _topics-images-thumbnails:

生成图片的缩略图
-------------------------------

The Images Pipeline can automatically create thumbnails of the downloaded
images.

.. setting:: IMAGES_THUMBS

In order use this feature, you must set :setting:`IMAGES_THUMBS` to a dictionary
where the keys are the thumbnail names and the values are their dimensions.

For example::

   IMAGES_THUMBS = {
       'small': (50, 50),
       'big': (270, 270),
   }

When you use this feature, the Images Pipeline will create thumbnails of the
each specified size with this format::

    <IMAGES_STORE>/thumbs/<size_name>/<image_id>.jpg

Where:

* ``<size_name>`` is the one specified in the :setting:`IMAGES_THUMBS`
  dictionary keys (``small``, ``big``, etc)

* ``<image_id>`` is the `SHA1 hash`_ of the image url

.. _SHA1 hash: https://en.wikipedia.org/wiki/SHA_hash_functions

Example of image files stored using ``small`` and ``big`` thumbnail names::

   <IMAGES_STORE>/full/63bbfea82b8880ed33cdb762aa11fab722a90a24.jpg
   <IMAGES_STORE>/thumbs/small/63bbfea82b8880ed33cdb762aa11fab722a90a24.jpg
   <IMAGES_STORE>/thumbs/big/63bbfea82b8880ed33cdb762aa11fab722a90a24.jpg

The first one is the full image, as downloaded from the site.

过滤掉小的图片
--------------------------

.. setting:: IMAGES_MIN_HEIGHT

.. setting:: IMAGES_MIN_WIDTH

When using the Images Pipeline, you can drop images which are too small, by
specifying the minimum allowed size in the :setting:`IMAGES_MIN_HEIGHT` and
:setting:`IMAGES_MIN_WIDTH` settings.

For example::

   IMAGES_MIN_HEIGHT = 110
   IMAGES_MIN_WIDTH = 110

.. note::
    The size constraints don't affect thumbnail generation at all.

It is possible to set just one size constraint or both. When setting both of
them, only images that satisfy both minimum sizes will be saved. For the
above example, images of sizes (105 x 105) or (105 x 200) or (200 x 105) will
all be dropped because at least one dimension is shorter than the constraint.

By default, there are no size constraints, so all images are processed.

允许重定向
---------------------

.. setting:: MEDIA_ALLOW_REDIRECTS

By default media pipelines ignore redirects, i.e. an HTTP redirection
to a media file URL request will mean the media download is considered failed.

To handle media redirections, set this setting to ``True``::

    MEDIA_ALLOW_REDIRECTS = True

.. _topics-media-pipeline-override:

扩展媒体管道
=============================

.. module:: scrapy.pipelines.files
   :synopsis: Files Pipeline

See here the methods that you can override in your custom Files Pipeline:

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
