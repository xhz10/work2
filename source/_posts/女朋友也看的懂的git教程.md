---
title: 女朋友也能看得懂的git教程
---
## 前言
在非编码行业，我们常常会遇到一种痛苦的情况——**电脑崩溃了，重要内容却未保存**。另外一个让人头疼的问题是，**想修改文件但担心改错，却没有备份版本可供恢复。**
通常，我们会费尽心思在WPS、Word等文件编写工具中搜寻**最近一次的保存记录**，试图恢复，然而往往会错失一些重要修改。
令人惊喜的是，有一群人不甘于现状，他们开发了版本控制工具，来帮助我们解决这些问题。在这个领域中，有一个璀璨登场的明星——**SVN**。
虽然我自己并未使用过SVN，但当Linus说它不好用时，我不禁心生疑虑。毕竟，Linus曾被问及这一点时，不畏麻烦地写了个版本控制工具，这位主角直到现在才闪亮登场——**Git。**
Git如同一位大明星，为我们带来了便利与解脱，让我们不再为版本控制的复杂而困扰。从此，我们可以**轻松管理文件版本、随意回退，找回丢失的内容**，真是犹如身临天堂。

## 关于git
### git是什么
Git作为一个版本控制工具于**2005年横空出世**，它的诞生是为了解决开发过程中的版本管理难题。随后，基于Git发展起来的GitHub平台扮演着至关重要的角色，为开源社区带来了革命性的影响，许多开发者从中受益，解决了各种开发中的困难。
无论身为程序员还是普通电脑使用者，我坚信Git的使用都应该被视为一项基本技能。它不仅使团队协作更**高效**，更让个人创作的**安全性和可控性**大大提升。因此，学会使用Git是当务之急。
言归正传，现在让我们先来欣赏一张初学者看了绝对看不懂的Git图表。这些看似晦涩的**箭头、分支和合并**，对新手来说确实有些晦涩难懂。不过，别担心，接下来我会用通俗易懂的语言带你逐步探索Git的奥妙，让Git这个神奇的工具不再让你望而生畏。让我们开始吧！
![git整体图](http://img.xuhongzhuo.com/speaive/image/show/2ef80b8afbed4ca6b582afb78e9885f6)

乍一看是不是**地铁？老人？手机？**

没关系，看完下面的文章，回过头你会发现这张图如此之简单


### 来自面试官的拷打
大三找工作的时候经常会遇见一道面试题----请问如何设计一个git？
这个问题在年幼的我心中作为职业生涯上最难的一个问题，经常让我百思不得其解！直到我真正工作那一天。
### 如何设计一个git
所以如何设计一个git呢？我们从我们的使用场景中开始看git。
我们都知道我们使用git是为了让我们的工作文件具有版本控制的作用，所以我们会有如下流程去进行工作
```bash
a. git add . # 将所有文件的更改放到暂存区
b. git commit -m "提交日志"  # 将我们git的提交到本地的Repoistory
c. git push  # 将我们的改动推送到远程的服务器
```
事实上针对个人使用git，那么记住这3个命令就能解决工作中60%的问题了，如果只是作为简单的数据快照存储，那么可以退出文档了(手动狗头)。
#### 根据用法去思考几个问题
当然我们的初衷是如何设计一个git，所以我们要去思考下面几个问题：

1. 我们每次的"**提交**"到底提交的什么东西？
2. 我们每次的提交都**存储**在了哪里（为什么我们能做到版本的回退？）
#### 针对上述问题的分析
##### 问题1
针对问题1，我们做出思考，我们都听说过每一次的commit都是一次文件的快照，那么既然是快照，我们需要每次commit都保存完整的文件内容嘛？答案显而易见不是，因为那样随着我们的commit增多，我们的存储成本则会越来越大！！！而git采用了一种**内容寻址的方式存储未改动文件的指针**，和改动文件的**diff**信息。所以我们知道了，**我们提交的是我们针对内容的更改**，但是我们保存了内容的**快照**！所以我们的Commit的数据结构如下：
```java
public class Commit {
    private String hash; // 提交的唯一哈希值
    private String message; // 提交信息
    private String author; // 作者信息
    private Date timestamp; // 提交时间戳
    private String treeHash; // 树对象的哈希值 文件系统的快照，包含了指向文件或其他树对象的引用。
    private Map<String, String> fileDiffs; // 文件的 diff 信息，文件路径对应 diff 内容
}
```
观察该数据结构不难发现git的每一次提交都包含了提交人的信息、提交的时间点、改动的内容、以及提交的备注。所以当我们每次在我们的本地进行commit的时候，我们都可以根据commit 的**hash**去查看我们提交的改动是哪些
```bash
# -----------------------这里开始提交内容--------------------------------------
$ git commit -m "第二次提交"  # 提交内容改动
[main 673a046] 第二次提交         # 生成改动的hash
 1 file changed, 1 insertion(+)  # 具体改动的文件和内容介绍
# -----------------------下面开始查看提交内容-----------------------------------
$ git show 673a046  # 根据提交的hash查询内容

commit 673a046a142330ddb0b84df7f9671de07cfc64be (HEAD -> main)   # 提交的hash值
Author: xuhongzhuo <speaive-hz@xxx.com>    # 提交的用户信息
Date:   Thu Apr 25 22:47:04 2024 +0800      # 提交的日期信息

    第二次提交   # 这是提交的message
 # 下面是diff结构
diff --git a/src/main/java/com/speaive/speaiveimg/service/domain/impl/ImageDomainServiceImpl.java b/src/main/java/com/speaive/speaiveimg/service/domain/impl/ImageDomainServiceImpl.java
index ce7e10b..5c334ee 100644
--- a/src/main/java/com/speaive/speaiveimg/service/domain/impl/ImageDomainServiceImpl.java
+++ b/src/main/java/com/speaive/speaiveimg/service/domain/impl/ImageDomainServiceImpl.java
@@ -84,6 +84,7 @@ public class ImageDomainServiceImpl implements ImageDomainService {
     public byte[] compressionImage(MultipartFile inputFile, int width, int height) {
         if (inputFile == null || inputFile.isEmpty()) {
             throw new RuntimeException("文件是空");
+  # 这个加号代表我在这里加了一行空格
         }
         try {
             File file = new File("/Users/xuhongzhuo/testImage/" + inputFile.getOriginalFilename());


```
当然，这还没结束，我们说过git的每一次提交是支持回滚到之前版本的，那么commit与commit之间又是怎样的关系呢？
实际上我们的每一次commit的数据结构都**包含了一个指向前一个commit的指针**，完整的数据结构如下
```java
public class Commit {
    private String hash; // 提交的唯一哈希值
    private String message; // 提交信息
    private String author; // 作者信息
    private Date timestamp; // 提交时间戳
    private String treeHash; // 树对象的哈希值 文件系统的快照，包含了指向文件或其他树对象的引用。
    private Map<String, String> fileDiffs; // 文件的 diff 信息，文件路径对应 diff 内容

    
    private Commit parentCommit;  // 指向上一次提交的指针
}
```
所以我好像还是没说这个指向上一次提交的指针作用是什么？实际上有以下几点好处：

1. 版本控制和历史记录：通过链式关系灵活的追踪内容更改链路
2. 根据父指针控制分支和合并：如果是普通的提交可能只有**一个父提交指针**，但是如果是两个分支合并到一起，会生成一个新的commit，这个新的commit则拥有**两个父提交**，所以我们的数据结构更正如下
```java
public class Commit {
    private String hash; // 提交的唯一哈希值
    private String message; // 提交信息
    private String author; // 作者信息
    private Date timestamp; // 提交时间戳
    private String treeHash; // 树对象的哈希值 文件系统的快照，包含了指向文件或其他树对象的引用。
    private Map<String, String> fileDiffs; // 文件的 diff 信息，文件路径对应 diff 内容

    private Commit[] parentCommits; // 父指针，可以是一个或多个父提交
}
```
##### 问题2
实际上了解完问题一后，我们可以总结出一件事
> git 的单个提交----> 分支 -----> 合并 会让我们的提交记录形成一个**有向无环图**的形式
> ps: 你可能很困惑为啥是无环，因为指针没有指向未来，所以是无环

```git
o<---o<---o<---o<---o(merge) <---o (branch A) # 那个merge就是拥有两个父亲的commit
         /          /
(拉新分支)o<---o<---o <---o<---o (branch B)
```
我们回到问题2我们的每次commit都存储在了哪里？
实际上git会维护一个简单的对象存储结构，以为k,v的形式存储commit的内容
```bash
# xuhongzhuo @ 192 in ~/IdeaProjects/speaive-img/.git/objects on git:main o [23:35:23] 
$ pwd
/project/.git/objects # 查看当前目录确实是 .git/objects 里面存储了对象结构

# xuhongzhuo @ 192 in ~/IdeaProjects/speaive-img/.git/objects on git:main o [23:35:24] 
$ ls # 下面都是对象存储的文件夹
00   13   21   2c   36   3f   46   4c   5b   61   68   6b   75   7b   7f   89   91   96   9c   a8   b2   ba   c8   ce   da   e6   ee   f8   pack
09   14   2a   2e   3a   42   47   52   5c   63   69   6d   77   7c   83   8d   93   97   a5   ae   b4   be   cc   cf   dc   e9   f1   fd
0e   1e   2b   33   3e   45   4a   54   5d   67   6a   70   7a   7e   84   90   94   9a   a6   b1   b8   c0   cd   d2   dd   ec   f6   info
# xuhongzhuo @ 192 in ~/IdeaProjects/speaive-img/.git/objects on git:main o [23:41:25] 
$ cd 67

# xuhongzhuo @ 192 in ~/IdeaProjects/speaive-img/.git/objects/67 on git:main o [23:41:27] 
$ ls
3a046a142330ddb0b84df7f9671de07cfc64be # 67 + 3axxxx 就是上文的提交记录

# git show 673a046a142330ddb0b84df7f9671de07cfc64be 就能看到刚才的内容了
```
#### 深入git基本操作
我们现在已经熟悉了git的每一次commit的基本原理，现在我们应该完全理解了git commit -b "message"这件事了。
那么回看开头的那张图，我们似乎理解下面几个命令也没有那么困难了
```git
$ git log  # 查看提交日志
# --------------------------------提交日志如下-----------------------------
commit c81264f2adab30c470de4b861f34984f395890c9 (HEAD -> main)
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Thu Apr 25 23:49:51 2024 +0800

    第三次提交

commit 673a046a142330ddb0b84df7f9671de07cfc64be
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Thu Apr 25 22:47:04 2024 +0800

    第二次提交

commit 4ad27839153952f25e587522eeebeac8bc6397fa
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Thu Apr 25 22:46:40 2024 +0800

    第一次提交
# --------------------------------尝试回退第三次提交----------------------------

$ git reset 673a046a142330ddb0b84df7f9671de07cfc64be
Unstaged changes after reset:
M       src/main/java/com/speaive/speaiveimg/service/domain/impl/ImageDomainServiceImpl.java

# 可以看到刚才的改动回来了
$ git log  # 查看提交日志
# --------------------------------提交日志如下-----------------------------
# 第三次提交不见啦
commit 673a046a142330ddb0b84df7f9671de07cfc64be (HEAD -> main)
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Thu Apr 25 22:47:04 2024 +0800

    第二次提交

commit 4ad27839153952f25e587522eeebeac8bc6397fa
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Thu Apr 25 22:46:40 2024 +0800
```
##### reset 和revert
刚才我们使用的是git reset 命令去查看我们的改动，我们发现我们的git log中第三次提交居然直接消失不见啦！！！这对我们的操作记录的保存是非常不利的，这是很危险的兄弟。
所以为了保证我们即使是回滚操作也要有操作记录，所以我们可以使用 git revert命令
```git
#--------------------------首先查看commit日志-------------------------
$ git log
commit 0ec415685058718ad33c951e2e8f2932282f48df (HEAD -> main)
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Fri Apr 26 00:04:33 2024 +0800

    新的第三次啦

commit fe2a96a98de2e3afeb29f7241b9d2fce55a13b08
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Fri Apr 26 00:04:07 2024 +0800

    新的第二次提交

commit 4ad27839153952f25e587522eeebeac8bc6397fa
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Thu Apr 25 22:46:40 2024 +0800

    第一次提交
#-----------------------------然后开始revert-------------------------------------
# 我们这次暴力的直接回滚到第一次的内容
$ git revert 4ad27839153952f25e587522eeebeac8bc6397fa # 我想直接回滚到第一次提交的时候
# 报错了，因为撤销的时候出现了冲突
# 根本原因是某些文件已经被我修改过，无法撤销过去的修改了
CONFLICT (modify/delete): src/main/java/com/speaive/speaiveimg/service/domain/impl/ImageDomainServiceImpl.java deleted in (empty tree) and modified in HEAD.  Version HEAD of src/main/java/com/speaive/speaiveimg/service/domain/impl/ImageDomainServiceImpl.java left in tree.
error: could not revert 4ad2783... 第一次提交
hint: After resolving the conflicts, mark them with
hint: "git add/rm <pathspec>", then run
hint: "git revert --continue".
hint: You can instead skip this commit with "git revert --skip".
hint: To abort and get back to the state before "git revert",
hint: run "git revert --abort".


#----------------------------重新查看log------------------------------------------
commit 833a215c98eff14a0fe232eef6ac3f19b7d5eb43
Author: xuhongzhuo <speaive-hz@xxx.com> (HEAD -> main)
Date:   Fri Apr 26 00:07:18 2024 +0800

    sorry 操作失败了
    
commit 0ec415685058718ad33c951e2e8f2932282f48df
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Fri Apr 26 00:04:33 2024 +0800

    新的第三次啦

commit fe2a96a98de2e3afeb29f7241b9d2fce55a13b08
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Fri Apr 26 00:04:07 2024 +0800

    新的第二次提交

commit 4ad27839153952f25e587522eeebeac8bc6397fa
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Thu Apr 25 22:46:40 2024 +0800

    第一次提交
# ----------------------------上面是我修复数据后的提交日志-------------------------
# 那我们尝试回滚最近的一次呢
$ git revert 0ec415685058718ad33c951e2e8f2932282f48df

[main 0ef4414] Revert "新的第三次啦" # 可以知道我们的内容已经回退到最近的版本了
 1 file changed, 1 deletion(-)

#-------------------------------查看日志-----------------------------------------
# 不好意思之前的日志不小心没留住
commit dbf0689211185cab7100d5e0128c98a55cff94f9 (HEAD -> main)
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Fri Apr 26 00:17:11 2024 +0800

    Revert "又咋了"
    
    This reverts commit d782bdf6dffe079bb141277f00a5971511c704e1.

commit d782bdf6dffe079bb141277f00a5971511c704e1
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Fri Apr 26 00:16:52 2024 +0800

    又咋了

```
但是我们观察我们的日志会发现，我们的revert本质上也是**一次提交（commit）**，而revert的概念是撤销！即撤销我们内容的改动，**并且生成一次commit**，那么有以下几个问题需要我们注意：

1. 我们撤销的改动是否已经不存在了？
   1. 观察下列流程
```git
A文件添加一行 sout("hello,world"); commit  hash = abcv
A文件修改刚才那行为 sout("how are you") commit hash = xzhb
A文件撤销 hash = abcv的修改 
  报错： 因为abcv的修改已经不存在了，无法被撤销
A文件撤销 hash = xzhb 的修改
  sout("how are you) ------> sout("hello,world") 并且 commit hash = kxjk
```

2. 第二点太困了忘了要说啥了 skip
##### head 是什么
我们观察上述reset 和revert 的操作，我们发现每次查看git log 的时候都有一个Head--->Main的字眼，那么这个奇怪的Head哥 到底是什么捏？
> 在 Git 中，HEAD 是一个指向当前所在分支的符号引用（symbolic reference），它通常指向当前所在的分支或者特定的提交（commit）。HEAD 代表当前工作目录所在的位置，可以理解为当前工作目录所在的“指针”。

1. 说白了，Head就是你当前commit的位置，**相当于你每一次commit，head都跟着你走**！
2. Head在哪你的代码改动就会生效在哪

所以我们回头看上述reset和revert两个操作中HEAD的位置，我们发现了两个情况

1. reset： reset的时候HEAD直接**暴力的指向了过去的commit **所以git log的时候之前的日志全没了
2. revert：revert的时候**新生成了一个commit并且Head指向了新的commit**，所以git log发现日志多了一个新的提交

所以是不是更加清楚明白reset和revert的区别啦！
> 讨论：什么时候用reset？什么时候用revert呢？

#### 中场休息
我们小小的总结一下过去，然后思考一下开发中的问题，最后再进一步的去聊git这个东西

1. git的每次commit都是 提交的“改动”-----> 衍生问题：冲突是什么？
2. git的reset和revert都能实现回滚但是HEAD位置不同----> 衍生问题：分支合并是否可以回滚？
### git多人协作
我们上一部分聊的是如何设计一个git？现在确将话题引到了git的多人协作，足以证明我们的文章性质已经发生了变化！但是换个角度想，中场休息的时候我们留下的思考问题，**难道哥几个就不好奇嘛**？
所以忘掉我们本来是想设计一个git的初衷，我们开始处理中场休息的遗留问题吧！
#### 分支
上文中简单的用图介绍过分支的样子，现在我们重新提问：什么是git的分支？
其实不好解释，但是我们举例说明！
在多人协作开发过程中

1. 首先我们拥有一个干净的、没有任何问题的目录 M
2. 我们每个开发者基于当前目录 M 去创建自己独属的快照 目录 D(n) {n ∈ 正整数}   ( >=< )
3. 每个人在自己的快照D(n)上开发自己的功能
4. 每个人将自己快照文件改动 添加到 目录M 中
5. 目录M 还是一个干净的没问题的目录，但是多了许多内容

在这个过程中，我们目录M可以理解成是git中的master分支的最新commit (ms)。而快照目录D(n)则是基于master的最新commit(ms)拉出的分支，并且创建了一个新的commit(dv)，此时我们知道 dv extends ms
```git
o<------o<-----o(ms) (master)
              /
             o(dv) (developer)
```
**这就是分支**!
所以我们回到开发中的流程问题：

1. 为什么从master拉分支？因为master最新的commit是最干净的
2. 为什么开发分支合并到master没有冲突？因为每个commit只是**改动**，我们相当于把**改动**带回原文件
##### 当然要给个例子看看拉
```git
# xuhongzhuo @ 192 in ~/IdeaProjects/speaive-img on git:main o [0:52:18] 
$ git checkout -b speaive/develop main
Switched to a new branch 'speaive/develop'

# xuhongzhuo @ 192 in ~/IdeaProjects/speaive-img on git:speaive/develop o [0:52:32] 
$ # 可以看到我们分支已经来到了speaive/develop这里
$ git log

# 观察日志我们发现三个分支都在这个提交点上，并且HeAD指向的是test1 说明我们在test1 分支上
commit dbf0689211185cab7100d5e0128c98a55cff94f9 (HEAD -> test1, speaive/develop, main)
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Fri Apr 26 00:17:11 2024 +0800

    Revert "又咋了"
    
    This reverts commit d782bdf6dffe079bb141277f00a5971511c704e1.

commit d782bdf6dffe079bb141277f00a5971511c704e1
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Fri Apr 26 00:16:52 2024 +0800

    又咋了

commit cfc5918f2c4c8f4760cac6c38dd6c9a182a53e9a
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Fri Apr 26 00:15:55 2024 +0800
#------------------------------------在test1上提交点东西看看----------------------

$ git commit -m "test1的第一次提交"
[test1 cbd4c65] test1的第一次提交
 1 file changed, 1 insertion(+), 1 deletion(-)
#------------------------查看日志如下-----------------------------------
$ git log
# 发现上次的dbxxx变成了我们的parent 此时head还是在我们这里
commit cbd4c6573595ad34d7955ba981bd848e296c000e (HEAD -> test1)
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Fri Apr 26 00:57:40 2024 +0800

    test1的第一次提交

commit dbf0689211185cab7100d5e0128c98a55cff94f9 (speaive/develop, main)
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Fri Apr 26 00:17:11 2024 +0800

    Revert "又咋了"
    
    This reverts commit d782bdf6dffe079bb141277f00a5971511c704e1.

commit d782bdf6dffe079bb141277f00a5971511c704e1
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Fri Apr 26 00:16:52 2024 +0800

```
#### 合并
分支不会有冲突，但是**合并会有冲突**，什么是合并？
合并就是我们两个或者多个分支的改动合并到一个commit的过程，并且合并分为两种：

1. **快速合并**：当两个分支有相同的直接祖先的时候，只需要移动head指针即可完成合并
2. **合并-提交**：当两个分支各自都有自己的改动的时候，需要新生成一个commit

举例说明以上两种情况
##### 快速合并
```git
$ git log
#-----------------------------查看当前日志---------------------------------------
# 得出以下结论
# 1. 我们当前分支是main 分支
# 2. 基于main分支曾经拉出去过一个speaive/develop分支
commit dbf0689211185cab7100d5e0128c98a55cff94f9 (HEAD -> main, speaive/develop)
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Fri Apr 26 00:17:11 2024 +0800

    Revert "又咋了"
    
    This reverts commit d782bdf6dffe079bb141277f00a5971511c704e1.

commit d782bdf6dffe079bb141277f00a5971511c704e1
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Fri Apr 26 00:16:52 2024 +0800

    又咋了

commit cfc5918f2c4c8f4760cac6c38dd6c9a182a53e9a
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Fri Apr 26 00:15:55 2024 +0800

    真拿你没办法啊

commit b61baed3376183d694459f3638ee6ba3b0c1c3c5
# ----------------------------------------开始实验-------------------------------
# 我们首先切换到speaive/develop 分支，并且进行内容的提交
$ git checkout speaive/develop
Switched to branch 'speaive/develop'

# xuhongzhuo @ 192 in ~/IdeaProjects/speaive-img on git:speaive/develop o [8:25:06] 
$ git add .

# xuhongzhuo @ 192 in ~/IdeaProjects/speaive-img on git:speaive/develop x [8:25:22] 
$ git commit -m "新的提交是develop的改动"
[speaive/develop f18e857] 新的提交是develop的改动
 1 file changed, 1 insertion(+), 1 deletion(-)

commit f18e8573d9f3f05f765dee126ca540bd70b45834 (HEAD -> speaive/develop)
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Fri Apr 26 08:25:42 2024 +0800

    新的提交是develop的改动

commit dbf0689211185cab7100d5e0128c98a55cff94f9 (main)
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Fri Apr 26 00:17:11 2024 +0800

    Revert "又咋了"
    
    This reverts commit d782bdf6dffe079bb141277f00a5971511c704e1.

commit d782bdf6dffe079bb141277f00a5971511c704e1
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Fri Apr 26 00:16:52 2024 +0800

    又咋了


# 此时我们的dev分支和main分支还是有同一个共同祖先的，此时执行合并就是快速合并
```
要开始合并了，这是关键操作，新开一个代码块来看
观察上面的最后一次日志，我们发现develop的新的提交是  f1  并且HEAD指针在 f1 的位置，main指针在 db的位置， f1  extends db 所以我们预测如果将develop merge 到 msater 则是main指针前移一位 不生成新的commit
```git
# xuhongzhuo @ 192 in ~/IdeaProjects/speaive-img on git:speaive/develop o [8:27:15] 
$ git checkout main    # 切换到分支main
Switched to branch 'main'

# xuhongzhuo @ 192 in ~/IdeaProjects/speaive-img on git:main o [8:27:20] 
$ git merge speaive/develop   # 将develop 合并到当前分支
Updating dbf0689..f18e857
Fast-forward
 src/main/java/com/speaive/speaiveimg/service/domain/impl/ImageDomainServiceImpl.java | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)

# -------------------------------------查看提交日志--------------------------------
$ git log
commit f18e8573d9f3f05f765dee126ca540bd70b45834 (HEAD -> main, speaive/develop)
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Fri Apr 26 08:25:42 2024 +0800

    新的提交是develop的改动

commit dbf0689211185cab7100d5e0128c98a55cff94f9
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Fri Apr 26 00:17:11 2024 +0800

    Revert "又咋了"
    
    This reverts commit d782bdf6dffe079bb141277f00a5971511c704e1.

commit d782bdf6dffe079bb141277f00a5971511c704e1
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Fri Apr 26 00:16:52 2024 +0800

    又咋了

```
事实上就是如此，我们的HEAD -> main  main处于的位置就是 f1 的内容提交上，这就是快速合并，这种快速合并会出现冲突吗？------- **不会** --------   可以停下来思考为什么不会有冲突
##### 合并-提交
真正会出现冲突的是下面的**带有提交的合并**，我们还是举例看什么时候会出现合并的时候会新生成一个commit
我们进行下面这样一个流程测试一下:

1. 从main 拉取一个feature-ABC 分支和feature-DEF 分支
2. 分别做改动
3. 将两个分支都合并到feature/develop 分支
```git
# xuhongzhuo @ 192 in ~/IdeaProjects/speaive-img on git:main o [8:42:15] 
$ git checkout -b feature-ABC main
Switched to a new branch 'feature-ABC'

# xuhongzhuo @ 192 in ~/IdeaProjects/speaive-img on git:feature-ABC o [8:42:31] 
$ git checkout -b feature-DEF main
Switched to a new branch 'feature-DEF'

# xuhongzhuo @ 192 in ~/IdeaProjects/speaive-img on git:feature-DEF o [8:42:40] 
$  # 分支拉完了
```
我们在两个分支分别改动如下
ABC分支新增内容
```java
try {
            System.out.println("真拿你没办法呢我爱了");
    // 下面这行就是新增的ABC的改动
            System.out.println("我是ABC分支的改动");  
            File file = new File("/Users/xuhongzhuo/testImage/" + inputFile.getOriginalFilename());
            inputFile.transferTo(file);
            ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
            Thumbnails.of(file).size(width, height).outputFormat("jpg").toOutputStream(outputStream);
            return outputStream.toByteArray();
        } catch (IOException e) {
            log.error("文件转换异常", e);
        }
        return null;
```
DEF分支新增内容
```java
try {
            System.out.println("真拿你没办法呢我爱了");
            File file = new File("/Users/xuhongzhuo/testImage/" + inputFile.getOriginalFilename());
            inputFile.transferTo(file);
            ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
            Thumbnails.of(file).size(width, height).outputFormat("jpg").toOutputStream(outputStream);
            return outputStream.toByteArray();
        } catch (IOException e) {
            log.error("文件转换异常", e);
    // 下面这行才是新增的DEF的改动
            System.out.println("我是DEF分支的改动");
        }
        return null;
```
此时我们每一个分支都会生成一个commit，并且commit之间不会产生影响，因为各自的改动没有互相覆盖的情况，所以我们还是不会出现冲突
那么我们现在思考：

1. 将ABC合并到develop 是什么合并？ 
2. 将DEF合并到develop 是什么合并？
```git
# ---------------------------当前develop分支的日志--------------------------------
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Fri Apr 26 08:25:42 2024 +0800

    新的提交是develop的改动

commit dbf0689211185cab7100d5e0128c98a55cff94f9
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Fri Apr 26 00:17:11 2024 +0800

    Revert "又咋了"
    
    This reverts commit d782bdf6dffe079bb141277f00a5971511c704e1.

commit d782bdf6dffe079bb141277f00a5971511c704e1
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Fri Apr 26 00:16:52 2024 +0800

    又咋了

commit cfc5918f2c4c8f4760cac6c38dd6c9a182a53e9a
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Fri Apr 26 00:15:55 2024 +0800

    真拿你没办法啊

commit b61baed3376183d694459f3638ee6ba3b0c1c3c5


# xuhongzhuo @ 192 in ~/IdeaProjects/speaive-img on git:speaive/develop o [8:52:08] 
$ git merge feature-ABC
Updating f18e857..18800a7
Fast-forward   # 发现是快速合并
 src/main/java/com/speaive/speaiveimg/service/domain/impl/ImageDomainServiceImpl.java | 1 +
 1 file changed, 1 insertion(+)
#----------------------------------合并日志---------------------------------------
commit 18800a73c9f9810c9b8bf81d03d617e64306f5f0 (HEAD -> speaive/develop, feature-ABC)
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Fri Apr 26 08:46:05 2024 +0800

    我是ABC分支的改动

commit f18e8573d9f3f05f765dee126ca540bd70b45834 (main)
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Fri Apr 26 08:25:42 2024 +0800

    新的提交是develop的改动

commit dbf0689211185cab7100d5e0128c98a55cff94f9
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Fri Apr 26 00:17:11 2024 +0800

    Revert "又咋了"
    
    This reverts commit d782bdf6dffe079bb141277f00a5971511c704e1.

commit d782bdf6dffe079bb141277f00a5971511c704e1
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Fri Apr 26 00:16:52 2024 +0800

    又咋了
```
将ABC合并到develop是一个快速合并，只切换了头指针而没有生成commit，因为ABC和的父亲就是develop
```git
# xuhongzhuo @ 192 in ~/IdeaProjects/speaive-img on git:speaive/develop o [8:52:29] 
$ git merge feature-DEF
Auto-merging src/main/java/com/speaive/speaiveimg/service/domain/impl/ImageDomainServiceImpl.java
Merge made by the 'ort' strategy.
 src/main/java/com/speaive/speaiveimg/service/domain/impl/ImageDomainServiceImpl.java | 1 +
 1 file changed, 1 insertion(+)

# 看起来不像快速合并
# ------------------------------------查看日志如下---------------------------------
commit f7128a3587084b0c4f9395beafbf358c3275a0c3 (HEAD -> speaive/develop)
Merge: 18800a7 56a4e4c
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Fri Apr 26 08:54:47 2024 +0800

    Merge branch 'feature-DEF' into speaive/develop

commit 18800a73c9f9810c9b8bf81d03d617e64306f5f0 (feature-ABC)
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Fri Apr 26 08:46:05 2024 +0800

    我是ABC分支的改动

commit 56a4e4c4057905884e07924d682e9a67284f7aec (feature-DEF)
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Fri Apr 26 08:45:26 2024 +0800

    DEF改动了内容

commit f18e8573d9f3f05f765dee126ca540bd70b45834 (main)
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Fri Apr 26 08:25:42 2024 +0800

    新的提交是develop的改动
```
我们可以看到我们新生成了一个commit位置去保存这次的合并操作，因为ABC的合并已经**更换了**develop的指针为ABC的commit，此时ABC的commit和DEF的commit并无父子关系，所以再次合并DEF的时候会新生成一个commit，改场景下**是否有出现冲突的可能**？-----------**有**----------------
##### 冲突的出现
试想一下如果我们的DEF分支当初修改的就是ABC分支修改的内容，那么如果你是git你会怎么处理合并呢？
我们新拉一个分支feature-HIJ 去修改ABC新增的那一行
```java
try {
            System.out.println("真拿你没办法呢我爱了");
            System.out.println("我就喜欢改这一行"); // 这一行是之前ABC改过的
            File file = new File("/Users/xuhongzhuo/testImage/" + inputFile.getOriginalFilename());
            inputFile.transferTo(file);
            ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
            Thumbnails.of(file).size(width, height).outputFormat("jpg").toOutputStream(outputStream);
            return outputStream.toByteArray();
        } catch (IOException e) {
            log.error("文件转换异常", e);
        }
        return null;
```
我们将该分支**合并到develop**
```git
# xuhongzhuo @ 192 in ~/IdeaProjects/speaive-img on git:speaive/develop o [9:04:23] 
$ git merge feature-HIJ
Auto-merging src/main/java/com/speaive/speaiveimg/service/domain/impl/ImageDomainServiceImpl.java
CONFLICT (content): Merge conflict in src/main/java/com/speaive/speaiveimg/service/domain/impl/ImageDomainServiceImpl.java
Automatic merge failed; fix conflicts and then commit the result.
```
```java

        try {
            System.out.println("真拿你没办法呢我爱了");
<<<<<<< HEAD
            System.out.println("我是ABC分支的改动");
=======
            System.out.println("我就喜欢改这一行");
>>>>>>> feature-HIJ
            File file = new File("/Users/xuhongzhuo/testImage/" + inputFile.getOriginalFilename());
            inputFile.transferTo(file);
            ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
            Thumbnails.of(file).size(width, height).outputFormat("jpg").toOutputStream(outputStream);
            return outputStream.toByteArray();
        } catch (IOException e) {
            log.error("文件转换异常", e);
            System.out.println("我是DEF分支的改动");
        }
        return null;
```
内容中告诉我们当前HEAD 分支的内容是 **5 **行的内容 但是** 7 **行的内容想覆盖
我们做一个取舍，就去第7行的内容吧
```git
# xuhongzhuo @ 192 in ~/IdeaProjects/speaive-img on git:speaive/develop x [9:05:04] C:1
$ git add .

# xuhongzhuo @ 192 in ~/IdeaProjects/speaive-img on git:speaive/develop x [9:06:01] 
$ git commit -m "处理冲突"
[speaive/develop 2e8a9ab] 处理冲突

#--------------------------------查看日志---------------------------
commit 2e8a9ab7204c583f697350d897ae00ad89fbba3e (HEAD -> speaive/develop)
Merge: f7128a3 953cde7
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Fri Apr 26 09:06:13 2024 +0800

    处理冲突

commit 953cde76de1afb1cd3638962c27cec48434b5ef3 (feature-HIJ)
Author: xuhongzhuo <speaive-hz@xxx.com>
Date:   Fri Apr 26 09:04:16 2024 +0800

    我要提交

```
##### 怎么下掉误合并的分支内容
开发的时候会有人把develop的分支合并到master分支上，这种行为无疑对于线上环境来说是致命的，这里读者回去问问自己的老师或者领导或者同事就知道了就不多赘述了。我们要聊的是面对这种情况我们该如何处理？
###### 自信的处理
根据当前head指向的commit以及对应的parent，一点一点的git reset  <commit_hash>
###### 稳妥的处理
我导师曾经因为我在那里用方法1 而说我是死脑瓜，他给出的方案如下：

1. 召集当天需要上线的开发人员
2. 寻找到上一次上线的release分支
3. 每个人向该分支提交MR，但是**不予合并**
4. 定位到有人的MR莫名奇妙多出来许多代码后 把他拉出来**鞭尸**
5. 其他人正常合并release分支
6. release分支变成新的master
   1. tips：前提是严格遵守上线规范，就是master不要多出来大领导的非上线时间的commit
##### merge 和 rebase的区别
上面说完了合并的几种情况后，我们来聊一个有趣的话题 rebase ！！
依稀记得我初入职场的时候有一个好朋友告诉我这样一句话被我沿用到现在
> 单分支操作用rebase 多分支操作用merge

我懂原理嘛？我不懂，**我懂个锤子**，但是我知道这样做不会出错！
真的不会出错嘛？rebase到底是什么？我们把二者放一起去比较一下
###### merge
我们刚才在使用的过程中会发现，merge操作会保留完整的历史记录（commit），并且会标记出每次的合并点
![image.png](http://img.xuhongzhuo.com/speaive/image/show/bd8c05165ac64105b192aa6de6b6383b)
这在多人协作的过程中是非常有价值的，尤其是抓**到底是谁把develop带到release分支**这件事上
###### rebase
rebase 就比较神奇了，如果说merge是**n个开发者的n个交叉出来的树枝**，那么rebase则是**把所有的commit根据时间排序好的一颗干净的直线**。可以用一个伪代码看一下rebase的工作
```java
TreeNode # 树的结构
List<Commit> allCommitList = new ArrayList();
for (Commit cm: TreeNode.getCommits()) {
    allCommitList.add(cm);
}
// 根据时间排序所有的commit历史
allCommitList = allCommitList.stream
                            .sort(Compartor
                            .compareDate(Commit::getCreateTime))
                            .collect(Collectors.toList());
// 根据时间倒序排序
Collections.reverse(allCommitList);

```
本质上就是把所有的commit整理到一个分支中，那么这个环节不可避免的就是冲突的处理！并且会让我们的一些操作记录诸如merge等小时掉！
**所以，公共分支一定不要用rebase，如果不懂就不要用rebase。**
### 结尾
没有讲**cherry-pick**是因为上述流程如果遵守规范不会出现异常情况
没有讲远程操作是因为上述原理懂了，后续读者可以自主学习远程操作
结束结束！

