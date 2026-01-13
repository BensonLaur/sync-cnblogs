# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 仓库概述

这是一个**博客园文章同步仓库**，用于管理发布到博客园（cnblogs）的 Markdown 博客文章。通过博客园的 GitHub App（cnblogs-sync），仓库中的 Markdown 文件会自动同步到博客园。

**注意**：这不是代码项目，没有构建、测试或运行命令。

## 文章结构

### 目录组织
- 根目录 `README.md` - 同步教程文章
- `工具教程/` - 技术工具类文章
- `生活随笔/` - 生活感悟类文章
- `resource/` - 各目录下存放对应文章的图片资源

### 文件命名规范
```
序号-日期-标题.md
```
示例：`01-20241017-VMWare安装与拖动文件到Win7虚拟机.md`

### Markdown FrontMatter 格式
每篇文章开头必须包含：
```yaml
---
title: 博文标题
description: 博文摘要
tags: 标签1, 标签2
category: 分类1, 分类2
---
```

## 同步机制

- Git push 到 GitHub 后，博客园 App 自动检测变更并同步
- 新文件创建时默认为草稿状态（需在博客园后台手动发布）
- 文件删除时取消发布（保留评论数据）
