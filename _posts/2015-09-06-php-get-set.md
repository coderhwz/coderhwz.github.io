---
layout: post
title: PHP 类中get set 数组属性操作
keyword: php,__set,__get,array,property
description: PHP 类中get set 数组属性操作
tags: 
    - PHP
---

自己写了一个`MongoL`的ODM库,近日要操作数组成员变,怎样操作都不能设置他的值.

其实道理也比较简单

```php

class BaseModel{

    protected $attributes;


    public function __get($key){
        return $this->$attributes[$key];
    }


    public function __set($key,$value){
        $this->attributes[$key] = $value;
    }
}


$model = new BaseModel();

$model->username = 'hwz';
$model->habbits = [
    'coding'
];
$model->save();


$model->habbits[] = 'basketball';
$model->save();

```

第二次的`$model->save()`是不起作用的,因为它并非使用`__set` 方法注入,`__set`只对当前的对象属性有效,对象属性内部是无效的.
解决这个问题我需要操作属性的"指针",就是引用,所以改造一下`__get`方法,方法前加上`&`符号,让他以引用的方式返回

```php

public function  &__get($key){
    return $this->attributes[$key];
}

```
