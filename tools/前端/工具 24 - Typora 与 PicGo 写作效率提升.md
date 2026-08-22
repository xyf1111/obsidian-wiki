---
title: "工具 24 - Typora 与 PicGo 写作效率提升"
date: 2026-08-22
tags: [Typora, PicGo, 图床, Markdown, 七牛云, 写作工具]
source: "鱼皮·编程导航 / codefather"
---

# 工具 24 - Typora 与 PicGo 写作效率提升

> Typora 所见即所得 Markdown 编辑器 + PicGo 图床工具 + 七牛云对象存储，实现文章中插图自动上传，免去逐张手动传图，写作发布效率大幅提升。

## 痛点：本地图片无法直接发布

- 在 Typora 中写的文章要发布到 B 站、知乎等平台，**本地图片没有 URL，无法直接复制粘贴**，只能一张张通过平台插入功能上传，非常麻烦
- 已上传到网络的图片则可以愉快地直接复制粘贴，平台会自动保存

## Markdown 与 Typora

- **Markdown**：轻量级标记语言，用易读易写的纯文本格式编写文档，如 `**加粗**` 两对星号实现加粗
- **Typora**：简洁的 Markdown 文本编辑器，自动利用 Markdown 统一排版，无需专门学习语法。官网直接下载安装即可：https://www.typora.io/

## PicGo + 七牛云：图片自动上传

**PicGo** 是图床工具，能自动把本地图片上传到网络并转换为可访问的链接；在 Typora 中配合 PicGo，插入图片时即可自动上传。

> **图床** = 存储上传图片的服务器（云存储空间）。

### 1. 下载安装 PicGo

- 下载地址：https://github.com/Molunerfinn/PicGo/releases
- 按操作系统（Win/Linux/Mac）选择安装包，安装后需先配置图床

### 2. 七牛图床配置

1. 打开七牛云官网申请存储空间（对象存储）：https://portal.qiniu.com/create ，先身份认证再添加对象存储
2. 创建存储空间：地域选离自己近的，**空间设为公开**（保证图片可公开访问）
3. 将**存储空间名称**与**存储区域代码**填入 PicGo 图床设置（华东 z0 / 华北 z1 / 华南 z2 / 北美 z3 / 东南亚 z4）
4. 复制空间临时访问域名（30 天有效）填入 PicGo 的「设定访问网址」
5. 从七牛云右上角头像进入密钥管理（https://portal.qiniu.com/user/key ），将 AccessKey 和 SecretKey 填入 PicGo 图床设置

### 3. Typora 开启图片自动上传

打开 Typora 偏好设置，按图配置（上传服务选择 PicGo）后，在文章中插入图片即自动上传得到 URL。

> 来源：鱼皮·编程导航 / codefather
