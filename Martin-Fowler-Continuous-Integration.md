
### 持续集成笔记
>Continuous Integration is a software development practice where members of a team integrate their work frequently, usually each person integrates at least daily - leading to multiple integrations per day. Each integration is verified by an automated build (including test) to detect integration errors as quickly as possible. Many teams find that this approach leads to significantly reduced integration problems and allows a team to develop cohesive software more rapidly. This article is a quick overview of Continuous Integration summarizing the technique and its current usage.  

* 一种软件开发的实践
* 适用于**频繁集成**的环境
* 包含自动化的构建、测试
* 目的在于**快速发现**集成错误
* 能够显著减少集成问题,加快开发节奏

---
<!-- #### Building a Feature with Continuous Integration -->
#### 持续集成中新功能的开发过程
* 步骤如下:
  1. 从源代码管理系统中**签出**(checking out)最新代码（mainline -> working copy）
  2. 完成功能开发,**添加/修改测试(利用XUnit测试框架)**
  3. 在**开发机**完成构建并进行自动化测试
  4. 考虑将代码提交时，鉴于在你做功能开发这段时间有人可能修改过mainline代码，所以应该**更新代码**，解决冲突
  5. 将**代码提交**到mainline
  6. mainline代码触发集成的构建，如果成功代表功能完毕，如果失败必须尽快修复

>The result of doing this is that there is a stable piece of software that works properly and contains few bugs. Everybody develops off that shared stable base and never gets so far away from that base that it takes very long to integrate back with it. Less time is spent trying to find bugs because they show up quickly.  

* 按照持续集成的实践，一直存在一份**可工作**的代码(虽然可能有些许bug存在),而众多开发者均是从最新的稳定代码签出并进行开发和**小步提交**。每次修改的代码量控制在了较小的范围内，从而一旦出问题，能够**更快的定位和解决**,从而大幅降低软件集成的难度。  

---
<!-- #### Practices of Continuous Integration -->
#### 持续集成实践
<!-- ##### Maintain a Single Source Repository. -->
##### 维护单一代码仓库
* 使用源代码管理工具
* 把所有构建需要依赖的文件都放进去（在一个干净的环境下能够重新构建完整应用）
* 应包含: 源代码、测试脚本、配置文件、数据库schema、安装脚本、第三方依赖
* 不应包含: 任何项目本身构建物(如包含了则代表团队缺失可靠重新构建的能力)
* 减少分支的滥用(合理的使用场景:先前版本的bugfix,临时性的实验)

<!-- ##### Automate the Build -->
##### 自动化构建
* 利用自动构建工具Maven,Ant,Rake之类
* 构建脚本应该包含所有(含数据库迁移等)
* 构建脚本应支持不同的构建目标(是否包含测试代码等)

<!-- ##### Make Your Build Self-Testing -->
##### 在构建中内建测试
* 利用XP或TDD生产自我测试代码(self-testing code)
* 让测试失败导致构建失败
* 使用XUnit测试框架
* 端到端测试(FIT,Selenium...)
* 测试并不意味着bug的缺席,但是有一定比没有好

<!-- ##### Everyone Commits To the Mainline Every Day   -->
##### 开发者每日提交到Mainline  
>Integration is primarily about communication. Integration allows developers to tell other developers about the changes they have made. Frequent communication allows people to know quickly as changes develop.  

* 软件集成本质上是沟通的问题,持续地集成意味着频繁的沟通,从而能够让开发者更快地知道代码的变化
* 提交的先决条件: 1. 成功构建; 2. 通过自我测试
* 频繁提交意味着问题更好定位

<!-- ##### Every Commit Should Build the Mainline on an Integration Machine   -->
##### 每一次提交都应在集成机器上触发构建  
* 构建失败,有可能是因为:   
    1. 纪律问题: 开发者并没有进行本地构建和测试
    2. 开发机和集成环境的差异
* 每次提交都要进行构建,如果构建失败,该次提交的开发者要对修复构建负责
* 使用CI Server
* 每日构建并不意味着持续集成(违背了尽快发现问题的原则)

<!-- ##### Fix Broken Builds Immediately -->
##### 失败的构建必须立即修复
* 持续集成的核心是在一份稳定的代码上做开发,所以一旦构建失败,必须马上修复
* 修复构建拥有第一优先级-Kent Beck
* 除非损坏原因太过明显,否则建议直接revert mainline,在开发机上做调试定位问题

<!-- ##### Keep the Build Fast -->
##### 缩短构建时间
* 快速反馈是持续集成的关键点
* 十分钟以内
* 关注测试花费的时间(尤其是那些需要调用外部服务的测试)
* 常用套路,分两个阶段:
   1. commit build(满足十分钟法则): 包含构建,单元测试,假数据
   2. 纯测试(真实数据,测试端到端行为): 如果此过程中发现bug,意味着commit build中可能需要加测试用于捕获它
* 可将两阶段扩展至多阶段,可考虑并行执行

<!-- ##### Test in a Clone of the Production Environment -->
##### 在类生产环境中测试
* 同版本同类型数据库;同操作系统;同样的依赖;甚至同IP同Port;同硬件;
* 有一定限制,权衡维护代价和开发/产品环境一致性
* 利用测试替身(test doubles)
* 利用虚拟化技术

<!-- ##### Make it Easy for Anyone to Get the Latest Executable -->
##### 让所有人都很方便的拿到最新的可执行程序
* 人类天性:很难提前预知自己确切需求,但看到一个近似的或不完美的,容易提出改进方向
* 项目所有相关人员都应该能快速地拿到最新的可执行的程序
* 使用制品库

<!-- ##### Everyone can see what's happening -->
##### CI信息的公开化
* 主线构建的状态
* 构建状态传感器:CI灯,CI铃,跳舞的兔子...
* CI Web界面可承载更多信息
* 监督需求(团队成员之外的)
* 物理墙上的CI状态有助于稳定构建的产生

<!-- ##### Automate Deployment -->
##### 自动化部署
* 同一构建物在不同环境中流转(commit tests, secondary tests...)
* 自动化部署脚本
* 自动rollback
* rolling update
* 金丝雀发布,灰度发布,A/B测试

<!-- #### Benefits of Continuous Integration -->
#### 持续集成的好处
* 最大好处是降低风险
* 延后的集成造成盲区,难以估量集成时间和现有进度
* 持续集成并不能消除bug,但可以让其易于定位和排查
* bug是可以累加和相互作用的,持续集成能够大幅降低bug引发的破窗效应
* 需要持续改进测试
* 持续集成是持续部署过程中的最大障碍

<!-- #### Introducing Continuous Integration -->
#### 如何引入持续集成
* 让构建自动化
* 添加自动化测试
* 加快commit build
* 新项目最好在开始的时候就引入
* 寻找智囊(PS:ThoughtWorks提供相应咨询服务)

[老马原贴地址](https://www.martinfowler.com/articles/continuousIntegration.html#EveryoneCommitsToTheMainlineEveryDay)  
