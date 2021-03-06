关于任务
===========

任务是被调度的最小单位。

任务
-----

* `taskid`是任务的唯一区分标识. (默认使用 url 的 md5 作为 taskid, 也可以重写`def get_taskid(self, task)`方法自定义`taskid`)
* 项目之间任务的`taskid`是独立的（不同 project 间 taskid 可以相同）。
* 任务有如下四种状态:
    - active   活动状态，表示任务在队列中等待被抓取（包括在队列中、抓取中、执行时间未到、重试中）
    - failed   经过重试后抓取失败
    - success  抓取成功状态
    - bad      损坏，暂未使用
* 只有处于 active 状态的任务才会得到调度，
* 任务会根据优先级 `priority` ，在有流量配额情况下，依次发起调度

调度
--------

当一个新的任务（没见过）来：

* 如果 `exetime` 没有到执行时间. 它会被放进队列等待时间。
* 否则会被接受执行。

当一个任务已经在队列当中:

* 默认会忽略，除非设置了 `force_update`

当一个完成的任务进来时:

* 如果设置了 `age` , 并且`last_crawl_time + age < now` 它会被接收。否则会被忽略。
* 如果 `itag` 被设置并且不等于它的值，则会被接受，否则会被忽略。
