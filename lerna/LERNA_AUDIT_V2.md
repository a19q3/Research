# Lerna Protocol 学术论文审计报告 V2

审计对象：`lerna/paper.tex`  
参考基线：`lerna/LERNA_AUDIT.md`  
审计日期：2026-06-05  
审计角色：资深密码学协议与 Cardano/Hydra 方向审稿人  
结论等级：**大修后再审，当前不应作为完整安全协议论文发表**

## 0. 执行摘要

新版论文相比旧审计对象有明显进步。它显著弱化了 V0 的安全声明，明确承认非 Head 虚拟参与者没有独立 L1 exit path，增加了 `Checkpoint-or-materialise liveness`、`Contestability`、`Materialisation capacity` 三个核心假设，并把 Hydra inclusion lemma 明确降格为 Hydra 机制的直接应用，而非 Lerna 独立安全定理。这些修改修复了旧报告中最严重的“过度声称”问题。

但从严格学术审稿角度看，论文仍然主要是一份**设计纪律和研究路线图**，不是完整密码学协议规范。其原创性集中在 Hydra/Cardano 语境下的组合性工程约束，而非新的密码学机制或可证明安全的 adjudication 协议。C1-C3 作为 “prototype obligations” 较诚实，但仍缺少形式化模型、状态机、有效性谓词、证明系统、具体 validator 语义、容量边界与实验数据。

最关键的剩余问题是：

1. **V0 安全仍依赖 Head-authorised actor 的在线性、授权提交能力和 Hydra snapshot inclusion。** 这不是一般虚拟通道的单边可执行安全，而是 watcher-assisted Head-local accounting。
2. **非干涉证明仍未形式化。** `rights_dependency_graph`、`unchanged_dependency_paths`、`conservation_witness` 等核心对象没有可检查语义。
3. **材料化算法比旧版更具体，但仍不是可执行规范。** `BudgetProfile`、`Fits_P(k)`、settlement descriptor、min-ADA、Hydra transaction construction、snapshot confirmation latency 等仍依赖部署测量和链下约定。
4. **“latest-state settlement” 仍有语义张力。** 论文虽承认不是 Lerna 安全定理，但章节名和安全性质表述仍容易让读者误以为存在 Lerna-level adjudication。
5. **与 Hydra/Interhead/Hydrozoa 的定位更谨慎，但比较仍不够严格。** 特别是缺少 timeout composition、custody/execution 接口、Hydrozoa rollout 机制与 Lerna inner materialisation 的形式化映射。

建议作者将论文明确定位为：**“Hydra 内部虚拟通道工厂的设计说明和原型验证计划”**。若要达到学术安全协议论文标准，必须补齐形式化定义、协议算法、安全定理、反例分析和原型测量。

---

## 1. 与旧审计相比的修改评估

### 1.1 已修复或显著改善的问题

新版较好吸收了旧审计中的以下批评：

#### A. V0 安全声明降级

旧审计指出 V0 的 safety 几乎完全外包给 Hydra 和 watcher liveness。新版在第 424-548 行明确写出：

- V0 没有独立 L1 dispute protocol。
- 虚拟参与者不会仅因参与 Lerna virtual channel 获得 independent L1 exit path。
- watcher 只有在拥有 authorised Head participant path 时才有 enforcement value。
- 如果所有可推进状态的 Head participant 腐化、被审查或离线，V0 没有单独补救。

这是重要修复。论文现在对 V0 的安全边界更加诚实。

#### B. Hydra inclusion lemma 被正确降格

第 1330-1355 行的 lemma 和 remark 明确承认：该引理只是 Hydra close/contest 机制应用于包含 Lerna 数据的 Head state，不证明非 Head 虚拟参与者可强制 inclusion，也不证明 proof material availability 或 materialisation capacity。这修复了旧审计中“把 Hydra 自身性质包装成 Lerna 定理”的问题。

#### C. 材料化容量风险被前置

新版在第 1569-1696 行加入了 deterministic materialisation、BudgetProfile、canonical sorting、largest fitting prefix、batch failure semantics 和 shutdown threshold。这比旧版更具体，也明确指出 compactness 是安全债务，不是免费压缩。

#### D. 失败场景表格更诚实

第 1784-1823 行列出 operator refusal、Head participant censorship、late watcher、proof withholding、same-number equivocation、too many outputs、receipt replay、native asset in V0 等失败场景。这显著增强了论文的严肃性。

