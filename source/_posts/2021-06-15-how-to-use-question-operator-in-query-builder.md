---
title: 怎样在QueryBuilder中使用PostgreSQL中的?操作符
date: 2021-06-15T21:51:54+09:00
author: epona
excerpt: 在我们使用Laravel的QueryBuilder中的whereRaw()的时候，? 会被当作Bindings来进行处理。但是在PostgreSQL中存在着许多?的操作符。本文将会介绍怎样处理?操作符。
layout: post
permalink: '/how-to-use-question-operator-in-query-builder/'
categories:
  - PROGRAMMING
tags:
  - Laravel
  - SQL
---

在我们使用Laravel的QueryBuilder中的`whereRaw()`的时候，`?` 会被当作Bindings来进行处理。但是在PostgreSQL中存在着许多`?`的操作符。本文将会介绍怎样处理`?`操作符。

## Laravel中whereRaw的基本使用

**这个章节中主要介绍whereRaw的基本使用方法**。`whereRaw`和`orWhereRaw`方法将原生的`where`注入到你的查询中。这两个方法的第二个参数还是可选项，值还是绑定参数的数组：

<pre><code class="PHP">$orders = DB::table('orders')
    -&gt;whereRaw('price &gt; IF(state = "TX", ?, 100)', [200])
    -&gt;get();
</code></pre>

通过示例代码可以看到`whereRaw`中的`?`方法是用来绑定参数的。

## PostgreSQL中?相关的使用。

**这个章节主要介绍PostgreSQL中有关`?`操作符的介绍。在PostgreSQL中的jsonb格式有很多操作符，其中有一个操作符是这样的。`?|`，表示左边的jsonb数据是否包含右边的jsonb数据中的任意一项。

    SELECT '{"a":1, "b":2, "c":3}'::jsonb ?| array['b', 'c']
    

## 在whereRaw中使用PostgreSQL的?操作符

很显然，我们直接在whereRaw中使用`?|`操作符是会报错的。

    Student::whereRaw("'{}'::jsonb ?| array[1, 2, 3]")->get();
    

结果:

    Illuminate/Database/QueryException with message 'SQLSTATE[42601]: Syntax error: 7 ERROR:  syntax error at or near "$1"
    LINE 1: select * from "students" where '{}'::jsonb $1| array[1, 2, 3...
                                                       ^ (SQL: select * from "students" where '{}'::jsonb ?| array[1, 2, 3])'
    

我们可以看到在执行查询的时候`?`已经被替换成binding了无法使用。

### 解决方案

我们可以使用原始定义的function来代替`?|`操作符。

    select oprname, oprcode from pg_operator where oprname = '?|';
    

结果如下：

<table>
  <colgroup> <col /> <col /> </colgroup> <tr>
    <th>
      oprname
    </th>
    
    <th>
      oprcode
    </th>
  </tr>
  
  <tr>
    <td>
      ?|
    </td>
    
    <td>
      point_vert
    </td>
  </tr>
  
  <tr>
    <td>
      ?|
    </td>
    
    <td>
      lseg_vertical
    </td>
  </tr>
  
  <tr>
    <td>
      ?|
    </td>
    
    <td>
      line_vertical
    </td>
  </tr>
  
  <tr>
    <td>
      ?|
    </td>
    
    <td>
      jsonb_exists_any
    </td>
  </tr>
  
  <tr>
    <td>
      我们可以看到可以将<code>?|</code>用<code>jsonb_exists_any</code>方法来替换。
    </td>
  </tr>
</table>

    Student::whereRaw("jsonb_exists_any('{}'::jsonb, array[1, 2, 3])")->get();
    

完美解决！