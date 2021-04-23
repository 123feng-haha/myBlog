# git stash 命令用法

# 1. 基本使用

`stash`命令可用于临时保存和回复修改，**可跨分支**。

> *** 注：在未`add`之前才能执行`stash`！！！！ ***

- `git stash [save message]`
   保存，`save`为可选项，`message`为本次保存的注释
- `git stash list`
   所有保存的记录列表
- `git stash pop stash@{num}`
   恢复，`num`是可选项，通过`git stash list`可查看具体值。**只能恢复一次**
- `git stash apply stash@{num}`
   恢复，`num`是可选项，通过`git stash list`可查看具体值。**可回复多次**
- `git stash drop stash@{num}`
   删除某个保存，`num`是可选项，通过`git stash list`可查看具体值
- `git stash clear`
   删除所有保存

# 2. 参考

1. [Git 保存和恢复工作进度(stash)](https://www.jianshu.com/p/1e65e938f93c)

**前提：必须是处于git下的文件，未add到git的文件无法使用。**

- 命令：`git stash`

  保存当前工作进度，将工作区和暂存区恢复到修改之前。

- 命令：`git stash save message`

  作用同上，message为此次进度保存的说明。

- 命令：`git stash list`

  显示保存的工作进度列表，编号越小代表保存进度的时间越近。

- 命令：`git stash pop stash@{num}`

  恢复工作进度到工作区，此命令的stash@{num}是可选项，在多个工作进度中可以选择恢复，不带此项则默认恢复最近的一次进度相当于`git stash pop stash@{0}`

- 命令：`git stash apply stash@{num}`

  恢复工作进度到工作区且该工作进度可重复恢复，此命令的stash@{num}是可选项，在多个工作进度中可以选择恢复，不带此项则默认恢复最近的一次进度相当于`git stash apply stash@{0}`

- 命令：`git stash drop stash@{num}`

  删除一条保存的工作进度，此命令的stash@{num}是可选项，在多个工作进度中可以选择删除，不带此项则默认删除最近的一次进度相当于`git stash drop stash@{0}`

- 命令：`git stash clear`

  删除所有保存的工作进度。