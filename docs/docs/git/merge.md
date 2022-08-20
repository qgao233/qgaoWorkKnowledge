## 5 git合并多次提交为1次

[git实用技巧：将多次commit合并为一次](https://blog.csdn.net/qq_45503196/article/details/123876803)

### 5.1 输入git rebase -i

`git rebase -i a77517ad7fda85ba98f207`: 合并`(a77517ad7fda85ba98f207, head所指向的历史提交]`为1次提交

### 5.2 弹出目的编辑窗口

除第一行外的`pick`改成`s`后wq保存退出，会弹出最后的成功提示，再次wq保存退出即可。

>可以看注释，有详细解释

### 5.3 push到远程

因为此时远程分支有多次提交，而本地已经合并为一次提交，所以直接push会失败，改用force参数强制用本地分支覆盖远程分支。

`git push --force-with-lease`

使用该命令在强制覆盖前会进行一次检查如果其他人在该分支上有提交会有一个警告，此时可以避免覆盖代码的风险。

### 5.4 pull到本地

拉取远程主机上被他人rebase操作然后强制推送的分支，直接使用：

`git pull --rebase`