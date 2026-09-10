---
title: 别再手忙脚乱了：用 Git Hooks 自动化你的代码检查
categories:
  - 瞎折腾系列
excerpt: 是不是也遇到过手一快，commit message 写错了格式？或者更惨，不小心把本地的密钥文件给提交上去了？原来 Git 自带一个叫 Git Hooks 的神器，能帮你从源头上杜绝这些问题。
date: 2025-08-27 18:30:00
tags:
  - Git
  - Git Hooks
  - 自动化
  - 代码规范
---

# 今天试用了新东西：用 Git Hooks 自动化你的代码检查

不知道你是不是也遇到过这样的场景：

代码写完了，心情激动，手指在键盘上翻飞，`git add .`，`git commit -m "fix bug"`，一气呵成！结果提交上去之后才发现，团队要求 commit message 必须是 `fix: 修复用户无法登录的问题` 这种格式……

更惨的是，有时候不小心把本地的配置文件 `local.env` 或者一些日志文件也 add 进去了，等 push 到远程仓库才发现，那叫一个尴尬。

当然了，以上场景在老手程序员里不太容易出现，但是有时候确实存在一些小的手误，或者你想规范跟你合作的“不成器”的兄弟的提交行为，基于此，之前就听说 Git 自带一个叫 **Git Hooks（Git 钩子）** 的神器，能帮你把这些检查工作自动化，可以从源头上杜绝这类“手快了”的低级错误。但是之前没有亲自去构建过，现在摸索尝试一下，记录并分享。

## 所以，Git Hooks 是个啥？

简单说，它就是一些在你执行 Git 命令时，能够被自动触发的脚本。

你可以把它想象成是你家门口的“保安”。比如，你想出门（`git commit`），保安（钩子脚本）会先把你拦下来，检查一下你的“出门条”（commit message）写得规不规范，看看你有没有携带“危险品”（不该提交的文件）。检查通过了，才放你出门。

这些“保安”脚本就存放在你项目里的 `.git/hooks` 目录下，而且**完全不用安装任何第三方工具，是 Git 原生自带的功能**。

## 核心玩法：让脚本帮你站岗

这里举个例子，我要求：

1.  **检查 Commit Message**：必须符合 `类别: 描述` 的格式。
2.  **拦截禁用文件**：不允许提交某些指定的文件。

下面我们就来配置两个“保安”脚本来完成这个任务。

#### 第一关：检查 Commit Message 格式

我们需要一个在“提交信息”写完后触发的钩子，它叫 `commit-msg`。

1.  **创建脚本文件**
    进入你的项目目录，找到 `.git/hooks` 文件夹，在里面创建一个名为 `commit-msg` 的文件（注意，没有扩展名）。

2.  **写入脚本内容**
    把下面的代码复制进去。

    ```sh
    #!/bin/sh
    # commit-msg 钩子: 检查 commit message 是否符合规范

    COMMIT_MSG_FILE=$1
    # 允许的类别
    COMMIT_MSG_REGEX="^(version|chore|config|feat|fix|doc|docs|scripts|widget): .+"
    FIRST_LINE=$(head -n1 $COMMIT_G_FILE)

    if ! [[ "$FIRST_LINE" =~ $COMMIT_MSG_REGEX ]]; then
      echo "!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!"
      echo "!!! 错误：Commit Message 格式不正确 !!!"
      echo "!!! 正确格式: '类别: 描述', 例如: 'feat: 添加登录功能'"
      echo "!!! 允许的类别: version, chore, config, feat, fix, doc, docs, scripts，widget"
      echo "!!! 您的提交信息: \"$FIRST_LINE\""
      echo "!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!"
      exit 1 # 退出并中断 commit
    fi

    exit 0 # 检查通过
    ```

3.  **添加执行权限（非Windows环境可能需要）**
    这是关键一步，不然脚本跑不起来。打开你的终端（推荐 Git Bash），执行：
    `chmod +x .git/hooks/commit-msg`

搞定！现在你再试试不按格式写 commit message，看看 Git 是不是会马上报错并拒绝你的提交。

#### 第二关：拦截不想提交的文件

这次我们需要一个在 `git commit` 一开始就运行的钩子，它叫 `pre-commit`。

1.  **创建脚本文件**
    同样，在 `.git/hooks` 目录下创建一个名为 `pre-commit` 的文件。

2.  **写入脚本内容**
    这个脚本可以让你自定义一个“黑名单”，所有在黑名单里的文件都会被拦截。

    ```sh
    #!/bin/sh
    # pre-commit 钩子: 检查是否有不允许提交的文件

    # --- 在这里配置你的文件黑名单 ---
    FORBIDDEN_FILES=(
      "config/local.env"
      "secrets.json"
      "*.log"
    )
    # --------------------------------

    ERROR_FOUND=0
    STAGED_FILES=$(git diff --cached --name-only)

    for pattern in "${FORBIDDEN_FILES[@]}"; do
      MATCHING_FILES=$(echo "$STAGED_FILES" | grep -E "^${pattern}$")
      if [ -n "$MATCHING_FILES" ]; then
        echo "!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!"
        echo "!!! 错误：检测到禁止提交的文件 (模式: $pattern) !!!"
        echo "$MATCHING_FILES"
        echo "!!! 请使用 'git reset HEAD <file>' 将其移出暂存区。"
        echo "!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!"
        ERROR_FOUND=1
      fi
    done

    if [ $ERROR_FOUND -eq 1 ]; then
      exit 1 # 发现黑名单文件，中断 commit
    fi

    exit 0 # 检查通过
    ```

3.  **添加执行权限（非Windows环境可能需要）**
    `chmod +x .git/hooks/pre-commit`

现在，如果你不小心 `git add` 了一个 `.log` 文件，`git commit` 的时候就会立刻被这位“保安”发现并拦下。这下不怕手滑了吧？

## 关键一步：如何让团队一起用？

你可能会问，`.git` 目录不是不会被提交到远程仓库吗？那团队其他人怎么用这些钩子呢？问得好！这正是团队协作的关键。

最好的方法是使用 Git 2.9 版本之后加入的一个新配置：`core.hooksPath`。我看了一下我的Git版本，已经是2.50.x了，看来已经是远古时代就增加了的功能了。

操作很简单：

1.  在你的项目**根目录**下创建一个新文件夹，比如叫 `.githooks`。
2.  把刚才写好的 `pre-commit` 和 `commit-msg` 两个脚本文件移动到 `.githooks` 文件夹里。
3.  把 `.githooks` 文件夹 `git add .` 并提交，这样团队所有人都能拉取到这些脚本了。
4.  最后，通知团队所有成员，在自己的电脑上，针对这个项目**执行一次**下面的命令：
    ```bash
    git config core.hooksPath .githooks
    ```
    这条命令会告诉 Git：“以后这个项目的钩子，别去默认的 `.git/hooks` 找了，都来根目录的 `.githooks` 文件夹里找！”

这样一来，不仅实现了团队钩子的统一，而且以后更新钩子逻辑，也只需要修改 `.githooks` 里的文件并提交就行，大家 `git pull` 之后就自动更新了。

## 总结一下

总之，今天又实践了一个新玩意，听说过的技术很多，有时候动手实践一次就会真正成为自己的经验了。用 Git Hooks 这么个小小的技巧，就能从源头上规范团队的提交行为，减少很多不必要的低级错误，大大提升了代码仓库的整洁度和协作效率。

这次只记录了最基础的用法，其实 Git Hooks 能做的事情还有很多，比如提交前自动跑测试、检查代码格式等等。以后有机会还要继续试试。
