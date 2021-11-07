---
title: Leetcode —— Palindrome Number
date: 2021-03-07T20:50:48+09:00
author: epona
excerpt: |
  Given an integer x, return true if x is palindrome integer.
  
  An integer is a palindrome when it reads the same backward as forward. For example, 121 is palindrome while 123 is not.
layout: post
permalink: /leetcode-palindrome-number/
categories:
  - LEETCODE
  - PROGRAMMING
tags:
  - easy
  - leetcode
  - palindrome number
---
### 问题描述

Given an integer x, return true if x is palindrome integer.

An integer is a palindrome when it reads the same backward as forward. For example, 121 is palindrome while 123 is not.

#### Example 1:

    Input: x = 121
    Output: true
    

#### Example 2:

    Input: x = -121
    Output: false
    Explanation: From left to right, it reads -121. From right to left, it becomes 121-. Therefore it is not a palindrome.
    

#### Example 3:

    Input: x = 10
    Output: false
    Explanation: Reads 01 from right to left. Therefore it is not a palindrome.
    

#### Example 4:

    Input: x = -101
    Output: false
    

### 自己写的解法

    # @param {Integer} x
    # @return {Boolean}
    def is_palindrome(x)
        x = x.to_s
        
        x.reverse == x
    end
    

用 Ruby 内置的 String 的 reverse 可以很方便的实现回文的判断