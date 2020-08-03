---
title: git简单使用记录
date: 2020-05-28 14:41:06
tags: git
---

### 创建版本库

> mkdir learngit
>
> cd learngit
>
> pwd
>
> git init         通过git init命令把这个目录变成Git可以管理的仓库
>
> git add readme.txt     用命令git add告诉Git，把文件添加到仓库
>
> git **commit** -m "wrote a readme file"     用命令git commit告诉Git，把文件提交到仓库

<!--more-->

### 版本回退

> git log                                                            命令可以告诉我们历史记录，在Git中，我们用git log命令查看
>
> git log --pretty=oneline                                          看得眼花缭乱的，可以试试加上--pretty=oneline参数
>
> git re**set** --hard HEAD^         上一个版本就是HEAD^，上上一个版本就是HEAD^^，当然往上100个版本写100个^					     比较容易数不过来，所以写成HEAD~100
>
> git re**set** --hard 1094a          通过 git log 查看commit id 返回到制定版本
>
> 现在，你回退到了某个版本，关掉了电脑，第二天早上就后悔了，想恢复到新版本怎么办？找不到新版本的commit id怎么办？
>
> 在Git中，总是有后悔药可以吃的。当你用$ git reset --hard HEAD^回退到add distributed版本时，再想恢复到append GPL，就必须找到append GPL的commit id。Git提供了一个命令git reflog用来记录你的每一次命令
>
> git reflog         终于舒了口气，从输出可知，append GPL的commit id是1094adb，现在，你又可以乘坐时光机回到未来了。
>
> git status     git status查看一下状态：



###  工作区和暂存区

> 工作区（Working Directory）
>
> 就是你在电脑里能看到的目录，比如我的learngit文件夹就是一个工作区
>
> 版本库（Repository）
>
> 工作区有一个隐藏目录.git，这个不算工作区，而是Git的版本库
>
> Git的版本库里存了很多东西，其中最重要的就是称为stage（或者叫index）的暂存区，还有Git为我们自动创建的第一个分支master，以及指向master的一个指针叫HEAD。

{% asset_img 1.png 1.png %}

> **git add 将工作区的文件添加到暂存区,git commit 一次性将暂存区的数据提交到默认的master分支上**



### 管理修改

> Git跟踪并管理的是修改，而非文件。你会问，什么是修改？比如你新增了一行，这就是一个修改，删除了一行，也是一个修改，更改了某些字符，也是一个修改，删了一些又加了一些，也是一个修改，甚至创建一个新文件，也算一个修改。
>
> 第一次修改 -> git add -> 第二次修改 -> git commit
>
> 你看，我们前面讲了，Git管理的是修改，当你用git add命令后，在工作区的第一次修改被放入暂存区，准备提交，但是，在工作区的第二次修改并没有放入暂存区，所以，git commit只负责把暂存区的修改提交了，也就是第一次的修改被提交了，第二次的修改不会被提交。
>
> git diff HEAD -- readme.txt命令可以查看工作区和版本库里面最新版本的区别
>
> 第一次修改 -> git add -> 第二次修改 -> git add -> git commit
>
> 好，现在，把第二次修改提交了，然后开始小结。
>
> 现在，你又理解了Git是如何跟踪修改的，每次修改，如果不用git add到暂存区，那就不会加入到commit中



### **撤销修改**

> git checkout -- file  可以丢弃工作区的修改
>
> 命令git checkout -- readme.txt意思就是，把readme.txt文件在工作区的修改全部撤销，这里有两种情况：
>
> 一种是readme.txt自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；
>
> 一种是readme.txt已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。
>
> 总之，就是让这个文件回到最近一次git commit或git add时的状态。
>
> git checkout -- file命令中的--很重要，没有--，就变成了“切换到另一个分支”的命令，我们在后面的分支管理中会再次遇到git checkout命令。
>
> 现在假定是凌晨3点，你不但写了一些胡话，还git add到暂存区了
>
> 庆幸的是，在commit之前，你发现了这个问题。用git status查看一下，修改只是添加到了暂存区，还没有提交
>
> Git同样告诉我们，用命令git reset HEAD <file>可以把暂存区的修改撤销掉（unstage），重新放回工作区
>
> git reset命令既可以回退版本，也可以把暂存区的修改回退到工作区。当我们用HEAD时，表示最新的版本。
>
> 再用git status查看一下，现在暂存区是干净的，工作区有修改
>
> 还记得如何丢弃工作区的修改吗？         git checkout *-- readme.txt*



### 删除文件

