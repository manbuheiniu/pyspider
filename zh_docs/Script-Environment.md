爬虫代码运行环境介绍
==================

变量
---------
* `self.project_name`  当前脚本名
* `self.project`       当前任务信息
* `self.response`      当前请求的response
* `self.task`          当前请求的原始task信息

关于爬虫代码
------------
* Handler(BaseHandler) 是脚本的主体，爬虫类中至少有一个类继承`BaseHandler`类
* 任务执行方法至少需要三个参数来指定任务。如: `def callback(self, response, task)`
* 默认 status_code 不为 200 的请求不会发给回调函数解析，请用 `@catch_status_code_error` 修饰回调函数，以捕获抓取异常。

关于执行环境
-----------------
* `logger` 和 `print` 的信息会被采集，请使用 `logger` 打印日志（默认 `logging` 已经 `import` 到环境中，请不要再次导入）.
* 可以通过 projects 伪模块将其他 project 导入成 `module`，例如 `from projects import some_project`

### Web view

* 在控制台可以通过view功能渲染任务页面，实时查看页面内容。

### HTML view

* HTML View功能可以实时查看回调函数 (index_page, detail_page, etc.)正处理的HTML代码。

### Follows view

* 此功能可实时查看HTML被分析后有哪些链接可以被添加到任务，及添加到任务时对应的回调函数。
* index_page 可分析页面并跟进链接 detail_page 可解析任务并获取需要的数据.

### Messages view

* 显示通过api发送过来的消息 [`self.send_message`](apis/self.send_message) API。

### Enable CSS Selector Helper

* 在WEB VIEW模式下可以打开css选择器助手，用于帮助用户快速选择需要的元素并生成css方法，当点击页面上对应元素时，框架会自动把cs方法添加到脚本中。
