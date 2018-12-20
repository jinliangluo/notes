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
git pull
```



## 5. 查看log

```
git log # 查看所有log
git log -3 #查看最新的3个log
```



## 6. 查看状态

```
git status #查看本地仓库是否与远程一致
git status -s #简单模式查看
```



## 7. 如何撤销已放入缓存区文件的修改

```
git rm –cached “文件路径” # 将文件从缓存区移除
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
git reset -hard commit_id #回退到上一个commit节点， 代码也发生了改变，变成上一次的

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

































