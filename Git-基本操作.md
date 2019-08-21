---
title: Git-基本操作
date: 2018-03-19 23:14:11
categories: 
- Git
tags:
- Git
---

继装Git就遇到很大的问题后，我发现我对Git并不是很了解……因为之前都是直接用sourcetree的

整理学习一下吧

当然，新手其实最需要的就是三个技能：clone，merge，push

<!--more-->

![git](http://www.ruanyifeng.com/blogimg/asset/2015/bg2015120901.png)

>- Workspace：工作区
>- Index / Stage：暂存区
>- Repository：仓库区（或本地仓库）
>- Remote：远程仓库



| method                             | 解释                                              | 其他                                                         |
| ---------------------------------- | ------------------------------------------------- | ------------------------------------------------------------ |
| `git remote`                       | 列出每个远程库的简短名字                          | 可加上 `-v` 选项（ `--verbose` 的简写），显示对应的克隆地址  |
| `git remote add [shortname] [url]` |                                                   |                                                              |
| `git fetch`                        | 同步远程分支                                      | 可能有人新建了远程分支，但是在本地上的Remote Branches中并没有远程分支 |
| `git revert`                       | 本地的文件修改后，想要恢复到本地git仓库上次的提交 |                                                              |
| `git clone`                        |                                                   |                                                              |
| `git reset`                        | 回退到某次提交                                    |                                                              |
|                                    |                                                   |                                                              |
|                                    |                                                   |                                                              |
|                                    |                                                   |                                                              |

### merge

A merge B是把A中的改动放到B分支上

例如把master分支上的改动移到分支orign上，可以这样merge。

1. 转到master分支上，然后更新master最新更新。
2. 再转到orign上，然后在Local Branches中选择master分支，选择merge，这样就把本地的master merge到 本地仓库的orign上，然后再选择git push ，这样就把远程master merge到 orign上, 并会提示: Merged master to orign。

### reset

```
$ git reset --hard HEAD^         回退到上个版本
$ git reset --hard HEAD~3        回退到前3次提交之前，以此类推，回退到n次提交之前
$ git reset --hard commit_id     退到/进到 指定commit的sha码
```



### cherry pick

使用情况：其中一个分支A的部分commit你想单独拿出来，放到另一个分支B上去。

这篇讲的很清晰，就不摘抄过来了。

>使用链接：[使用IDEA进行git cherry-pick操作](https://yq.aliyun.com/articles/640890)





### 参考资料

 [Intellij IDEA 中的git操作 ](https://blog.csdn.net/lovesummerforever/article/details/50032937)