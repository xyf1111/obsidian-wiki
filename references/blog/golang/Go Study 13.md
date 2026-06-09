---
title: "Go Study 13"
date: 2021-02-21
tags: [golang]
source: "https://xyf1111.github.io/go-study-13/"
aliases:
  - "Go Study 13"
---

# Go Study 13

> 原文：[https://xyf1111.github.io/go-study-13/](https://xyf1111.github.io/go-study-13/)

## 测试


- testing.T的应用
- 运行测试


### 代码覆盖率


- 使用IDE查看代码覆盖
- 使用go test获取代码覆盖报告
- 使用go tool cover查看代码覆盖报告


### 性能测试


- testing.B的使用


### 使用pprof进行性能调优


1. 终端进入有测试文件的项目目录下
2. go test -bench . -cpuprofile cpu.out生成cpu.out
3. go tool pprof cpu.out查看cpu.out
4. 在交互式命令行输入web进行查看


## 案例