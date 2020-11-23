---
title: CodeReview代码审查
date: 2020-05-09 21:43:53
tags: 
	- 项目管理
	- 代码审查
categories: 
	- 项目管理
---

# Code Review

## 什么是Code Review?

Code Review, 意即代码审查,是指一种有意识和系统的召集其他程序员来检查彼此的代码是否有错误的地方.

通常进行Code Review会有以下效果:

- 提高代码质量和可维护性, 可读性等.
- 查漏补缺, 发现一些潜在的问题点等.
- 最佳实践, 能够更好更快的完成任务的方法.
- 知识分享, Review他人代码时, 其实也是一个学习的过程, 自己也会反思&总结.

## 为什么要 Code Review?

要不要Code Review, 需要结合当前的环境和形势来决定. 如果项目组开发任务极其紧张, 此时再进行Code Review可能会收到不利的效果. 因势利导,求同存异是Code Review的核心所在。

Code Review尤其需要和项目组成员沟通和配合, 同时也在检验各成员的技术水平.

> 尤其需要注意的是, 人都有惰性, 每个人都有自己的一套行为准则和规范, 要想把他们的行为或规范统一起来, 很容易使他们产生抗拒心理.

在Code Review之前, 和项目成员沟通好, 有一个共同的愿景, 并辅助以相应的培训, 把部分人员的短板补齐. 会减轻项目成员的一些不适感, 也会使得项目运行更加顺畅。

## Code Review的前提条件

Code Review本身属于"事后"工作, 可以起到查漏被缺的作用.

在推行Code Review时, 需要先提前准备以下几个工作, 以便团队能够更快更好的接受和实施.

- 建立规范, 包含:
  - 编码规范, 如变量命名, 文件命名规范等
  - 设计原则, 如单一职责原则, 最少知识原则等
  - 分支管理策略
- 完善的文档, 方便查阅. 文档内容最好能共建, 千万不可出现"一言堂"
- 制定Code Review流程&目标, 以及实施周期.

根据困难程度, Code Review实施分为3个阶段：

- 第一阶段, 重点关注, 恰到好处的函数注释, "硬编码"问题, 常见变量命名规则等, 预期实施周期为1~3个月
- 第二阶段, 重点关注, 代码耦合性, 单一职责、最少知识原则, 潜在隐患, 性能问题等, 预期实施周期为3~6个月
- 第三阶段, 重点关注, 模块实现方案, 设计模式, 最佳实践, 代码重构等.

在实施过程中, 如果遇到比较大的阻力或困难, 推行Code Review所得到的收益较低时, 可以考虑适当"休息"一段时间.

## Code Review如何实施?

从当前项目的实施过程的体会, 以及结合其他人的一些经验来看, 在Code Review时建议:

- 单次查看代码不多于500行, 人的精力有限, 一次审查太多的代码, 收益可能不理想.
- 单次审查建议不要超过30分钟

## 常见的Code Review项

### git使用规范

git commit提交规范, 如:

- 不标注信息
- 不及时commit
- 提交时标注的信息不规范等

都是严重阻扰review的因素之一,除了常规的描述信息外,还可以按类型等备注:

- feat: 新特性
- fix: 修改问题
- refactor: 代码重构
- docs: 文档修改
- chore: 其他修改
- test: 测试用例修改
- style: 代码格式修改等等

