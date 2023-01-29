#### Git

```
git status 查看当前分支状态
git add src/test/first 提交到暂存区 ，文件路径参考git status 打印出来的文件路径
git stash -u -k 忽略其他文件，把已经修改的隐藏起来
git commit -m "modify xx" 提交到本地仓库
git pull origin [name] 拉取合并（为了避免冲突）<git push -u origin master>
git push origin [name] 提交到远程分支
git stash pop 恢复之前忽略的文件（重要)

```

