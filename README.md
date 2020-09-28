##GItHub 协作常用操作

GitHub作为优秀的协同开发工具，成熟了多年且被众多企业用于开发协作，但在实验室中实际应用却少得可怜。首先简要介绍GitHub可以做到什么：

1. 记录所有修改历史，标记重要修改方便随时回滚。任何过去能够实现的东西，后期都能找回来，避免改自己的代码改到不认识。
2. 可与自动化CI（Continues Integration 持续集成）工具较好地协同，能够很方便地在开发新功能的同时测试已有功能是否遭到破坏。
3. 多人协作，分配任务时可以同时有多个分支并行开发，有完善的合并机制。
4. 代码审计功能，写代码的指导可以更方便。



针对上述功能，一些常用的指令如下：

##### 1.建立空白目录，同步已有的github项目

将当前文件夹初始化为git仓库(repo)

`git init`



指明远程仓库相关信息

`git remote add origin git@github.com:UserID/RepoName.git `

其中`UserID/RepoName.git`可在GitHub上获取：

![](https://raw.githubusercontent.com/HaojieYuan/img_links/master/imgs/Xnip2020-09-17_15-25-39.png)



更新远程仓库到本地：

`git pull origin master`

通过该指令，本地会有完整的主分支代码。

`origin`是仓库代名，在指明远程repo相关信息中也出现过，可以采用别的名字，但默认的存于GitHub上的仓库都被称为`origin`。`master`是分支名，指主分支。通常我们保证主分支上的代码是随时可以运行的。在进行你的修改之前，**切记切换到新的分支**，保证你本地有一个干净的主分支。在新的分支上完成开发后，通过后续分支合并操作来更新到主分支。关于分支操作详见第3节。下节主要介绍如何保存、撤回修改。



##### 2.修改本地仓库后的保存提交与撤回

简单来讲，想要提交本地仓库中的改动到GitHub服务器，要经历三步：`git add`， `git commit`和`git push`。为什么提交改动还要分三步？原因是Git需要管理所有细微改动显然不现实，否则修改历史记录将会有大量冗余，因此，它把选择记录哪些修改的工作交给了用户，这也就是`git add`指令的功能，如`git add file1.txt`可以将`file1.txt`中的修改记录下来。可只记录哪个文件改了什么，这还不够，我们希望知道这次修改的目的是什么，比如是开发了什么新功能，或者修复了什么错误等。于是，需要`git commit`来整合之前所有没被`commit`，但被`add`过的改动。如在`add`了`file1.txt`和`file2.py`后，我们实现了功能`func1`，这个时候就可以`git commit -m 'Func1功能实现。'`。在`commit`完成后，本地仓库就有了一次完整的改动记录，利用`git push origin master`便可以将改动更新至GitHub服务器上。



下面讲讲怎么撤回修改，根据修改被提交的进度分成几种情况：

I. 修改未执行`add`操作

`git stash`将撤回所有未`add`的修改，对所有文件生效，撤回过程可逆，也就是说这些修改还能找回来

`git checkout -- file1.txt`将撤回`file1.txt`上所有未`add`的修改

`git reset --hard`同样将撤回所有未`add`的修改，对所有文件生效，但撤回过程不可逆



II.修改已被`add`，但尚未`commit`

`git reset HEAD file1.txt`将撤销对于`file1.txt`的`add`操作

`git reset`将撤销所有尚未`commit`的`add`操作

撤销`add`操作之后，便回到了未执行`add`操作的情况



III.修改已被`commit`，但未被`push`

`git rebase -i`可以交互式的选择放弃某些`commit`操作，迫不得已再用



IV.修改已被`push`

这撤回就麻烦了，所以，直接开放`push`权限会存在远程修改需撤回的麻烦，所以，后面我们来讲怎么利用多分支和`pull request `来更好地管理修改。





##### 3.多分支协同修改、集成测试与代码审计

接下来介绍的是完整工作流：

在已有的GitHub远程仓库的基础上，首先通过步骤1同步到本地。



在本地创建新分支以便修改，**不要在`master`分支上直接改**。

`git branch`可以列出当前所有分支，

`git checkout branch1`可以切换到已有的`branch1`上，

`git checkout -b branch1`可以新建一个名为`branch1`的分支，

一个简单示例：

![](https://raw.githubusercontent.com/HaojieYuan/img_links/master/imgs/20200917164429.png)



分支建立完成后，开始正常的Coding工作，例如我在`code1.py`中实现了一个加法函数`func1`，`code2.py`中实现了一个乘法函数`func2`:

![image-20200917164940788](/Users/haojieyuan/Library/Application Support/typora-user-images/image-20200917164940788.png)



一定要写测试函数，有些代码实现很容易，但暗藏bug，甚至有很多同学喜欢一遍写完，然后能运行就运行，不能运行再去调试，这是很不好的习惯。编写测试文件带来的收益远绝对值得上付出的时间。测试文件主要在两个地方发挥作用：1.在你commit之前，验证自己实现的功能有没有问题；2.在后续你或者别人拓展、修改你之前的写好的功能时，保证其原有功能不被破坏。前者需要你在`commit`之前自行验证，后者一般通过自动化测试工具完成。目前最主流的自动测试工具是TravisCI，配置文件例子如下：

![](https://raw.githubusercontent.com/HaojieYuan/img_links/master/imgs/20200917170033.png)