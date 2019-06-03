# 常用 Git 命令

```shell
# 查看状态
git status

# 暂存
git stash

# 创建分支
git checkout -b (branch name)

# 撤销制定文件的修改 -- 慎用!
git checkout -- (file name)

# 撤销所有文件的修改 -- 慎用！
git checkout .

# 已add 未commit 执行后返回add前状态，不清空修改
git reset HEAD (file name) 

# 已经commit (回退版本，不清空修改)
git reset HEAD~1 (log 版本号)

# 拉取远程更新到本地，--all为所有分支，--prune为修剪本地分支，就是会把本地多余的分支删除到和远程一致
git fetch --all --prune
#  简写方式
git fetch -a -p

# 拉取远程cuhk到当前分支
git rebase origin/cuhk
```

**常用套路**

以 CUDO-666分支，xx文件为例

```shell
git checkout -b CUDO-666
# 此处在编辑器一顿狂按，修改了xx文件
git add xx
git commit -m "CUDO-666 I change xx"
git stash
git fetch --all --prune 
git rebase origin/cuhk
# 此处是为了检查合完代码有无冲突
git status
git push origin/CUDO-666
```











<https://msdn.itellyou.cn/>