> 在Git中，删除也是一个修改操作，我们实战一下，先添加一个新文件test.txt到Git并且提交
>
> 一般情况下，你通常直接在文件管理器中把没用的文件删了，或者用rm命令删了    rm test.txt
>
> 这个时候，Git知道你删除了文件，因此，工作区和版本库就不一致了，git status命令会立刻告诉你哪些文件被删除了
>
> 现在你有两个选择，一是确实要从版本库中删除该文件，那就用命令git rm删掉，并且git commit
>
> git rm test.txt           git **commit** -m "remove test.txt"
>
> 现在，文件就从版本库中被删除了。
>
> 另一种情况是删错了，因为版本库里还有呢，所以可以很轻松地把误删的文件恢复到最新版本
>
> git checkout -- test.txt
>
> git checkout其实是用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以“一键还原”



### 添加远程库

> 现在的情景是，你已经在本地创建了一个Git仓库后，又想在GitHub创建一个Git仓库，并且让这两个仓库进行远程同步，这样，GitHub上的仓库既可以作为备份，又可以让其他人通过该仓库来协作，真是一举多得。
>
> 首先，登陆GitHub，然后，在右上角找到“Create a new repo”按钮，创建一个新的仓库：
>
> 在Repository name填入learngit，其他保持默认设置，点击“Create repository”按钮，就成功地创建了一个新的Git仓库
>
> 目前，在GitHub上的这个learngit仓库还是空的，GitHub告诉我们，可以从这个仓库克隆出新的仓库，也可以把一个已有的本地仓库与之关联，然后，把本地仓库的内容推送到GitHub仓库。
>
> 现在，我们根据GitHub的提示，在本地的learngit仓库下运行命令
>
> **git remote add origin git@github.com:michaelliao/learngit.git**
>
> 请千万注意，把上面的michaelliao替换成你自己的GitHub账户名，否则，你在本地关联的就是我的远程库，关联没有问题，但是你以后推送是推不上去的，因为你的SSH Key公钥不在我的账户列表中。
>
> 添加后，远程库的名字就是origin，这是Git默认的叫法，也可以改成别的，但是origin这个名字一看就知道是远程库。
>
> 下一步，就可以把本地库的所有内容推送到远程库上：
>
> **git push -u origin master**
>
> 把本地库的内容推送到远程，用git push命令，实际上是把当前分支master推送到远程。
>
> 由于远程库是空的，我们第一次推送master分支时，加上了-u参数，Git不但会把本地的master分支内容推送的远程新的master分支，还会把本地的master分支和远程的master分支关联起来，在以后的推送或者拉取时就可以简化命令
>
> 推送成功后，可以立刻在GitHub页面中看到远程库的内容已经和本地一模一样：
>
> 从现在起，只要本地作了提交，就可以通过命令
>
> **git push origin master**
>
> 把本地master分支的最新修改推送至GitHub，现在，你就拥有了真正的分布式版本库！



### 从远程库克隆

> 上次我们讲了先有本地库，后有远程库的时候，如何关联远程库。
>
> 现在，假设我们从零开发，那么最好的方式是先创建远程库，然后，从远程库克隆。
>
> 首先，登陆GitHub，创建一个新的仓库，名字叫gitskills
>
> 我们勾选Initialize this repository with a README，这样GitHub会自动为我们创建一个README.md文件。创建完毕后，可以看到README.md文件
>
> 现在，远程库已经准备好了，下一步是用命令git clone克隆一个本地库
>
> git clone git@github.com:michaelliao/gitskills.git
>
> 注意把Git库的地址换成你自己的，然后进入gitskills目录看看，已经有README.md文件了
>
> 如果有多个人协作开发，那么每个人各自从远程克隆一份就可以了。
>
> 你也许还注意到，GitHub给出的地址不止一个，还可以用https://github.com/michaelliao/gitskills.git这样的地址。实际上，Git支持多种协议，默认的git://使用ssh，但也可以使用https等其他协议。
>
> 使用https除了速度慢以外，还有个最大的麻烦是每次推送都必须输入口令，但是在某些只开放http端口的公司内部就无法使用ssh协议而只能用https。



### 创建与合并分支

