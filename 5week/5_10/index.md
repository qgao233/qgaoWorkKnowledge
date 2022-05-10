# 第5周-5.10

## git cherry-pick

将其它分支上的某个commit历史提交复制到当前分支上来。

>即并不是要把其它分支上的所有提交全部合并到当前分支。

例：将`dev`分支上`commit_id`为`f99f2b57b7ee72d55a08e699fbeec34cbac96cb8`的提交合并到`master`分支：

1. 切换到`master`分支：`git checkout master`
2. 执行`cherry-pick`命令：`git cherry-pick f99f2b57b7ee72d55a08e699fbeec34cbac96cb8`
3. 推送到远程`master`仓库：`git push`

注意`master`上新的`commit id`与`dev`上的`id`并不相同，即只是将`dev`上的修改拷贝过来作为一个新的提交。