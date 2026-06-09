---
title: "Go Study 06"
date: 2021-02-16
tags: [golang]
source: "https://xyf1111.github.io/go-study-06/"
aliases:
  - "Go Study 06"
---

# Go Study 06

> 原文：[https://xyf1111.github.io/go-study-06/](https://xyf1111.github.io/go-study-06/)

Go Study 06-rune字符串

rune相当于go的char

使用range遍历pos，rune对 s := "Yes我是你爸爸!" for _, b := range []byte(s) { fmt.Printf("%X ", b) } fmt.Println() for i, ch := range s { fmt.Printf("(%d %X) ", i, ch) }

使用utf8.RuneCountInString获得字符数量 fmt.Println("Rune count:", utf8.RuneCountInString(s))

使用len获得字节长度 fmt.Println(len(s))

使用[]byte获得字节 bytes := []byte(s) for len(bytes) > 0 { ch, size := utf8.DecodeRune(bytes) bytes = bytes[size:] fmt.Printf("%c ", ch) }

其他字符串操作

Fields，Split，Join

Contains，Index

ToLower，ToUpper

Trim，TrimRight，TrimLeft
