---
title: Git 使用指南
published: 2026-05-09
description: 常用的 Git 功能使用教程
pinned: true    # 置顶
tags: [Foo, Bar]
category: Git
draft: false
lang: zh-CN      # 仅当文章语言与 `siteConfig.ts` 中的网站语言不同时需要设置
comment: true    # 是否允许评论
---

## 绑定账户

配置本地 Git 身份：

```bash
# 1. 设置你的GitHub用户名（就是你GitHub主页的用户名）
git config --global user.name "你的GitHub用户名"

# 2. 设置你的GitHub绑定邮箱（就是你注册GitHub用的邮箱）
git config --global user.email "你的GitHub邮箱"
```

检测是否成功

```bash
git config --list

git config --global user.name
git config --global user.email
```

---

## 项目开始

方式一：克隆 GitHub 上已有的项目

```bash
git clone https://github.com/用户名/项目名.git
```

方式二：本地新建项目，关联到 GitHub

```bash
# 1. 进入你的项目文件夹
cd 你的项目路径

# 2. 初始化 Git
git init

# 3. 关联远程 GitHub 仓库（先在 GitHub 新建空仓库）
git remote add origin https://github.com/用户名/项目名.git
```

---

## 拉取和推送操作

拉取最新代码：

```bash
git pull
```

查看文件修改状态：

```bash
git status
```

添加修改到「暂存区」：

```bash
# 添加所有修改
git add .

# 添加某个文件
git add 文件名
```

提交到本地仓库并推送：

```bash
git commit -m "修改内容"
# 推送
git push origin 本地分支名
# 推送过后可以选择更简明的方式推送
git push
```

---

## 分支操作

优先新建分支进行代码修改，确保代码无误后切换本地主分支，进行分支合并后推送到远程仓库。

查看当前分支：

```bash
git branch
```

创建并切换到新分支：

```bash
git checkout -b 分支名
# 切换分支
git checkout 分支名
```

把分支合并到主分支：

```bash
# 先切回主分支
git checkout main

# 合并你的功能分支
git merge 分支名
```

删除本地分支：

```bash
git branch -d 分支名
```

---

## Fork 项目同原作者更新

完整流程：

```bash
# 1. 添加上游仓库（原作者项目）【只做一次】
git remote add upstream https://github.com/原作者/项目名.git

# 查看是否存在
git remote -v

# 2. 拉取原作者更新
git fetch upstream

# 3. 合并到你的本地主分支
git merge upstream/main

# 4. 推送到你的 GitHub
git push origin main
```



 
