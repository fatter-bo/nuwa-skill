# Agent 5：决策记录与行动

## 调研范围
Stardew Valley开发过程中的关键决策、Haunted Chocolatier、与Chucklefish的合作与分手、技术选型

## 关键决策清单

### 决策1：选择做一个"大项目"而非小游戏
**背景**：此前多个小项目未完成（包括2008年的17CF Quest、2011年的Wumbus World漫画、2014年的Air Pear手游）
**决策**：不再做小项目，做一个大型项目并"拒绝放弃直到完成"
**逻辑**：小项目缺乏足够的承诺感，大项目的沉没成本反而成为持续动力
**结果**：4.5年持续开发，最终完成

### 决策2：以Harvest Moon为蓝本开发
**背景**：Barone感觉Harvest Moon系列在Back to Nature之后"progressively worse"
**决策**：做一个Harvest Moon的精神续作，但加入他认为缺失的元素
**逻辑**："My whole reason for making Stardew Valley was to eventually add all the things that I wished Harvest Moon did."
**关键取舍**：有一个明确的参照物降低了设计决策的不确定性，但也带来了"被约束"的感觉——他后来说Haunted Chocolatier没有特定参照物，反而更自由

### 决策3：选择C#+XNA而非Unity
**背景**：2012年Unity已经存在但不如今天成熟；Barone刚从UW Tacoma毕业，学的是CS
**决策**：用C#+XNA框架，后迁移至MonoGame
**逻辑**：
- 初始目的是练习C#编程以充实简历
- XNA提供底层控制，比引擎更适合理解每一行代码在做什么
- 后来选择MonoGame是因为跨平台、开源、有活跃社区
- 2025年12月向MonoGame捐赠$125,000+每月$1,250持续捐赠
**关键取舍**：更多控制力 vs 更多开发时间；选择了控制力

### 决策4：接受Chucklefish出版合作
**背景**：2013年Steam Greenlight期间，Chucklefish主动联系
**决策**：接受出版合作，但保留完整创意控制权
**逻辑**：作为行业新人需要出版经验，自己无法处理商务事务
**结果**：2016年成功发布；2018年开始逐步收回出版权

### 决策5：与Chucklefish分手，转向自出版
**背景**：Barone积累了足够的行业经验和资金
**决策**：逐步收回所有平台的出版权（2018 Switch → 2021 iOS/Wiki → 2022 Android，全部完成）
**逻辑**："I think self-publishing is the end-goal of most indie developers, and I'm happy to be in a place where that's possible."
**附加因素**：2019年Chucklefish被指控使用未付薪劳动，Barone公开保持距离

### 决策6：坚持免费更新，永不收费DLC
**背景**：游戏持续更新8年，每次更新都是大型内容包
**决策**：所有更新永久免费
**逻辑**："My goal in life isn't about making money. I want to create things and share them with the world."
**关键取舍**：放弃了数百万美元的DLC收入，换取了社区信任和"传奇"地位

### 决策7：宣布Haunted Chocolatier（2021年10月）
**背景**：新游戏开发始于2020年底
**决策**：早期公布概念和预告视频
**反思**："I know, I shouldn't have announced the game so early."
**教训**：过早公布创造了外部压力，与他偏好的"先做后说"风格矛盾

### 决策8：1.7更新 — 从独裁到创意总监
**背景**：Stardew Valley仍在增长（5000万份），不更新是浪费
**决策**：让团队承担更多开发工作，自己转为"creative director"角色
**逻辑**：平衡Stardew Valley持续维护和Haunted Chocolatier开发
**关键取舍**：放弃部分直接控制权，但保留音乐等核心创意元素

### 决策9：Haunted Chocolatier从零开发引擎
**背景**：可以复用Stardew Valley的代码基础
**决策**："Haunted Chocolatier is written from scratch; it's not the same 'engine' as Stardew Valley."
**逻辑**：新游戏需要新的技术基础，不愿被旧代码限制；也想通过重新构建来实现更好的架构

### 决策10：反复重做而非"够好就行"
**背景**：角色立绘重画约10次，对话反复重写
**决策**：不断返回"已完成"的部分重新制作
**逻辑**："I just persevered and forced myself to learn. You realize the thing that you thought was good actually isn't. You realize why and you improve on it. And that's just an endless cycle."
**关键取舍**：完美主义导致开发时间翻倍，但最终产品质量成为护城河

## 决策模式总结

1. **承诺锁定**：通过公开承诺或大规模投入创造"不可退出"的条件
2. **控制权优先**：在效率和控制之间，始终选择控制
3. **直觉驱动**：没有方法论，靠"intuition as to what was the next important thing"
4. **反思性修正**：承认错误（如过早公告HC），但不过度自责
5. **玩家体验为终极判断标准**："Make it work, make it fun for the player—that's my angle."
