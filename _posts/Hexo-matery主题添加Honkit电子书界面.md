---
title: Hexo-matery主题添加Honkit电子书界面
date: 2023-04-19 15:09:27
img: https://i.gyazo.com/c08f3d4a4e07d7fc93ef2a5d3a41e27b.png
top: false
summary: Honkit是一款基于Node.js的静态网站生成器，可以将Markdown文件转换为漂亮的静态网站或电子书形式。本文将介绍如何在Hexo-matery主题中添加Honkit电子书界面。
categories: Hexo
tags:
- Honkit
- 静态网站
- 电子书
- Shell
- Node.js
- Git
---
### 一、写在前面
Honkit是一款基于Node.js的静态网站生成器，可以将Markdown文件转换为漂亮的静态网站或电子书形式。Honkit提供了多种主题和插件，并且具有自动化和灵活性强的特点，可以满足用户对文档撰写、发布和分享的需求，广泛应用于知识管理、技术文档等领域。

在Hexo-matery主题中添加Honkit电子书界面，可以让我们的博客同时拥有博客界面和电子书界面，这样既可以让我们的博客更加美观，又可以让我们的博客更加方便，让我们的博客更加有价值。 但是hexo只提供使用在hexo中使用Honkit主题的教程，而没有提供在Hexo-matery主题中添加Honkit电子书界面的教程。

作者在网上探索了好久，发现没有一篇教程能够让我在Hexo-matery主题中添加Honkit电子书界面，所以我决定自己写一篇教程，希望能够帮助到大家。


### 二、使用Honkit主题

如果你使用 Hexo 搭建博客，可以通过添加 Honkit 插件来支持 GitBook 风格的文档。下面是一些基本步骤：

1. 安装 hexo-plugin-honkit 插件：

   在命令行中执行以下命令：
    ```shell
    npm install hexo-plugin-honkit --save
    ```
2. 配置 _config.yml

   打开你的博客根目录下的 _config.yml 文件，找到 plugins 属性并将其值设置为 honkit：
    ```yaml
    plugins:
      - hexo-plugin-honkit
    ```
3. 添加 GitBook 风格的页面

   在 source 目录下创建一个名为 docs 的文件夹，并在其中添加 Markdown 格式的文档。目录结构应该类似于以下示例：
    ```shell
    - source
    - docs
        - getting-started.md
        - installation.md
        - ...
          生成静态文件并启动服务器
          在命令行中执行以下命令以生成静态文件：
    ```
4. 生成静态文件并启动服务器
   在命令行中执行以下命令以生成静态文件：
    ```shell
    hexo generate
    ```
   完成之后执行以下命令以启动服务器并预览更改：
    ```shell
    hexo server
    ```
   此时，可以通过访问本地服务器地址（默认为 [http://localhost:4000](http://localhost:4000)）来查看你的博客和 GitBook 风格的页面。

请注意，以上步骤不会影响你现有的博客主题和文章，只会添加 GitBook 风格的页面。

### 三、在Hexo-matery主题中添加Honkit电子书界面

作者在网上探索了好久，发现没有一篇教程能够让我在Hexo-matery主题中添加Honkit电子书界面，在多次的尝试下，最终也是曲线成功了，下面是详细的教程：

1. 安装Honkit

   在命令行中执行以下命令：
    ```shell
    npm init --yes
    npm install honkit --save-dev
    ```
   安装完成后，执行以下命令：
    ```shell
    npx honkit init
    ```

2. 编写电子书

   具体编写方式大家可以参考[官方文档](https://honkit.netlify.app/)，这里不再赘述。

3. 搭建Honkit仓库

   在GitHub上创建一个新的仓库，并将我们的电子书仓库推送到GitHub上。

   这里我没又找到合适的办法在`Hexo generate`的时候自动将电子书仓库推送到GitHub上，所以我只能手动推送了，如果有大佬知道怎么做的话，欢迎在评论区留言告诉我。

4. 添加Honkit电子书界面

   这里的做法其实很简单，就是在`Hexo generate`生成静态页面后将Honkit生成的静态文件放到Hexo-matery主题的`public`目录下，然后在`Hexo s`或者`Hexo d`的时候就可以将Honkit生成的静态展示到我们的博客上了。具体做法如下：

   编写`honkit-scripts.js`脚本：
   ```js
   'use strict';
   const fs = require('fs');
   const path = require('path');
   const {copyDir} = require('hexo-fs');
   
   // 指定需要复制的源目录和目标目录
   const srcPath = path.join(__dirname, 'Honkit/_book');
   const destPath = path.join(__dirname, 'public/book');
   
   const {exec} = require('child_process');
   
   
   // 复制目录
   try {
   // 执行 git 命令来拉取仓库的内容 username:你的GitHub用户名 proname:你的仓库名
   exec('git clone git@github.com:<username>/<pronanme>.git --sparse&&cd Honkit&&git sparse-checkout init --cone&&git sparse-checkout set _book', (error, stdout, stderr) => {
   if (error) {
   console.error(`Honkit clone 执行出错：${error}`);
   return;
   }
   console.log(`Honkit clone 执行成功：${stdout}`);
   if (fs.existsSync(srcPath)) {
   copyDir(srcPath, destPath, (err) => {
   if (err) {
   console.error(`Copy file failed!`);
   return;
   }
   });
   console.log(`Copy file successfully!`);
   // 删除临时文件
   // exec('rm-rf Honkit', (error, stdout, stderr) => {
   exec('rd/s/q Honkit', (error, stdout, stderr) => {
   if (error) {
   console.error(`Honkit文件夹删除出错：${error}`);
   return;
   }
   console.log(`Honkit文件夹删除成功：${stdout}`);
   });
   } else {
   console.log(`Source path '${srcPath}' does not exist!`);
   }
   });
   } catch (err) {
   console.error(err);
   }
   ```

4. 效果查看

   博客的首页
   ![Honkit_2023-04-19_15-44-16](https://wangxs020202.gitee.io/pbad/new/Honkit_2023-04-19_15-44-16.png)

   电子书的首页
   ![Honkit_2023-04-19_15-44-42](https://wangxs020202.gitee.io/pbad/new/Honkit_2023-04-19_15-44-42.png)

### 四、结尾

虽然我们并没有在hexo-matery主题中添加Honkit电子书界面部署，但是我们通过脚本的方式将Honkit生成的静态文件放到了Hexo-matery主题的public目录下，然后在`Hexo s`或者`Hexo d`的时候就可以将Honkit生成的静态展示到我们的博客上了。

这里的教程只是简单的介绍了如何在Hexo-matery主题中添加Honkit电子书界面，如果大家有什么问题或者建议，欢迎在评论区留言，我会尽快回复大家的。
