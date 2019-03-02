# git 常用命令



## 1. 初始化

```
git init
```



## 2. 添加远程仓库

```
git clone https://github.com/hujia-team/uds_iso14229 #示例
```



## 3. 提交修改

```
# 添加指定文件
git add file1 file2
#添加所有文件
git add .

#写日志
git commit -m "日志信息"

#提交
git push
```



## 4. 从远程仓库更新本地

```
git fetch	# 相当有从远程获取最新版本到本地，不会自动merge，更安全
git pull	# 相当有从远程获取最新版本并自动merge到本地，相对与"git fetch" + "git merge"
git pull origin master
```



## 5. 查看log

```shell
git log # 查看所有log
git log -3 #查看最新的3个log
git log -p #查看log的详细修改信息(如查看最近一次提交的详细修改信息，git log -p -1)
```



## 6. 查看状态

```
git status #查看本地仓库是否与远程一致
git status -s #简单模式查看
```



## 7. 如何撤销已放入缓存区文件的修改

```
git rm -–cached “文件路径” # 将文件从缓存区移除
git rm -f “文件路径” #将文件从缓存区移除，并将物理文件删除

git reset HEAD #撤销上一次git add的文件
git reset # 没有带参数的 git reset 命令，默认执行了 –mixed 参数，即用reset版本库到指定版本，并重置缓存区

git checkout one.txt # 切回某个文件（还原工作区某个文件的更改）
```



## 8. git commit  错误

如果不小心 弄错了  git add后 ， 又  git commit 了

```
#先使用 git log 查看节点 
git log

#然后
git reset commit_id #回退到上一个 提交的节点 代码还是原来你修改的
git reset --hard commit_id #回退到上一个commit节点， 代码也发生了改变，变成上一次的

```



## 9.还原已经提交的修改

```
#还原已经提交的修改
#此次操作之前和之后的commit和history都会保留，并且把这次撤销作为一次最新的提交


#git revert 是提交一个新的版本，将需要revert的版本的内容再反向修改回去，版本会递增，不影响之前提交的内容
git revert HEAD #撤销前一次 commit
git revert HEAD^ #撤销前前一次 commit
git revert commit-id #(撤销指定的版本，撤销也会作为一次提交进行保存）
```



## 10.分支开发

1. 下载远程分支开发并推送

   + 同步远程数据到本地 `git fetch origin`

   + 在远程分支(remotebranch)的基础上分化一个本地分支  `git checkout -b localbranch origin/remotebranch`

   + 推送到远程分支 

     ```
     git add test.c
     git commit -m "test"
     git push origin remotebranch	# remotebranch指远程分支
     ```

2. 查看本地分支： `git branch`

3. 查看远程分支： `git branch -r`

4. 切换分支： `git checkout -b newbranch origin/remotebranch`

5. 查看所属分支：`git branch -a`

6. 回退命令: 
   回退到上一版本: `git reset --hard HEAD^` ，
   回退到前3此提交前: `git reset --hard HEAD~3`
   回退到指定commit_id: `git reset --hard commit_id`

分支使用示例：

```
# 分支创建并切换、开发、提交
git branch <分支名>	# 创建新分支,如 git branch bugfix01
git checkout <分支名>	# 切换分支,如 git checkout bugfix01
# 切换到分支后使用git add,git commit ...等命令开发
# 推送本地分支到远程
git push origin <分支名>
```



分支合并和删除

```
# 1、切换到master分支
git checkout master
# 2、将bugfix01合并到master
git merge bugfix01
# 3、删除被合并前的分支名
git branch -d bugfix01
# 4、如果想删除远程分支
git push origin --delete <分支名>
```



## 11. fetch更新本地仓库的两种方式

```
//方法一
$ git fetch origin master //从远程的origin仓库的master分支下载代码到本地的origin master

$ git log -p master.. origin/master//比较本地的仓库和远程参考的区别

$ git merge origin/master//把远程下载下来的代码合并到本地仓库，远程的和本地的合并

//方法二
$ git fetch origin master:temp //从远程的origin仓库的master分支下载到本地并新建一个分支temp

$ git diff temp//比较master分支和temp分支的不同

$ git merge temp//合并temp分支到master分支

$ git branch -d temp//删除temp
```



























