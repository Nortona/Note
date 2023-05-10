## 推送至远端
将工作区本地文件修改放入暂存区
```shell
git add .
```
提交到分支
```shell
git commit -m "message"
```
提交到远端仓库
```shell
git remote origin main
```



## 从远端拉取合并到本地

```shell
git pull origin main
git merge main
```


若本地工作区有修改,并且没有add和commit
1. 舍弃本地修改
```shell
git checkout -- <文件>
#把工作区的文件checkout拉出覆写本地的文件
#checkout与add是反义词。都是操作暂存区的。

git reset --hard main
# 直接从仓库中拉出覆写暂存区和工作区。
```
2. 贮藏修改
```shell
git stash
```

3. 合并
```shell
git merge origin/main
```
 4. 弹出贮藏
```shell
git stash pop
```