#### E. V0-ADA 范围更清楚

论文反复限制 V0 为 ADA-only，并把 native assets 放入 extension target。考虑 Hydra known limitations，这一降级是正确的。

### 1.2 仍未修复的问题

旧审计中的核心技术短板仍存在：

- 没有完整状态机。
- 没有形式化 adversarial execution model。
- 没有可执行的 non-interference predicate。
- 没有具体 Plutus/Aiken validator 成本论证。
- 没有证明材料可用性机制。
- 没有材料化容量的数值上界。
- 没有与 Perun/channel factories 的充分形式化比较。

### 1.3 新版本引入的新张力

1. **贡献变成“义务清单”而不是协议结果。** C1-C3 被称为 prototype obligations，这更诚实，但也削弱了可发表贡献强度。
2. **`validator or transaction-rule enforced` 语义混杂。** 第 2021 行将 validator-enforced 与 transaction-rule-enforced 并列，但二者信任边界不同。
3. **same-number equivocation 的处理是 freeze/repair，而非协议化 adjudication。** 这可能导致恶意方用同编号冲突头造成 liveness griefing。
4. **BudgetProfile 使用固定 witness-size estimate。** 若估计保守不足，材料化计划可能在真实 ledger 参数或签名集合变化下失败；若过度保守，则严重降低容量。

---

## 2. 新颖性评估：哪些贡献真正原创？

### 2.1 真正有原创潜力的部分

#### C1. Hydra Head 内部的 factory-rights discipline

把虚拟通道权利表示为 Head-local compact rights，而不是每个临时通道都成为 Head-visible UTxO，这是合理的 Cardano/Hydra 专用设计方向。它与传统 channel factory 概念相关，但引入了 Hydra fanout、eUTxO、min-ADA、datum/reference script、Head snapshot 等具体约束。

严格说，这不是新的密码学原语，也不是新的 virtual channel adjudicator，而是**Cardano/Hydra 下的状态表示纪律**。其原创性强度为中等偏弱，但有工程研究价值。

#### C2. Inner materialisation before outer fanout

论文反复强调 compactness 是 debt，必须在外层 Hydra fanout 或 Hydrozoa-style rollout 前 repayment。这是本文最清晰、最有价值的安全设计原则。它准确抓住了 Hydra 语境中的关键问题：Hydra 最终 fanout 的对象仍是普通 UTxO set，紧凑权利不能魔法般逃避最终展开。

但该点更像**必要安全约束**，不等于已解决机制。若没有容量上界和算法实现，它只能算设计洞察。

#### C3. Release receipts 与 factory-local non-interference 的组合

release receipts 用于防止同一 right 同时作为 compact entry 和 materialised output 被声明，这在 eUTxO 工程中有实用价值。将 receipts 与 touched leaves、rights-dependency graph、local exit 绑定，是有潜力的协议机制。

然而，receipt/nullifier/spent marker 类机制本身并非新概念。其原创性取决于是否能给出 Cardano/Hydra 下的具体、低成本、可验证编码。当前尚未做到。

### 2.2 不应声称原创的部分

以下内容不应被包装成原创贡献：

- virtual channels。
- channel factories。
- monotonic state numbers。
- latest-state supersession。
- watcher-assisted contest。
- deterministic rollout 的一般思想。
- compact commitment + Merkle proof 的状态压缩。
- non-punitive stale-state replacement。

论文现在已经较好避免这类过度声明，但仍应在摘要和结论中进一步强调“组合性设计纪律”而非“协议发明”。

### 2.3 学术新颖性强度评级

| 贡献 | 新颖性 | 当前成熟度 | 审稿评价 |
|---|---:|---:|---|
| C1 Hydra 内 compact factory rights | 中等 | 低到中 | 有 Cardano 专用价值，但缺完整语义 |
| C2 materialisation debt before fanout | 中等 | 中 | 最清晰贡献，但目前是约束而非完成算法 |
| C3 non-interference + receipts | 中等偏弱 | 低 | 潜力存在，形式化不足 |
| V0 latest-state safety | 低 | 低 | 主要是 Hydra contest 的包含性应用 |
| V1 claim UTxO | 未能评价 | 很低 | 明确为 future work，不能算贡献 |

---

## 3. 技术正确性审计

