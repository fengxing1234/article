---
title: merge与rebase区别
date: 2020-05-09 22:46:17
tags: 
	- 命令行
	- git
categories: 
	- git
---

先看merge，官方文档给的说明是：

```
git-merge - Join two or more development histories together
```

顾名思义，当你想要两个分支交汇的时候应该使用merge。

根据官方文档给的例子，是master merge topic，如图：

```
										 A---B---C topic
                    /         \
               D---E---F---G---H master
```

然而在实践中，在H这个commit上的merge经常会出现merge conflict。为了避免解决冲突的时候引入一些不必要的问题，工程中一般都会规定no conflict merge。比如你在github上发pull request，如果有conflict就会禁止merge。

这种情况下用merge当然是一个选项。用merge代表了topic分支与master分支交汇，并解决了所有合并冲突。**然而merge的缺点是引入了一次不必要的history join。**如图：

```
							      A--B--C-X topic
                    /       / \
               D---E---F---G---H master
```

其实仔细想一下就会发现，**在**引入master分支的F、G commit这个问题上，**我们并没有要求两个分支必须进行交汇(join)，我们只是想避免最终的merge conflict而已。**



rebase是另一个选项。rebase的含义是改变当前分支branch out的位置。这个时候进行rebase其实意味着，将topic分支branch out的位置从E改为G，如图：

```
							               A---B---C topic
                            /         
               D---E---F---G master
```

在这个过程中会解决引入F、G导致的冲突，同时没有多余的history join。但是rebase的缺点是，**改变了当前分支branch out的节点。**如果这个信息对你很重要的话，那么rebase应该不是你想要的。rebase过程中也会有多次解决同一个地方的冲突的问题，不过可以用squash之类的选项解决。个人并不认为这个是rebase的主要问题。

综上，其实选用merge还是rebase取决于你到底是以什么意图来避免merge conflict。实践上个人还是偏爱rebase。一个是因为branch out节点不能改变的情况实在太少。另外就是频繁从master merge导致的冗余的history join会提高所有人的认知成本。