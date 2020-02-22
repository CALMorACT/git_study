# Git 笔记记录


## Git 基操

1. `git init` 初始化版本库
2. `git status -sb` 显示当前所有文件状态
3. `git diff` 查看文件更改记录
4. `git add <文件路径>` 将文件加入暂存区
   - 文件路径为 . 时, 及将所有文件加入
5. `git commit -m <提交信息>` 进行版本提交和记录

   - 提交信息有一定规则, 以下为基础标题格式
   - `<type>(<scope>): <subject>`

     - type: 类型, 区分提交的格式

       ```text
       feat：新功能（feature）
       fix：修补bug
       docs：文档（documentation）
       style： 格式（不影响代码运行的变动）
       refactor：重构（即不是新增功能，也不是修改bug的代码变动）
       test：增加测试
       chore：构建过程或辅助工具的变动
       ```

     - scope: 用于说明 `commit` 影响的范围，比如数据层(data layer) 控制层(control layer) 视图层(view layer)等等，视项目不同而不同

     - subject: 用于对 commit 内容的简单解释
       1. 以动词开头，使用第一人称现在时，比如 change，而不是 changed 或 changes
       2. 第一个字母小写
       3. 结尾不加句号 (.)

6. `git log` 查看变更日志

## 版本回退

- `git reset --hard <commitid>`

  - 使用 commitid 则为指定版本的 id 记录(一串 hash)
  - 不使用 commitid 则为上一个版本
  - `reset` 操作会让本地代码库丧失掉对舍弃版本的控制, 对于在 log 中舍弃的版本(比如从版本三跳到了版本二, 想要再跳到版本三)需要使用 reflog 查找每一步操作的 Hash 值, 即`git reflog`
  - --hard 参数用于控制硬链接, 而非仅仅软链接

- `git revert <HEAD>/<commitid>`

  - 使用 commitid 指定版本
  - HEAD 即上一个版本 HEAD^上上个版本
  - 对于 revert 命令, 其特色是并不回退而是将之前某次的提交作为一次新的提交产生回退的效果, 但所有的操作都可以直接通过 log 查看, 而无需 reflog

- 按照具体情形来说
  - 如果已经进行了 commit, 想要回退, 一般会选择在本地版本调整时使用 `reset` (方便精简远程分支), 而在远程分支调整时使用 `revert`
  - 如果文件没有加入暂存区(`git add`)但你进行了更改想要回退, 直接进行 `git checkout -- <filename>`, 将文件切回到上一次提交的状态
  - 如果文件加入了暂存区, 先丢弃暂存 `git reset HEAD <filename>`, 再执行 `git checkout -- <filename>` 切回到原状态

## 引用

- 策略一: 直接引用 Hash 值 即`<commitid>`
- 策略二: 相对引用, 分为两种
  1. 使用 `^` 表示上跳一级(可以累加)
  2. 使用 `~<num>` 表示上跳 num 级
- 策略三: 让分支基于一个 commit 或者基于一个 branch 进行跳转使用 `git branch -f <branchname> <commitid/branchname><^/~num>`

## 删除文件

- `git rm <filename>` 这样会直接删除文件, 并将其删除状态放入暂存区
- 如果发现删错了, 可以直接通过切到上一次提交的版本即`git checkout -- <filename>`

## 分支

- 对于分支有 `git checkout <branchname>` `git switch <branchname>` `git branch <branchname>`
- 创建新分支: `checkout -b` `switch -n` (-b 即 branch 的意思指代 new-branch ), checkout 本质上就是对 HEAD 的操作 `-b` 参数指的是新建分支的同时也激活新分支, 不带 `-b` 参数的话就仅仅是创建新分支, HEAD 指针不会变动
- 删除分支: `branch -d` (合并后删除), 如果未合并就要删除就使用 `-D` 参数
- 合并分支: `merge`, 合并分支时，加上`--no-ff`参数就可以用普通模式合并(默认以 fast-forward 模式合并)，合并后的历史有分支，即使删除分支也能看出来曾经做过合并，而 fast forward 合并就看不能
- 团队分支管理
  - master 分支作为最稳定的分支, 只有版本功能确定时候更改
  - 而开发在 dev 分支(或者其他名字)上进行
  - 所有团队成员的操作在 dev 分支上在创建自己的分支来增加功能
- Bug 修复

  1. 先冻结开发分支 `git stash`
  2. 切到 master 上, 再从 master 上创建新分支修复 Bug
  3. 修复好后, 提交到 master 上(使用 `--no-ff` )

- 远程协作
  - 分支在远程并且和本地有链接, 且分支在本地为最新无冲突
    - 使用 `git push origin <branchname>`
  - 分支在远程并且和本地有链接, 但分支在本地非最新且无冲突
    - 使用 `git pull` 进行更新, 然后检查再推送
  - 分支在远程并且和本地有链接, 但远程分支和本地有冲突
    - 在本地整理自己的版本, 然后再推送
  - 分支不在远程/和自己本地版本库无链接 (提示 'no tracking information')
    - 将云端分支和本地进行链接 `git branch --set-upstream <branchname> origin/<branchname>`
    - 在本地创建和远端关联的分支 `git branch -b <branchname> origin/<branchname>`