当然, 也要以利用一些工具, 来协助我们完成基本的约束和检查. 如: [优雅的使用Git](https://juejin.im/post/5de280cfe51d450f0a2006f1)

### 风格

#### 1. 可读性

衡量可读性, 有很好的实践标准, 即Code Review时能否非常容易的理解代码逻辑, 如果不能, 那意味着代码的可读性要进行改进.

#### 2. 命名

- 命名对可读性非常重要, 我个人倾向于函数名/类名长一点都没关系, 但必须能清晰的表明函数/类的作用.
- 英语用词尽量准确, 哪怕需要借助翻译工具, 也是值得的. 但有一点需要注意, 如果使用的单词很冷门, 都没人认识, 那就不要使用.

#### 3. 函数体长度/类长度

- 函数体太长, 不好阅读, 一般建议不要超过50行
- 类太长, 如超过10000行, 那可能就要看下是否违反了"单一职责"原则.

#### 4. 参数个数

不要太多, 一般不要超过5个, 超过5个, 建议使用对象

#### 5. 注释

恰到好处的注释, 能够帮助我们理解函数/类的作用。

### 架构/设计

#### 1. 单一职责原则

这是经常被违背的原则, 也是最难运用好的原则.

- 一个类只做一类相关的事情
- 一个函数/方法, 最好只做一件事情

#### 2. 行为是否统一

什么是行为统一? 例如:

- 错误处理是否统一
- 错误提示是否统一
- 弹出框是否统一
- ...

同一逻辑/行为, 有没有执行同样的代码路径:

- **低质量**的代码一个特征是, 同一逻辑/行为, 在不同的地方或不同的方式触发时, 没有执行同样的代码路径(产生出不同的结果), 或者是各处copy一份实现, 导致非常难以维护。

#### 3. 代码污染

代码有没有对其他模块强耦合

#### 4. 重复代码

主要看有没有把公用组件, 可复用的代码、函数抽取出来

#### 5. 开放-封闭原则

简单理解是, 看代码好不好扩展

#### 6. 健壮性

- 核心数据有没有强制校验?
- 对业务有没有考虑完整, 逻辑是否健壮
- 有没有潜在的bug?
- 有没有内存泄露?有没有循环依赖?

#### 7. 错误处理

有没有很好的Error Handling? 如网络出错, IO出错等。

#### 8. 面向接口编程/面向对象接口编程

主要看有没有进行合适的抽象, 把一些行为抽象为接口

#### 9. 效率/性能

- 客户端程序对频繁的消息和较大数据等耗时操作是否处理得当

- 关键算法的时间复杂度是多少? 有没有潜在的性能瓶颈? 

#### 10. 代码重构

- 新的改动是打补丁, 让代码质量继续恶化, 还是对代码质量提升有帮助?

### 低质量代码的常见特征

- 违反"单一职责"原则
- 同一逻辑/行为, 通过不同的方式触发时, 不会执行同样的代码路径
- 缺少注释
- 函数/类命名乱, 词不达意, 或"挂羊头卖狗肉", 例如

### 下面推荐一些 Code Review 工具：

- **[Crucible](https://link.zhihu.com/?target=https%3A//www.atlassian.com/software/crucible)：**Atlassian 内部代码审查工具；
- **[Gerrit](https://link.zhihu.com/?target=https%3A//www.gerritcodereview.com/)：**Google 开源的 git 代码审查工具；
- **[GitHub](https://link.zhihu.com/?target=https%3A//github.com/)：**程序员应该很熟悉了，上面的 "Pull Request" 在代码审查这里很好用；
- **[LGTM](https://link.zhihu.com/?target=https%3A//lgtm.com/)：**可用于 GitHub 和 Bitbucket 的 PR 代码安全漏洞和代码质量审查辅助工具；
- **[Phabricator](https://link.zhihu.com/?target=https%3A//www.phacility.com/phabricator/)：**Facebook 开源的 git/mercurial/svn 代码审查工具； 
- **[PullRequest](https://link.zhihu.com/?target=https%3A//www.pullrequest.com/)：**GitHub pull requests 代码审查辅助工具；
- **[Pull Reminders](https://link.zhihu.com/?target=https%3A//pullreminders.com/)：**GitHub 上有 PR 需要你审核，该插件自动通过 Slack 提醒你；
- **[Reviewable](https://link.zhihu.com/?target=https%3A//reviewable.io/)：**基于 GitHub pull requests 的代码审查辅助工具；
- **[Sider](https://link.zhihu.com/?target=https%3A//sider.review/)：**GitHub 自动代码审查辅助工具；
- **[Upsource](https://link.zhihu.com/?target=https%3A//www.jetbrains.com/upsource/)：**JetBrain 内部部署的 git/mercurial/perforce/svn 代码审查工具。

