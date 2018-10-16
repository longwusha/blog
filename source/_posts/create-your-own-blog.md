---
title: 搭建并使用属于自己的博客
date: 2018-10-16 23:51:28
tags: [Hexo, Blog, Deepin Linux]
categories: [操作记录]
---

博客主题：Aria
博客搭建工具：Hexo
博客发布平台：Github Pages
系统信息：Deepin Linux 15.7, 64位

<!--more-->

## 环境搭建

### Node.js安装

- [官网](https://nodejs.org/en/)下载最新稳定版安装包

- 解压到安装文件夹(eg: /tools/node)

- 添加node及全局包安装路径到PATH，全局包安装路径可通过命令 npm config set prefix 修改
``` bash
export PATH="/tools/node/bin:$PATH"
export PATH="/tools/node/global/npm/bin:$PATH"
```

- 配置国内镜像，加快包安装速度
``` bash
npm config set registry https://registry.npm.taobao.org
```

- （可选）安装yarn并添加yarn全局包安装路径到PATH，全局包安装路径可通过命令 yarn config set prefix 修改
``` bash
npm install -g yarn
export PATH="/tools/node/global/yarn/bin:$PATH"
```

### Git安装

- 安装git
``` bash
sudo apt install git
```

- 配置git全局用户名与邮箱
``` bash
git config --global user.name "username"
git config --global user.email "yourmail@emailhost.com"
```

### Hexo安装

- 使用npm安装Hexo
``` bash
npm install -g hexo-cli
```

- 使用yarn安装Hexo
``` bash
yarn global add hexo-cli
```

## 博客创建与发布

### 本地博客创建与预览

- 创建本地博客文件夹
``` bash
mkdir -p /media/lsw/work/longwusha
```

- 使用git对本地博客文件夹进行版本管理
``` bash
git init
```

- 初始化该文件夹为Hexo文件夹
``` bash
hexo init
yarn
```

- 编译并生成网站文件
``` bash
hexo g
hexo generate
```

- 在本地预览，预览地址 http://localhost:4000/
``` bash
hexo s
hexo server
```

### 博客主题选择与设置

- [官方主题网站](https://hexo.io/themes/)选择喜欢的主题，如[Aria](https://github.com/AlynxZhou/hexo-theme-aria)
- 参照主题官网中的指南对主题进行设置

主题选择注意事项：
1. 主题使用人数，方便出现问题后在社区寻求帮助
2. 主题维护活跃度，方便在需要对主题进行定制时寻求原主题作者帮助
3. 主题支持的额外功能，如博客搜索、博客访问统计、数学公式编辑和博客评论
4. 主题设置简洁，方便博客后续维护

### Github Pages发布平台配置

- 新建用于发布博客的[Github](https://github.com/)项目，项目名使用 username.github.io

- 添加ssh公钥到Github

- 测试公钥访问Github
``` bash
ssh -T git@github.com
```

- 配置Deployment，在本地博客文件夹中，找到_config.yml文件，修改repo值为博客Github项目的ssh访问地址

### 远程博客发布与查看

- 安装Hexo部署插件
``` bash
yarn add hexo-deployer-git
```

- 新建博客，博客文件路径为 ./source/_posts
``` bash
hexo new “博客名”
hexo new post “博客名”
```

- 编译并部署博客
``` bash
hexo d -g
```
