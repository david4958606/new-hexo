---
title: 重构博客
date: 2024-10-27 15:09:00
tags: [nodejs, 日志]
description: 重构博客
---

## 2024/10/27 更新

- 原本的 GitHub Action Workflow 不工作了，fork 了一份并做了对应的修改，具体可看 [david4958606/hexo-action](https://github.com/david4958606/hexo-action)；
- 修正一些拼写错误；
- 修改url。

## 为什么要重构

笔者之前的博客是在一台腾讯云学生机上部署了宝塔面板和 WordPress，这俩结合起来的优点一是部署方式傻瓜，二是在 WordPress 上写作和管理都有比较完备的 WebUI，非常方便。但这个方案的缺点也很明显，就是过于臃肿了，而且 WordPress 在国内的网络环境下也很难用，主题和插件安装都很慢。所以在经历了一次删库风波后，我就暂时停用了这个博客。

这次重新开始写博客，主要是响应 NetUnion 制作主页和期刊的号召，也是想着把之前落下的写作习惯找补回来。

## 关于博客系统的选择

现在比较流行的搭建个人博客的方式主要有两种，一是我上文提到的在服务器上或者利用 SaaS 部署类似于 WordPress 这类基于 LAMP 的博客，第二种就是生成静态页面并直接显示（比如利用 GitHub Pages）的模式，这种方式的主要代表就是 Hexo。

考虑到笔者的博客以静态页面为主且服务器资源较少，笔者采用了 Hexo + GitHub Pages 的方案，同时将 Hexo 的运行环境也上传至 GitHub，使用 GitHub Action 生成并部署页面。

## 重构流程

### 安装 nodejs 和 npm

Windows 用户直接在官网下载安装包后一路 Next 就好，安装程序会自动将安装目录添加至 Path 环境变量；Linux 用户直接使用包管理器即可。

安装好后，新建文件夹 `hexo-env` 这是我们本地的 hexo 环境所属的文件夹。

输入下列命令，下载并构建环境

```powershell
cd hexo-env
npm install hexo-cli -g
hexo init .
npm install
```

等待进度条走完，我们的本地环境就搭建好了。

### 上传 Git

随后，需要将本地环境上传到 GitHub。在 GitHub 新建仓库 `hexo-env` 然后在本地执行下列命令。

```bash
git init
git remote add origin <git url>
git branch -M master
git add .
git commit
git push -u origin master
```

### 编辑配置文件并更换主题

Hexo 的配置文件是根目录的 `_config.yml`，读者可以按需对他进行修改，具体的配置内容请参考 [官方文档](https://hexo.io/zh-cn/docs/configuration)

我选择了 Next 作为主题，并把它作为一个子模块导入我们的 Git 仓库中。

在本地执行命令 `git submodule add https://github.com/next-theme/hexo-theme-next.git theme/next`

在根目录的 `_config.yml` 中修改 `theme: next`，并执行如下命令

```bash
hexo clean
hexo g
hexo s
```

如果一切正常的话，终端的输出中就会显示 Next 的 Logo 了。访问 `http://localhost:4000` 查看主题效果。

如果需要修改 Next 的配置，不能直接修改子模块中的内容，而需要在根目录下新建 `_config.next.yml` 文件并将 `theme/next/_config.yml` 中内容完整复制到其中，以后所有的配置修改需要在该文件中进行。

### 使用 GitHub Action 自动构建并部署到 GitHub Pages

首先新建 GitHub 仓库，命名为 `<username>.github.io`，并生成 README。同时启用 GitHub Pages。

准备一对 SSH 密钥，将公钥作为 Deploy Key 添加到 Pages 仓库，将私钥作为 Secret 添加到 `hexo-env` 并命名为 `HEXO_DEPLOY_PRI`。

在 Hexo 文件夹中安装插件 `npm install hexo-deployer-git --save`，并按照 [官方文档](https://hexo.io/zh-cn/docs/one-command-deployment.html#Git) 修改配置。

在根目录中新建 `.github\workflows\main.yml` 并写入以下内容（参考：[sma11black/hexo-action](https://github.com/sma11black/hexo-action)）

```yaml
name: Deploy

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    name: A job to deploy blog.
    steps:
    - name: Checkout
      uses: actions/checkout@v1
      with:
        submodules: true # Checkout private submodules(themes or something else).
    
    # Caching dependencies to speed up workflows. (GitHub will remove any cache entries that have not been accessed in over 7 days.)
    - name: Cache node modules
      uses: actions/cache@v4
      id: cache
      with:
        path: node_modules
        key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
        restore-keys: |
          ${{ runner.os }}-node-
    - name: Install Dependencies
      if: steps.cache.outputs.cache-hit != 'true'
      run: npm ci
    
    # Deploy hexo blog website.
    - name: Deploy
      id: deploy
      uses: david4958606/hexo-action@v1
      with:
        deploy_key: ${{ secrets.HEXO_DEPLOY_PRI }}
        # user_name: your github username  # (or delete this input setting to use bot account)
        # user_email: your github useremail  # (or delete this input setting to use bot account)
        commit_msg: ${{ github.event.head_commit.message }}  # (or delete this input setting to use hexo default settings)
    # Use the output from the `deploy` step(use for test action)
    - name: Get the output
      run: |
        echo "${{ steps.deploy.outputs.notify }}"
```

这样，每次将修改 push 到 `hexo-env` 时就会自动生成静态页面并部署到 Pages 仓库了。

## 从 Git 上获取并在本地重构 Hexo 环境并开始写作

如果只需要在不同工作环境中完成写作任务，首先克隆 `hexo-env` 仓库，并执行命令

```bash
npm install
hexo clean
```

要开始写作，执行 `hexo new post "<tile of your post>"` 后，会在 `source\_posts` 中生成 Markdown 文件，正常写作即可。

如果需要在另外的工作环境中本地预览页面，还需要导入对应的子模块。首先克隆仓库到本地，然后执行

```bash
cd themes/next
git submodule init
git submodule update
```

对于其他的子模块同理。