> 在[版本回退](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/0013744142037508cf42e51debf49668810645e02887691000)里，你已经知道，每次提交，Git都把它们串成一条时间线，这条时间线就是一个分支。截止到目前，只有一条时间线，在Git里，这个分支叫主分支，即master分支。HEAD严格来说不是指向提交，而是指向master，master才是指向提交的，所以，HEAD指向的就是当前分支。
>
> 一开始的时候，master分支是一条线，Git用master指向最新的提交，再用HEAD指向master，就能确定当前分支，以及当前分支的提交点：
>
> 每次提交，master分支都会向前移动一步，这样，随着你不断提交，master分支的线也越来越长
>
> 当我们创建新的分支，例如dev时，Git新建了一个指针叫dev，指向master相同的提交，再把HEAD指向dev，就表示当前分支在dev上：
>
> 你看，Git创建一个分支很快，因为除了增加一个dev指针，改改HEAD的指向，工作区的文件都没有任何变化！
>
> 不过，从现在开始，对工作区的修改和提交就是针对dev分支了，比如新提交一次后，dev指针往前移动一步，而master指针不变
>
> 假如我们在dev上的工作完成了，就可以把dev合并到master上。Git怎么合并呢？最简单的方法，就是直接把master指向dev的当前提交，就完成了合并：
>
> 所以Git合并分支也很快！就改改指针，工作区内容也不变！
>
> 合并完分支后，甚至可以删除dev分支。删除dev分支就是把dev指针给删掉，删掉后，我们就剩下了一条master分支
>
> 首先，我们创建dev分支，然后切换到dev分支：
>
> git checkout -b dev
>
> git checkout命令加上-b参数表示创建并切换，相当于以下两条命令
>
> git branch dev         git checkout dev
>
> 然后，用git branch命令查看当前分支
>
> git branch命令会列出所有分支，当前分支前面会标一个*号
>
> 然后，我们就可以在dev分支上正常提交，比如对readme.txt做个修改，加上一行
>
> git add            git commit
>
> 现在，dev分支的工作完成，我们就可以切换回master分支
>
> git checkout master
>
> 切换回master分支后，再查看一个readme.txt文件，刚才添加的内容不见了！因为那个提交是在dev分支上，而master分支此刻的提交点并没有变：
>
> 现在，我们把dev分支的工作成果合并到master分支上
>
> git merge dev
>
> git merge命令用于合并指定分支到当前分支。合并后，再查看readme.txt的内容，就可以看到，和dev分支的最新提交是完全一样的。
>
> 注意到上面的Fast-forward信息，Git告诉我们，这次合并是“快进模式”，也就是直接把master指向dev的当前提交，所以合并速度非常快。
>
> 当然，也不是每次合并都能Fast-forward，我们后面会讲其他方式的合并。
>
> 合并完成后，就可以放心地删除dev分支了
>
> git branch -d dev
>
> 删除后，查看branch，就只剩下master分支了：
>
> 因为创建、合并和删除分支非常快，所以Git鼓励你使用分支完成某个任务，合并后再删掉分支，这和直接在master分支上工作效果是一样的，但过程更安全。



### 解决冲突

> 人生不如意之事十之八九，合并分支往往也不是一帆风顺的。
>
> 准备新的feature1分支，继续我们的新分支开发 git checkout -b feature1 修改readme.txt最后一行，改为    
>
> Creating a new branch is quick AND simple.
>
> 在feature1分支上提交 git add readme.txt       git commit -m "AND simple" 切换到master分支 git
>
> checkout master Git还会自动提示我们当前master分支比远程的master分支要超前1个提交。
>
> 在master分支上把readme.txt文件的最后一行改为：                Creating a new branch is quick & simple.
>
> 提交 git add readme.txt      git commit -m "& simple" 现在，master分支和feature1分支各自都分别有新
>
> 提交，变成了这样：

{% asset_img 2.png 2.png %}



> 这种情况下，Git无法执行“快速合并”，只能试图把各自的修改合并起来，但这种合并就可能会有冲突，我们试试看 git merge
>
> feature1 果然冲突了！Git告诉我们，readme.txt文件存在冲突，必须手动解决冲突后再提交。git status也可以告诉我们冲突的文
>
> 我们可以直接查看readme.txt的内容： Git用<<<<<<<，=======，>>>>>>>标记出不同分支的内容，我们修改如下后保存 Creating
>
> a new branch is quick and simple.
>
> 再提交： git add readme.txt           git commit -m "conflict fixed" 现在，master分支和feature1分支变成了下图所示

{% asset_img 3.png 3.png %}

> 用带参数的git log也可以看到分支的合并情况 
>
> 最后，删除feature1分支： 
>
> git branch -d feature1



### 分支管理策略 

