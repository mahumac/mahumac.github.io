# 完整 Git 工作流程指南：从初学者到专家

掌握 Git 工作流程，包含 48 个基本命令和可视化图表。从初学者到专家的完整指南，包含分支、合并和故障排除。

发表于2023年1月31日，更新于2025年10月2日

[![完整的Git工作流程和命令可视化指南](https://sagarnikam123.github.io/assets/img/posts/20230131/git-workflows-guide.webp)](https://sagarnikam123.github.io/assets/img/posts/20230131/git-workflows-guide.webp)完整的Git工作流程和命令可视化指南

作者*[：萨加尔·尼卡姆](https://github.com/sagarnikam123/)*

*阅读42分钟*

掌握从基础命令到高级技巧的**Git工作流程**。这份全面的**Git指南**涵盖了开发者高效**版本控制**和团队协作所需的一切。

无论你是初学者，还是实施复杂**Git工作流**的专家，本指南都提供了48个基本的Git命令、故障排除解决方案以及现代软件开发的最佳实践。`git commit`

## 目录

- [入门指南](https://sagarnikam123.github.io/posts/git-workflows-guide/#getting-started)
- [基本 Git作](https://sagarnikam123.github.io/posts/git-workflows-guide/#basic-git-operations)
- [Git 工作流程概述](https://sagarnikam123.github.io/posts/git-workflows-guide/#git-workflow-overview)
- [分行管理](https://sagarnikam123.github.io/posts/git-workflows-guide/#branch-management)
- [远程存储库作](https://sagarnikam123.github.io/posts/git-workflows-guide/#remote-repository-operations)
- [故障排查指南](https://sagarnikam123.github.io/posts/git-workflows-guide/#troubleshooting-guide)
- [历史与搜寻行动](https://sagarnikam123.github.io/posts/git-workflows-guide/#history--search-operations)
- [高级 Git 技术](https://sagarnikam123.github.io/posts/git-workflows-guide/#advanced-git-techniques)
- [Git 工作流程与策略](https://sagarnikam123.github.io/posts/git-workflows-guide/#git-workflows--strategies)
- [维护与清理](https://sagarnikam123.github.io/posts/git-workflows-guide/#maintenance--cleanup)
- [安全性与性能](https://sagarnikam123.github.io/posts/git-workflows-guide/#security--performance)
- [Git 钩子与自动化](https://sagarnikam123.github.io/posts/git-workflows-guide/#git-hooks--automation)
- [高级场景](https://sagarnikam123.github.io/posts/git-workflows-guide/#advanced-scenarios)
- [现实世界的例子](https://sagarnikam123.github.io/posts/git-workflows-guide/#real-world-examples)
- [快速参考](https://sagarnikam123.github.io/posts/git-workflows-guide/#quick-reference)
- [最佳实践总结](https://sagarnikam123.github.io/posts/git-workflows-guide/#best-practices-summary)
- [常见问题解答](https://sagarnikam123.github.io/posts/git-workflows-guide/#frequently-asked-questions)

## 入门指南

### 什么是Git？

Git 是一个分布式版本控制系统，用于跟踪多个开发者文件的变更和工作坐标。它对现代软件开发至关重要。

### 📝 完整 Git 命令快速引用（48 条命令）

| 指挥                             | 描述               | 用途                             |
| -------------------------------- | ------------------ | -------------------------------- |
| **设置与配置（4）**              |                    |                                  |
| ⚙️`git config --global`           | 全局配置 Git       | 设置用户名、邮箱和偏好设置       |
| 🔧`git --version`                 | 查看Git版本        | 验证 Git 安装                    |
| 📋`git config --list`             | 视图配置           | 查看所有当前的Git设置            |
| 🌿`git config init.defaultBranch` | 设置默认分支       | 配置主主机与主机                 |
| **基本作（7）**                  |                    |                                  |
| 🆕`git init`                      | 初始化仓库         | 创建新的 Git 仓库                |
| 🌍`git clone`                     | 克隆仓库           | 本地复制远程仓库                 |
| 📁`git status`                    | 检查文件状态       | 看看哪些是分段、修改或未被追踪的 |
| ➕`git add`                       | 舞台变换           | 准备提交文件                     |
| 💾`git commit`                    | 存档更改           | 创建带有消息的快照               |
| 📜`git log`                       | 查看历史           | 浏览提交历史和消息               |
| 🔍`git diff`                      | 参见变更           | 比较工作目录与分期               |
| **分行管理（6）**                |                    |                                  |
| 🌿`git branch`                    | 列表/创建分支      | 管理分支运营                     |
| 🔄`git checkout`                  | 切换分支           | 更改现役分支                     |
| ✨`git checkout -b`               | 创建与切换         | 一个命令中的新分支               |
| 🔀`git merge`                     | 合并分支           | 联合军种变更                     |
| 🗑️`git branch -d`                 | 删除分支           | 移除本地分支                     |
| 📝`git branch -m`                 | 更名分支           | 更改分支名称                     |
| **远程行动（6）**                |                    |                                  |
| 🔗`git remote add`                | 添加遥控器         | 连接到远程仓库                   |
| 🔍`git remote -v`                 | 查看遥控器         | 列表配置的远程                   |
| 🚀`git push`                      | 上传更改           | 向远程发送提交                   |
| 🚀`git push -u`                   | 上游推             | 设置追踪和推送到远程             |
| 📥`git pull`                      | 下载更新           | 取用并合并的更改                 |
| 📦`git fetch`                     | 只用取球           | 下载而不合并                     |
| **文件与变更管理（8）**          |                    |                                  |
| 📦`git stash`                     | 暂时节省工作       | 存储未提交的更改                 |
| 🔍`git show`                      | 显示提交详情       | 查看具体提交更改                 |
| 📝`git mv`                        | 移动/重命名文件    | git 类文件作                     |
| 🗑️`git rm`                        | 删除文件           | 从Git追踪中删除文件              |
| 🔄`git restore`                   | 还原文件           | 现代的弃变方法                   |
| 🧹`git clean`                     | 删除未被追踪的文件 | 干净的工作目录                   |
| 📋`git ls-files`                  | 列表追踪文件       | 版本控制下的显示文件             |
| 🔍`git blame`                     | 显示文件注释       | 看看是谁更改了每一句话           |
| **历史与搜寻（6）**              |                    |                                  |
| 🔍`git reflog`                    | 查看所有变更       | 查看完整历史日志                 |
| 🔍`git grep`                      | 在仓库中搜索       | 在所有文件中查找文本             |
| 🔍`git log --grep`                | 搜索提交消息       | 通过消息查找提交                 |
| 🔍`git log --author`              | 按作者筛选         | 按特定作者查找提交               |
| 🔍`git log --since`               | 按日期筛选         | 查找日期范围内的提交             |
| 🔍`git bisect`                    | 查找漏洞介绍       | 问题提交的二分搜索               |
| **高级技巧（6）**                |                    |                                  |
| 🔄`git rebase`                    | 历史重写           | 创建线性提交历史                 |
| 🍒`git cherry-pick`               | 应用特定提交       | 复制提交到当前分支               |
| ⏪`git reset`                     | 撤销提交           | 将HEAD移到上一个状态             |
| 🏷️`git tag`                       | 标记版本           | 创建释放标记                     |
| 🔄`git revert`                    | 安全撤销           | 创建新的提交以撤销更改           |
| 🔄`git rebase -i`                 | 交互式基底         | 交互式编辑提交历史               |

### 初始git设置

```
# configure your identity
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# set default branch name to 'main' (GitHub's new standard)
git config --global init.defaultBranch main

# verify installation
git --version

# check configuration
git config --list
```

## 基本 Git作

### 仓库初始化

```
# create new repository
git init

# clone existing repository
git clone <repository_URL>
```

### 文件作

```
# add files to staging area
git add <file_name>
git add .  # add all files
git add *.js  # add specific file types
git add -p  # interactively stage parts of files

# commit changes
git commit -m "descriptive commit message"
git commit -a -m "add and commit in one step"
git commit --amend  # modify last commit

# view commit history variations
git log --oneline  # compact view
git log --graph    # visual branch history
git log --stat     # show file statistics
git log -p         # show patch/diff for each commit
```

### File Management Commands

```
# move/rename files (Git-aware)
git mv old_filename.txt new_filename.txt
git mv file.txt subfolder/file.txt

# remove files from Git tracking
git rm file.txt                    # delete file and remove from Git
git rm --cached file.txt           # remove from Git but keep local file
git rm -r folder/                  # remove directory recursively

# restore files (Git 2.23+)
git restore file.txt               # discard changes in working directory
git restore --staged file.txt      # unstage file
git restore --source=HEAD~1 file.txt  # restore from specific commit

# clean untracked files
git clean -n                       # dry run - show what would be deleted
git clean -f                       # remove untracked files
git clean -fd                      # remove untracked files and directories
git clean -fx                      # remove untracked and ignored files

# list tracked files
git ls-files                       # show all tracked files
git ls-files --others              # show untracked files
git ls-files --ignored             # show ignored files
```

### Viewing Changes and Information

```
# show specific commit details
git show <commit_hash>             # show commit details and diff
git show HEAD~1                   # show previous commit
git show --name-only <commit>      # show only changed file names
git show --stat <commit>           # show file statistics

# file annotations (who changed what)
git blame file.txt                 # show line-by-line authorship
git blame -L 10,20 file.txt        # blame specific lines
git blame -w file.txt              # ignore whitespace changes
```

## Git Workflow Overview

Now that you know the basic commands, let’s understand how Git works with this visual overview:

```
🌐 Remote Environment (Cloud/Server)
🏠 Local Environment (Your Machine)
1️⃣ git add
Stage changes
2️⃣ git commit
Save snapshot
3️⃣ git push
Upload to remote
4️⃣ git pull/fetch
Download updates
5️⃣ git checkout
Switch/restore
💻 Working Directory
(Your local files)
Edit, create, modify files
🎨 Staging Area
(Index/Cache)
Prepared for commit
📦 Local Repository
(Your .git folder)
Committed snapshots
☁️ Remote Repository
(GitHub/GitLab)
Shared with team
```

## Branch Management

### Main vs Master Branch

**Important:** GitHub changed the default branch name from to in 2020. New repositories use by default, but older repositories may still use . Both work identically - it’s just a naming convention.`master``main``main``master`

```
# if working with older repositories using 'master'
git checkout master
git pull origin master

# if working with newer repositories using 'main'
git checkout main
git pull origin main

# rename master to main in existing repository
git branch -m master main
git push -u origin main
```

### Creating and Switching Branches

```
# list all branches
git branch -a
# create new branch
git branch <branch_name>
# switch to branch
git checkout <branch_name>
# create and switch in one command
git checkout -b <new_branch>
```

### Branch Operations

```
# delete remote branch
git push origin --delete <branch_name>

# advanced branch operations
git branch -v                      # show last commit on each branch
git branch --merged                # show branches merged into current
git branch --no-merged             # show unmerged branches
git branch --contains <commit>     # branches containing specific commit
git branch -u origin/main          # set upstream tracking

# branch comparison
git diff main..feature-branch      # compare branches
git log main..feature-branch       # commits in feature not in main
git merge-base main feature-branch # find common ancestor
```

### Working with Specific Branches

```
# pull specific branch
git clone <remote_url>
git checkout <remote_branch_name>
git pull origin <branch_name>

# create branch from existing branch
git checkout <existing-branch>
git checkout -b <new-branch>
git push -u origin <new-branch>
```

## Remote Repository Operations

### Basic Remote Commands

```
# push with upstream tracking
git push -u origin <branch_name>  # set upstream

# fetch without merging
git fetch origin
git fetch --all                   # fetch from all remotes
git fetch --prune                 # remove deleted remote branches

# advanced remote operations
git remote show origin             # detailed remote information
git remote rename origin upstream  # rename remote
git remote set-url origin <new_url>  # change remote URL

# push variations
git push --force-with-lease        # safer force push
git push --all                     # push all branches
git push --tags                    # push all tags
git push origin --delete <branch>  # delete remote branch
```

### Fork Synchronization

**Fork** is your personal copy of someone else’s repository where you can make changes without affecting the original.

```
# add upstream remote for forks
git remote add upstream <original_repo_url>

# sync fork with upstream
git fetch upstream
git checkout main
git merge upstream/main
git push origin main

# alternative: rebase approach
git rebase upstream/main
```

## Troubleshooting Guide

```
📤 Can't push
⚔️ Merge conflicts
👻 Detached HEAD
💔 Lost commits
❌ Wrong commit










🚨 Git Issue Detected
Something went wrong?
🤔 What's the problem?
Choose your issue
🔍 1️⃣ Check Remote
Verify connection
✏️ 2️⃣ Resolve Conflicts
Manual editing required
🌿 3️⃣ Create Branch
Save your work
🔍 4️⃣ Use Reflog
Find lost commits
⏪ 5️⃣ Reset/Revert
Undo changes
📋 git remote -v
Check URL & permissions
🛠️ Edit → Add → Commit
Resolve markers manually
🆕 git checkout -b new-branch
Create branch from HEAD
📜 git reflog → checkout hash
Recover lost work
🔄 git reset --soft HEAD~1
Keep changes staged
✅ Issue Resolved!
Back to coding 🎉
```

### 常见问题与解决方案

#### 分离的主控

**分离的HEAD**意味着你不在任何分支上，只是查看某个特定的提交。

```
# create branch from detached HEAD
git checkout -b new-branch-name

# return to main branch
git checkout main
```

#### 恢复丢失的提交

```
# find lost commits
git reflog
git log --all --full-history

# recover specific commit
git checkout <commit_hash>
git checkout -b recovered-branch
```

#### 合并冲突

合并**冲突**发生在 Git 无法自动合并不同分支的变更时。

```
# resolve conflicts manually, then:
git add <resolved_files>
git commit -m "resolve merge conflicts"

# abort merge if needed
git merge --abort
```

#### 仓库问题

```
# check repository integrity
git fsck --full

# cleanup and optimize
git gc --aggressive --prune=now

# remove untracked files
git clean -f    # files only
git clean -fd   # files and directories
```

#### 行尾问题

```
# configure line endings
git config --global core.autocrlf true   # Windows
git config --global core.autocrlf input  # Mac/Linux
git config --global core.autocrlf false  # no conversion
```

## 历史与搜寻行动

### 检索仓库

```
# search for text in files
git grep "search_term"             # search in working directory
git grep "search_term" HEAD~1      # search in specific commit
git grep -n "search_term"          # show line numbers
git grep -i "search_term"          # case insensitive search
git grep -w "search_term"          # match whole words only

# search commit messages
git log --grep="bug fix"           # find commits with specific message
git log --grep="feature" --grep="bug" --all-match  # multiple criteria

# search by author and date
git log --author="John Doe"        # commits by specific author
git log --since="2023-01-01"       # commits since date
git log --until="2023-12-31"       # commits until date
git log --since="2 weeks ago"      # relative date

# advanced log filtering
git log --oneline --since="1 month ago" --author="$(git config user.name)"
git log --stat --since="1 week ago"
git log -p --grep="fix" --since="1 month ago"
```

### Binary Search for Bugs

**Git bisect** helps you find the exact commit that introduced a bug using binary search.

```
# start bisect session
git bisect start
git bisect bad                     # current commit is bad
git bisect good v1.0.0             # known good commit

# Git will checkout middle commit
# test your code, then mark as good or bad
git bisect good                    # if this commit works
git bisect bad                     # if this commit has the bug

# continue until Git finds the problematic commit
git bisect reset                   # end bisect session

# automated bisect with script
git bisect start HEAD v1.0.0
git bisect run ./test_script.sh    # script returns 0 for good, 1 for bad
```

## Advanced Git Techniques

### Stashing Changes

**Stashing** temporarily saves your uncommitted changes so you can switch branches or pull updates without losing work.

```
# temporarily save changes
git stash
git stash save "work in progress"

# apply stashed changes
git stash pop
git stash apply stash@{0}

# manage stashes
git stash list
git stash drop stash@{0}
git stash clear
```

### Rebase Operations

**Rebase** rewrites commit history by moving your commits to a new base, creating a cleaner linear history.

```
⚡ git rebase main
📏 AFTER REBASE: Clean Linear History



🌿 A
main
🌿 B
main
✨ C'
rebased
✨ D'
rebased
🔀 BEFORE REBASE: Messy History



🌿 A
main
🌿 B
main
✨ C
feature
✨ D
feature
```

### Merge vs Rebase: Visual Comparison

#### Starting Scenario

**Common situation**: You have a feature branch that needs to be integrated with the main branch.

```
🎯 Starting Point



🌿 A
main
🌿 B
main (latest)
✨ C
feature
✨ D
feature (your work)
```

#### Merge Approach Result

**`git merge feature`** - Preserves history with merge commit:

```
🔀 MERGE RESULT





🌿 A
main
🌿 B
main
✨ C
feature
✨ D
feature
🔀 M
merge commit
```

#### 重定法结果

**`git rebase 主版`**——创建干净的线性历史：

```
📏 REBASE RESULT



🌿 A
main
🌿 B
main
✨ C'
rebased
✨ D'
rebased
```

### 详细对比表

#### 📏 历史与结构

| 相位             | 🔀 **合并**           | 📏 **重新基底**       |
| ---------------- | -------------------- | -------------------- |
| **历史保护**     | ✓ 保留原始提交历史   | ✗ 重写提交历史       |
| **提交结构**     | ✗ 创建合并提交       | ✓ 无额外提交         |
| **时间线准确性** | ✓ 显示真实开发时间线 | ✗ 创造人为线性时间线 |
| **提交哈希**     | ✓ 原始哈希保存       | ✗ 新生成的哈希       |
| **图复杂度**     | ✗ 复分支图           | ✓ 简单线性图         |

#### 🚀 团队协作

| 相位         | 🔀 **合并**               | 📏 **重新基底**         |
| ------------ | ------------------------ | ---------------------- |
| **共享分支** | ✓ 公共分支安全           | ✗ 共用分支上的危险     |
| **团队协作** | ✓ 多位贡献者友好         | ✗ 最佳单一贡献者       |
| **代码审查** | ✓ 作为单元易于查看的功能 | ✓ 清理提交以供审核     |
| **回滚安全** | ✓ 易于还原合并           | ✗ 复基复合体           |
| **冲突解决** | ✓ 一次性冲突解决         | ✗ 可能需要多项冲突修复 |

#### 🎯 何时使用每种方法

| 剧情         | 🔀 **使用合并**   | 📏 **使用Rebase**   |
| ------------ | ---------------- | ------------------ |
| **分支类型** | 公营/共享分支    | 私人特色分支       |
| **球队规模** | 多位贡献者       | 单显影剂           |
| **历史偏好** | 想保留准确的历史 | 想要干净的线性历史 |
| **项目阶段** | 制作发行         | 开发清理           |
| **合作**     | 团队功能开发     | 个人专题作品       |
| **代码审查** | 基于专题的评论   | 基于提交的评审     |

### 指挥摘要

```
# MERGE APPROACH
git checkout main
git merge feature-branch          # creates merge commit
git merge --no-ff feature-branch  # force merge commit

# REBASE APPROACH
git checkout feature-branch
git rebase main                   # rewrite commits on new base
git rebase -i HEAD~3             # interactive rebase (edit history)

# HANDLING CONFLICTS
# During merge:
git merge --abort                 # cancel merge

# During rebase:
git rebase --continue            # continue after fixing conflicts
git rebase --abort               # cancel rebase

# BEST PRACTICE WORKFLOW
# For private feature branches:
git checkout feature-branch
git rebase main                  # clean up before sharing
git checkout main
git merge feature-branch         # fast-forward merge
```

### 精选与重置：视觉对比

#### 起始场景

**常见情况**：你需要应用特定的提交或撤销 Git 历史中的更改。

```
🎯 Starting Point




🌿 A
main
🌿 B
main
✨ C
feature
✨ D
feature
✨ E
hotfix
```

#### 精选结果

**`git cherry-pick E`** - 复制特定提交到当前分支：

```
🍒 CHERRY-PICK RESULT





🌿 A
main
🌿 B
main
✨ E'
cherry-picked
✨ C
feature
✨ D
feature
✨ E
original
```

#### Reset Result

**`git reset --hard HEAD~1`** - Moves HEAD back and discards changes:

```
⏪ RESET RESULT




🌿 A
main
🌿 B
HEAD moved here
✨ C
feature
✨ D
feature
✨ E
hotfix
❌ X
discarded commit
```

### Detailed Comparison Tables

#### 🎯 Purpose & Functionality

| Aspect                  | 🍒 **Cherry-Pick**         | ⏪ **Reset**                  |
| ----------------------- | ------------------------- | ---------------------------- |
| **Primary Purpose**     | Copy specific commits     | Undo commits/move HEAD       |
| **Direction**           | Forward (adds commits)    | Backward (removes commits)   |
| **Commit Creation**     | ✓ Creates new commit      | ✗ No new commits             |
| **Original Commits**    | ✓ Preserves originals     | ✗ May discard commits        |
| **Selective Operation** | ✓ Choose specific commits | ✗ Affects all recent commits |

#### 🛠️ Safety & Impact

| Aspect                | 🍒 **Cherry-Pick**        | ⏪ **Reset**               |
| --------------------- | ------------------------ | ------------------------- |
| **Data Safety**       | ✓ Non-destructive        | ✗ Can be destructive      |
| **Reversibility**     | ✓ Easy to undo           | ✗ Hard to recover (–hard) |
| **Working Directory** | ✓ Preserves changes      | ✗ May discard changes     |
| **Staging Area**      | ✓ Preserves staged files | ✗ May clear staging       |
| **Risk Level**        | ✓ Low risk               | ✗ High risk (–hard)       |

#### 🎯 When to Use Each Approach

| Scenario                  | 🍒 **Use Cherry-Pick**                 | ⏪ **Use Reset**              |
| ------------------------- | ------------------------------------- | ---------------------------- |
| **Hotfix Application**    | Apply urgent fix to multiple branches | Undo recent commits          |
| **Feature Extraction**    | Extract specific features             | Remove unwanted commits      |
| **Bug Fix Propagation**   | Copy bug fixes across branches        | Clean up commit history      |
| **Selective Integration** | Pick useful commits from experiments  | Reset to stable state        |
| **Cross-Branch Work**     | Share commits between branches        | Local development cleanup    |
| **Production Fixes**      | Apply tested fixes                    | Never use on shared branches |

### Commands Summary

```
# CHERRY-PICK COMMANDS
git cherry-pick <commit_hash>        # copy specific commit
git cherry-pick <hash1> <hash2>      # copy multiple commits
git cherry-pick <start>..<end>       # copy range of commits
git cherry-pick --no-commit <hash>   # copy without committing
git cherry-pick --continue           # continue after conflicts
git cherry-pick --abort              # cancel cherry-pick

# RESET COMMANDS
git reset --soft HEAD~1              # undo commit, keep changes staged
git reset --mixed HEAD~1             # undo commit, unstage changes
git reset --hard HEAD~1              # undo commit, discard all changes
git reset <commit_hash>              # reset to specific commit
git reset --hard origin/main         # reset to remote state

# REVERT COMMANDS (SAFER ALTERNATIVE)
git revert <commit_hash>             # create new commit that undoes changes
git revert --no-commit <hash>        # revert without committing
git revert -m 1 <merge_commit>       # revert merge commit
git revert HEAD~3..HEAD              # revert range of commits

# SAFETY COMMANDS
git reflog                           # find lost commits after reset
git checkout <commit_hash>           # recover lost commit
git checkout -b recovery-branch      # create branch from lost commit

# WORKING DIRECTORY CLEANUP
git checkout -- <file_name>          # discard file changes (legacy)
git restore <file_name>              # Git 2.23+ restore file
git clean -f                         # remove untracked files
git clean -fd                        # remove untracked files & directories
```

### Git Revert vs Reset: Visual Comparison

#### Starting Scenario

**Common situation**: You have commits you want to undo, but need to choose the right approach.

```
🎯 Starting Point



📝 A
good commit
📝 B
good commit
❌ C
bad commit
📝 D
current HEAD
```

#### Revert Approach Result

**`git revert C`** - Creates new commit that undoes changes:

```
🔄 REVERT RESULT




📝 A
good commit
📝 B
good commit
❌ C
bad commit
📝 D
current
✅ E
revert commit
undoes C
```

#### Reset Approach Result

**`git reset --hard B`** - Moves HEAD back and discards commits:

```
⏪ RESET RESULT

📝 A
good commit
📝 B
HEAD moved here
❌ C
discarded
📝 D
discarded
```

### Git Revert vs Reset: When to Use Each

#### 🔄 Git Revert (Recommended for Shared Repositories)

**Revert** creates a new commit that undoes the changes from a previous commit, making it safe for shared repositories.

```
# revert single commit
git revert abc123                   # creates new commit undoing abc123

# revert merge commit (specify parent)
git revert -m 1 <merge_commit_hash>  # revert to first parent
git revert -m 2 <merge_commit_hash>  # revert to second parent

# revert multiple commits
git revert --no-commit HEAD~3..HEAD  # revert last 3 commits
git commit -m "revert last 3 commits"

# interactive revert
git revert --edit <commit_hash>      # edit revert commit message
```

#### ⏪ Git Reset (Use with Caution)

**Reset** moves the branch pointer and can discard commits permanently. Only use on private branches.

```
# reset types
git reset --soft HEAD~1    # keep changes staged
git reset --mixed HEAD~1   # unstage changes (default)
git reset --hard HEAD~1    # discard all changes

# reset to specific commit
git reset --hard abc123    # reset to commit abc123

# reset specific files
git reset HEAD file.txt    # unstage specific file
```

### Cherry-Pick and Reset: Advanced Usage

#### Interactive Rebase: Editing History

**Interactive rebase** allows you to edit, reorder, squash, or delete commits in your history.

```
# start interactive rebase for last 3 commits
git rebase -i HEAD~3

# interactive rebase options in editor:
# pick = use commit as-is
# reword = use commit but edit message
# edit = use commit but stop for amending
# squash = combine with previous commit
# fixup = like squash but discard commit message
# drop = remove commit entirely

# example interactive rebase session:
# pick abc123 Add user authentication
# squash def456 Fix typo in auth
# reword ghi789 Update documentation
# drop jkl012 Debug print statements

# continue after making changes
git rebase --continue

# abort if something goes wrong
git rebase --abort
```

### Interactive Cherry-Pick

```
# cherry-pick with edit opportunity
git cherry-pick --edit <commit_hash>

# cherry-pick without creating commit (for modifications)
git cherry-pick --no-commit <commit_hash>
git add <modified_files>
git commit -m "modified cherry-picked commit"
```

### Tag Operations

**Tags** mark specific points in Git history, typically used for release versions (v1.0, v2.0).

```
# list tags
git tag -l
git tag -l "v1.*"  # filter tags

# create tags
git tag v1.0.0
git tag -a v1.0.0 -m "version 1.0.0 release"

# push tags
git push --tags
git push origin v1.0.0

# checkout specific tag
git checkout tags/v1.0.0
```

## Git Workflows & Strategies

### Git Flow Workflow

**Git Flow** is a branching model with separate branches for features, releases, and hotfixes.

```
Git Flow Workflow
1️⃣ merge when complete
2️⃣ create when ready
3️⃣ merge to production
4️⃣ merge back changes
5️⃣ branch for urgent fix
6️⃣ merge fix
7️⃣ merge to develop
🌿 main
(production ready)
✨ feature/*
(new features)
🔧 develop
(integration branch)
🚀 release/*
(prepare release)
🚨 hotfix/*
(urgent fixes)
# initialize git flow
git flow init

# feature development
git flow feature start new-feature
git flow feature finish new-feature

# release management
git flow release start 1.0.0
git flow release finish 1.0.0

# hotfix process
git flow hotfix start critical-fix
git flow hotfix finish critical-fix
```

### GitHub Flow

**GitHub Flow** is a simple workflow where you create feature branches and merge them via pull requests.

```
1️⃣ Create branch
2️⃣ Open PR when ready
3️⃣ Merge after review
4️⃣ Auto-deploy
GitHub Flow Process




🌱 1. Create feature branch from main
📝 2. Add commits with descriptive messages
🔄 3. Open Pull Request for discussion
👀 4. Review, discuss, and iterate
✅ 5. Merge and deploy immediately
🌿 main branch
(always deployable)
Production ready code
✨ feature branch
(new development work)
Isolated feature development
📝 Pull Request
(code review process)
Team collaboration & quality
🚀 Deploy to Production
(continuous deployment)
Automated release process
# simple GitHub workflow
git checkout main
git pull origin main
git checkout -b feature/new-feature

# develop and commit
git commit -m "implement new feature"
git push -u origin feature/new-feature

# create pull request on GitHub
# merge via GitHub interface
git checkout main
git pull origin main
git branch -d feature/new-feature
```

### Team Collaboration Best Practices

```
# daily workflow
git checkout main
git pull origin main
git checkout feature/my-feature
git rebase main
git push --force-with-lease origin feature/my-feature
```

## Maintenance & Cleanup

### Repository Maintenance

```
# garbage collection and optimization
git gc                           # basic garbage collection
git gc --aggressive              # thorough cleanup (slower)
git gc --prune=now              # remove unreachable objects immediately

# repository integrity and repair
git fsck                         # check repository integrity
git fsck --full                  # thorough integrity check
git fsck --unreachable          # find unreachable objects

# count objects and disk usage
git count-objects               # count loose objects
git count-objects -v            # verbose object count
git count-objects -vH           # human-readable sizes

# pack files optimization
git repack -a -d                # repack all objects
git repack -a -d -f --window=250 --depth=250  # aggressive repacking

# prune operations
git prune                       # remove unreachable objects
git prune --expire="2 weeks ago" # prune objects older than 2 weeks
git remote prune origin         # remove stale remote branches
```

### Cleanup Commands

```
# clean working directory
git clean -n                    # dry run - show what would be deleted
git clean -f                    # remove untracked files
git clean -fd                   # remove untracked files and directories
git clean -fx                   # remove untracked and ignored files
git clean -fX                   # remove only ignored files

# reset and cleanup
git reset --hard HEAD           # discard all working directory changes
git checkout -- .               # discard all working directory changes (legacy)
git restore .                   # discard all working directory changes (modern)

# branch cleanup
git branch --merged | grep -v "\*\|main\|develop" | xargs -n 1 git branch -d
git remote prune origin         # remove stale remote tracking branches
git branch -vv | grep ': gone]' | awk '{print $1}' | xargs git branch -D

# reflog cleanup
git reflog expire --expire=30.days --all
git reflog expire --expire-unreachable=7.days --all
```

### Archive and Backup

```
# create archive of repository
git archive --format=zip --output=backup.zip HEAD
git archive --format=tar.gz --output=backup.tar.gz HEAD
git archive --format=tar --output=backup.tar HEAD^{tree}  # without .git

# create bundle (portable Git repository)
git bundle create backup.bundle --all
git bundle create feature.bundle main..feature-branch

# verify and use bundle
git bundle verify backup.bundle
git clone backup.bundle restored-repo
```

## Security & Performance

### Git Security

#### Personal Access Token Setup

**Personal Access Token (PAT)** is a secure alternative to passwords for GitHub authentication.

For secure GitHub authentication, use Personal Access Tokens instead of passwords:

```
# configure credential helper for macOS
git config --global credential.helper osxkeychain

# when prompted for password, use your Personal Access Token
# generate token at: https://github.com/settings/personal-access-tokens
```

**Generate Personal Access Token:**

1. Go to [GitHub Settings > Personal Access Tokens](https://github.com/settings/personal-access-tokens)
2. Click “Generate new token (classic)”
3. Select required scopes: , , `repo``workflow``write:packages`
4. Copy the generated token (save it securely)
5. Use this token as your password when Git prompts for authentication

#### GPG Commit Signing

**GPG signing** cryptographically proves that commits came from you, adding security and authenticity.

```
# generate GPG key
gpg --gen-key

# configure Git to use GPG
git config --global user.signingkey <GPG_KEY_ID>
git config --global commit.gpgsign true

# sign commits
git commit -S -m "signed commit"
```

#### Credential Management

```
# store credentials securely (macOS)
git config --global credential.helper osxkeychain

# cache credentials temporarily
git config --global credential.helper 'cache --timeout=3600'

# store credentials in file (less secure)
git config --global credential.helper store
```

### Performance Optimization

```
# Git LFS for large files
git lfs install
git lfs track "*.psd"
git lfs track "*.zip"
git lfs track "*.mp4"
git lfs ls-files                # list LFS tracked files
git lfs migrate import --include="*.zip"  # migrate existing files to LFS

# shallow clone for speed
git clone --depth 1 <repository_URL>
git clone --depth 10 <repository_URL>     # last 10 commits
git fetch --unshallow                     # convert to full repository

# partial clone (Git 2.19+)
git clone --filter=blob:none <repo_url>   # clone without file contents
git clone --filter=tree:0 <repo_url>      # clone only commits
git clone --filter=blob:limit=1m <repo_url>  # exclude files larger than 1MB

# performance configuration
git config --global core.preloadindex true
git config --global core.fscache true
git config --global core.untrackedCache true
git config --global gc.auto 256
git config --global pack.threads 0        # use all available CPU cores
git config --global pack.windowMemory 256m
git config --global pack.packSizeLimit 2g

# index optimization
git update-index --split-index             # split index for better performance
git update-index --untracked-cache         # cache untracked files
```

### `.gitignore` Best Practices

```
# Node.js
node_modules/
npm-debug.log*
.env
dist/

# Python
__pycache__/
*.pyc
.venv/
*.egg-info/

# Java
*.class
target/
*.jar
*.war

# IDE and OS
.vscode/
.idea/
*.swp
.DS_Store
Thumbs.db
```

## Git Hooks & Automation

### Pre-commit Hooks

**Hooks** are scripts that run automatically at specific Git events (before commit, after push, etc.).

#### Using Pre-commit Framework

[Pre-commit](https://pre-commit.com/) is a framework for managing multi-language pre-commit hooks that automatically formats code, checks syntax, and runs tests before commits.

```
# install pre-commit (if not already present)
pip install pre-commit

# or via homebrew
brew install pre-commit

# add .pre-commit-config.yaml into your root of Git repository
cd <your_repo>
touch .pre-commit-config.yaml    # add content from below section

# install hooks in repository (note <your_repo>/.pre-commit-config.yaml be present)
cd <your_repo>
pre-commit install

# if you get below error on above command, then unset hooksPath
# [ERROR] Cowardly refusing to install hooks with `core.hooksPath` set.
git config --unset-all core.hooksPath

# Optionally, check all files now by running
pre-commit run --all-files

# if you want to skip pre-commit hooks for a commit
git commit --no-verify -m "your commit message" <file-name>
```

#### Pre-commit Configuration

Create in your repository root:`.pre-commit-config.yaml`

```
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.4.0
    hooks:
      - id: trailing-whitespace        # removes trailing whitespace
      - id: end-of-file-fixer          # ensures files end with newline
      - id: fix-byte-order-marker      # fixes byte order marker
      - id: mixed-line-ending          # ensures consistent line endings
      - id: check-yaml                 # validates YAML syntax
      - id: check-json                 # validates JSON syntax
      - id: check-xml                  # validates XML syntax
      - id: pretty-format-json         # formats JSON files
      - id: check-merge-conflict       # prevents merge conflict markers
      - id: detect-aws-credentials     # detects AWS credentials
      - id: detect-private-key         # detects private keys
      - id: check-added-large-files    # prevents large files from being added
```

Explore more pre-built hooks at [pre-commit-hooks repository](https://github.com/pre-commit/pre-commit-hooks).

#### Manual Pre-commit Hook

```
# create custom pre-commit hook
cat > .git/hooks/pre-commit << 'EOF'
#!/bin/sh
npm test
if [ $? -ne 0 ]; then
  echo "Tests failed. Commit aborted."
  exit 1
fi
EOF
chmod +x .git/hooks/pre-commit
```

### Post-commit Hooks

**Post-commit hooks** run after a successful commit and are useful for notifications, deployments, or logging.

```
# notification hook
cat > .git/hooks/post-commit << 'EOF'
#!/bin/sh
echo "Commit completed: $(git log -1 --pretty=%B)"
EOF
chmod +x .git/hooks/post-commit
```

## Advanced Scenarios

### Multiple Remotes

```
# work with multiple remotes
git remote add upstream <original_repo>
git remote add fork <your_fork>
git remote add backup <backup_repo>

# push to specific remote
git push fork main
git push upstream main
git push --all backup        # push all branches to backup

# fetch from multiple remotes
git fetch --all              # fetch from all remotes
git fetch upstream           # fetch from specific remote

# track different remotes for different branches
git branch --set-upstream-to=upstream/main main
git branch --set-upstream-to=fork/feature feature-branch

# push to multiple remotes simultaneously
git remote set-url --add --push origin <repo1_url>
git remote set-url --add --push origin <repo2_url>
```

### Monorepo Management

**Monorepo** is a single repository containing multiple projects/applications, allowing shared code, unified tooling, and coordinated releases across teams.

```
# subtree operations
git subtree add --prefix=libs/shared <repo_url> main
git subtree pull --prefix=libs/shared <repo_url> main
git subtree push --prefix=libs/shared <repo_url> main
git subtree split --prefix=libs/shared -b shared-branch

# sparse checkout (partial clone)
git clone --filter=blob:none --sparse <repo_url>
git sparse-checkout init --cone
git sparse-checkout set frontend backend/api
git sparse-checkout add docs/
git sparse-checkout list
git sparse-checkout disable

# worktree management (multiple working directories)
git worktree add ../feature-branch feature-branch
git worktree add ../hotfix-branch -b hotfix-branch
git worktree list
git worktree remove ../feature-branch
git worktree prune
```

### Submodules

**Submodules** allow you to include external Git repositories as subdirectories within your main repository, keeping them as separate, independently versioned projects.

```
# add submodule
git submodule add <repo_url> <path>
git submodule add -b <branch> <repo_url> <path>  # track specific branch

# initialize and update
git submodule update --init --recursive
git submodule update --init --recursive --jobs 4  # parallel updates

# update submodules
git submodule update --remote              # update to latest remote
git submodule update --remote --merge      # merge remote changes
git submodule update --remote --rebase     # rebase on remote changes

# submodule management
git submodule status                       # show submodule status
git submodule foreach git pull origin main # run command in all submodules
git submodule foreach git status          # check status of all submodules

# remove submodule
git submodule deinit <path>
git rm <path>
rm -rf .git/modules/<path>
```

## 现实世界的例子

### 开源贡献

```
# 1. Fork repository on GitHub
# 2. Clone your fork
git clone <your_fork_url>
cd <repository>

# 3. Add upstream remote
git remote add upstream <original_repo_url>

# 4. Create feature branch
git checkout -b fix/issue-123

# 5. Make changes and commit
git add .
git commit -m "fix: resolve issue #123"

# 6. Push to your fork
git push origin fix/issue-123

# 7. Create Pull Request on GitHub
```

### 热修复部署

```
# emergency hotfix workflow
git checkout main
git pull origin main
git checkout -b hotfix/critical-bug

# implement fix
git add .
git commit -m "hotfix: resolve critical security issue"

# merge to main
git checkout main
git merge hotfix/critical-bug

# merge to develop
git checkout develop
git merge hotfix/critical-bug

# tag and deploy
git tag -a v1.0.1 -m "hotfix release v1.0.1"
git push origin main develop --tags

# cleanup
git branch -d hotfix/critical-bug
```

## 快速参考

### 基本指令

```
# Status and Information
git status              # working directory status
git status -s           # short status format
git log --oneline       # commit history
git log --graph --all   # visual branch history
git diff                # show changes
git diff --staged       # staged changes
git diff HEAD~1         # compare with previous commit
git blame <file>        # file annotations
git show <commit>       # show commit details

# Branching Quick Commands
git branch -a           # list all branches
git branch -r           # remote branches only
git branch -v           # branches with last commit
git checkout -          # switch to previous branch
git switch <branch>     # modern branch switching (Git 2.23+)
git switch -c <branch>  # create and switch to new branch
git merge --no-ff       # merge with merge commit

# Remote Operations
git fetch --all         # fetch all remotes
git fetch --prune       # remove deleted remote branches
git remote prune origin # clean remote references
git push --force-with-lease  # safer force push
git pull --rebase       # pull with rebase instead of merge

# File Operations Quick Reference
git add -A              # add all changes (new, modified, deleted)
git add -u              # add only modified and deleted files
git add -p              # interactively add parts of files
git restore <file>      # discard changes (Git 2.23+)
git restore --staged <file>  # unstage file
git rm --cached <file>  # remove from Git but keep local file
```

### 有用的别名

```
# create helpful aliases
git config --global alias.st "status"
git config --global alias.co "checkout"
git config --global alias.sw "switch"
git config --global alias.br "branch"
git config --global alias.cm "commit -m"
git config --global alias.ca "commit --amend"
git config --global alias.lg "log --oneline --graph --all"
git config --global alias.ll "log --oneline --graph --decorate --all"
git config --global alias.unstage "reset HEAD --"
git config --global alias.last "log -1 HEAD"
git config --global alias.visual "!gitk"
git config --global alias.type "cat-file -t"
git config --global alias.dump "cat-file -p"
git config --global alias.hist "log --pretty=format:'%h %ad | %s%d [%an]' --graph --date=short"
```

### Git 工具集成

```
# VS Code integration
git config --global core.editor "code --wait"

# Built-in GUI
gitk --all              # repository browser
git gui                 # commit tool
```

**常用图形界面工具：**

- [SourceTree](https://www.sourcetreeapp.com/)（免费）
- [GitKraken](https://www.gitkraken.com/)（付费）
- [GitHub桌面](https://desktop.github.com/)版（免费）
- 塔[台](https://www.git-tower.com/)（付费）
- [分叉](https://git-fork.com/)（付费）

## 常见问题解答

### Git 和 GitHub 有什么区别？

**Git** 是本地跟踪代码变更的版本控制系统。**GitHub** 是一个基于云的托管服务，用于 Git 仓库，增加了协作功能，如拉取请求、问题和项目管理工具。

### 我该如何在Git中撤销上一次提交？

用来撤销上一次提交，同时保持更改分阶段，或者完全移除提交和所有更改。`git reset --soft HEAD~1``git reset --hard HEAD~1`

### git pull 和 git fetch 有什么区别？

- **`git fetch`** 会从远程仓库下载更改，但不会合并到你当前的分支中
- **`git pull`** 会下载更改并自动合并到你当前的分支中 （`git pull = git fetch + git merge`)

### 我如何在Git中解决合并冲突？

1. Git 会用冲突标记冲突文件（， ，`<<<<<<<``=======``>>>>>>>`)
2. 手动编辑文件以解决冲突
3. 移除冲突标记
4. 跑`git add <resolved-files>`
5. 完成合并后`git commit`

### 我应该用合并还是重新基准来整合变更？

- 在公共分支和想保留提交历史时**使用merge**
- 私有功能分支**使用rebase**，可以创建更清晰、线性的历史
- 永远不要重基被推送到共享仓库的提交

### 我如何在本地远程删除 Git 分支？

```
# Delete local branch
git branch -d branch-name

# Delete remote branch
git push origin --delete branch-name
```

### 什么是分离的HEAD状态，我该如何解决？

分离主干部发生在你检查某个具体提交而不是分支时。要修正：

```
# Create a new branch from current position
git checkout -b new-branch-name

# Or return to main branch
git checkout main
```

### 我怎么看到 Git 里文件变了？

```
git status          # See staged/unstaged changes
git diff            # See unstaged changes
git diff --staged   # See staged changes
git log --stat      # See files changed in commits
```

### 团队使用的最佳Git工作流程是什么？

- **GitHub Flow**：简单、持续部署（功能分支→主功能）
- **Git Flow**：带有开发/主分支的结构化发布
- **GitLab Flow**：基于环境的分支（生产、预设、功能）

请根据您的团队规模、发布频率和部署策略进行选择。

### 我第一次配置Git该怎么做？

```
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
git config --global init.defaultBranch main
git config --global credential.helper osxkeychain  # macOS
```

### 我可以在 Git 中恢复被删除的提交吗？

是的，用来查找提交哈希，然后：`git reflog`

```
git reflog                    # Find lost commit
git checkout <commit-hash>    # Go to that commit
git checkout -b recovery-branch  # Create branch to save it
```

### 我该如何忽略已经被追踪的文件？

```
# Add file to .gitignore first
echo "filename.txt" >> .gitignore

# Remove from tracking but keep local file
git rm --cached filename.txt
git commit -m "Stop tracking filename.txt"
```

## 最佳实践总结

### 提交指南

- 使用命令语气：“添加功能”而不是“添加功能”
- 首行保持在50个字符以内
- 使用常规提交：， ， ， ，`feat:``fix:``docs:``refactor:`
- 参考问题：“修复 #123”
- 做原子提交（每次提交一次逻辑更改）

### 分支策略

- 使用描述性分支名称：`feature/user-authentication`
- 保持分支小而集中
- 请立即删除合并的分支
- 定期与主分支同步
- 在团队环境中使用分支保护规则

### 安全清单

- 切勿提交秘密或凭证
- 敏感文件的使用`.gitignore`
- 用GPG签署重要提交
- 推送前请先审查更改
- 远程连接时使用HTTPS或SSH
- 启用双因素认证

### 表演技巧

- 对于大型仓库，使用浅克隆
- 配置二进制文件的 Git LFS
- 定期维护：`git gc`
- 对单仓库使用稀疏借出
- 优化 Git 配置以适应你的工作流程

## 结论

掌握**Git工作流程**对于现代软件开发和团队协作至关重要。这份全面指南涵盖了从基础Git命令到高级技巧的方方面面，为你提供了48个必备的Git命令，以及应对各种版本控制场景的知识。

### 主要要点：

- **从基础开始**：先掌握基础Git作，再逐步进入复杂的工作流程
- **选择合适的工作流程**：结构化发布用 Git Flow，持续部署用 GitHub Flow
- **优先考虑安全**：使用个人访问令牌和GPG签名以实现安全开发
- **优化性能**：利用 Git LFS、浅克隆和正确的配置
- **有效排查**：了解如何从常见的Git问题中恢复
- **维护仓库**：定期清理和优化能让 Git 顺畅运行
- **高效搜索**：使用 git grep、git log filters 和 git bisect 进行有效调试

### 下一步：

1. **定期练习**——每天使用Git来建立肌肉记忆
2. **探索高级功能**——尝试钩子、子模块和自动化
3. **加入社区**——为开源项目贡献力量，积累真实世界经验
4. **保持更新**——关注Git发布和新的工作流程模式

### 相关开发者资源：

- **[macOS 全新安装安装指南](https://sagarnikam123.github.io/posts/macos-fresh-install-setup-guide/)**——完整的开发环境设置，包括 Git 配置
- **[Ubuntu 全新安装安装指南](https://sagarnikam123.github.io/posts/ubuntu-fresh-install-setup-guide/)**——包含 Git、Java 和 DevOps 工具的基础 Ubuntu 开发环境
- **[Linux 故障排除命令指南](https://sagarnikam123.github.io/posts/linux-troubleshooting-commands/)** - 管理 Git 服务器和开发环境的必备 Linux 命令

**准备好提升你的Git技能了吗？**收藏这份指南，与你的团队分享，并从今天就开始在你的项目中实施这些工作流程。高效的版本控制是成功软件开发的基础。

祝你版本制作愉快！🌿✨