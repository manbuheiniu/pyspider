命令行参数详解
============

全局配置
-------------

你可以通过输入`pyspider --help` 来获取帮助，也可以使用`pyspider all --help`获取子命令帮助。

全局参数适用于所有子命令

```
用法: pyspider [OPTIONS] COMMAND [ARGS]...

  A powerful spider system in python.

Options:
  -c, --config FILENAME    a json file with default values for subcommands.
                           {"webui": {"port":5001}}
  --debug                  debug mode
  --queue-maxsize INTEGER  maxsize of queue
  --taskdb TEXT            database url for taskdb, default: sqlite
  --projectdb TEXT         database url for projectdb, default: sqlite
  --resultdb TEXT          database url for resultdb, default: sqlite
  --amqp-url TEXT          amqp url for rabbitmq, default: built-in Queue
  --phantomjs-proxy TEXT   phantomjs proxy ip:port
  --data-path TEXT         data dir path
  --version                Show the version and exit.
  --help                   Show this message and exit.
```

#### --config

配置文件为JSON格式，适用于配置全局参数和子命令的参数。 [example](/Deployment/#configjson)

``` json
{
  "taskdb": "mysql+taskdb://username:password@host:port/taskdb",
  "projectdb": "mysql+projectdb://username:password@host:port/projectdb",
  "resultdb": "mysql+resultdb://username:password@host:port/resultdb",
  "amqp_url": "amqp://username:password@host:port/%2F",
  "webui": {
    "username": "some_name",
    "password": "some_passwd",
    "need-auth": true
  }
}
```

#### --queue-maxsize

任务队列长度限制,值为0时为不限制长度。

#### --taskdb, --projectdb, --resultdb

```
mysql:
    mysql+type://user:passwd@host:port/database
sqlite:
    # relative path
    sqlite+type:///path/to/database.db
    # absolute path
    sqlite+type:////path/to/database.db
    # memory database
    sqlite+type://
mongodb:
    mongodb+type://[username:password@]host1[:port1][,host2[:port2],...[,hostN[:portN]]][/[database][?options]]
    more: http://docs.mongodb.org/manual/reference/connection-string/
sqlalchemy:
    sqlalchemy+postgresql+type://user:passwd@host:port/database
    sqlalchemy+mysql+mysqlconnector+type://user:passwd@host:port/database
    more: http://docs.sqlalchemy.org/en/rel_0_9/core/engines.html
local:
    local+projectdb://filepath,filepath
    
type:
    should be one of `taskdb`, `projectdb`, `resultdb`.
```


#### --amqp-url

队列服务 [https://www.rabbitmq.com/uri-spec.html](https://www.rabbitmq.com/uri-spec.html)

#### --phantomjs-proxy

phantomjs代理地址。使用前你需要有一个phantomjs已经被安装并且需要运行命令: [`pyspider phantomjs`](#phantomjs).

#### --data-path

SQLite数据库文件保存路径，默认是安装目录下的data目录。


all
---

```
用法: pyspider all [OPTIONS]

  在线程或进程中运行所有组件。

Options:
  --fetcher-num INTEGER         instance num of fetcher
  --processor-num INTEGER       instance num of processor
  --result-worker-num INTEGER   instance num of result worker
  --run-in [subprocess|thread]  run each components in thread or subprocess.
                                always using thread for windows.
  --help                        Show this message and exit.
```


单用户模式（one）
---

```
用法: pyspider one [OPTIONS] [SCRIPTS]...

    单模式启动,所有的事件都在tornado.ioloop中运行，此模式常用于调试。

参数:
  -i, --interactive  enable interactive mode, you can choose crawl url.
  --phantomjs        enable phantomjs, will spawn a subprocess for phantomjs
  --help             Show this message and exit.
```

**注意: 单模式下web控制台不会启动。**

在单模式下爬取结果会输出到标准设备（终端）上，如果想保存输出结果请使用输出重定向命令 `pyspider one > result.txt`.

#### [SCRIPTS]

项目的状态为RUNNING时，`rate` 和 `burst` 可以设置脚本的执行速度。

```
# rate: 1.0
# burst: 3
```

当脚本设置好并启动后，`taskdb` 和 `resultdb` 默认使用内存数据库sqlite来保存数据 (当然你可以使用全局配置`--taskdb`, `--resultdb`修改这两个数据库的保存方法。). 当 on_start 执行时会自动启动sqlite数据库服务程序。

#### -i, --interactive

互动模式。互动模式将会启动一个交互的终端，每一次操作都需要用户输入命令或参数。在互动模式下你可以使用：

``` python
crawl(url, project=None, **kwargs)
    Crawl given url, same parameters as BaseHandler.crawl

    url - url or taskid, parameters will be used if in taskdb
    project - can be omitted if only one project exists.
    
quit_interactive()
    Quit interactive mode
    
quit_pyspider()
    Close pyspider
```

在脚本互动模式中你可以使用 `pyspider.libs.utils.python_console()` 打开一个新的交互终端。

bench
-----

```
用法: pyspider bench [OPTIONS]

  用来做压力测试。在测试过程中，pyspider会使用内存模式的sqlite，而不是硬盘下的sqlite。

参数:
  --fetcher-num INTEGER         instance num of fetcher
  --processor-num INTEGER       instance num of processor
  --result-worker-num INTEGER   instance num of result worker
  --run-in [subprocess|thread]  run each components in thread or subprocess.
                                always using thread for windows.
  --total INTEGER               total url in test page
  --show INTEGER                show how many urls in a page
  --help                        Show this message and exit.
```


调度器（scheduler）
---------

```
用法: pyspider scheduler [OPTIONS]

  运行调度程序，不管什么部署方式只允许有一个调度器运行。

参数:
  --xmlrpc / --no-xmlrpc
  --xmlrpc-host TEXT
  --xmlrpc-port INTEGER
  --inqueue-limit INTEGER  size limit of task queue for each project, tasks
                           will been ignored when overflow
  --delete-time INTEGER    delete time before marked as delete
  --active-tasks INTEGER   active log size
  --loop-limit INTEGER     maximum number of tasks due with in a loop
  --scheduler-cls TEXT     scheduler class to be used.
  --help                   Show this message and exit.
```

#### --scheduler-cls

这个选项是使用自定义调度器类。

JS解析框架（phantomjs）
---------

```
用法: pyspider phantomjs [OPTIONS]

  Run phantomjs fetcher if phantomjs is installed.

参数:
  --phantomjs-path TEXT  phantomjs path
  --port INTEGER         phantomjs port
  --help                 Show this message and exit.
```

任务抓取器（fetcher）
-------

```
用法: pyspider fetcher [OPTIONS]

  Run Fetcher.

参数:
  --xmlrpc / --no-xmlrpc
  --xmlrpc-host TEXT
  --xmlrpc-port INTEGER
  --poolsize INTEGER      max simultaneous fetches
  --proxy TEXT            proxy host:port
  --user-agent TEXT       user agent
  --timeout TEXT          default fetch timeout
  --fetcher-cls TEXT      Fetcher class to be used.
  --help                  Show this message and exit.
```

#### --proxy

这个可以设置fetcher的代理服务器，优先级比`self.crawl`的选项低，可以被`self.crawl`选项中的代理配置覆盖。 [DOC](apis/self.crawl/#fetch)


任务处理器（processor）
---------

```
用法: pyspider processor [OPTIONS]

  Run Processor.

参数:
  --processor-cls TEXT  Processor class to be used.
  --help                Show this message and exit.
```

结果处理器（result_worker）
-------------

```
用法: pyspider result_worker [OPTIONS]

  Run result worker.

参数:
  --result-cls TEXT  ResultWorker class to be used.
  --help             Show this message and exit.
```


web控制台（webui）
-----

```
用法: pyspider webui [OPTIONS]

  Run WebUI

参数:
  --host TEXT            webui bind to host
  --port INTEGER         webui bind to host
  --cdn TEXT             js/css cdn server
  --scheduler-rpc TEXT   xmlrpc path of scheduler
  --fetcher-rpc TEXT     xmlrpc path of fetcher
  --max-rate FLOAT       max rate for each project
  --max-burst FLOAT      max burst for each project
  --username TEXT        username of lock -ed projects
  --password TEXT        password of lock -ed projects
  --need-auth            need username and password
  --webui-instance TEXT  webui Flask Application instance to be used.
  --help                 Show this message and exit.
```

#### --cdn

JS/CSS 库的CDN服务, URL必须兼容 [cdnjs](https://cdnjs.com/)

#### --fercher-rpc

XMLRPC服务器的路径，用于与调度器交互获取信息（多用于WEB控制台与调度器不在同一台服务器的情况）. 如果没有设置会使用本地地址。

#### --need-auth

是否需要认证，如果为true，访问WEB控制平台时会要求用户输入账号和密码。账号和密码请使用 `--username` 和 `--password` 设置。
