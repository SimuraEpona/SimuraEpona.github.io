---
title: 使用 chunkById 方法的时候请不要进行排序
date: 2020-10-31T13:42:59+09:00
author: epona
layout: post
permalink: '/do-not-use-sort-in-chunk-by-id/'
categories:
  - PROGRAMMING
---
# 使用 chunkById 方法的时候请不要进行排序

> 最近在做开发任务的时候碰到了个诡异的问题，于是分享给大家

## 问题说明

由于需要批量处理数据，并且这个数据的量很大，一次全部取出然后执行是不现实的，幸运的是 Laravel 为我们提供了 `chunkById` 方法来让我们方便的处理。伪代码如下

<pre><code class="php">Student::query()
    -&gt;where('is_delete', false)
    -&gt;orderBy('id', 'DESC')
    -&gt;chunkById(200, function($students) {
            // 在这里进行逻辑处理
    });
</code></pre>

咋一眼看上去，并没有什么问题，但是实际执行代码的时候会发现chunkById只会执行第一次，第二次以后由于某种原因会停止执行。

## 查找原因

Laravel的源码中 `chunkById` 代码如下

<pre><code class="php">    public function chunkById($count, callable $callback, $column = null, $alias = null)
    {
        $column = is_null($column) ? $this-&gt;getModel()-&gt;getKeyName() : $column;

        $alias = is_null($alias) ? $column : $alias;

        $lastId = null;

        do {
            $clone = clone $this;


            $results = $clone-&gt;forPageAfterId($count, $lastId, $column)-&gt;get();

            $countResults = $results-&gt;count();

            if ($countResults == 0) {
                break;
            }


            if ($callback($results) === false) {
                return false;
            }

            $lastId = $results-&gt;last()-&gt;{$alias};

            unset($results);


        } while ($countResults == $count);

        return true;
    }
</code></pre>

看起来没什么问题，由于while 循环是根据 `$countResults == $count` 来判断的，那么我们 dump 一下这两个变量就会发现， 第一次这两个是一致的，第二次由于数据不一致导致程序停止。

在上面的代码中， `$count` 是由`$results = $clone->forPageAfterId($count, $lastId, $column)->get();` 来获得的，

继续查看 `forPageAfterId`方法

    public function forPageAfterId($perPage = 15, $lastId = 0, $column = 'id')
    {
        $this->orders = $this->removeExistingOrdersFor($column);
    
        if (! is_null($lastId)) {
            $this->where($column, '>', $lastId);
        }
    
        return $this->orderBy($column, 'asc')
                    ->take($perPage);
    }
    

我们可以看到，在这里返回的结果是 `orderBy` 进行升序排列的， 而我们的原始代码是进行降序排列，就会导致count不一致，从而使`chunkById`结束执行。

## 解决方案

把之前的`orderBy('id', 'desc')`移除即可。

    Student::query()
        ->where('is_delete', false)
        ->chunkById(200, function($students) {
                // 在这里进行逻辑处理
        });
    

## 总结

  * 以后使用 `chunkById` 或者 `chunk` 方法的时候不要添加自定义的排序
  * 骚到老，学到老。。。