>  通常，合并分支时，如果可能，Git会用Fast forward模式，但这种模式下，删除分支后，会丢掉分支信息。
>
> 如果要强制禁用Fast forward模式，Git就会在merge时生成一个新的commit，这样，从分支历史上就可以看出分支信息。
>
> 下面我们实战一下--no-ff方式的git merge： 首先，仍然创建并切换dev分支： git checkout -b dev 修改readme.txt文件，并提交
>
> 个新的commit： git add readme.txt                git commit -m "add merge" 现在，我们切换回master： git checkout master 准
>
> 合并dev分支，请注意--no-ff参数，表示禁用Fast forward： git merge --no-ff -m "merge with no-ff" dev 因为本次合并要创建一
>
> 新的commit，所以加上-m参数，把commit描述写进去。
>
> 合并后，我们用git log看看分支历史： 在实际开发中，我们应该按照几个基本原则进行分支管理： 首先，master分支应该是非常
>
> 定的，也就是仅用来发布新版本，平时不能在上面干活； 那在哪干活呢？干活都在dev分支上，也就是说，dev分支是不稳定的，
>
> 某个时候，比如1.0版本发布时，再把dev分支合并到master上，在master分支发布1.0版本； 你和你的小伙伴们每个人都在dev分
>
> 上干活，每个人都有自己的分支，时不时地往dev分支上合并就可以了。
>
> 所以，团队合作的分支看起来就像这样：

{% asset_img 4.png 4.png %}



### Bug分支 

> 软件开发中，bug就像家常便饭一样。有了bug就需要修复，在Git中，由于分支是如此的强大，所以，每个bug都可以通过一个新的临时分支来修复，修复后，合并分支，然后将临时分支删除。
>
> 当你接到一个修复一个代号101的bug的任务时，很自然地，你想创建一个分支issue-101来修复它，但是，等等，当前正在dev上进行的工作还没有提交： 并不是你不想提交，而是工作只进行到一半，还没法提交，预计完成还需1天时间。但是，必须在两个小时内修复该bug，怎么办？
>
> 幸好，Git还提供了一个stash功能，可以把当前工作现场“储藏”起来，等以后恢复现场后继续工作： git stash 现在，用git status查看工作区，就是干净的（除非有没有被Git管理的文件），因此可以放心地创建分支来修复bug。
>
> 首先确定要在哪个分支上修复bug，假定需要在master分支上修复，就从master创建临时分支：
>
>  git checkout master git checkout -b issue-101 现在修复bug，需要把“Git is free software ...”改为“Git is a free software ...”，然后提交： git add readme.txt  git commit -m "fix bug 101" 修复完成后，切换到master分支，并完成合并，最后删除issue-101分支： git checkout master git merge --no-ff -m "merged bug fix 101" issue-101 太棒了，原计划两个小时的bug修复只花了5分钟！现在，是时候接着回到dev分支干活了！
>
> git checkout dev git status 工作区是干净的，刚才的工作现场存到哪去了？用git stash list命令看看： 工作现场还在，Git把stash内容存在某个地方了，但是需要恢复一下，有两个办法： 一是用git stash apply恢复，但是恢复后，stash内容并不删除，你需要用git stash drop来删除； 另一种方式是用git stash pop，恢复的同时把stash内容也删了： git stash pop 再用git stash list查看，就看不到任何stash内容了： git stash list 你可以多次stash，恢复的时候，先用git stash list查看，然后恢复指定的stash，用命令： git stash apply stash@{0}



### Feature分支 

> 软件开发中，总有无穷无尽的新的功能要不断添加进来。
>
> 添加一个新功能时，你肯定不希望因为一些实验性质的代码，把主分支搞乱了，所以，每添加一个新功能，最好新建一个feature分支，在上面开发，完成后，合并，最后，删除该feature分支。
>
> 现在，你终于接到了一个新任务：开发代号为Vulcan的新功能，该功能计划用于下一代星际飞船。
> 于是准备开发： git checkout -b feature-vulcan 5分钟后，开发完毕： git add vulcan.py git status git commit -m "add feature vulcan" 切回dev，准备合并： git checkout dev 一切顺利的话，feature分支和bug分支是类似的，合并，然后删除。
> 但是！
>
> 就在此时，接到上级命令，因经费不足，新功能必须取消！
> 虽然白干了，但是这个包含机密资料的分支还是必须就地销毁： git branch -d feature-vulcan 销毁失败。Git友情提醒，feature-vulcan分支还没有被合并，如果删除，将丢失掉修改，如果要强行删除，需要使用大写的-D参数。。
> 现在我们强行删除： git branch -D feature-vulcan 开发一个新feature，最好新建一个分支； 如果要丢弃一个没有被合并过的分支，可以通过git branch -D <name>强行删除。