### 3.1 V0 enforcement 不是虚拟通道单边安全

V0 的 enforcement path 是：

```text
latest Lerna evidence
  -> authorised Head-local checkpoint/materialisation transaction
  -> Hydra snapshot inclusion
  -> Hydra close/contest/fanout
```

如果中间任何一步失败，Lerna V0 没有独立补救。尤其是：

- 虚拟参与者知道最新状态不等于能强制它进入 Head。
- watcher 观察到 stale close 不等于能 contest。
- 有签名 header 不等于有 proof material。
- 有 proof material 不等于材料化交易能 fit。
- 有材料化交易不等于 Hydra participants 会签出新 snapshot。

因此，V0 的准确安全描述应是：在存在至少一个有证据、有权限、有时间、有交易容量、且能促成 Hydra snapshot 的诚实 actor 时，Lerna compact rights 可通过 Hydra 机制被间接执行。这比通常 state channel / virtual channel 论文中的 adjudicator-enforced safety 弱得多。

### 3.2 假设块清楚但极强

第 496-533 行三个假设修复了旧版模糊性，但这些假设恰好覆盖最难的问题：

- 谁能把 Lerna state 放入 Hydra snapshot？
- 谁能及时 contest？
- materialisation workload 是否 fits？

从证明角度，这会导致安全性质接近条件化真理：如果最新状态已成功变成 Hydra 可 contest snapshot，则旧 snapshot 不会胜出。这本质上是 Hydra 的安全性。

建议：论文应将这三个假设提升为**部署准入条件**，并在 abstract 和 contribution 中明确：C1-C3 的 V0 安全只在这些条件下成立。

### 3.3 Head snapshot inclusion 的细节仍不足

论文说 authorised actor 可以 checkpoint/materialise 并获得 Hydra snapshot，但没有说明：

- Hydra snapshot 的多签生成何时可能被少数参与者阻塞。
- Lerna transaction 在 Head 内的 validity 与 snapshot confirmation 的关系。
- 如果 Head participants 对 Lerna transaction 有不同 local validation view，如何处理。
- snapshot hint 是否有任何强制语义，还是只是辅助字段。
- watcher 如何获得 newer Hydra snapshot，是必须先有 snapshot 还是可以提交 Head transaction 触发 snapshot。

这部分对 V0 是核心，而目前仍停留在口头模型。

### 3.4 State monotonicity 与 equivocation 仍未充分解决

第 1315-1328 行定义固定 scope 下高 state number supersedes 低 state number。第 1811-1813 行说 same-number equivocation 会 freeze affected scope until repair。

问题包括：

1. **freeze 是 liveness 攻击面。** 恶意参与者可用同编号不同 hash 阻塞退出和材料化。
2. **repair state 的 required signers 未定义。** 若需要全体签名，攻击者可拒签；若只需部分签名，可能破坏安全。
3. **scope 偏序未定义。** factory scope、virtual channel scope、reserve scope、materialisation plan scope 之间是否存在依赖关系不清楚。

建议定义 `state_scope` 类型系统、每个 transition family 的 parent state relation、same-number conflict 的 admissible repair transition，以及 equivocation evidence 的处理规则。

### 3.5 Non-interference proof 仍不是 proof

第 978-991 行给出 `NonInterferenceProof` 字段，但字段不是语义。当前没有定义足够的可检查谓词。

缺失内容包括：

- rights dependency graph 的节点集合。
- 边的语义：value dependency、exit dependency、liveness dependency、fee sponsorship dependency 是否区分。
- untouched rights 的不变性到底是什么：金额不变、可退出路径不变、材料化顺序不变、min-ADA reserve 不变、deadline 不变，还是全部。
- shared reserve 被局部 action 改变时如何证明未触及方不受影响。
- operator budget 与 participant balance 分离如何被 proof 检查，而不是部署承诺。
- 插入和删除节点如何影响 absence proofs。
- settlement descriptor 变化是否会间接影响所有 rights。

当前 C3 中“reduced-signature local exits”不能被认为安全。论文较诚实地说 V0 prototype 应 fallback to full authorization，但这意味着 C3 的强版本仍未实现。

### 3.6 SettlementDescriptor 过于抽象

