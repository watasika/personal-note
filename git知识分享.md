#  git高级命令

## 变基

### 基础概念

​	变基（rebase）：即**改变**当前分支的**基础分支**版本（父版本）

​	通俗的说：如同盖高楼一般，最下面的是地基，一层层往上盖，每一层都可以称为下一层的基。虽然现实中改变地基是比较困难的事，但是在git中，能够方便使用rebase命令进行变基操作。

### 为什么要使用变基操作

​	有一个显而易见的原因是——代码项目的开发跟盖楼是差不多的，项目复杂度上升就难免会有错误产生，有时候已经产生了数个提交，但发现之前的某次提交中修改的文件有一定的问题，此时可以使用rebase命令方便地更改。

​	另一个原因则是基于程序员的“洁癖”，它能让提交的代码历史记录显得格外清晰，并非说这种习惯是不好的，相反，正是由于这种良好的“洁癖”，才能在将来遇到某些事件需要回看历史记录的时候能够更清晰定位到问题的所在。

### 具体操作

- 基础用法：

  ​	rebase的基础用法之一就是可以像merge一样合并分支。

  ​	语法：`git rebase <basebranch> [<topicbranch>]`

  ​	`<basebranch>`：作为基底的提交记录（基底分支）

  ​	`<topicbranch>`：将要变基的提交记录（主题分支）

  ​	如果想要分支*bugFix*合并到分支*feature*，可以键入`git rebase bugFix feature`，合并完成之后HEAD会到`topicbranch`分支上。

  ​	如果省去`topicbranch`参数（即去掉feature），则以当前分支作为目标分支。

  ​	rebase将一个分支并到另一个分支发生的事：从分支指向的提交记录向前追溯，直到找到两个分支的交汇点（即公共的父节点），此时将这若干提交记录作为主题分支的“基”，变动到主题分支上，从宏观上来看，更像是将主题分支的提交记录“嫁接”到基底分支上。

  ​	git会如此做的原因是：我们总是希望自己的更改是基于其他人的更改之后，以此来避免错误的发生。

  <img src="D:\soft\gif录制工具\保存文件\rebase合并.gif" alt="rebase合并" style="zoom: 67%;" />

- 衍生命令：`git pull --rebase`

  ​	在需要与远程同步的时候，使用–rebase子命令会使得提交记录形成简洁的线性发展，但有个缺点就是：rebase以后就无法得知当前分支最早的真正“基底”分支了。 

  > C4												（develop分支）
  >
  > ​	↘						
  >
  > ​		C3->C2->C1
  >
  > ​	↗
  >
  > C5												（featrue分支）

  ​	同时，如果是git pull，那么合并后的历史记录可能会呈现这样的现象：

  > 以线状图看：
  >
  > C6→C5→C4→C3→C2→C1			（featrue分支）
  >
  > 以树状图看（见下gif图）
  >
  > 
  >
  > C6这个新提交是自动生成的，用于提示merge的产生，同时包含了合并后的内容

  ​	如果是rebase方式合并，那么是这样的：

  > C5’→C4→C3→C2→C1				（featrue分支）

  ​	两种方式拉取合并各有优缺点，很难说哪种更好，实际上不过是两种风格的差异：简洁派——更想以完美的形式呈现提交历史，而非冗杂的草稿；严谨派——目的是记录过程，无论是好是坏，都有其存在的意义。

  | `git pull`          | ![pull-merge](D:\soft\gif录制工具\保存文件\pull-merge.gif)   |
  | ------------------- | ------------------------------------------------------------ |
  | `git pull --rebase` | <img src="D:\soft\gif录制工具\保存文件\pull-rebase.gif" alt="pull-rebase"  /> |

  

