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

## 一、绑定账户

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

## 二、本地仓库操作

### 1. 初始化本地仓库

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

### 2. 查看工作区状态

无论做什么操作，建议先查看工作区状态，确认当前文件的修改情况，避免误操作。

```bash
git status
```

- 未修改：提示 `nothing to commit, working tree clean`（工作区干净，没有任何修改）；

- 已修改但未提交到暂存区：文件名称显示为 `modified`（红色）；

- 已提交到暂存区但未提交到版本库：文件名称显示为 `staged`（绿色）。


### 3. 文件提交到暂存区

在工作区修改了文件（如新增、修改、删除文件），需要先提交到暂存区，再准备提交到版本库。

```bash
# 方式 1：提交所有修改的文件（最常用）
git add .

# 方式 2：提交指定文件（适用于只提交部分修改）
git add 文件名（如：git add index.html）

# 方式 3：提交指定文件夹下的所有文件
git add 文件夹名（如：git add src/）
```

### 4. 暂存区的文件提交到本地版本库

提交后，修改会被永久保存到本地版本库，生成一条新的版本记录。

```bash
git commit -m "提交说明"
```

### 5. 查看提交历史记录

查看之前的提交记录，包括提交者、提交时间、提交说明、版本号（用于回滚版本）。

```bash
# 方式 1：查看完整历史记录（详细）
git log

# 方式 2：查看简洁历史记录（只显示版本号和提交说明，推荐）
git log --oneline

# 方式 3：查看所有提交记录（包括回滚的记录）
git reflog
```

### 6. 版本回滚（恢复到历史版本）

修改出错、代码丢失，需要恢复到之前的某个稳定版本

```bash
# 第一步：查看历史版本，获取要回滚的版本号（前7位即可）
git log --oneline

# 第二步：回滚到指定版本（两种常用方式）

# 方式 1：彻底回滚，删除回滚版本之后的所有提交记录（谨慎使用，适合个人开发）
git reset --hard 版本号（如：git reset --hard a1b2c3d）

# 方式 2：安全回滚，保留回滚版本之后的修改（推荐，适合多人协作）
git revert 版本号（如：git revert a1b2c3d）
```

### 7. 撤销修改

在工作区修改了文件，但还没执行 `git add`，想放弃修改，恢复到上一次提交后的状态。

```bash
# 撤销单个文件的修改
git checkout -- 文件名（如：git checkout -- index.html）

# 撤销所有工作区的修改（未 add 的）
git checkout .
```

执行了 `git add`，将修改提交到了暂存区，但还没执行 `git commit`，想撤销暂存，重新修改。

```bash
git reset HEAD 文件名（如：git reset HEAD index.html）

# 撤销所有暂存区的修改
git reset HEAD .
```

将暂存区的修改撤销回工作区，此时文件状态会从“绿色（staged）”变回“红色（modified）”，之后可以重新修改、重新 add。

---

## 三、远程合作

### 1. 拉取和推送

拉取最新代码：

```bash
# 绑定远程仓库
git remote add origin 远程仓库地址

# 查看已关联远程库
git remote -v

git pull

git pull origin 分支名（如：git pull origin develop）
```


提交到本地仓库并推送：

```bash
# 第一次推送（绑定本地分支和远程分支，后续可直接用 git push）
git push -u origin main（或 master）

# 后续推送（已绑定分支）
git push origin main（或 master）

# 方式2：推送本地指定分支到远程指定分支
git push origin 本地分支名:远程分支名（如：git push origin develop:develop）
```

### 2. 分支操作

优先新建分支进行代码修改，确保代码无误后切换本地主分支，进行分支合并后推送到远程仓库。

查看当前分支：

```bash
git branch

# 查看本地和远程所有分支
git branch -a

# 查看远程所有分支
git branch -r
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

## 四、Fork 项目同原作者更新

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



 