`SettlementDescriptor` 被称为 bounded rule set，不是 application runtime。但字段如 `output_derivation_rules`、`asset_allocation_rules`、`min_ada_rules`、`datum_and_reference_script_rules`、`local_exit_rules`、`rebalance_and_splice_rules` 都没有语法、解释器、确定性约束或版本化语义。

这会影响多个核心性质：

- materialisation equivalence 无法检查。
- receipt 的 `replacement_output_hash` 是否正确无法判断。
- `Fits_P(k)` 无法在各方之间确定性计算。
- descriptor upgrade 是否影响旧 rights 未定义。
- descriptor body withholding 会阻塞验证。

建议至少为 V0-ADA 定义一个极小 descriptor grammar，例如仅支持 participant destination key hash、lovelace amount、fixed datum/no datum、no reference script、fixed min-ADA rule、no native assets。

### 3.7 Materialisation algorithm 有改进但仍不可执行

第 1599-1664 行算法是新版最大技术增强，但仍存在严肃缺口：

1. **`BudgetProfile` 与真实 ledger 参数的关系未定义。** Cardano min-ADA、tx size、execution units、reference script、datum encoding、fee model、protocol params 变化都可能影响 `Fits_P(k)`。
2. **固定 witness-size estimate 不足以保证 fit。** 签名集合、proof 大小、receipt 数量、datum 长度都可能可变。
3. **largest prefix 策略可能非最优。** 若某个早期 right 过大导致 no k fits，后续小 rights 也被阻塞。canonical prefix 有确定性，但可能带来 griefing。
4. **pending suffix 在 V0 close 中仍危险。** 论文承认若 Head closes before batch i，pending suffix has no independent L1 claim。这意味着算法并未解决最坏情况下的退出安全。
5. **并发规则只按 sourceFactoryStateHash 限制。** 若有更新产生新 source state，同时旧 materialisation plan 仍在传播，冲突处理需要更细的 state transition relation。
6. **batch failure recovery 依赖 cooperative progress。** 失败后 same batch can be reconstructed 不等于可以被强制执行。

### 3.8 Release receipt 机制方向正确但仍需细化

release receipt 是新版中比较实用的机制，但还缺：

- receipt signer set 的精确定义。
- receipt 与 old state/new state 的约束方向。
- receipt root 的增长控制和压缩策略。
- receipt rollback 与 Hydra snapshot rollback 的关系。
- materialised output 被 decommit 或再次转移时，receipt 的生命周期。
- receipt id 是否足以抵抗跨 plan、跨 epoch、跨 descriptor 的 replay。

当前 receipt 机制足以作为设计想法，不足以作为安全证明基础。

### 3.9 Aiken/Plutus/Head-local validator sketch 仍不能证明可实现性

论文把许多检查列为 validator or transaction-rule enforced、off-chain construction enforced、test-only until benchmarked。这个分类诚实，但暴露出安全边界混乱：

- 真正链上 validator 可被 L1 强制。
- Hydra Head 内的普通 script 受 Hydra ledger 语义约束。
- Application-specific transaction rule 可能只是参与者约定。
- Off-chain construction enforced 不是安全 enforcement，只是 honest implementation behavior。
- Test-only assertions 不能出现在安全性质的机制列表中。

建议论文为每个 property 标出 enforcement layer：L1 Plutus、Hydra ledger script、Hydra node/application rule、off-chain protocol、test-only。否则审稿人无法判断信任边界。

---

## 4. 逻辑一致性审计

### 4.1 总体定位更一致

新版在多个位置重复强调：

- 不替代 Hydra。
- 不是 eltoo for Cardano。
- V0 不提供 non-Head user 独立 L1 claim。
- V1 claim UTxO 不声称可部署。
- compactness 是 materialisation debt。

这些表述总体一致，明显优于旧版。

### 4.2 仍存在的主要张力

#### 张力一：章节名和 safety property 仍可能过强

虽然第 1349-1355 行说 inclusion lemma 不是 Lerna security theorem，但 `Latest-state Settlement Semantics` 和 `State monotonicity` 的表述仍容易让读者误解为 Lerna V0 有独立 latest-state settlement。建议改名为：

- `Latest-state Checkpoint Discipline in V0`
- `Hydra-mediated inclusion safety`

#### 张力二：compactness 缓解 fast path 压力，但增加 tail risk

论文现在承认 compact rights 最终必须展开。但这意味着系统可能把 UTxO explosion 从正常运行期推迟到关闭前的最脆弱时刻。若没有严格 admission control，compactness 可能增加而非降低风险。

