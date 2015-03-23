pyspider [![Build Status][Build Status]][Travis CI] [![Coverage Status][Coverage Status]][Coverage] [![Try][Try]][Demo]
========

pyspider 是一个基于Python强大的爬虫框架. **[在线demo][Demo]**

- 基于Python强大的API接口开发爬虫代码。
- 兼容Python 2和Python 3
- 拥有强大的可视脚本编辑器，任务监控，项目管理和结果管理功能。
- 支持Javascript动态生成的页面数据抓取!
- 支持流行的数据库，如：MySQL, MongoDB, SQLite, PostgreSQL等 
- 支持任务优先队列，重试，周期执行，过期执行等常用爬虫功能.
- 支持分布式部署


简单实例 
-----------

```python
from pyspider.libs.base_handler import *


class Handler(BaseHandler):
    crawl_config = {
    }

    @every(minutes=24 * 60)
    def on_start(self):
        self.crawl('http://scrapy.org/', callback=self.index_page)

    @config(age=10 * 24 * 60 * 60)
    def index_page(self, response):
        for each in response.doc('a[href^="http"]').items():
            self.crawl(each.attr.href, callback=self.detail_page)

    def detail_page(self, response):
        return {
            "url": response.url,
            "title": response.doc('title').text(),
        }
```

[![Demo][Demo Img]][Demo]


安装
------------

* 执行 `pip install pyspider` 安装pyspider.
* 运行命令 `pyspider`启动pyspider, visit [http://localhost:5000/](http://localhost:5000/)

如何参与及互动
----------

* 使用程序
* 查看github的[Issue], 也可以发送 PR
* 访问论坛[User Group]


授权协议
-------
Licensed under the Apache License, Version 2.0


[Build Status]:         https://img.shields.io/travis/binux/pyspider/master.svg?style=flat
[Travis CI]:            https://travis-ci.org/binux/pyspider
[Coverage Status]:      https://img.shields.io/coveralls/binux/pyspider.svg?branch=master&style=flat
[Coverage]:             https://coveralls.io/r/binux/pyspider
[Try]:                  https://img.shields.io/badge/try-pyspider-blue.svg?style=flat
[Demo]:                 http://demo.pyspider.org/
[Demo Img]:             imgs/demo.png
[Issue]:                https://github.com/binux/pyspider/issues
[User Group]:           https://groups.google.com/group/pyspider-users
