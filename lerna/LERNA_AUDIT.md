# Lerna Protocol 学术论文审计报告

审计对象：`paper.tex` 与 `metadata.yaml`  
审计日期：2026-06-05  
审计角色：密码学协议与 Cardano/Hydra 方向严格审稿人

## 1. 总体结论

本文是一份定位谨慎、工程意识较强的“研究设计说明”，但尚不构成可发表的完整密码学协议论文。其核心思想是把 Morph Channel 的若干抽象迁移到 Cardano Hydra 环境中，在 Hydra Head 内部维护紧凑的虚拟通道工厂状态，并要求在 Hydra fanout 前将紧凑权利确定性材料化为普通 Head UTxO。

严格评价如下：

- **作为问题定位和研究路线图：有价值。** 论文准确意识到 Hydra fanout、Cardano eUTxO、min-ADA、native assets、datum、脚本预算和 contestation deadline 对虚拟通道工厂的限制。
- **作为协议规范：明显不足。** 多数关键对象仅是字段列表，没有状态机、有效性谓词、敌手模型、消息流程、签名语义、证明系统或可验证安全定理。
- **作为密码学安全论文：不合格。** 所谓 “latest-state safety” 是对 Hydra contest 机制的直接复述，并把最关键的可用性、材料化容量、观察者在线性、授权提交能力作为假设。该命题没有证明 Lerna 自身的新安全性质。
- **作为工程白皮书：尚可，但应降级表述。** 当前贡献更像“Hydra 内部虚拟通道工厂的设计纪律与开放问题清单”，不应声称已经提出可安全部署的协议。

建议结论：**弱拒 / 大修后再审**。若投向学术会议，需要补齐形式化模型、安全定义、协议算法、实现可行性数据和与已有 channel factory / virtual channel / Hydra / Interhead / Hydrozoa 工作的严格比较。

## 2. 新颖性评估

### 2.1 可能的新意

论文的新颖点主要在于 Cardano/Hydra 语境下的组合，而不是底层密码学机制：

1. **Hydra Head 内部的紧凑 factory rights discipline**：将多个虚拟通道权利以紧凑根或状态对象表示，而非每个通道都成为 Head-visible UTxO。
2. **fanout 前的 inner materialisation 纪律**：明确指出紧凑状态不能直接逃避 Hydra fanout 的 UTxO 语义，必须先展开成普通 Head UTxO 或达成释放。
3. **release receipts 防止紧凑权利与材料化输出双重声明**：这是一个合理的工程安全机制，适合 Cardano eUTxO 语境。
4. **将 Morph 的 CKB 概念谨慎映射到 Hydra/Cardano**：论文没有简单声称 “eltoo for Cardano”，这一点比很多 L2 设计叙述更诚实。

### 2.2 新颖性不足

但这些新意目前停留在架构性组合，学术贡献强度有限：

- **channel factories、virtual channels、latest-state adjudication、non-punitive supersession** 均有大量既有工作，如 channel factories、Perun、eltoo、Hydra、Interhead。
- “紧凑承诺 + 局部证明 + 材料化”并非新密码学技术。论文没有提出新的 accumulator、证明系统、adjudication protocol 或复杂度改进。
- “材料化前置”是正确的工程约束，但更像安全需求而非已解决的机制。
- release receipt 概念有实用价值，但没有形式化到足以判断是否区别于一般 spent/nullifier/receipt/withdrawal marker 机制。

### 2.3 对贡献表述的审计意见

当前 C1-C3 的表述过强。建议改为：

- C1：提出一种 **设计框架**，而非定义完整协议。
- C2：提出一种 **必要安全纪律**，而非证明可总是满足。
- C3：提出一种 **待形式化的非干涉与释放收据机制**，而非已完成安全证明。

## 3. 技术正确性审计

### 3.1 最大技术问题：V0 安全几乎完全外包给 Hydra 和 watcher liveness

论文承认 V0 没有独立 L1 exit path。非 Head 虚拟参与者若无法自己提交 Hydra contest 或材料化交易，其安全依赖 Head participant/operator/watcher。