#### 张力三：V0 说不修改 Hydra，但依赖 Head-local validator 或 transaction rule

如果 Lerna 规则只是普通 Hydra ledger transaction 中的 script，则较清楚；如果是 Hydra application rule 或 node plugin，则未必被外层 fanout 强制。论文需要明确哪些规则会体现在可 fanout 的 UTxO/validator 中，哪些只是应用层约定。

#### 张力四：virtual participants 被纳入协议，但安全权利较弱

非 Head 用户如果不能自己 submit、contest 或 trigger materialisation，则其地位更接近由 Head participant/operator 代表的应用用户，而非拥有完整 state-channel 保障的参与者。论文现在承认这一点，但应在 contribution 和 abstract 中更醒目。

#### 张力五：V0-ADA 与论文多资产语言并存

论文已将 native asset 放入 extension，但文中仍多次出现 asset registry、asset allocation、native-asset extension 等字段。作为 V0 设计 note，可以保留；作为 protocol specification，会造成 claim surface 膨胀。建议把 extension 字段移到附录或 future profile。

---

## 5. 缺陷与风险清单

### 5.1 安全风险

1. **无独立退出路径。** V0 中非 Head 参与者没有 L1 claim。
2. **watcher 失败即安全失败。** 观察、证明获取、交易构造、提交、snapshot inclusion、contest 任一步失败都会让 stale state 可能 finalise。
3. **Head participant censorship。** 若授权 Head actor 不推进 Lerna transaction，Lerna 无 V0 补救。
4. **材料化容量攻击。** 恶意或不谨慎操作可累积过多 compact rights，使关闭前无法展开。
5. **大 right prefix griefing。** canonical prefix 中早期大输出可能阻塞后续小输出材料化。
6. **proof withholding。** 签署高编号 header 后拒绝提供 touched leaves、descriptor body、receipt witnesses，会阻塞验证。
7. **same-number equivocation。** 当前 freeze 策略可能被用作 liveness attack。
8. **descriptor withholding 或升级攻击。** 若 descriptor body 不可得或升级影响旧 rights，材料化可能失败。
9. **release receipt 膨胀。** 长期 factory 的 receipt root/datum 可能膨胀，影响 Head-local transaction fit。
10. **operator 权限混淆。** operator 负责 liveness 与 fee/liquidity，但“不拥有资金权限”的边界尚未协议化。
11. **min-ADA reserve 与 business balance 混淆。** 论文意识到问题，但 V0 仍需要强制机制。
12. **Hydra snapshot availability。** Lerna state 有效但未进入 snapshot 时没有外层可 contest evidence。
13. **privacy 泄漏。** access manifests、touched leaves、materialisation order 会泄漏关系图和活动信息。
14. **native asset extension 风险。** Hydra known limitations 下，未燃尽 token 或大 bundle 可破坏 finalisation。

### 5.2 工程风险

- Head-local transaction bytes、datum bytes、execution units 未测量。
- Proof scheme 未选型，无法估算 proof size。
- Hydra partial fanout、deposit/decommit、snapshot confirmation latency 与 Lerna 交互未实测。
- Aiken/Plutus 多签、Merkle proof、receipt root、ADA conservation 组合成本可能过高。
- 分批材料化期间的用户体验、失败恢复、并发更新策略未定义。

### 5.3 学术风险

- 主要安全论证依赖假设而非机制。
- 没有 formal model，难以同行验证。
- 与 Perun/general state channels 的比较仍过浅。
- Hydrozoa 引用的是 pre-alpha/WIP，适合作为设计启发，不适合作为强学术基础。
- C1-C3 当前是 design obligations，尚非 proven contributions。

---

## 6. 与引用工作的比较

### 6.1 Hydra Head

论文对 Hydra 的基本定位总体准确：Hydra 提供外层 snapshot/close/contest/fanout，Lerna V0 不修改 Hydra。对 Hydra known limitations 的讨论也更谨慎。

不足：

- 未具体说明 Lerna checkpoint 如何进入 Hydra snapshot。
- 未分析 Hydra multisignature snapshot 生成失败场景。
- 未给出 contestation period 与材料化批次数的量化关系。
- 对 partial fanout 的依赖保持谨慎，但缺少测试和接口说明。

### 6.2 Interhead Hydra

