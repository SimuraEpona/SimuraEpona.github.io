---
title: 在PHP中生成日期数组
date: 2020-03-12 22:21:04
tags: [PHP]
---

最近在工作中碰到了这样一个需求，根据开始日期和结束日期生成一个按天计的数组，虽然我们可以通过`foreach`办到，但是有没有什么更加优雅的方法呢？经过一番搜索之后，找到了这样的一个类——[`DatePeriod`](https://www.php.net/manual/zh/class.dateperiod.php)。

`DatePeriod`的简介如下：

>DatePeriod 类表示一个时间周期。
>一个时间周期可以用来在给定的一段时间之内， 以一定的时间间隔进行迭代。

<!--more-->

## 一般的使用方法

假设我们的开始日期为`2020-03-01`，结束日期为`2020-03-10`。那么我们可以用下面的代码来生成一个时间周期

```php
$period = new DatePeriod(
     new DateTime('2020-03-01'),
     new DateInterval('P1D'),
     new DateTime('2020-03-11') //最后一天的日期不会被包含，所以要加1天
);

// 遍历$perid即可获得每天的日期
foreach ($period as $key => $value) {
    $value->format('Y-m-d')       
}

// 当然我们也可以

```

## 使用CarbonPeriod

在Laravel中默认引入了Carbon这个类，那么我们可以使用`CarbonPeriod`类来更加方便可读的生成我们需要的内容。

```php
use Carbon\CarbonPeriod;

$period = new CarbonPeriod('2020-03-01', '1 day', '2020-03-10');


foreach ($period as $key => $value) {
    echo $value->format('Y-m-d').PHP_EOL;
}
```

当然，我们有另外的一种写法

```
$period = Carbon::parse('2020-03-01')->daysUntil('2020-03-10');
```

其他的写法我们可以在[Carbon 的文档](https://carbon.nesbot.com/docs/#api-period)中找到。

PS：其实Carbon的写法是我在写这篇文章的时候才发现的😂，于是明天把已经写好的代码换成Carbon写法。
