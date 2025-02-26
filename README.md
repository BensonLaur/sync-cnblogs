---
title: 使用 GitHub 跟踪博文数据，并自动同步到 cnblogs
description: 本文将说明如何使用 GitHub 跟踪博文数据并自动同步到 cnblogs 的方法，并做相关的个人使用心得分享
#多个标签请使用英文逗号分隔或使用数组语法
tags: GitHub, cnblogs, 跟踪博文, 版本控制
#多个分类请使用英文逗号分隔或使用数组语法，暂不支持多级分类
category: 版本控制, 教程
---

## 导读
开通了博客园的 VIP，发现了会员服务中的 GitHub 跟踪和同步博文数据的功能，测试发现习惯使用 GitHub 的话，这个还是一个挺不错的功能。

本文主要分两部分，一是介绍说明如何使用该功能，并做相关个人的经验分享，二是作为个人 GitHub 同步 cnblogs 的文章的索引

## 教程：如何使用 

### 适合人群

1. 想要使用博客园 cnblogs 来发布技术博客或其他随笔文章
2. 想要对博文数据进行版本控制（记录和查看文章历史）
3. 熟练使用 GitHub 基本功能，或不排斥学习和使用 GithHub （需要学习：版本控制软件 git，md 文件格式等）
4. 有意愿开通博客园 VIP 会员（以使用其会员服务提供的 GitHub App 同步博文功能）

### 教程步骤

