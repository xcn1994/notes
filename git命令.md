# git命令

## 基本

```cmd
--初始化当前目录为git仓库
git init

--将仓库中目录下的文件添加到传输器
git add testFile.txt

--将传输器中的文件提交到仓库
git commit -m "a test file."

--回退版本
git reset --hard 1011b

--将当前目录下的文件从暂存区中去除
git rm -r --cached ./
```

## 查看信息

```cmd
--查看仓库状态，是否有文件改动
git status

--查看文件具体改动内容
git diff testFile.txt

--查看历史修改信息
git log
git log --pretty=oneline
git log --oneline

--查看分支图
git log --graph --pretty=oneline --addrev-commit
```



## 分支

```cmd
--查看分支
git branch

--创建分支
git branch dev

--创建并切换到分支
git checkout -b dev

--分支切换
git checkout dev
git checkout master

--将当前分支与另一分支合并
git merge dev

--不使用fast forward合并分支
git merge --no-ff -m "merge with no-ff" dev
```

## 远程仓库

```cmd
--删除绑定远程仓库连接
git remote rm origin

--从远程仓库获取最新版本，不会合并本地版本
git fetch

--从远程仓库获取最新版本并合并本地版本（merge）
git pull

```