论文区分 Interhead virtualises Heads、Lerna virtualises rights，这一点清晰且合理。第 303-319 行对 Interhead 的描述没有明显过度声称。

不足：

- “Lerna 可以存在于 Interhead virtual Head 内”只是构想，没有 timeout composition。
- Interhead 的 collateral/conversion 机制不能自动支持 Lerna local rights，论文应明确组合条件。
- 若 Interhead 失败并转换为 regular Hydra Head，Lerna pending materialisation 的状态如何迁移未定义。

### 6.3 Hydrozoa

论文对 Hydrozoa 的使用比旧版更谨慎，承认其 pre-alpha 和 public specifications in flux。将 Hydrozoa 的 deterministic rollout 作为“设计伦理”借鉴是合理的。

不足：

- Hydrozoa outer rollout 与 Lerna inner materialisation 只是类比，没有形式化接口。
- 若 Hydrozoa 自身已有 withdrawals/rollout/treasury state，Lerna 应说明为何不是直接作为 Hydrozoa ledger object 实现。
- “execution/custody separation” 在 Lerna V0 中并未真正实现为强制机制，因为 V0 仍依赖 authorised Head actor。

### 6.4 Eltoo

论文正确避免把自己包装成 “eltoo for Cardano”。但仍需强调：eltoo 的关键是底层可执行替换机制；Lerna V0 没有独立替换/adjudication，只是通过 Hydra snapshot contest 间接实现。

### 6.5 Channel Factories 和 Perun

这是相关工作最薄弱处。Perun/general state channels 对 virtual channel、adjudicator、subchannel、dispute handling 有成熟形式化。论文只用一段概括，不足以显示 Lerna 与其差异。

建议增加比较表：

| 维度 | Channel factories/Perun | Hydra Head | Lerna V0 | Lerna V1 target |
|---|---|---|---|---|
| 谁能单边退出 | 通常有 adjudicator 路径 | Head participant | 非 Head 用户不能 | 取决于 claim UTxO |
| 最新状态执行 | adjudication | snapshot contest | Hydra-mediated | validator-mediated |
| 资金表示 | shared funding | Head UTxO set | compact rights then materialised UTxOs | claim UTxO + withdrawals |
| watcher 假设 | 取决于协议 | Head participant online | watcher plus authorised path | watcher plus L1 claim path |
| 主要瓶颈 | dispute complexity | fanout size | materialisation capacity | Plutus budget |

### 6.6 Morph Channel

旧审计指出 Morph-to-Lerna mapping 容易让贡献看起来像术语迁移。新版 paper.tex 当前相关表述减少，主要强调 C1-C3 Cardano/Hydra obligation，这是改善。但若其他文档继续以 Morph 为背景，建议明确标注：哪些概念来自 Morph，哪些是 Hydra/Cardano 专有新增。

---

## 7. 写作质量审计

### 7.1 优点

- 语气比旧版更谨慎，避免过度营销。
- 结构完整，覆盖 related work、system model、threat model、formal objects、lifecycle、security properties、failure scenarios、evaluation agenda。
- Cardano/Hydra 术语总体准确。
- 对 V0/V1 边界、ADA-only 限制、Hydra fanout 风险、native asset 风险的表述较诚实。
- 失败场景表格非常有价值，应保留。

### 7.2 主要写作问题

1. **篇幅较长但协议核心仍不够具体。** 很多章节解释“为什么重要”，但没有给出 exact predicate。
2. **形式对象像字段清单，不像规范。** 定义缺少 validity、transition、encoding、canonicalization、error cases。
3. **extension surface 太多。** V0-ADA、native assets、Interhead、Hydrozoa-style ledgers、V1 claim UTxO 同时出现，容易稀释主线。
4. **机制层级混杂。** L1 validator、Hydra script、Head-local transaction rule、off-chain protocol、test-only assertion 应严格分层。
5. **某些措辞仍偏强。** “Security Properties” 中的 properties 应改为 “Target Properties and Enforcement Status”，并标注 conditional/test-only/future。

### 7.3 建议的措辞收缩

- 将 “latest-state settlement” 改为 “Hydra-mediated latest-state checkpointing”。
- 将 “Factory non-interference” 改为 “Non-interference target; full-authorisation fallback in V0”。
- 将 “Safe materialisation” 改为 “Conditionally safe materialisation under measured capacity bounds”。
- 将 “No accidental Head-level UTxO explosion” 改为 “Fast-path UTxO compactness; shutdown expansion remains required”。