这导致严重问题：

- 如果 Head 参与者串谋或离线，非 Head 虚拟用户没有直接 L1 补救。
- 如果 watcher 能观察最新 Lerna 状态但不能让其进入有效 Hydra snapshot，则所谓最新状态无效。
- 如果 Hydra snapshot 的生成需要 Head 多方协作，而协作方拒绝签名，则 Lerna 状态无法桥接到外层 contest。
- 论文的命题只证明：“如果更新状态已经进入更新的 Hydra snapshot 且能及时 contest，则旧 snapshot 不最终化。”这基本是 Hydra 自身性质，不是 Lerna 的新安全性。

因此 V0 更准确地说是 **cooperative or watcher-assisted Head-local accounting layer**，而不是一般意义上的虚拟通道强制执行协议。

### 3.2 “Checkpoint-or-materialise liveness” 假设过强

第 397-447 行附近的 V0 假设块是诚实的，但太强：

- 它假设至少一个授权方能及时 checkpoint/materialise。
- 它假设材料化工作量适配交易大小、执行预算、min-ADA、native asset 和 contestation 窗口。
- 它假设 watcher 能获得足够证明材料和最新状态。

这些正是协议最难保障的部分。将其作为假设会使安全定理变弱为：“如果所有关键安全桥都成功，则安全”。学术上必须进一步说明这些条件在何种网络模型、签名模型和 Hydra 实现流程中可以成立。

### 3.3 非干涉证明未定义

论文反复强调 NonInterferenceProof，但实际只有字段列表：

```text
NonInterferenceProof {
  access_manifest_hash
  rights_dependency_graph_hash
  untouched_rights_commitment
  unchanged_dependency_paths
  conservation_witness
}
```

缺失内容包括：

- rights dependency graph 的节点、边和语义。
- touched/untouched 的精确定义。
- “不影响”是 value-preservation、exit-preservation、membership-preservation 还是 strategy-preservation？
- reserve、operator budget、min-ADA reserve、shared liquidity 改变时如何判定间接影响。
- 插入、删除、缺席证明如何组合。
- reduced signing set 与 proof predicate 的 soundness 关系。
- 对并发更新、同 state number 冲突、proof withholding 的处理。

因此当前非干涉部分不能支撑 reduced-signature safety。若没有完整形式化，保守安全策略只能是 full factory authorization。

### 3.4 材料化算法不完整

第 1173 行后的 V0 Algorithm 给出了排序、派生输出、最大前缀分批、更新 pending root 的框架，但缺少可执行规范：

- settlement descriptor 没有语法、确定性解释器或受限语言。
- “largest prefix that fits” 依赖交易构造、费用、脚本执行单位和 Hydra 实现参数，可能不是各方离线一致可计算的纯函数。
- 如果分批材料化中途 Head 被关闭，pending suffix 的安全语义不明确。
- 如果有多个并发材料化计划，如何避免重复 receipt 或 output ordering 冲突？
- release receipt 是在 compact state 中、独立 UTxO 中还是 witness 中，论文自己列为开放问题。
- min-ADA 变动、ledger 参数更新、reference script 成本变化会影响 deterministic materialisation。

材料化是论文核心贡献之一，但目前只是原则性描述。

### 3.5 Latest-state settlement 语义不足

论文使用 monotonic state numbers，但没有解决：

- 相同 state number、不同 state hash 的 equivocation 处理。
- 不同 scope 之间的偏序关系。
- factory state number 与 virtual channel state number 的同步关系。
- previous_state_hash 是否强制形成线性链，是否允许 DAG 或并行子通道更新。
- 谁确定 required_signers，如何防止 transition family 欺骗性降低签名集合。
- 高 state number 签名但 proof body 被 withholding 时如何处理。

这些都是 virtual channel adjudication 的基础问题。

### 3.6 Head-local validator sketch 太抽象

Aiken-like pseudocode 有助于理解，但无法判断可实现性：

