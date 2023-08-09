---
layout: blog
title: "Kubernetes 1.17：稳定"
date: 2019-12-09T13：00：00-08：00
slug: kubernetes-1-17-release-announcement
evergreen: true
---

**作者:** [Kubernetes 1.17发布团队](https://github.com/kubernetes/sig-release/blob/master/releases/release-1.17/release_team.md)

我们高兴的宣布Kubernetes 1.17版本的交付，它是我们2019年的第四个也是最后一个发布版本。Kubernetes v1.17包含22个增强功能：有14个增强已经逐步稳定(stable)，4个增强功能已经进入公开测试版(beta)，4个增强功能刚刚进入内部测试版(alpha)。
## 主要的主题
### 云服务提供商标签基本可用
作为公开测试版特性添加到 v1.2 ，v1.17 中可以看到云提供商标签达到基本可用。
### 卷快照进入公开测试版
在 v1.17 中，Kubernetes卷快照特性是公开测试版。这个特性是在 v1.12 中以内部测试版引入的，第二个有重大变化的内部测试版是 v1.13 。
## 容器存储接口迁移公开测试版
在 v1.17 中，Kubernetes树内存储插件到容器存储接口(CSI)的迁移基础架构是公开测试版。容器存储接口迁移最初是在Kubernetes v1.14 中以内部测试版引入的。
## 云服务提供商标签基本可用
当节点和卷被创建，会基于基础云提供商的Kubernetes集群打上一系列标准标签。节点会获得一个实例类型标签。节点和卷都会得到两个描述资源在云提供商拓扑的位置标签,通常是以区域和地区的方式组织。

Kubernetes组件使用标准标签来支持一些特性。例如，调度者会保证pods和它们所声明的卷放置在相同的区域；当调度部署的pods时，调度器会优先将它们分布在不同的区域。你还可以在自己的pods标准中利用标签来配置，如节点亲和性，之类的事。标准标签使得你写的pod规范在不同的云提供商之间是可移植的。

在这个版本中，标签已经达到基本可用。Kubernetes组件都已经更新，可以填充基本可用和公开测试版标签，并对两者做出反应。然而，如果你的pod规范或自定义的控制器正在使用公开测试版标签，如节点亲和性，我们建议你可以将它们迁移到新的基本可用标签中。你可以从如下地方找到新标签的文档：

- [实例类型](/zh-cn/docs/reference/labels-annotations-taints/#nodekubernetesioinstance-type)
- [地区](/zh-cn/docs/reference/labels-annotations-taints/#topologykubernetesioregion)
- [区域](/zh-cn/docs/reference/labels-annotations-taints/#topologykubernetesiozone)

## 卷快照进入公开测试版
在 v1.17 中，Kubernetes卷快照是是公开测试版。最初是在 v1.12 中以内部测试版引入的，第二个有重大变化的内部测试版是 v1.13 。这篇文章总结它在公开版本中的变化。
### 卷快照是什么？
许多的存储系统(如谷歌云持久化磁盘，亚马逊弹性块存储和许多的内部存储系统)支持为持久卷创建快照。快照代表卷在一个时间点的复制。它可用于配置新卷(使用快照数据提前填充)或恢复卷到一个之前的状态(用快照表示)。
### 为什么给Kubernetes加入卷快照？
Kubernetes卷插件系统已经提供了功能强大的抽象用于自动配置、附加和挂载块文件系统。

支持所有这些特性是Kubernetes负载可移植的目标：Kubernetes旨在分布式系统应用和底层集群之间创建一个抽象层,使得应用可以不感知其运行集群的具体信息并且部署也不需特定集群的知识。

Kubernetes存储特别兴趣组(SIG)将快照操作确定为对很多有状态负载的关键功能。如数据库管理员希望在操作数据库前保存数据库卷快照。

在Kubernetes接口中提供一种标准的方式触发快照操作，Kubernetes用户可以处理这种用户场景，而不必使用Kubernetes API(并手动执行存储系统的具体操作)。

取而代之的是，Kubernetes用户现在被授权以与集群无关的方式将快照操作放进他们的工具和策略中，并且确信它将对任意的Kubernetes集群有效，而与底层存储无关。

此外，Kubernetes 快照原语作为基础构建能力解锁了为Kubernetes开发高级、企业级、存储管理特性的能力:包括应用或集群级别的备份方案。

你可以阅读更多关于[发布容器存储接口卷快照公开测试版](https://kubernetes.io/blog/2019/12/09/kubernetes-1-17-feature-cis-volume-snapshot-beta/)
## 容器存储接口迁移公测版
### 为什么我们迁移内建树插件到容器存储接口？
在容器存储接口之前，Kubernetes提供功能强大的卷插件系统。这些卷插件是树内的意味着它们的代码是核心Kubernetes代码的一部分并附带在核心Kubernetes二进制中。然而，为Kubernetes添加插件支持新卷是非常有挑战的。希望在Kubernetes上为自己存储系统添加支持(或修复现有卷插件的bug)的供应商被迫与Kubernetes发行进程对齐。此外，第三方存储代码在核心Kubernetes二进制中会造成可靠性和安全问题，并且这些代码对于Kubernetes的维护者来说是难以(一些场景是不可能)测试和维护的。在Kubernetes上采用容器存储接口可以解决大部分问题。

随着更多容器存储接口驱动变成生产环境可用，我们希望所有的Kubernetes用户从容器存储接口模型中获益。然而，我们不希望强制用户以破坏现有基本可用的存储接口的方式去改变负载和配置。道路很明确，我们将不得不用CSI替换树内插件接口。什么是容器存储接口迁移？

在容器存储接口迁移上所做的努力使得替换现有的树内存储插件，如`kubernetes.io/gce-pd`或`kubernetes.io/aws-ebs`，为相应的容器存储接口驱动成为可能。如果容器存储接口迁移正常工作，Kubernetes终端用户不会注意到任何差别。迁移过后，Kubernetes用户可以继续使用现有接口来依赖树内存储插件的功能。

当Kubernetes集群管理者更新集群使得CSI迁移可用，现有的有状态部署和工作负载照常工作；然而，在幕后Kubernetes将存储管理操作交给了(以前是交给树内驱动)CSI驱动。

Kubernetes组非常努力地保证存储接口的稳定性和平滑升级体验的承诺。这需要细致的考虑现有特性和行为来确保后向兼容和接口稳定性。你可以想像成在加速行驶的直线上给赛车换轮胎。

你可以在这篇博客中阅读更多关于[容器存储接口迁移成为公开测试版](https://kubernetes.io/blog/2019/12/09/kubernetes-1-17-feature-csi-migration-beta/).
## 其它更新
### 稳定💯
- [按条件污染节点](https://github.com/kubernetes/enhancements/issues/382)
- [可配置的Pod进程共享命名空间](https://github.com/kubernetes/enhancements/issues/495)
- [采用kube-scheduler调度DaemonSet Pods](https://github.com/kubernetes/enhancements/issues/548)
- [动态卷最大值](https://github.com/kubernetes/enhancements/issues/554)
- [Kubernetes容器存储接口支持拓扑](https://github.com/kubernetes/enhancements/issues/557)
- [在SubPath挂载提供环境变量扩展](https://github.com/kubernetes/enhancements/issues/559)
- [为Custom Resources提供默认值](https://github.com/kubernetes/enhancements/issues/575)
- [从频繁的Kublet心跳到租约接口](https://github.com/kubernetes/enhancements/issues/589)
- [拆分Kubernetes测试Tarball](https://github.com/kubernetes/enhancements/issues/714)
- [添加Watch书签支持](https://github.com/kubernetes/enhancements/issues/956)
- [行为驱动一致性测试](https://github.com/kubernetes/enhancements/issues/960)
- [服务负载均衡终结保护](https://github.com/kubernetes/enhancements/issues/980)
- [避免每一个Watcher独立序列化相同的对象](https://github.com/kubernetes/enhancements/issues/1152)

### 主要变化
- [添加IPv4/IPv6双栈支持](https://github.com/kubernetes/enhancements/issues/563)

### 其它显著特性
- [拓扑感知路由服务(内部测试版)](https://github.com/kubernetes/enhancements/issues/536)
- [为Windows添加RunAsUserName](https://github.com/kubernetes/enhancements/issues/1043)

### 可用性
Kubernetes 1.17 可以[在GitHub下载](https://github.com/kubernetes/kubernetes/releases/tag/v1.17.0)。开始使用Kubernetes，看看这些[交互教学](https://kubernetes.io/docs/tutorials/)。你可以非常容易使用[kubeadm](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/)安装1.17。
### 发布团队
正是因为有上千人参与技术或非技术内容的贡献才使这个版本成为可能。特别感谢由Guinevere Saenger领导的[发布团队](https://github.com/kubernetes/sig-release/blob/master/releases/release-1.17/release_team.md)。发布团队的35名成员在发布版本的多方面进行了协调，从文档到测试，校验和特性的完善。
随着Kubernetes社区的成长，我们的发布流程是在开源软件协作方面惊人的示例。Kubernetes快速并持续获得新用户。这一成长产生了良性的反馈循环，更多的贡献者贡献代码创造了更加活跃的生态。Kubernetes已经有超过[39000位贡献者](https://k8s.devstats.cncf.io/d/24/overall-project-statistics?orgId=1)和一个超过66000人的活跃社区。
### 网络研讨会
2020年1月7号，加入Kubernetes 1.17发布团队，学习关于这次发布的主要特性。[这里](https://zoom.us/webinar/register/9315759188139/WN_kPOZA_6RTjeGdXTG7YFO3A)注册。
### 参与其中
最简单的参与Kubernetes的方式是加入其中一个与你兴趣相同的[特别兴趣组](https://github.com/kubernetes/community/blob/master/sig-list.md)（SIGs)。有什么想要广播到Kubernetes社区吗？通过如下的频道，在每周的[社区会议](https://github.com/kubernetes/community/tree/master/communication)分享你的声音。感谢你的贡献和支持。

- 在Twitter上关注我们[@Kubernetesio](https://twitter.com/kubernetesio)获取最新的更新
- 在[Discuss](https://discuss.kubernetes.io/)参与社区的讨论
- 在[Slack](http://slack.k8s.io/)加入社区
- 在[Stack Overflow](http://stackoverflow.com/questions/tagged/kubernetes)发布问题(或回答问题)
- 分享你的Kubernetes[故事](https://docs.google.com/a/linuxfoundation.org/forms/d/e/1FAIpQLScuI7Ye3VQHQTwBASrgkjQDSS5TP0g3AXfFhwSM9YpHgxRKFA/viewform)