## 标签管理

- 作用: 为指定的 commit 做标记, 可以作为版本标记使用

- 创建 TAG

  - `git tag <tagname> <commitid>`, commitid 不指定的话默认为 HEAD
  - 增加 TAG 描述信息, `git tag -a <tagname> -m <info>`
  - 增加 TAG PGP 签名, `git tag -s <tagname> -m <PGP>`
  - 增加信息和创建过程合并: `git tag -a <tagname> -m <info> <commitid>`

- 参看 TAG: `git tag`

- 操作标签
  - 推送到远程
    - `git push` 参数 `<place>` 设置成 `<tagname>`, 即 `git push <remote> <tagname>`
    - `git push <remote> --tags` 把本地所有 tag 都推送上去
  - 删除本地
    - `git tag -d <tagname>`
  - 删除远程
    1. `git tag -d <tagname>`
    2. `git push origin :refs/tags/<tagname>` refs/tags 指的是路径

## 远程仓库

- `git clone`
  - 从远程获得 `origin/<branchname>` 等远程分支
  - 克隆获得远程版本库, 如果版本库不是自己的无法获取的话, 得不到 `origin/<branchname>` 分支
- `git fetch`
  - 让本地的 `origin/<branchname>` 分支和远程同步
  - 仅仅同步本地 origin 的分支中的修改记录, 并不会修改本地文件, 即本地 master 分支没有变动
  - 不指定具体的分支的话是同步所有的 `origin/..` 分支
- `git pull`
  - 让本地的 `origin/<branchname>` 分支和远程同步, 同时将本地分支和`origin/..`分支合并
  - 等同于 `git fetch` + `git merge origin/master`

## 本地和远程发生冲突

- 当远程改变而本地没有变且进行了开发, 就需要进行更改操作

  - 需要先把 origin 分支同步到和远程一致 (`git fetch`), 手动处理好合并冲突的部分

  - 然后将 origin 分支和本地开发后的 master 分支合并, 有以下方案

    - 使用 rebase 将开发后的分支和 origin 合并
    - 使用 merge 将开发后的分支和 orgin 合并

  - 然后推送到云端 (`git push`)

    - 使用 rebase 合并的话, 如果推送到远程, 仅仅会推送目前操作的 branch, 丢弃掉之前操作的未更新的本地冲突 commit
      ![2020-02-20 git pull --rebase](https://i.loli.net/2020/02/20/8D6V5KnpGS2H9kf.png)
    - 使用 merge 合并的话, 如果推送到远程, 会全部保留提交过程, 暴扣之前操作的未更新的本地冲突 commit
      ![2020-02-20 git pull](https://i.loli.net/2020/02/20/Uw6Hc8pJCSjsfRt.png)

  - 命令合并:

    - `git pull --rebase` = `git fetch` + `git rebase`
    - `git pull` = `git fetch` + `git merge`

  - 两种方式的区别
    - `rebase` 可以保持更加干净的远程提交树, 更好观察, 但会在远程丢失完整的提交记录
    - `merge` 可以保证远程分支同样保持完整的提交记录, 但会使得远程分支太过复杂

## 远程追踪

- 建立新分支追踪远程的分支 `git checkout -b <branchname> origin/<branchname>`
- 让已有分支追踪远程分支 `git branch -v <branchname> origin/<branchname>`

## git push/pull/fetch 参数

- 统一格式 `git push/pull/fetch <remote> <place>`

  - `<place>`: 切到本地仓库中的 `<place>` 分支，获取所有的提交，再到远程仓库 `<remote>` 中找到和 `<place>` 追踪的远程分支(不指定的话和 `<place>` 相同), 对两者进行对应操作
  - `<remote>`: 一般情况指远程仓库的名称, 一般为 origin
  - 如果不指定 `<place>` 参数的话, 会认为是当前 HEAD 指向的位置
  - `<place>` 参数不仅仅可以使用分支也可以使用 commit, 同样操作到具体的位置
  - 如果不想设定异姓追踪后再操作, 可以直接将 `<place>` 替换为 `<source>:<destination>` 参数

- 独特的 `<source>` 参数

  - 一旦 `<source>` 参数置空, push 就会删掉远程的 `<destination>` 分支, fetch 就会在本地创建 `<destination>` 分支

- `git pull <remote> <place>` = `git fetch <remote> <place>` + `git merge <place>`
- `git pull <remote> <source>:<destination>` = `git fetch <remote> <source>:<destination>` + `git merge <destination>`

## 一些对于 git 的特殊描述文件

- .gitignore
  - [github: A collection of useful .gitignore templates](https://github.com/github/gitignore)
- .gitattributes
  - [github: Configuring Git to handle line endings](https://help.github.com/cn/github/using-git/configuring-git-to-handle-line-endings)

---

**部分摘自** [掘金: 从只会 git add. 的菜鸟到基本掌握 git 命令](https://juejin.im/post/5abef8356fb9a028df22bd78#heading-8)
**部分总结自** [Learning Git Branching](https://learngitbranching.js.org/)