### 多人协作 

> 当你从远程仓库克隆时，实际上Git自动把本地的master分支和远程的master分支对应起来了，并且，远程仓库的默认名称是origin。
>
> 要查看远程库的信息，用git remote： git remote 或者，用git remote -v显示更详细的信息： 上面显示了可以抓取和推送的origin的地址。如果没有推送权限，就看不到push的地址。



### 推送分支 

> 推送分支，就是把该分支上的所有本地提交推送到远程库。推送时，要指定本地分支，这样，Git就会把该分支推送到远程库对应的远程分支上： git push origin master 如果要推送其他分支，比如dev，就改成： git push origin dev 但是，并不是一定要把本地分支往远程推送，那么，哪些分支需要推送，哪些不需要呢？
>
> master分支是主分支，因此要时刻与远程同步； dev分支是开发分支，团队所有成员都需要在上面工作，所以也需要与远程同步； bug分支只用于在本地修复bug，就没必要推到远程了，除非老板要看看你每周到底修复了几个bug； feature分支是否推到远程，
>
> 决于你是否和你的小伙伴合作在上面开发。
> 总之，就是在Git中，分支完全可以在本地自己藏着玩，是否推送，视你的心情而定！



### 抓取分支 

> 多人协作时，大家都会往master和dev分支上推送各自的修改。
>
> 现在，模拟一个你的小伙伴，可以在另一台电脑（注意要把SSH Key添加到GitHub）或者同一台电脑的另一个目录下克隆： 
>
> git clone git@github.com:michaelliao/learngit.git 
>
> 当你的小伙伴从远程库clone时，默认情况下，你的小伙伴只能看到本地的master分支。不信可以用git branch命令看看： 现在，你的小伙伴要在dev分支上开发，就必须创建远程origin的dev分支到本地，于是他用这个命令创建本地dev分支 
>
> git checkout -b dev origin/dev 
>
> 现在，他就可以在dev上继续修改，然后，时不时地把dev分支push到远程：
>
>  git add env.txt git commit -m "add env" git push origin dev 
>
> 你的小伙伴已经向origin/dev分支推送了他的提交，而碰巧你也对同样的文件作了修改，并试图推送： 
>
> git add env.txt git commit -m "add new env" git push origin dev 推送失败，
>
> 因为你的小伙伴的最新提交和你试图推送的提交有冲突，解决办法也很简单，Git已经提示我们，先用git pull把最新的提交从origin/dev抓下来，然后，在本地合并，解决冲突，
>
> 再推送： git pull git pull也失败了，原因是没有指定本地dev分支与远程origin/dev分支的链接，根据提示，设置dev和origin/dev的链接： 
>
> git branch --set-upstream-to=origin/dev dev git pull 这回git pull成功，
>
> 但是合并有冲突，需要手动解决，解决的方法和分支管理中的解决冲突完全一样。
>
> 解决后，提交，再push： git commit -m "fix env conflict" git push origin dev 因此，多人协作的工作模式通常是这样：
>
> + 首先，可以试图用git push origin <branch-name>推送自己的修改；
> + 如果推送失败，则因为远程分支比你的本地更新，需要先用git pull试图合并；
> + 如果合并有冲突，则解决冲突，并在本地提交；
> + 没有冲突或者解决掉冲突后，再用git push origin <branch-name>推送就能成功！
>
> 如果git pull提示no tracking information，则说明本地分支和远程分支的链接关系没有创建，用命令git branch --set-upstream-to <branch-name> origin/<branch-name>。
>
> 这就是多人协作的工作模式，一旦熟悉了，就非常简单。



### Rebase 

