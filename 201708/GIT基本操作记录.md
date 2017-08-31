### GIT基本操作记录
1. 创建git仓库： 
	
	mkdir gitname
	cd gitname
	pwd // 用来显示仓库的路径

2. 将新建的仓库转变成git可管理的仓库： 
	
	git init

3. 给git仓库添加文件：
	
	git add filename //将文件添加到仓库中，在仓库中就可以查看此文件。
	git commit -m "提交的相关描述" //将文件提交到仓库中，并在log信息中可以查看到本次提交的相关信息及描述

4. 对仓库中的文件进行回滚：
	
	git reset --hard HEAD^ //回滚至上一次提交的版本。
	git log //查看提交日志（从时间最近到最远进行排序）
	git log -- pretty=oneline //查看记录日志信息（简洁化版本）
	git reset -- hard 231434 //回滚至制定版本
	git reset git push-f //先回滚，再强推。可以回滚远程仓库的版本

5. 查看指令：
	
	git status //查看文件有没有被修改过
	git diff //查看文件修改的内容
	git diff HEAD -- filename //查看工作区里面的内容和版本库里面最新版本的区别
	cat filename //查看文件内容

6. 撤销修改： 
	
	git checkout --filename //撤销工作区的修改
	git reset HEAD filename //将已经提交至暂存区的修改从暂存区撤回

7. 删除操作： 
	
	git rm filename //删除版本库中的文件(删除后记得使用 git commit -m"remove filename"进行提交)
	//当工作区文件是误删时，可以使用git checkout --filename 来进行删除操作的撤销。
	//但只能回复到上一次提交的版本，删除前在工作区中的修改将会丢失。

8. 关联一个远程库： 
	
	git remote add origin [repository address]

9. 使用分支：
	
	git branch //查看分支
	git branch<name> //创建分支
	git checkout<name> //切换分支
	git checkout -b <name> //创建并切换分支
	git merge <name> //合并分支(将该分支合并至主分支)
	git branch -d <name> //删除分支
	git branch -D <name> //强行删除分支

10. 其他命令	
	
	git pull //取回远程主机某个分支的更新,再与本地的指定分支合并

	git stash :可用来暂存当前正在进行的工作， 比如想pull 最新代码， 又不想加新commit， 或者另外一种情况，为了fix 一个紧急的bug, 先stash, 使返回到自己上一个commit, 改完bug之后再stash pop, 继续原来的工作
	
	基础命令：

		$git stash
	
		$do some work

		$git stash pop
