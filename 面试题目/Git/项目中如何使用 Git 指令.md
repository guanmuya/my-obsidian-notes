# 项目中如何使用 Git 指令

Git 是项目开发中最常用的版本控制工具。对一个项目而言，Git 不只是用来“保存代码”，更重要的是记录每一次修改、支持多人协作、方便回滚错误，并让项目可以稳定地发布和维护。

本文按照一个项目从创建到日常开发、多人协作、问题回退和版本发布的流程，总结常见 Git 指令的使用方式。

## 一、Git 在项目中的作用

在项目开发中，Git 主要解决以下问题：

1. 记录项目每一次有意义的修改。
2. 可以查看谁在什么时候改了什么内容。
3. 可以在不同功能之间使用分支隔离开发。
4. 可以把本地代码同步到 GitHub、Gitee 等远程仓库。
5. 出现错误时，可以回退到之前的稳定版本。

一个好的习惯是：不要等到写完一大堆内容才提交，而是每完成一个相对完整的小功能或修改，就提交一次。

## 二、项目初始化

如果一个文件夹还不是 Git 仓库，可以进入项目目录后执行：

```bash
git init
```

这个命令会在当前目录下创建一个 `.git` 隐藏文件夹，用来保存 Git 的版本记录。

如果项目已经存在于 GitHub，可以直接克隆远程仓库：

```bash
git clone 仓库地址
```

例如：

```bash
git clone https://github.com/用户名/仓库名.git
```

克隆后会得到一个已经关联远程仓库的本地项目。

## 三、查看项目状态

开发过程中最常用的命令是：

```bash
git status
```

它可以告诉我们：

1. 哪些文件被修改了。
2. 哪些文件是新建但还没有被 Git 跟踪的。
3. 哪些文件已经被暂存。
4. 当前分支是否领先或落后远程分支。

简洁版状态可以使用：

```bash
git status --short
```

如果想看当前在哪个分支上，可以使用：

```bash
git branch
```

或者：

```bash
git branch --show-current
```

## 四、查看修改内容

在提交之前，应该先检查自己具体改了什么。

查看工作区中还没有暂存的修改：

```bash
git diff
```

查看已经暂存、准备提交的修改：

```bash
git diff --staged
```

查看某一个文件的修改：

```bash
git diff 文件路径
```

这一步很重要，因为它可以避免把临时调试代码、隐私信息或错误文件提交进去。

## 五、暂存文件

Git 的提交分为两步：先暂存，再提交。

暂存某个文件：

```bash
git add 文件路径
```

暂存当前目录下所有修改：

```bash
git add .
```

暂存整个仓库的所有新增、修改和删除：

```bash
git add -A
```

一般项目中推荐先用 `git status` 和 `git diff` 检查，再用 `git add` 暂存需要提交的文件。

## 六、提交修改

暂存完成后，使用 `git commit` 生成一次版本记录：

```bash
git commit -m "提交说明"
```

例如：

```bash
git commit -m "新增用户登录页面"
```

提交说明应该简洁明确，能说明这次提交做了什么。

比较好的提交说明示例：

```bash
git commit -m "修复登录接口参数错误"
git commit -m "新增 Git 学习笔记"
git commit -m "优化首页布局"
```

不太好的提交说明：

```bash
git commit -m "修改"
git commit -m "更新一下"
git commit -m "111"
```

## 七、连接远程仓库

如果本地项目还没有连接 GitHub 仓库，可以添加远程地址：

```bash
git remote add origin 仓库地址
```

查看当前远程仓库：

```bash
git remote -v
```

如果远程地址写错了，可以修改：

```bash
git remote set-url origin 新仓库地址
```

其中 `origin` 是远程仓库的常见默认名称，也可以使用其他名称。

## 八、推送到远程仓库

本地提交完成后，可以推送到远程仓库：

```bash
git push
```

如果是第一次推送当前分支，通常需要指定远程分支并建立关联：

```bash
git push -u origin main
```

含义是：把本地 `main` 分支推送到远程 `origin`，并让以后直接执行 `git push` 就能推送到这个远程分支。

如果当前项目使用的是 `master` 分支，则命令可能是：

```bash
git push -u origin master
```

## 九、拉取远程更新

多人协作时，别人可能已经把新代码推送到了远程仓库。自己开发前，最好先拉取最新代码：

```bash
git pull
```

它相当于从远程获取更新，并合并到当前分支。

也可以分成两步：

```bash
git fetch
git merge
```

`git fetch` 只获取远程更新，不会直接修改当前代码；`git merge` 才会把远程更新合并进当前分支。

## 十、分支管理

分支可以让不同功能互不影响。实际项目中，通常不会直接在主分支上开发新功能，而是创建功能分支。

查看所有本地分支：

```bash
git branch
```

创建新分支：

```bash
git branch 分支名
```

切换分支：

```bash
git switch 分支名
```

创建并切换到新分支：

```bash
git switch -c 分支名
```

例如：

```bash
git switch -c feature/login
```

当功能开发完成后，可以切回主分支：

```bash
git switch main
```

然后合并功能分支：

```bash
git merge feature/login
```

删除已经合并的本地分支：

```bash
git branch -d feature/login
```

如果分支没有合并但仍要强制删除：