- `check_domain_separated_signatures` 在 Cardano/Plutus/Aiken 中成本如何？签名方案是什么？
- `check_touched_roots_recompute` 对 SMT/KZG/自定义 map 的成本未知。
- `check_non_interference` 是最重的部分，但完全未定义。
- `check_min_ada_for_materialised_outputs` 可能需要 ledger 参数，如何在脚本中访问或承诺？
- “validator-compatible transaction rule used by a Hydra application” 与真正链上 validator 的信任边界不同，不能混用。

附录承认许多检查是 off-chain 或 test-only，这削弱了“validator sketch”的安全意义。

## 4. 逻辑一致性审计

### 4.1 论文定位总体一致

论文反复强调：

- 不替代 Hydra。
- 不是 eltoo for Cardano。
- 不声称 V1 可部署。
- V0 必须在 Head 内部完成材料化。

这一定位基本一致，优于过度营销式白皮书。

### 4.2 但存在若干张力

#### 张力一：声称“latest-state settlement”，但 V0 实际没有 Lerna-level adjudication

V0 中最新状态是否胜出，取决于是否进入 Hydra snapshot 并成功 contest。若最新 Lerna 状态只存在于虚拟参与者之间，且无法转为 Hydra snapshot，则并不 enforceable。

建议将 “latest-state settlement” 改为 “latest-state checkpoint discipline under Hydra contestability assumptions”。

#### 张力二：声称 compactness 缓解 fanout 压力，但最终仍需展开

论文正确承认 compactness 只是延期而非消除 UTxO expansion。但这意味着系统高峰压力可能被推迟到关闭前最脆弱时刻。除非有严格容量控制和持续材料化策略，否则 compactness 可能增加而非降低尾部风险。

#### 张力三：V0 说不修改 Hydra，但又依赖 Head-local validator 或 transaction rule

若只是普通 Hydra 内部交易，脚本语义可以被 Head ledger 执行；若是 Hydra application rule 或 node plugin，则不一定被外层 fanout 强制。论文需要明确哪些规则最终会体现在可 fanout 的 UTxO/validator 中，哪些只是应用层约定。

#### 张力四：非 Head 用户被纳入 virtual participants，但安全权利较弱

论文承认非 Head 用户需要 watcher/operator 代表，但这会改变通道工厂的安全承诺。若用户不能单边退出或 contest，则他们更像托管/半托管应用用户，而非拥有完整 state-channel 保障的参与者。

## 5. 缺陷与风险清单

### 5.1 安全风险

1. **无独立退出路径**：V0 中非 Head 参与者没有 L1 claim。
2. **watcher 失败即安全失败**：观察、构造、提交、contest 任一步失败都会让 stale state finalise。
3. **材料化容量攻击**：恶意或不谨慎操作可累积过多 compact rights，使关闭前无法全部展开。
4. **proof withholding**：签署较高 state 后拒绝提供 touched leaves 或 non-interference proof，可能阻塞材料化。
5. **state equivocation**：同编号不同状态的处理未正式定义。
6. **receipt 膨胀与双花风险**：release receipts 若存储不当，会导致 datum bloat 或无法防止重复声明。
7. **operator 权限混淆**：operator 既负责 liveness 又不能成为 custodian，边界尚未协议化。
8. **min-ADA 与 business ADA 混淆**：论文意识到该问题，但没有给出强制机制。
9. **native asset fanout 失败**：Hydra known issues 中的 token fanout 风险可能直接破坏材料化策略。
10. **descriptor 可编程性风险**：SettlementDescriptor 若过于通用，等同嵌入未验证应用运行时；若过于弱，则无法支持复杂 dApp channel。

### 5.2 工程风险

- Head-local transactions 的大小、执行预算和 snapshot 频率没有数据。
- proof scheme 未选型，无法估算交易体积。
- Hydra 当前实现细节、partial fanout 限制和动态功能变化可能改变假设。
- Aiken/Plutus 验证多签、Merkle proof、native asset conservation 的组合成本可能过高。
- 分批材料化期间的用户体验和失败恢复未定义。

