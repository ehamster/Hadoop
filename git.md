```bash
1.git status 查看是否有未提交
2.git init 把当前目录变为管理仓库
```
修改完文件后
----------
```bash
1. git add a.txt
2. 这时候可以查看状态
3. git commit -m "注释" 
```
回滚版本
-----------
```bash
1.git diff a.txt 查看未提交修改了啥
2.git log 查看历史记录和作者
3.git log --pretty=oneline 只查看注释
4.git reset --hard HEAD~2  回滚上两个版本
5.或者用 git reflog 查看所有版本号
6.git reset --hard id号 来回滚到任意版本
7.git checkout -- a.txt   将a.txt恢复为当前缓存区里的样子，add过就是add后的样子，没add就是版本样子
```

删除文件
---------------
```bash
1.rm b.txt
2.确认要删除缓存区里的b   git rm b.txt
然后git commit -m "rm b"
```

管理分支-自己用一个分支，全部改好再同步到master
-------
```bash
1.创建并切换分支dev
  git checkout -b dev
2.也可以单独创建和切换
  git branch dev
  git checkout dev
3.正常修改文件，不影响master里的版本
  切换回master git checkout master
  发现再dev修改的内容没有了，因为还没同步到master
4.同步回master
  git merge dev - 将dev同步到当前分支(master)
5.删除分支dev
  git branch -d dev
```