1. 确保**开通博客园会员**：「 [会员购买链接](https://cnblogs.vip) 」相关拓展背景 「 [求救信：救下园子，保住这块开发者的天地](https://www.cnblogs.com/cmt/p/18302049) 」
2. 确保**能够登录GitHub** 「[GitHub是一个面向开源及私有软件项目的托管平台](https://github.com/)」
3. 查看博客园官方博客提供的 「 [GitHub 同步功能使用帮助](https://www.cnblogs.com/cmt/p/17648901.html) 」，**逐步完成**：

   A. 在 cnblogs 的 [账户中心](https://account.cnblogs.com/settings/account) 绑定 GitHub 账号
   
     <img src="resource\教程步骤-3-A-绑定Github账号.png" alt="绑定Github账号" width="500">
    
   B. 在博客后台，根据指引安装 `GitHub App（博客园提供的 cnblogs-sync）`，完成 Github 和 博客园的关联

   在博客后台-同步选项卡下，点击「`添加 GitHub 源`」按钮，弹出创建 GitHub 同步源对话框，点击 「`GitHub 账户`」右侧的数据框，点击「`安装 GitHub App`」按钮
    
   <img src="resource\教程步骤-3-B-添加GitHub源.png" alt="添加GitHub源" width="500">
   <br/>
   <img src="resource\教程步骤-3-B-点击安装GitHubApp.png" alt="点击安装GitHubApp" width="400">
   
   览器将自动跳转到 GitHub（如果您的浏览器提示「是否允许新建窗口」，请选择允许），请在跳转后的页面选择您希望同步的 repo 所属账号，然后点击「`Install`」

   <img src="resource\教程步骤-3-B-安装GitHubApp.png" alt="安装GitHubApp" width="400">

   安装成功后，页面上会出现安装成功的提示：

   <img src="resource\教程步骤-3-B-GitHubApp安装成功效果.png" alt="GitHubApp安装成功效果" width="400">

   C. 回到博客后台，账号列表应该会出现您刚刚绑定的账号（如果没有，请刷新页面重试，多次尝试仍然没有的话，请参见[使用帮助文章](https://www.cnblogs.com/cmt/p/17648901.html)后面的「*重新绑定 GitHub 来源*」一节）

   <img src="resource\教程步骤-3-C-选择账号和仓库.png" alt="选择账号和仓库" width="400">

   D. 选择同步行为 “偏好” 和 "**启用 Markdown FrontMatter**"

   <img src="resource\教程步骤-3-D-同步偏好和FrontMatter.png" alt="同步偏好和FrontMatter" width="400">

### 几个使用心得

1. **创建 GitHub 同步源** 时，“`文件夹`” 是指跟踪 git 仓库中的哪个文件夹的 md 文件的变化，比如想跟踪`整个仓库的 md 文件`，那么就写根目录 “`/`”; 如果只跟踪 git 仓库的文件夹下 `/docs`，那么就填写 “`/docs`”

       这里根据个人文章规划情况，先决定是要同步整个目录，还是只同步具体子目录。比如我这里，README.md 我也要发布，同时想根据不同的类型的文章，在根目录下分不同的子目录来写 md 文件，所以我选择填写 “/”

2. **“当新文件创建时”**：可选项有 “`无操作`”、“`新建草稿`”、“`新建随笔并发布`”。个人建议使用 “`新建草稿`”

       “无操作” ：不做任何动作，感觉选这个那么同步就失去了意义。可能一些特殊情况才会用到。

       “新建草稿” ：个人推荐选项，因为这样可以不至于一创建文件，一在 git 提交 commit 就立刻发布了文章。可以分多次提交，当作临时保存而不至于立刻发布。而且，选择该选项后，当在博客园发布了草稿，那么下次修改文件 commit 之后，也会保持文章的 “已发布” 状态，不会重新变回 “未发布” 的草稿状态。

       “新建随笔并发布” ：这个一创建文件就发布了，只适合一次性就写完文章，这个可能不太适合我。我一般写博客习惯会分多次编辑修订细节之后，再正式发布。
    

3. **“当源文件被删除时”**：可选项有 “`无操作`”、“`取消发布`”、“`删除博文`”。个人建议使用 “`取消发布`”

       “无操作” ：不做任何动作，感觉选这个那么同步就失去了意义。可能一些特殊情况才会用到。

       “取消发布” ：个人推荐选项，因为文章如果有评论的话，直接 “删除博文” 会导致评论永久性丢失。

       “删除博文” ：如文字所述，删除 md 文件将直接将所有数据包括评论全部删除。

4. **"启用 Markdown FrontMatter"**, 这个可以根据定制的模板，自动让 cnblogs 确认 “`文章标题`”、 “`博客预览摘要`”、“`使用的标签`” 和 “`使用的文章分类`”

   启用之后，按默认格式，只要在 md 文章开头写下如下内容，即可自定义上面提到的各个项

    `````
    ---
    title: 博文标题
    description: 博文摘要
    #多个标签请使用英文逗号分隔或使用数组语法
    tags: 标签1, 标签2
    #多个分类请使用英文逗号分隔或使用数组语法，暂不支持多级分类
    category: 分类1, 分类2
    ---

    正文内容
    `````

    具体使用示例参考 [[个人文章列表-1](#个人文章列表-1)]，本 cnblogs 博文就是通过 GitHub 编写的 md 文件自动生成

5. **本地 Markdown 工具**

    在 GitHub 网页上也可以直接写文章，不过 “**切换文件时的响应速度**” 以及 “**Markdown 效果实时预览**” 可能不是很理想。

    下面这篇文章列出了一些可用的本地 PC 端 Markdown 编辑工具的推荐

    *[Markdown 教程-Markdown编辑工具推荐 - 阿鬼学长的文章 - 知乎](https://zhuanlan.zhihu.com/p/672257191)*

    由于我本人电脑已经安装了 “`VS Code`”，它提供了对 Markdown 的原生支持，所以直接使用它，目前使用起来感觉十分方便。

    **理由主要有二：**

    一、完善的 git 功能的原生支持，可以十分方便地进行 git 所有相关操作（提交，同步等）

   <img src="resource\几个使用心得-5-VsCode-git-支持.png" alt="安装GitHubApp" width="400">

    二、不错的实时预览效果。按快捷键 `Ctrl+K` + `V` 即可在右侧显示实时预览效果

   <img src="resource\几个使用心得-5-VsCode-md实时预览.png" alt="安装GitHubApp" width="400">

## 个人文章列表

<div id="个人文章列表-1"></div>

1. [使用 GitHub 跟踪博文数据，并自动同步到 cnblogs - GitHub:sync-cnblogs/README.md](https://github.com/BensonLaur/sync-cnblogs/blob/main/README.md) 

2. [可怕的甲醛 - GitHub:生活随笔/01-20240801-可怕的甲醛.md](https://github.com/BensonLaur/sync-cnblogs/blob/main/%E7%94%9F%E6%B4%BB%E9%9A%8F%E7%AC%94/01-20240801-%E5%8F%AF%E6%80%95%E7%9A%84%E7%94%B2%E9%86%9B.md)

3. [VMWare安装与拖动文件到虚拟机 - GitHub:工具教程/01-20241017-VMWare安装与拖动文件到Win7虚拟机.md](https://github.com/BensonLaur/sync-cnblogs/blob/main/%E5%B7%A5%E5%85%B7%E6%95%99%E7%A8%8B/01-20241017-VMWare%E5%AE%89%E8%A3%85%E4%B8%8E%E6%8B%96%E5%8A%A8%E6%96%87%E4%BB%B6%E5%88%B0Win7%E8%99%9A%E6%8B%9F%E6%9C%BA.md)

## 参考文章

* [博客园同步GitHub功能 - xbotter](https://www.cnblogs.com/xbotter/p/17522977.html)

-------------------------

*本文源地址：https://www.cnblogs.com/BensonLaur/p/18306067*

  