```bash
git branch -D 分支名
```

强制删除要谨慎使用。

## 十一、解决冲突

当多人修改了同一个文件的同一部分时，合并或拉取代码可能出现冲突。

出现冲突后，先查看状态：

```bash
git status
```

冲突文件中通常会出现类似内容：

```text
<<<<<<< HEAD
当前分支的内容
=======
其他分支的内容
>>>>>>> 分支名
```

解决方式是手动编辑文件，保留正确内容，并删除这些冲突标记。

解决后执行：

```bash
git add 冲突文件
git commit
```

如果是在 `git pull` 或 `git merge` 过程中产生的冲突，提交后合并过程就完成了。

## 十二、查看提交历史

查看提交记录：

```bash
git log
```

简洁查看：

```bash
git log --oneline
```

查看带分支图的历史：

```bash
git log --oneline --graph --decorate --all
```

查看某个文件的历史：

```bash
git log 文件路径
```

查看某次提交的具体内容：

```bash
git show 提交哈希
```

提交哈希就是 `git log` 中每次提交前面的一串字符。

## 十三、撤销和回退

### 1. 撤销工作区修改

如果某个文件改错了，还没有暂存，可以恢复到上一次提交的状态：

```bash
git restore 文件路径
```

恢复所有未暂存修改：

```bash
git restore .
```

这个命令会丢弃当前修改，使用前要确认不再需要这些内容。

### 2. 取消暂存

如果已经 `git add`，但还没有提交，可以取消暂存：

```bash
git restore --staged 文件路径
```

取消所有暂存：

```bash
git restore --staged .
```

### 3. 回退到某个提交

如果想创建一个新的提交来撤销之前某次提交，推荐使用：

```bash
git revert 提交哈希
```

`git revert` 不会破坏提交历史，适合已经推送到远程仓库的项目。

如果只是本地提交，还没有推送，可以使用：

```bash
git reset --soft 提交哈希
```

它会回到指定提交，但保留修改内容在暂存区。

也可以使用：

```bash
git reset --hard 提交哈希
```

这个命令会彻底丢弃指定提交之后的修改，非常危险，使用前一定要确认。

## 十四、标签和版本发布

当项目发布一个稳定版本时，可以打标签：

```bash
git tag v1.0.0
```

查看标签：

```bash
git tag
```

推送标签到远程：

```bash
git push origin v1.0.0
```

推送所有标签：

```bash
git push origin --tags
```

标签常用于标记正式版本，例如 `v1.0.0`、`v1.1.0`、`v2.0.0`。

## 十五、项目中的推荐工作流程

一个比较常见的个人项目流程：

```bash
git status
git pull
git switch -c feature/功能名
```

开发完成后：

```bash
git status
git diff
git add .
git commit -m "说明本次修改"
git push -u origin feature/功能名
```

如果功能需要合并到主分支：

```bash
git switch main
git pull
git merge feature/功能名
git push
```

一个比较常见的团队项目流程：

1. 从主分支拉取最新代码。
2. 创建自己的功能分支。
3. 在功能分支上开发和提交。
4. 推送功能分支到远程仓库。
5. 在 GitHub 上创建 Pull Request。
6. 代码审查通过后合并到主分支。
7. 删除已经合并的功能分支。

## 十六、常用命令速查

```bash
# 初始化仓库
git init

# 克隆仓库
git clone 仓库地址

# 查看状态
git status

# 查看修改
git diff

# 暂存文件
git add 文件路径
git add .
git add -A

# 提交修改
git commit -m "提交说明"

# 查看远程仓库
git remote -v

# 添加远程仓库
git remote add origin 仓库地址

# 拉取远程更新
git pull

# 推送到远程
git push
git push -u origin main

# 查看分支
git branch

# 创建并切换分支
git switch -c 分支名

# 切换分支
git switch 分支名

# 合并分支
git merge 分支名

# 查看提交历史
git log --oneline

# 查看某次提交
git show 提交哈希

# 撤销未暂存修改
git restore 文件路径

# 取消暂存
git restore --staged 文件路径

# 撤销某次提交
git revert 提交哈希

# 打标签
git tag v1.0.0
```

## 十七、使用 Git 的注意事项

1. 提交前先执行 `git status` 和 `git diff`。
2. 不要把密码、密钥、Token、`.env` 等敏感文件提交到仓库。
3. 每次提交只做一类事情，不要把无关修改混在一起。
4. 提交说明要能看懂，不要写成“修改”“更新”“测试”。
5. 多人协作时，开发前先 `git pull`。
6. 新功能尽量在独立分支开发。
7. 已经推送到远程的提交，不要随意使用会改写历史的命令。
8. `git reset --hard` 和强制推送要非常谨慎。

## 十八、总结

对一个项目而言，Git 的核心流程可以概括为：

```text
查看状态 -> 检查修改 -> 暂存文件 -> 提交记录 -> 推送远程
```

多人协作时，再加上：

```text
拉取更新 -> 创建分支 -> 开发功能 -> 提交推送 -> 合并分支
```

真正掌握 Git，不是背下所有命令，而是理解每个命令在项目流程中的位置。只要养成“先查看、再修改、提交前检查、提交后同步”的习惯，就能在大多数项目中安全、高效地使用 Git。
