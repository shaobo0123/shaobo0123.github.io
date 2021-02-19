---
title: Hexo 使用 GitHub Actions 自动部署
---

# 1. 创建仓库


GitHub 博客创建步骤非本文重点，请自行搜索。
推荐使用 `master`/`main` 分支作为最终部署分支。
> 注意你的主分支是main还是master，若为main，则替换master为main，现在新建仓库主分支都为main，后续一定要注意

这里的关键是要创**另一个分支**，我这里创建的是 `hexo`。


# 2. 生成公私钥


源码分支中通过下面命令生成公钥和私钥。
```
 ~ ssh-keygen -t rsa -b 4096 -C "$(git config user.email)" -f github-deploy-key -N ""
```
> 若无源码分支可查看文章末尾



目录中生成两个文件：


- `github-deploy-key.pub` --- 公钥文件
- `github-deploy-key` --- 私钥文件



> 文件在你的本地文件夹生成



# 3.GitHub 添加公钥


在 GitHub 中博客工程中按照 `Settings`->`Deploye keys`->`Add deploy key` 找到对应的页面，然后进行公钥添加。该页面中 `Title` 自定义即可，`Key` 中添加 `github-deploy-key.pub` 文件中的内容。


![](https://segmentfault.com/img/remote/1460000022360772#align=left&display=inline&height=445&margin=%5Bobject%20Object%5D&originHeight=445&originWidth=800&status=done&style=none&width=800)


> 注意：切记不要多复制空格!!!
切记要勾选 `Allow write access`，否则会出现无法部署的情况。



# 4.GitHub 添加私钥


在 GitHub 中博客工程中按照 `Settings`->`Secrets`->`Add a new secrets` 找到对应的页面，然后进行私钥添加。该页面中 `Name` 自定义即可，`Value` 中添加 `github-deploy-key` 文件中的内容。


![](https://segmentfault.com/img/remote/1460000022360773#align=left&display=inline&height=563&margin=%5Bobject%20Object%5D&originHeight=563&originWidth=800&status=done&style=none&width=800)


> 注意：切记不要多复制空格!!!



# 5. 创建workflows脚本


在博客源码分支 (我这里是 hexo 分支) 中创建 `.github/workflows/HexoCI.yml` 文件，内容如下：


```
name: CI
on:
  push:
    branches:
      - hexo
jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout source
        uses: actions/checkout@v1
        with:
          ref: hexo
      - name: Use Node.js ${{ matrix.node_version }}
        uses: actions/setup-node@v1
        with:
          version: ${{ matrix.node_version }}
      - name: Setup hexo
        env:
          ACTION_DEPLOY_KEY: ${{ secrets.HEXO_DEPLOY_PRI }}
        run: |
          mkdir -p ~/.ssh/
          echo "$ACTION_DEPLOY_KEY" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan github.com >> ~/.ssh/known_hosts
          npm install --save hexo-deployer-git
          git config --global user.email "123456789@qq.com"
          git config --global user.name "yourname"
          npm install hexo-cli -g
          npm install
      - name: Hexo deploy
        run: |
          hexo clean
          hexo d
```


> 注意：
> 代码第5行的hexo为你的分支名称；
> git config --global user.email "123456789@qq.com"  中的`123456789@qq.com`改为自己的邮箱；
> git config --global user.name "yourname"中的 `yourname`改为自己的用户名；
> `secrets.HEXO_DEPLOY_KEY` 就是对应我们之前设置的私钥，所以名字一定不要搞错。

# 6`_config.yml`文件配置


在项目**根目录**中修改 `_config.yml` ，**末尾**增加如下内容：


```
deploy:
  type: 'git'
  repo: git@github.com:yourname/yourname.github.io.git
  branch: master
```


> 再次注意你的主分支是`main`还是`master`，若为`main`，则替换`master`为`main；`
> yourname替换为自己的用户名；
> 每个 :(冒号)后面都有一个空格(英文在状态下)
> 这里的 repo 要填写 ssh 的形式，使用 http 形式可能会有问题。



# 7. 验证


现在 Hexo 已经和 GitHub Actions 已经集成了，接下来在博客源码分支上推送代码即可自动编译部署。具体
执行过程可以在 Actions 中查看：


![](https://segmentfault.com/img/remote/1460000022360774#align=left&display=inline&height=373&margin=%5Bobject%20Object%5D&originHeight=373&originWidth=800&status=done&style=none&width=800)




# 8. 部署完后如何写代码


若更换了设备，或重装了系统，在配置环境十分困难，这里只需要将hexo分支代码下载到本地，修改，上传，自动部署到master/main分支。
或者在GitHub中直接修改也可以自动部署。


#### 如何将代码下载到本地
```
git init  //初始化git仓库
git remote add origin https://github.com/yourname/yourname.github.io.git  //然后连接至远程仓库
git pull origin hexo  //下载到本地
```
> yourname替换为你的用户名

#### 上传到远程仓库
```
git add .
git commit -m "message"
git push -u origin master:hexo  //上传到远程仓库，冒号左边为本地分支名称，冒号右边为要部署到的远程分支名
```
> 注意：origin名称可随意， hexo为分支名称



本文引用自[https://segmentfault.com/a/1190000022360769](https://segmentfault.com/a/1190000022360769)