---

## 8. 具体可改进之处

### 8.1 立即可做的论文修改

1. 在摘要中加入一句：**V0 does not provide unilateral L1 exit for non-Head virtual participants.**
2. 将 `Latest-state Settlement Semantics` 改名为 `Hydra-mediated Latest-state Checkpoint Semantics`。
3. 为每个安全性质增加 enforcement layer：L1、Hydra ledger、Head application rule、off-chain、test-only、future。
4. 把 native-asset 字段移到 extension appendix，V0 主体只保留 ADA-only。
5. 为 V0 settlement descriptor 给出极小 grammar。
6. 为 `AccessManifest` 和 `TouchedLeaves` 定义 exact validity predicates。
7. 将 non-interference 明确标为 future proof mode，V0 默认 full factory authorization。
8. 增加 Hydra snapshot inclusion 流程图和失败路径。
9. 增加与 Perun/channel factories 的比较表。
10. 增加 `Materialisation admission invariant`：任何时候 pending work 必须小于 measured shutdown envelope。

### 8.2 需要补齐的形式化内容

1. **状态机**：FactoryState、VirtualChannelState、ReceiptState、MaterialisationPlanState。
2. **转移关系**：OpenVC、UpdateVC、Checkpoint、LocalExit、MaterialiseBatch、ShutdownPrepare、CloseReady、RepairEquivocation。
3. **有效性谓词**：signature validity、touched root recomputation、conservation、receipt freshness、descriptor determinism、materialisation equivalence。
4. **不变量**：ADA conservation、receipt uniqueness、no unresolved unilateral claims at V0 fanout、materialisation margin invariant、descriptor immutability。
5. **安全定理**：conditional anti-replay、conditional no-double-materialisation、conditional balance conservation、conditional latest-state inclusion safety。
6. **反例讨论**：无 Head-authorised actor 时 V0 不可能保护 non-Head user；materialisation capacity 是 safety prerequisite；无 non-interference proof 的 reduced-signature local exit 不安全。
7. **实验验证**：Hydra local cluster measurements、tx size、ex-units、datum size、max outputs per batch、snapshot confirmation latency、contestation-period sensitivity。

### 8.3 对 V0 原型的最小建议

V0 原型应极度收缩：

- ADA-only。
- Two or three Head participants。
- All virtual participants initially also Head participants，或明确 watcher delegation。
- Full factory authorisation for all local exits。
- No native assets。
- No dynamic membership。
- Fixed descriptor grammar。
- One active materialisation plan。
- Explicit receipt root。
- One stale snapshot contest demo。

在 reduced-signature non-interference 没有形式化和 benchmark 前，不应把它作为 demo 的安全功能，只能作为 future target。

### 8.4 对论文结构的建议

建议重排如下：

1. Introduction and claim scope。
2. Non-claims and V0 limitation table。
3. Hydra interface assumptions。
4. V0-ADA protocol only。
5. Formal objects for V0 only。
6. V0 algorithms。
7. Conditional security properties。
8. Failure scenarios。
9. Related work comparison。
10. Extensions: non-interference、native assets、V1 claim UTxO、Hydrozoa-style ledgers。

这样可避免读者把未来扩展误读为当前贡献。

---

## 9. 严格结论

新版论文比旧审计对象更加诚实、严谨，也更适合进入研究讨论。尤其是它承认：

- V0 没有独立 L1 exit。
- watcher 必须有 submission path。
- materialisation capacity 是安全前提。
- Hydra inclusion lemma 不是 Lerna 独立安全定理。
- V1 claim UTxO 不可声称可部署。

这些修改修复了旧版最严重的过度声明问题。

但从密码学协议审稿标准看，论文仍未达到“完整协议”或“可证明安全系统”的水平。它目前最准确的身份是：

> 一份针对 Cardano Hydra 的 Head-local virtual-channel factory 设计纪律、威胁边界说明和原型验证计划。

最终审稿意见：**大修后再审**。若作为 workshop design note，可在进一步收缩 claim、补充 V0 trace 和 comparison table 后考虑接收；若作为正式安全协议论文，当前应拒绝，直到形式化模型、协议算法、安全定理和原型测量完成。
