---
title: Leetcode —— two_sum
date: 2021-02-25T22:37:46+09:00
author: epona
excerpt: |
  <!-- wp:paragraph -->
  <p>Given an array of integers&nbsp;<code>nums</code>&nbsp;and an integer&nbsp;<code>target</code>, return&nbsp;<em>indices of the two numbers such that they add up to&nbsp;<code>target</code></em>.</p>
  <!-- /wp:paragraph -->
  
  <!-- wp:paragraph -->
  <p>You may assume that each input would have&nbsp;<strong><em>exactly</em>&nbsp;one solution</strong>, and you may not use the&nbsp;<em>same</em>&nbsp;element twice.</p>
  <!-- /wp:paragraph -->
  
  <!-- wp:paragraph -->
  <p>You can return the answer in any order.</p>
  <!-- /wp:paragraph -->
layout: post
permalink: /leetcode-two_sum/
categories:
  - LEETCODE
  - PROGRAMMING
tags:
  - easy
  - leetcode
---
 

## 问题描述

Given an array of integers&nbsp;`nums`&nbsp;and an integer&nbsp;`target`, return&nbsp;_indices of the two numbers such that they add up to&nbsp;`target`_.

You may assume that each input would have&nbsp;**_exactly_&nbsp;one solution**, and you may not use the&nbsp;_same_&nbsp;element twice.

You can return the answer in any order.

**Example 1:**

<pre class="wp-block-code"><code>&lt;strong>Input:&lt;/strong> nums = &#91;2,7,11,15], target = 9
&lt;strong>Output:&lt;/strong> &#91;0,1]
&lt;strong>Output:&lt;/strong> Because nums&#91;0] + nums&#91;1] == 9, we return &#91;0, 1].</code></pre>

**Example 2:**

<pre class="wp-block-code"><code>&lt;strong>Input:&lt;/strong> nums = &#91;3,2,4], target = 6
&lt;strong>Output:&lt;/strong> &#91;1,2]</code></pre>

**Example 3:**

<pre class="wp-block-code"><code>&lt;strong>Input:&lt;/strong> nums = &#91;3,3], target = 6
&lt;strong>Output:&lt;/strong> &#91;0,1]</code></pre>

**Constraints:**

  * `2 <= nums.length <= 10<sup>3</sup>`
  * `-10<sup>9</sup> <= nums[i] <= 10<sup>9</sup>`
  * `-10<sup>9</sup> <= target <= 10<sup>9</sup>`
  * **Only one valid answer exists.**

## 自己写的暴力解法

<pre class="wp-block-code"><code>def two_sum(nums, target)
  for i in 0...nums.size
    for j in (i+1)...nums.size
      if nums&#91;i] + nums&#91;j] == target
        puts i, j
        return &#91;i, j]
      end
    end
  end
end</code></pre>

虽然能用，但是不优雅

## 别人家的优雅解法

<pre class="wp-block-code"><code>def two_sum(nums, target)
  map = {}

  nums.each_with_index { |val, index|
    n = target - val
    if map.include?(n)
      return &#91;map&#91;n], index]
    else
        map&#91;val] = index
    end
  }
end</code></pre>