### 5.3 学术风险

- 主要安全论证依赖假设而非机制。
- 没有 formal model，难以同行审稿。
- 与 Perun/general state channels 的比较过浅。
- Morph 是作者本人的 local draft，作为主要灵感来源会削弱外部可信度。
- Hydrozoa 引用的是 WIP/pre-alpha，依赖其“设计伦理”可以，但不能作为强学术基础。

## 6. 与引用工作的比较

### 6.1 Hydra Head

论文正确指出 Hydra 提供外层 snapshot/close/contest/fanout。Lerna V0 不修改 Hydra，这是合理定位。

不足：

- 未具体说明 Hydra snapshot 如何包含 Lerna checkpoint，何时被认为 valid。
- 未分析 Hydra multisignature snapshot 的生成失败场景。
- 未给出 contestation period 与材料化批次数的量化关系。
- 对 partial fanout 的依赖保持谨慎，但没有定量测试。

### 6.2 Interhead Hydra

论文区分 Interhead virtualises Heads、Lerna virtualises rights，这一点清晰。

不足：

- “Lerna 可以存在于 Interhead virtual Head 内”只是构想，没有 timeout composition、安全组合定理或接口定义。
- Interhead 的转换/抵押机制不能自动支持 Lerna，论文也承认，但没有给出组合安全条件。

### 6.3 Hydrozoa

论文从 Hydrozoa 借鉴 execution/custody separation 和 deterministic rollout 的思想，方向合理。

不足：

- Hydrozoa 仍是 WIP/pre-alpha，引用应弱化。
- “inner rollout before outer rollout”是有用类比，但没有严格机制映射。
- 若 Hydrozoa 自身已有 withdrawals/rollout 机制，Lerna 需要说明为什么不直接作为 Hydrozoa ledger 对象实现。

### 6.4 eltoo

论文正确避免把自己包装成 eltoo for Cardano。

不足：

- 最新状态取代旧状态的核心难点在于 adjudication。eltoo 依赖底层可执行的替换/更新机制，而 Lerna V0 依赖 Hydra snapshot。比较应更明确指出 V0 没有独立 eltoo-like enforcement。

### 6.5 Channel Factories 和 Perun

这是相关工作中最薄弱的部分。

- Perun/general state channels 对 virtual channel、adjudicator、subchannel、dispute handling 有成熟形式化。论文只用一段概括，不足以显示 Lerna 与其差异。
- Channel factories 中共享 funding、off-chain 更新、unilateral exit 的权衡与 Lerna 高度相关，比较不够深入。
- 应比较：参与者集合、退出保证、链上争议复杂度、资金锁定、虚拟用户安全、watchtower 假设、状态空间复杂度。

### 6.6 Morph Channel

Morph-to-Lerna mapping table 有帮助，但风险是 Lerna 的许多核心概念看起来直接移植自 Morph。

建议：明确哪些是 Morph 已有，哪些是 Cardano/Hydra 特有新增；否则“新颖性”容易被认为主要是术语迁移。

## 7. 写作质量审计

### 7.1 优点

- 语气谨慎，避免过度承诺。
- 结构完整，覆盖 Introduction、Related Work、System Model、Formal Objects、Lifecycle、Limitations、Evaluation Agenda、Appendix。
- Cardano/Hydra 术语使用总体准确。
- 对 V1 不可部署、证明大小未知、材料化容量未知等问题诚实。
- 表格和伪代码有助于读者理解设计方向。

### 7.2 问题

- “Formal Objects” 并不 formal，只是字段枚举。
- 许多关键词重复出现，但没有收敛为严谨定义：rights、claims、reserve、membership、local exit、descriptor、dependency graph。
- 论文篇幅较长，但技术密度不足，存在设计宣言式重复。
- 相关工作比较偏叙述，缺少对已有安全模型和定理的精确比较。
- 命题与证明太弱，容易被审稿人认为是 trivial restatement of Hydra contestation。
- metadata.yaml 非常简略，只说明是 first research draft。若作为论文提交，需要补充版本、作者机构、artifact 状态、依赖资料、claim scope。

