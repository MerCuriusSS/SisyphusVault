---
tags:
  - Areas/Coder/工程化思维
category: 思维
status: 加工
project:
application:
source:
---
[「记忆规律」与「知识体系建设」](「记忆规律」与「知识体系建设」.md)

```
L1:
为什么团队协作需要Git？Git能解决哪些问题？

L2：
记录代码变化
隔离母版
多人合并
远程同步 
错误恢复

L3：
代码变化-> 工作区（红）、暂存区（绿）、提交区（黑）
隔离母版-> 分支机制
多人合并-> merge / rebase / conflict
远程同步-> origin / remote / fetch / pull / push
错误恢复-> restore / reset / revert / reflog机制

L4：
代码变化：
|--git status 查看工作区（红色文件）、暂存区（绿色文件）文件状态(总览对比)
|--git diff 比较哪些内容没有进入到暂存区（内容对比）
|--git add . / {file-name}   全部文件/制定文件 加入到暂存区
|--git commit -m "提交内容..."  加入到提交区
|--git log --oneline 查看内容（精简模式）

隔离母版：
|--git branch 查看所有分支，带*表示当前分支
|--git switch {feature-name} 切换已有分支
|--git switch -c {feature-name} 新建并切换到分支


多人合并：
|--git merge {feature-name} 将功能分支合并到当前分支（保留分叉，新增合并commit记录保留在当前分支）
|--git rebase {feature-name} 将功能分支合并到当前分支（先拷贝分叉，衔接到当前分支，再删除原来分叉记录，整个历史记录是一条直线）
|--git branch ‑d {feature-name} 将功能分支合并到当前分支后直接删除
|--conflict：表示合并时出现冲突，带"<<<<<<"标记，需解决后，执行 "git add 冲突文件"告知冲突已被解决；再执行“git commit（旧版）/git merge --continue（新版 ）” 

远程同步：
|--origin：远程仓库地址的指代地址（再clone中已经确定）
|--git remote -v：查看关联的远程仓库地址
|--git fetch origin {name}：拉取远程代码到本地远程版本副本（origin/main、origin/feature）,不修改本地代码
|--git pull origin {name}：拉取远程代码到本地远程版本副本（origin/main、origin/feature）同时修改本地代码
|--git push origin {name}：推送代码到远程仓库的main分支
|--git push -u origin {name}：第一次推送时使用 -u（upstream）设置上游关联

错误恢复：
|--git restore . / {filename}：将工作区代码还原到最近一次 commit 版本
本地撤销：
|--git reset --soft {ver-id}：将提交区(commit)内容撤销回暂存区(add)
|--git reset {ver-id}：将提交区内容撤销回工作区
|--git reset --hard {ver-id}：清空所有本地内容
远程撤销：
|--git revert {commitId}：撤销远程提交——新建一个反向提交抵消之前修改
|--git revert {a}..{b}:多远程提交记录时依次撤销远程提交，左开右闭(a,b]
|--git reflog：查看HEAD所有历史记录，拯救误 reset‑hard 丢失提交

L5：
我刚改完代码，准备提交
我要开发一个新功能
别人更新了 main，我的分支落后了
我提交错了文件
我合并时出现冲突

```


```
reset 功能：抹除原先的本地提交记录，回滚到指定点，然后 soft / mixed / hard 则是指回滚前的代码放到暂存区、工作区、还是完全丢弃；

比如本地提交记录为“A->B->C->D”,现在需要 “回滚到B”, 如果执行 reset B --soft,则C,D提交的代码回滚后放到暂存区，如果是mixed则放到工作区，如果是hard则直接丢弃（连通当前在工作区的代码）

revert 功能：不改动原有历史，新增一个反向提交抵消之前的修改；比如将远程提交记录为“A->B->C->D” 回滚到B（撤销C修改），则在本地日志中找C的commitid，执行revert <c-commitid>，自动生成新提交 E，此时本地只有D的代码，再执行 git add . -> git push 完成修改，同时生成E历史。而B的代码存在历史记录中，通过 git show <C的commitId> 重新找回；或者重新 revert <e-commitid> 找回 B的代码
```