- 交互式界面：`git rebase -i <startpoint> [endpoint]` 

  - `~<num>`与 ^ 的简单介绍

   ​	通常提交记录的哈希值有很长一串（如：fed2da64c0efc5293610bdd892f82a58e8cbc5d8），尽管可以通过前几个字符来唯一标识，但对于在命令行中键入多个哈希值仍然是一件麻烦事。如果通过git提交记录的哈希值来设定某些操作的步长或者影响范围感到不方便的时候，可以使用相对引用（^或`~<num>`）。

    - 使用 ^ 向前移动 1 个提交记录
    - 使用 `~<num>` 向前移动多个提交记录，如 `~3`

    ​	例如：`git checkout main^`或者`git branch -f main HEAD~3`（将 main 分支强制指向 HEAD 的第 3 级父提交）

    ​	同样，在`git rebase -i`中也可以通过此方法来快速界定影响的commit提交记录的范围，只需要输入例如`git rebase -i HEAD~3`。

  - rebase交互式界面

    语法中：`<startpoint> <endpoint>`区间指定的是一个前开后闭的区间，如果不指定`[endpoint]`，则该区间的终点默认是当前分支`HEAD`所指向的`commit`
    
    | commands | 解释                                                         |
    | :------- | ------------------------------------------------------------ |
    | `pick`   | `pick`只是意味着包括提交。重新进行命令时，重新安排`pick`命令的顺序会更改提交的顺序。如果选择不包括提交，则应删除整行。 |
    | `reword` | 该`reword`命令与相似`pick`，但是使用后，重新设置过程将暂停并为您提供更改提交消息的机会。提交所做的任何更改均不受影响。 |
    | `edit`   | 如果您选择`edit`提交，则将有机会修改提交，这意味着您可以完全添加或更改提交。您还可以进行更多提交，然后再继续进行变基。这使您可以将大型提交拆分为较小的提交，或者删除在提交中所做的错误更改。 |
    | `squash` | 该命令使您可以将两个或多个提交合并为一个提交。提交被压缩到其上方的提交中。Git使您有机会编写描述这两个更改的新提交消息。 |
    | `fixup`  | 这类似于`squash`，但是要合并的提交已丢弃其消息。提交仅合并到其上方的提交中，并且较早提交的消息用于描述这两个更改。 |
| `exec`   | 这使您可以对提交运行任意的Shell命令。                        |
    | `drop`   | 删除该commit                                                 |
    
    ​	如果需要对commit的顺序进行调整，只需要将最上面的几条commit信息以vim命令手动调整（剪切【dd】粘贴【p/P】）即可，命令模式下wq保存，cq退出vim，如果仍然在rebase过程中，还需执行`git rebase --abort`中断rebase。
    
    ​	演示操作：首先`git rebase -i HEAD~2`代表前两次提交做rebase，通过vim修改pick为edit或者e，命令模式下wq保存，此时进入rebasing状态中，这个阶段可以直接对文件更改（或是执行`git commit --amend`直接更改`commit message`），然后`git add .`推入暂存区，此时是不能进行`git commit`的，需要输入`git rebase --continue`来说明对工作满意，然后继续前进，此时会再有一个vim弹出，对`commit message`做更改，更改完毕之后wq保存退出，至此rebase完成。

### 警告：

​	变基最强大也是最危险之处就在于：它能**改变原commit的hashId**。而一旦对历史提交做出改变，那么**从变基那个节点开始往后的所有节点的commit id 都会发生变化**。这对线上环境来说是不可控的。

​	关于git rebase的黄金法则就是*永远不要在公共分支上使用它*

>如果提交存在于你的仓库之外，而别人可能基于这些提交进行开发，那么不要执行变基。
>
>如果你遵循这条金科玉律，就不会出差错。

​	***为什么？***

>变基操作的实质是丢弃一些现有的提交，然后相应地新建一些内容一样但实际上不同的提交。 如果你已经将提交推送至某个仓库，而其他人也已经从该仓库拉取提交并进行了后续工作，此时，如果你用 `git rebase` 命令重新整理了提交并再次推送，你的同伴因此将不得不再次将他们手头的工作与你的提交进行整合，如果接下来你还要拉取并整合他们修改过的提交，事情就会变得一团糟。

## 修改历史提交

### 修改最近一次提交

- `git commit --amend`

​	amend 的意思是补丁，**它可以把我们这一次的修改合并到上一条历史记录当中，同时该操作会改变原来的commit id**。键入命令进入vim编辑模式，对应操作即可。

​	如果只修改commit信息，可以使用`git commit --amend -m "提交描述"` （一般只做这个操作）。

### 修改多次提交

