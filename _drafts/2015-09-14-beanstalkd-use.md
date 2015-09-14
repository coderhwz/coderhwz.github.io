---
layout: post
title: Beanstalkd的使用过程
keyword: php,Beanstalkd,队列服务器
description: 基于内存的队列服务器Beanstalkd使用
tags: 
    - PHP
---

最近公司安排了一个任务，群发信息，由于环境的原因，我们无法使用被动请求的方式响应消息，而是使用主动发送的方式。项目中原来是存在这要一个服务的，只不过是使用轮询数据库的方式。这一次公司提出要改良这个群发的效率，于是我选择队列来做。

第一步就是选择队列服务器了，我选择了Beanstalkd服务器，因为它基于内存，非常快，具备队列服务的各种操作。与Reids的Lpush Rpop,相比Beanstalkd只专注队列，这也就意味着它在队列方面是非常专业的。

Beanstalkd的基本使用比较简单，生产消息

```php
<?php

    $queue = new Client();
    $queue->connect();
    $queue->useTube('example');
    $message = 'hello world';
    $queue->put(0,0,0,$message);

```

消费消息:

```php
<?php 
    $queue = new Client();
    $queue->watch('example');
    $job = $queue->reserve();
    if( $job ){
        echo "消息ID：{$job['id']} 消息体：{$job['body']}";
    }

```

一次这样的例子使用并没有什么，可以完美跑起来。试试下面这段代码会怎么样？

```php
<?php
    //文件A
    $queue = new Client();
    $queue->connect();
    $queue->useTube('example');
    $message = 'hello world';
    $queue->put(0,0,0,$message);

    //文件B
    $queue = new Client();
    $queue->watch('example);
    while(true){
        $job = $queue->reserve();
        if($job){
            // handle job
        }else{
            echo 'not a job';
        }

    }
```

有任务时，以上代码可以正常运行，但没有任务的时候，就要出问题了,它会一直输出`not a job`，为什么呢？有两个原因：
1. `$queue->reserve()`没有设置超时时间，没有设置超时间的情况下，Beanstalkd会一直返回状态。
2. `$queue->put()` 没有设置任务允许执行时间。



生产消息时，`put` 方法有四个参数，其中
