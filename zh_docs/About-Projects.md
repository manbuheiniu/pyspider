关于项目
==============

在大多数情况下，一个网站对应一个项目即一个脚本。

* 每个项目是独立的，但是可以通过 `from projects import other_project`导入另一个项目模块。
* 每个项目有五种状态，分别是: `TODO`, `STOP`, `CHECKING`, `DEBUG`, `RUNNING`
    - `TODO` - 当脚本被创建时自动变为这个状态.
    - `STOP` - 你可以设置一个项目为 `STOP` ，这个状态的项目会停止运行 (= =).
    - `CHECKING` - 当一个正运行的项目被编辑后，为了避免不完全的修改，项目状态会自动变为 `CHECKING` 。
    - `DEBUG`/`RUNNING` -  这两个状态就如字面意思，都可以正常运行，只是建议在正常运行之前最好先`DEBUG`好后再正常`RUNNING`。
* 通过 `rate` and `burst` 来控制抓取进程数和速度。 其实是控制令牌环 [token-bucket](http://en.wikipedia.org/wiki/Token_bucket) 。
    - `rate` -  每秒请求数。
    - `burst` - 进程数。
如, `rate/burst = 0.1/3`, 这样的设置是每10秒抓执行一个任务。当所有的任务完成时，项目会每分钟检查一下最后更新时间，假如突然有三个新任务，这三个新任务不需要等3*10秒，而是第四个任务才需要等10秒。
* 删除一个项目, 设置 组名（`group`）为 `delete` 并设置项目状态为 `STOP`, 设置完成24小时后会自动删除。
