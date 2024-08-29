## 工作流

### GitHub 工作流

### Gerrit 工作流

### AGit-Flow工作流

“AGit-Flow”是在 CGit 的基础上创建的一个集中式 Git 工作流，使用 AGit-Flow 工作流，无需创建派生仓库，也无需在仓库中创建特性分支，只读用户就可以通过`git push`命令创建代码评审。

![img](git.assets/p95624.png)

为此，就有了配套的命令行工具 “git-repo”，既能在单仓库下工作，又支持类似 Android 的多仓库项目协同。

**AGit-Flow 工作流**

单仓库下 AGit-Flow 工作流如下图所示：

![img](git.assets/p95625.png)图中的两个角色，一个是开发者，另外一个是评审者。

开发者通过如下操作，创建和更新 pull request：

1. 开发者克隆仓库。
2. 本地仓库内开发，创建提交。
3. 工作区中执行` git pr `命令，推送本地提交到服务器。
4. 服务器自动创建新的代码评审（例如：pull request #123）。
5. 开发者根据评审意见，在本地工作区继续开发，新增或修改提交。
6. 工作区中再次执行 `git pr` 命令，推送本地提交到服务器。
7. 服务器发现目标分支上已经存在来自同一用户、同一本地分支的 pull request，因此用户此次推送没有创建新的 pull request，而是更新已经存在的 pull request。

代码评审者，不但可以给出评审意见，也可以直接发起对评审代码的修改，更新 pull request：

1. 代码评审者执行` git download 123 `下载编号为 123 的 pull request 到本地仓库。
2. 代码评审者本地修改代码后，执行` git pr --change 123` 命令，将本地修改推送到服务端。
3. 服务端接收到代码评审者的特殊 `git push `命令，更新之前由开发者创建的 pull request。
4. 项目管理者通过点击 pull request 评审界面的合并按钮，将 pull request 合入 master 分支。master 分支被更新，同时关闭 pull request。

下面是单仓库下 AGit-Flow 工作流的演示：


![git-repo-single](git.assets/1.jpg)

![git-repo-single](git.assets/2.jpg)

![git-repo-single](git.assets/3.jpg)

![git-repo-single](git.assets/4.jpg)

![git-repo-single](git.assets/5.jpg)

![git-repo-single](git.assets/6.jpg)

![git-repo-single](git.assets/7.jpg)

多仓库协同演示参见：https://git-repo.info/zh_cn/docs/multi-repos/overview/