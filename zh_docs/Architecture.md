PYSPIDER架构
============

本文档介绍了我为什么会开发pyspider和pyspider架构流程。

为什么开发pyspider?
---
pyspider 来源于以前做的一个垂直搜索引擎使用的爬虫后端:

1. 我们需要从200个站点（由于站点失效，不是都同时啦，同时有100+在跑吧）采集数据
> 灵活的抓取控制是必须的。同时，由于100个站点，每天都可能会有站点失效或者改版，所以需要能够监控模板失效，以及查看抓取状态。

2. 要求在5分钟内将对方网站的更新更新到库中
> 为了达到5分钟更新，我们使用抓取最近更新页上面的最后更新时间，以此来判断页面是否需要再次抓取。  
> **因为www始终在变化，所以pyspider不会停止（爬虫）**

此外，为了满足合作伙伴的需求，我们需要更高级的APIs,如：POST请求，使用代理，授权访问等. 全面的脚本功能比全局配置更有效。 

概述
--------
下图显示了pyspider的组件之间的关系和信息流的方向。pyspider的开发是基于这个框架基础之上的。理解了这个会很快入手pyspider的开发部署。

![pyspider](imgs/pyspider-arch.png)

各个组件间使用消息队列连接，除了scheduler是单点的，fetcher 和 processor 都是可以多实例分布式部署的。 scheduler 负责整体的调度控制 [benchmarking](https://gist.github.com/binux/67b276c51e988f8e2c31#comment-1339242)。任务由 scheduler 发起调度，fetcher 抓取网页内容， processor 执行预先编写的python脚本，输出结果或产生新的提链任务（发往 scheduler），形成闭环。每个脚本可以灵活使用各种python库对页面进行解析，使用框架API控制下一步抓取动作，通过设置回调控制解析动作。

组件介绍
----------

### 调度器（Scheduler）
调度器从任务队（`newtask_queue`）列接收任务。决定任务是否是新的或需要重新抓取。同时实现任务的优先级，周期定时，流量控制([token bucket](http://en.wikipedia.org/wiki/Token_bucket) algorithm)，基于时间周期 或 前链标签（例如更新时间）的重抓取调度等调度功能。

通过 `self.crawl` [API](apis/)生成新任务实现自动发现链接并跟进。 

注意：当前程序中只有且只能有一个调试组件来实现调度功能。其它组件可按需求部署。

### 抓取器（Fetcher）
抓取器负责任务的数据抓取功能，可模拟浏览器从网络上获取任务对应的内容。并把抓取的结果传递给处理器（`Processor`）。抓取器支持 [Data URI](http://en.wikipedia.org/wiki/Data_URI_scheme) ，支持渲染JS生成的页面( 通过 [phantomjs](http://phantomjs.org/)实现js渲染). 支持通过[API](apis/self.crawl/#fetch)自定义请求方法，请求头，设置cookies，代理等常用功能。

### JS渲染器（Phantomjs Fetcher）
JS渲染器（Phantomjs Fetcher）其实是一个代理，JS渲染器（Phantomjs Fetcher）以代理的模式替抓取器（Fetcher）请求网络信息并实现js引擎实现页面渲染。最后把结果返回给抓取器（Fetcher）:

```
scheduler -> fetcher -> processor
                |
            phantomjs
                |
             internet
```

### 处理器（Processor）
处理器（Processor）负责用户脚本中的数据处理，从任务结果中使用[PyQuery](https://pythonhosted.org/pyquery/)提取需要的信息。脚本是全模式下的python，所以你也可以导入自己喜欢的HTML解析器来提取信息。

处理器（Processor）可以实现日志功能和异常捕获。还可以生成新任务发送给调度器（`scheduler`），最后把提取的信息发送给结果处理器（`Result Worker`）。

### 结果处理器（Result Worker (可选)）
结果处理器（`Result Worker`)的功能是接收处理器（`Processor`）发送过来的结果并保存到数据库（`resultdb`）。你也可以重写结果处理器把结果保存到自定义的数据库中。

### WEB控制台（WebUI）
WEB控制台（`WebUI`）是具有管理功能的web页面，对pyspider的多数操作都是在本页面上完成。WEB控制台（`WebUI`）的功能是：:

* web脚本编写，单步调试
* 项目管理 
* web的可视化任务监控
* 异常捕获、log捕获，print捕获等

数据流
---------
The data flow in pyspider is just as your seen in diagram above:

1. 1. Each script has a callback named `on_start`, when you press the `Run` button on WebUI. A new task of `on_start` is submitted to Scheduler as the entries of project.
2. Scheduler dispatches this `on_start` task with a Data URI as a normal task to Fetcher.
3. Fetcher makes a request and a response to it (for Data URI, it's a fake request and response, but has no difference with other normal tasks), then feeds to Processor.
4. Processor calls the `on_start` method and generated some new URL to crawl. Processor send a message to Scheduler that this task is finished and new tasks via message queue to Scheduler (here is no results for `on_start` in most case. If has results, Processor send them to `result_queue`).
5. Scheduler receives the new tasks, looking up in the database, determine whether the task is new or requires re-crawl, if so, put them into task queue. Dispatch tasks in order.
6. The process repeats (from step 3) and wouldn't stop till WWW is dead ;-). Scheduler will check periodic tasks to crawl latest data.
