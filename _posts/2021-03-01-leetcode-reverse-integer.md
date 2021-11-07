---
id: 126
title: Leetcode —— Reverse Integer
date: 2021-03-01T22:17:42+09:00
author: epona
excerpt: 'Given a signed 32-bit integer x, return x with its digits reversed. If reversing x causes the value to go outside the signed 32-bit integer range [-231, 231 - 1], then return 0.'
layout: post
guid: https://wp.epona.me/?p=126
permalink: /2021/03/01/leetcode-reverse-integer/
image: /wp-content/uploads/2021/02/LeetCode_Sharing.png
categories:
  - LEETCODE
tags:
  - easy
  - leetcode
---
### 问题描述

Given a signed 32-bit integer `x`, return `x` with its digits reversed. If reversing `x` causes the value to go outside the signed 32-bit integer range `[-231, 231 - 1]`, then return ``.

**Assume the environment does not allow you to store 64-bit integers (signed or unsigned).**

#### Example 1:

    Input: x = 123
    Output: 321
    

#### Example 2:

    Input: x = -123
    Output: -321
    

#### Example 3:

    Input: x = 120
    Output: 21
    

#### Example 4:

    Input: x = 0
    Output: 0
    

### 自己写的解法

    def reverse(x)
    
      return 0 if x == 0
    
      is_negative = x < 0
    
      a = x.to_s.reverse!.to_i
    
      a = -a if is_negative
    
      return 0 if a > (2**31 - 1) || a < (-2**31)
    
      a
    end
    

将`integer`转成`string`然后用`reverse`方法即可

### 别人家的解法

    def reverse(x)
            if x.negative?
            x = (x.to_s.reverse!.to_i * -1)
        else
            x = x.to_s.reverse!.to_i
        end
    
        x.bit_length > 31 ? 0 : x
    end
    

别人写的更加简洁，并且使用到了`bit_length`，`negative?`等内置方法