## 8. 需要补强的形式化内容

若要达到密码学协议论文标准，至少需要补齐：

1. **参与方模型**：Head participants、virtual participants、operators、watchers 的权限、腐化模型、在线性要求。
2. **账本模型**：Hydra snapshot、Head-local transaction、fanout UTxO、L1 validator 的抽象接口。
3. **状态机**：Factory init/open/update/rebalance/local exit/materialise/shutdown/fanout 的状态转移。
4. **有效性谓词**：ValidFactoryTransition、ValidMaterialisation、ValidReleaseReceipt、ValidNonInterference。
5. **安全定义**：安全性应至少包括 no double claim、balance conservation、latest-state finality under assumptions、non-interference、exit liveness。
6. **敌手能力**：stale close、withholding proofs、operator censorship、Head participant collusion、oversized materialisation attack。
7. **定理与证明**：不能只证明 Hydra 已有性质，必须证明 Lerna 转换保持安全性质。
8. **复杂度和预算**：proof size、validator ex units、transaction count、materialisation latency。

## 9. 优先改进建议

### P0：必须修改

1. 将论文定位从“protocol”降级为“design note / research agenda”，除非补上形式化协议。
2. 重写 V0 safety theorem，明确它依赖 Hydra snapshot inclusion，并说明非 Head 用户没有单边安全。
3. 给出最小 ADA-only V0 的完整状态机和确定性材料化函数。
4. 定义 release receipt 的存储、消费和防重放规则。
5. 对 non-interference 暂时采用 full authorization fallback，除非能形式化证明系统。
6. 加入 failure scenarios：watcher late、operator refuses、proof withheld、too many outputs、same-number equivocation。
7. 给出最小原型 benchmark 目标，而不仅是 evaluation agenda。

### P1：强烈建议

1. 与 Perun 和 channel factories 做表格化比较。
2. 明确 SettlementDescriptor 的受限语言或 schema。
3. 给出 Hydra contestation window 与 materialisation workload 的参数化公式或示例。
4. 对 native assets 先降级为 future work，首版只声称 ADA-only。
5. 区分 validator-enforced、Hydra-enforced、off-chain-enforced、social/operator-enforced。

### P2：后续工作

1. 研究 V1 claim UTxO，但不要在主贡献中过多依赖。
2. 比较 SMT、sorted Merkle map、KZG 在 Plutus/Aiken 下的可行性。
3. 探索 watchtower/delegation 协议，但需避免 operator custody。
4. 设计 receipt compaction 或 epoch rollover，避免长期 datum bloat。

## 10. 具体措辞风险

建议避免或弱化以下表述：

- “latest-state settlement” 改为 “latest-state checkpointing under Hydra contestability”。
- “defines compact rights” 改为 “sketches compact rights”。
- “factory-local non-interference strengthened by release receipts” 改为 “proposes a non-interference proof obligation and release receipt mechanism”。
- “Lerna Protocol” 若用于学术提交，应说明是 draft protocol framework。

## 11. 审计结论

本文最有价值的部分是它明确指出：在 Cardano/Hydra 中，虚拟通道工厂不能只停留在 Merkle root 或抽象权利上，最终必须面对 eUTxO、min-ADA、native assets、datum、脚本预算和 Hydra fanout。这是正确且重要的系统洞见。

但从严格密码学协议审稿标准看，当前稿件仍缺少可审计协议的核心要素。V0 的安全性依赖过强的 liveness 和材料化假设，V1 明确未解决，非干涉证明和材料化算法未形式化。因此论文目前适合作为研究路线图或架构设计草案，不适合作为声称安全的新协议论文。

最终建议：**大修。先实现并形式化 ADA-only V0，再逐步扩展 native assets、reduced-signature non-interference 和 V1 claim UTxO。**