> 在上一节我们看到了，多人在同一个分支上协作时，很容易出现冲突。即使没有冲突，后push的童鞋不得不先pull，在本地合并，然后才能push成功。
>
> 每次合并再push后，分支变成了这样： git log --graph --pretty=oneline --abbrev-commit 总之看上去很乱，有强迫症的童鞋会问
>
> 为什么Git的提交历史不能是一条干净的直线？
>
> 其实是可以做到的！
>
> Git有一种称为rebase的操作，有人把它翻译成“变基”。
>
> 先不要随意展开想象。我们还是从实际问题出发，看看怎么把分叉的提交变成直线。
>
> 在和远程分支同步后，我们对hello.py这个文件做了两次提交。用git log命令看看： 
>
> git log --graph --pretty=oneline --abbrev-commit 注意到Git用(HEAD -> master)和(origin/master)标识出当前分支的HEAD和远程origin的位置分别是582d922 add author和d1be385 init hello，本地分支比远程分支快两个提交。
>
> 现在我们尝试推送本地分支： git push origin master 很不幸，失败了，这说明有人先于我们推送了远程分支。按照经验，先pull一下：
>
>  git pull 再用git status看看状态： 
>
> git status 加上刚才合并的提交，现在我们本地分支比远程分支超前3个提交。
>
> 用git log看看： git log --graph --pretty=oneline --abbrev-commit 对强迫症童鞋来说，现在事情有点不对头，提交历史分叉了。如果现在把本地分支push到远程，有没有问题？有！什么问题？不好看！有没有解决方法？有
>
> 这个时候，rebase就派上了用场。我们输入命令git rebase试试： git rebase 输出了一大堆操作，到底是啥效果？再用git log看看： 
>
> git log --graph --pretty=oneline --abbrev-commit 原本分叉的提交现在变成一条直线了！这种神奇的操作是怎么实现的？其实原理非常简单。我们注意观察，发现Git把我们本地的提交“挪动”了位置，放到了f005ed4 (origin/master) set exit=1之后，这样，整个提交历史就成了一条直线。rebase操作前后，最终的提交内容是一致的，但是，我们本地的commit修改内容已经变化了，它们的修改不再基于d1be385 init hello，而是基于f005ed4 (origin/master) set exit=1，但最后的提交7e61ed4内容是一致的。
>
> 这就是rebase操作的特点：把分叉的提交历史“整理”成一条直线，看上去更直观。缺点是本地的分叉提交已经被修改过了。
> 最后，通过push操作把本地分支推送到远程： git push origin master 再用git log看看效果： git log --graph --pretty=oneline -
>
> abbrev-commit 远程分支的提交历史也是一条直线。
>
> rebase操作可以把本地未push的分叉提交历史整理成直线； rebase的目的是使得我们在查看历史提交的变化时更容易，因为分叉的提交需要三方对比。



### 创建标签 

> 在Git中打标签非常简单，首先，切换到需要打标签的分支上： 
>
> git branch git checkout master 然后，敲命令git tag <name>就可以打一个新标签： 
>
> git tag v1.0 可以用命令git tag查看所有标签：
>
>  git tag 默认标签是打在最新提交的commit上的。有时候，如果忘了打标签，比如，现在已经是周五了，但应该在周一打的标签没有打，怎么办？
>
> 方法是找到历史提交的commit id，然后打上就可以了： 
>
> git log --pretty=oneline --abbrev-commit 比方说要对add merge这次提交打标签，它对应的commit id是f52c633，敲入命令：
>
>  git tag v0.9 f52c633 再用命令git tag查看标签：
>
>  git tag 注意，标签不是按时间顺序列出，而是按字母排序的。
>
> 可以用git show <tagname>查看标签信息：
>
>  git show v0.9 可以看到，v0.9确实打在add merge这次提交上。
>
> 还可以创建带有说明的标签，用-a指定标签名，-m指定说明文字：
>
>  git tag -a v0.1 -m "version 0.1 released" 1094adb 
>
> 用命令git show <tagname>可以看到说明文字： 
>
> git show v0.1 命令git tag <tagname>用于新建一个标签，默认为HEAD，
>
> 也可以指定一个commit id；
>
>  命令git tag -a <tagname> -m "blablabla..."可以指定标签信息； 
>
> 命令git tag可以查看所有标签。



### 操作标签 

> 如果标签打错了，也可以删除： 
>
> git tag -d v0.1 因为创建的标签都只存储在本地，不会自动推送到远程。所以，打错的标签可以在本地安全删除。
>
> 如果要推送某个标签到远程，使用命令git push origin <tagname>：
>
>  git push origin v1.0 或者，一次性推送全部尚未推送到远程的本地标签： 
>
> git push origin --tags 如果标签已经推送到远程，要删除远程标签就麻烦一点，
>
> 先从本地删除： git tag -d v0.9 然后，从远程删除。删除命令也是push，
>
> 但是格式如下： git push origin :refs/tags/v0.9 命令git push origin <tagname>可以推送一个本地标签； 
>
> 命令git push origin --tags可以推送全部未推送过的本地标签；
>
>  命令git tag -d <tagname>可以删除一个本地标签；
>
>  命令git push origin :refs/tags/<tagname>可以删除一个远程标签。