#git

### 把本地项目提交到Github(初始)仓库

1. git init 
2. git add .
3. git commit -m "init"
4. git remote add  origin <git/http地址>
5. git pull --rebase origin main
6. git push -u origin main

### 切换本地分支到一个远程分支

1. git stash
2. git reset --hard   放弃所有未提交的更改（包括暂存的和未暂存的）
3. git fetch origin
4. git checkout -b <new-branch-name> --track origin/<remote-branch-name>
	     使用 `git checkout` 命令结合 `-b` 选项来创建一个新的本地分支，并让它基于远程分支。你还可以使用 `--track` 选项来明确指定跟踪关系（虽然这在 Git 2.23 及更高版本中是可选的，因为 `git checkout -b` 会自动设置跟踪关系）。

5. 再次拉取
6. git fetch origin 
7. git pull 