- `git rebase -i <startpoint> [endpoint]`

  上面已经叙述过了，详见[变基操作](###具体操作)

## 时空穿梭

### 概念

​	即版本回退。同时以下两个命令可以用作修改历史提交

### 基本用法

- `git reset [--soft | --mixed | --hard] [HEAD | commit id | hashcode]`

  `git reset`命令用于回退版本，可以指定退回某一次提交的版本，也可使用相对定位

  1、git reset --mixed：

  ​	此为默认方式，等同于不带任何参数的git reset。

  2、git reset --soft：

  ​	回退到某个版本，只回退了commit的信息，如果还要提交，直接commit即可（修改的内容变成未add的状态），索引（暂存区）和工作目录的内容是不变的，在三个命令中对现有版本库状态改动最小。

  3、git reset --hard：

  ​	彻底回退到某个版本，本地的源码也会变为上一个版本的内容，所有修改的内容都会丢失（修改的代码不会变成未add的状态）。![点击查看图片来源](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fimg2020.cnblogs.com%2Fblog%2F615156%2F202012%2F615156-20201228110755674-1288639130.png&refer=http%3A%2F%2Fimg2020.cnblogs.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1668814089&t=ebbf7545172d44da96cd2d2196fb7184)

- `git revert <commit id>`

  ​	git 会生成一个新的commit，将指定的commit内容从当前分支上撤除（在回滚的同时保留 commit 记录）。

  ​	当公共分支commit信息混合到一起后，此时使用`git reset`命令很难找到对应的父节点，但是`git revert`只会反做commit id对应的内容，然后重新生成一个新的commit，不会影响其他的commit。

  ​	所以如果要推送到远程服务器的话，`git reset`命令需要强制推送-f，而`git revert`只影响commit id的特性，普通的git push就可以解决。
  
<div><img src="C:\Users\15724\Pictures\Saved Pictures\git reset.png" style="width:50%" /><img src="C:\Users\15724\Pictures\Saved Pictures\git reset.png" style="width:50%" /></div>

> ​	大部分的时候都是修改最近的一次或几次提交，其实这个时候reset命令就直接可以通过相对定位来做，这样最简单，但有时候会遇到这样的一种情况：你提交了某次错误的commit，但是第二天又提交了几个正确的commit，紧接着你还提交了几个commit，这时突然发现之前的commit有错误的地方，但是它被夹在中间，单纯的reset已经无法胜任这个工作（因为它仅能回退到某一次commit，如果这样做，会把正确的commit给回退了）。
>
> ​	那么rebase命令就体现出优势了：可以通过下面的命令把指定版本的某个文件检出来，然后基于这个去改。`git checkout <commitId>  path/to/file`，把不想提交的文件的上一个版本checkout出来，然后添加到暂存区，再`git rebase --continue`。

### 进阶操作

​	如果只会上面两个命令，那么无异于是捡芝麻丢西瓜，时空穿梭的真正核心命令在于：

​	语法：`git reflog`

​	`reflog`是`Reference logs`（参考日志）的缩写，这条命令翻译过来就是——显示可引用的历史版本记录。

 - 跟`git log`命令有什么区别？

​	 `git log`命令能看到很多东西，但不会把所有的东西放在里面，如果版本发生过回退操作，则可能会出现，HEAD指针之后仍存在历史提交版本的情况。

​	`git log`和`git reflog`的直观对比就好像一个是记录了提交历史的《史记》，一个是记录了几乎所有你在仓库中的修改操作的星轨图。

 - 有什么用？

​	假如使用了`git reset`的硬回退（`--hard`），那么你的提交和修改将会荡然无存，这时一拍脑袋，突然意识到之前的还有用，那么怎么办？这时可以使用`git reflog`找到你之前的操作，通过哈希值来定位，再次使用`git reset <hashcode>`可以撤销到之前的操作，有趣的是，这样的操作同样会被记录在reflog中，于是借助reflog，可以真正实现在git中时空穿梭，reflog就是锚点定位器。

- 具象化

  ```bash
  3a709f8 (HEAD -> bugFix2) HEAD@{0}: commit (amend): 第二个bug修改了第一个文件，也许会发生冲突
  5014cb3 HEAD@{1}: rebase (finish): returning to refs/heads/bugFix2
  5014cb3 HEAD@{2}: rebase (pick): 第二个bug修改了第一个文件，可能会发生冲突
  dc2408b HEAD@{3}: rebase (pick): 修复第二个bug
  6554fcb (bugFix) HEAD@{4}: rebase (start): checkout HEAD~2
  d5922e3 HEAD@{5}: rebase (finish): returning to refs/heads/bugFix2
  d5922e3 HEAD@{6}: rebase (pick): 修复第二个bug
  64aa46e HEAD@{7}: rebase (pick): 第二个bug修改了第一个文件，可能会发生冲突
  6554fcb (bugFix) HEAD@{8}: rebase (start): checkout HEAD~2
  3530706 HEAD@{9}: checkout: moving from feature to bugFix2
  953257e (origin/develop, feature, develop) HEAD@{10}: reset: moving to HEAD^
  ```
  `HEAD{}`语法用于引用存储在reflog中的提交，用法与相对定位差不多，但无法用在`git reset`中

*提示：善用`git status`查看仓库当前状态，`git diff`查看修改信息，`git reflog`查看命令历史，可用于回到最新版本，`git log`查看版本信息*

## cherry-pick

语法：`git cherry-pick <commit id>...`

### 是什么？

​	直译为“遴选”，实际上与rebase的所作的事情差不多，但更为简单与直接。它允许将**除已经存在于改动集中之外**（当前分支的历史提交记录之外）的一些提交复制到当前所在的位置（`HEAD`）。

![git cherry-pick](C:\Users\15724\Pictures\Saved Pictures\git cherry-pick.png)

### 如何使用

​	它的基础用法十分简单易懂，按照语法来即可。

- 进阶语法：`git cherry-pick <commit id1>..<commit id100>`

​	如果需要cherry-pick的提交很少则无所谓，一旦达到一定数量，一个个操作必定是十分痛苦的事，那么git提供了一个连续区间操作方法，但注意这是一个左开右闭区间，即commit id1不会被合并，而commit id100则会。

### 注意事项

​	有必要提到的一个参数是 `-n`， 是`--no-commit`的缩写，即只保留文件改动,不做额外`commit`。在项目开发的某些时候，可能会需要这个参数作用，特别是在**回归测试阶段**bug修复过程中发现有一些改动时可以借用其他分支在**开发阶段**某次commit的内容，那么cheery-pick是最优选择，但如果不注意使用这个参数，会将commit message放在错误的地方。

​	当需要cherry-pick多个提交时，最好带上这个参数，否则每拣选一个commit就会提交一次生成一个新的commit id，加上-n参数之后，在全部操作结束之后，可以手动commit

## 远程和本地同步

**有关于同步的操作，之前已经提到了一点（[git pull](###具体操作)）实际上还有另外的几种情况需要单独区分，下面把它们全部罗列出来**

- 本地有新分支/新提交，远程仓库没有

​	如果有追踪源，则直接进行`git push`操作即可；若没有远程分支，则可以通过`git push <远程主机名> <本地分支名> [<远程分支名>] `创建一个远程分支。有时可能会需要用到`--force`子命令强制推送，但是除非你知道你在干什么，否则一般不建议使用。在此之前还需要注意是否存在下面的第二点情况，若存在，则需先完成第二点

- 远程仓库有新分支/新提交，本地没有

​	进行之前提到的`git pull [--rebase]`操作，如果是新分支可以使用`git checkout -b <本地分支名> <remote>/<远程分支名>`

- 本地删除了分支，远程也想删除

​	通常来说可以直接到gitlab或者github之类的托管平台进行删除，或者执行命令`git push origin -d <branch>`，该命令只会对远程仓库作用，本地不删则仍然存在。

------

  常常存在这种情况：*假如我直接到gitlab/github删除了某个分支，我在本地使用`git branch -a`查看远程分支，依然存在并且可以切换使用。*

  可以输入`git remote prune origin`来删除远程仓库已经删除过的分支。

- 远程删除了分支，本地也想删除

  `git branch -d <branch>`删除本地分支

  `git branch -D <branch>`强制删除本地分支，如果该分支有提交未进行合并，也会删除成功。

### 设置同步追踪

​	本地仓库的分支是如何与远程仓库相关联的呢？

​	如果你的远程仓库有一个main分支，并且本地也有一个基于这个分支而来的main分支，那么你会在项目的.git隐藏文件夹中找到远程仓库的分支（路径为.git/refs/remotes）。没错，远程仓库的分支在你的本地也有一份。在命令行中表现为`origin/<branch>`（如果你的源仓库默认名称origin）。

​	`main` 和 `origin/main` 的关联关系就是由分支的“remote tracking”属性决定的。当克隆仓库或者基于分支检出（checkout）的时候，git会自动关联同名分支，而我们自己也可以手动设置这种跟踪属性。

------

​	语法：`git branch -u <origin/branch> [<branch>]`或者`git branch --set-upstream <origin/branch> [<branch>]`

​	如果缺省参数`<branch>`则默认当前所在分支，可以对不同名的分支做追踪，追踪成功之后，push或者pull都将基于追踪来进行。

### 扩展

​	Git 有两种关于 `<source>` 的用法是比较诡异的，即你可以在 git push 或 git fetch 时不指定任何 `source`，方法就是仅保留冒号和 destination 部分，source 部分留空。

- `git push origin :<remotebranch>`

  原语法：`git push <远程仓库源> <本地分支>:<远程分支>`

  push 空 到远程仓库指定分支，会将远程仓库的分支删除

- `git fetch origin :<localbranch>`

  原语法：`git fetch <远程仓库源> <远程分支>:<本地分支>`

  fetch 空 到本地仓库指定分支，会在本地创建一个新分支