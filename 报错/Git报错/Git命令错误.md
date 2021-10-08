##### 1

出现`failed to push some refs to`，是因为远程仓库和本地仓库代码不一致，原因是在GitHub远程仓库创建的readme.md文件，在本地并没有。

使用命令：`git pull --rebase origin master`即可统一远程仓库和本地仓库。

然后`git push origin master`即可。

