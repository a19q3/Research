# Lerna Protocol 学术论文审计报告 V3

审计对象：`lerna/paper.tex`  
审计日期：2026-06-05  
审计角色：资深密码学协议审稿人，Cardano eUTxO / Hydra / channel factories 方向  
审计方法：独立通读 `paper.tex` 当前全文，并参考 `LERNA_AUDIT.md`、`LERNA_AUDIT_V2.md` 作为修订对照，但本报告不复刻前两版结论。  
结论等级：**弱拒 / 大修后再审**。作为 Hydra 原生设计说明有价值，作为顶级会议密码学协议论文仍明显不足。

---

## 0. 执行摘要

当前版本相比早期版本有实质改进。论文已经基本去除 CKB/Morph 迁移痕迹，将定位重新收束为 **Hydra-native virtual-channel factory discipline**，并且明确承认 V0 是 ADA-only、Head-local、watcher-assisted 的设计，而非给非 Head 用户提供独立 L1 unilateral exit 的完整虚拟通道协议。这使论文在叙事上比 V1 审计时更自洽。

最值得肯定的变化包括：

1. 摘要和引言把贡献限定为 C1-C3 的 prototype obligations，而非完成的安全定理。
2. 威胁模型明确承认 operator refusal、Head participant censorship、watcher lateness、proof withholding、same-number equivocation、materialisation overload 等失败模式。
3. 新版补入 `Rights dependency graph`、`Touched closure`、`ValidNonInterference` 的定义，比 V2 中单纯字段列表更接近协议语义。
4. 材料化算法新增 canonical ordering、`BudgetProfile`、`Fits_P(k)`、batch failure semantics 和 concurrency rule，较旧版更可审计。
5. 与 Hydra、Interhead、Hydrozoa、eltoo 的关系表述更谨慎，基本避免了“eltoo for Cardano”或“替代 Hydra/Hydrozoa”的误导。

但是，按顶级会议审稿标准，论文仍不满足完整协议论文要求。主要原因是：

- V0 的核心安全仍是条件式的：只要某个 authorised actor 能及时把最新 Lerna 状态 checkpoint 或 materialise 到 Hydra snapshot，并且 contest 成功，则 stale snapshot 不最终化。这几乎完全依赖 Hydra 的已有安全机制和部署层 liveness。
- 非 Head virtual participant 的权利不是 self-enforcing，而是 delegation / watcher / operator assisted。论文承认这一点，但标题中的 “virtual channel factories” 仍容易让读者期待传统 channel factory / Perun 式单边可执行安全。
- 状态机仍是 admissibility summary，不是完整状态转换函数。缺少错误状态、并发语义、scope 偏序、repair transition、delegation protocol、proof availability protocol。
- Non-interference 虽已形式化为图闭包条件，但仍没有给出 commitment scheme、证明语法、验证算法、复杂度和 soundness 定理。
- Materialisation capacity 是安全假设而非协议保证。论文没有给出任何数值边界或实验数据，无法判断 V0 是否在现实 Hydra 参数下可部署。
- `validator or transaction-rule enforced` 仍混淆脚本强制与应用规则强制，两者安全边界不同。

总体判断：**该论文现在是一份严肃、诚实、Cardano/Hydra 原生的研究设计说明和原型路线图；尚不是可发表的完整密码学协议或可部署安全系统说明。** 若投稿，应明确类别为 systems design note / position paper / prototype agenda，而非 full cryptographic protocol paper。

---

## 1. 新颖性评估

### 1.1 真正有原创潜力的贡献

#### C1. Hydra Head 内部的 compact factory rights discipline

论文提出在 Hydra Head 或 Head-like ledger 内用 compact factory object 表示多组 virtual-channel rights，而不是让每个临时通道都成为 Head-visible UTxO。这个方向在 Cardano/Hydra 语境下有原创潜力，原因是它直接针对 Hydra fanout 的 UTxO 集合限制、min-ADA、datum、reference script、native asset、Hydra snapshot 等具体约束。

但严格来说，这不是新的密码学原语，也不是新的虚拟通道 adjudicator。它更像是：

- channel factory 思想在 Hydra Head 内部的状态表示纪律；
- Merkle/commitment-style compact state 在 eUTxO 结算语义下的工程化约束；
- 对 Hydra fanout debt 的显式建模。

新颖性等级：**中等**。原创性来自 Cardano/Hydra 约束下的组合和边界定义，而非底层密码学。

#### C2. Deterministic inner materialisation before outer fanout / rollout

这是当前论文最清楚、最有价值的观点。论文反复强调 compactness is debt：紧凑权利不能直接逃逸到 Hydra fanout，必须先展开为 ordinary Head UTxOs、被 fresh agreement reabsorbed，或被 release receipt 释放。

该点是 Hydra 原生论文定位最站得住的部分，因为 Hydra fanout 结算的是 UTxO 集，不是应用私有 Merkle root。若 Lerna 在 Head 内维护 compact rights，那么在 outer settlement 前必须有 inner rollout。这一点与 Cardano eUTxO 和 Hydra 机制紧密相关，不是简单套用 Bitcoin/eltoo 概念。

但它仍主要是**必要安全纪律**，不是已完成机制。论文没有证明任意 admissible factory state 都能在 contestation envelope 内 materialise，也没有给出 admission control 算法确保 fanout debt 始终 bounded。

新颖性等级：**中等偏强的系统设计洞察，但机制成熟度中低**。

#### C3. Release receipts + factory-local non-interference

release receipts 用于防止 same right 同时存在于 compact active rights 和 materialised output 中。这在 eUTxO 场景中很实用，尤其因为 fanout 后普通 UTxO 与 Head-local compact state 的边界必须明确。

新版将 non-interference 从纯字段列表推进到 `Rights dependency graph`、`Touched closure` 和 `ValidNonInterference` 条件，这是实质进步。它正确认识到：balance proof 不足以证明 local exit 安全，因为 shared reserve、membership、descriptor、operator budget 变化都可能间接影响 untouched participants。

但是：

- receipt/nullifier/spent marker 类思想不是新概念；
- dependency graph + closure 是常见信息流 / frame condition / separation logic 风格证明义务；
- 当前没有具体证明系统、验证成本和 soundness theorem。

新颖性等级：**中等偏弱到中等**，取决于未来是否能给出 Cardano/Hydra 下低成本可验证实现。

### 1.2 明确不是原创的内容

以下内容不应被论文暗示为原创：

- state channels；
- virtual channels；
- channel factories；
- Perun-style virtual channel adjudication；
- latest-state supersession；
- monotonic state numbers；
- non-punitive replacement，eltoo 的核心直觉；
- watcher-assisted close/contest；
- compact commitments + Merkle proofs；
- deterministic rollout / batching 的一般思想。

当前版本整体已经较好避免直接声称这些是原创，但结论中的 “making virtual channels safer, compact, and auditable” 仍可进一步降级为 “providing a design discipline and prototype agenda”。

### 1.3 新颖性总评

| 项目 | 新颖性 | 当前成熟度 | 审稿评价 |
|---|---:|---:|---|
| Hydra 内 compact factory rights | 中 | 中低 | 有 Cardano/Hydra 专用价值，但缺完整语义 |
| Inner materialisation debt | 中偏强 | 中 | 最清晰贡献，仍缺 admission control 和测量 |
| Release receipts | 中低 | 中 | 工程上必要，但概念上类似 nullifier/spent marker |
| Non-interference graph discipline | 中 | 低到中 | 新版显著改善，但缺 proof system |
| V0 latest-state settlement | 低 | 低 | 主要依赖 Hydra snapshot contest |
| V1 claim UTxO | 不能计为贡献 | 很低 | 明确 future work，不能作为本文贡献 |

---

## 2. 技术正确性审计

### 2.1 V0 安全声明较诚实，但仍不是完整虚拟通道安全

论文现在明确写出 V0 的 enforcement path：

```text
latest Lerna evidence
  -> authorised checkpoint/materialisation inside Hydra Head
  -> Hydra snapshot inclusion
  -> Hydra close/contest/fanout
```

这一路径技术上是合理的，但它不是传统 channel factory / virtual channel 论文中的 self-enforcing path。关键差异是：

- 非 Head virtual participant 不能自己向 L1 提交 Lerna claim；
- watcher 只有观察能力不够，必须有 authorised Head submission path；
- 最新 Lerna header 若未进入 Hydra snapshot，对 fanout 没有直接约束；
- proof body、touched leaves、receipt witnesses 若被 withheld，高编号 header 也无法验证；
- materialisation 若无法 fit，签名正确也没有实际安全意义。

因此 V0 的准确安全类别是：

> watcher-assisted / authorised-actor-assisted Head-local accounting discipline under Hydra contestability assumptions。

而不是：

> general virtual channel factory with unilateral exit for all virtual participants。

论文已经承认这一点，但标题和 “virtual-channel factory” 术语仍可能造成过强期待。建议在标题、副标题或摘要中加入 “Head-local assisted” 或 “prototype discipline” 类限制词。

### 2.2 Hydra inclusion lemma 技术上正确但安全含量很弱

`Hydra inclusion lemma for V0` 的证明是正确的：如果一个更新的 Hydra snapshot 已经包含最新 Lerna checkpoint/materialisation，并且可以在 contestation deadline 前 contest，那么 Hydra final fanout 应遵循更新 snapshot。

但该 lemma 几乎是 Hydra close/contest 机制的直接实例化。它不证明：

- 非 Head 用户能强制 inclusion；
- Head participants 不会 censorship；
- watcher 拿得到 proof material；
- materialisation 能在剩余窗口内 fit；
- Lerna transition 的 validator semantics 已被 Hydra 节点一致执行；
- off-chain virtual state 与 Head-local factory root 的关系始终可恢复。

论文已经在 remark 中承认 “Not a Lerna security theorem”，这是正确修复。审稿建议是：把该 lemma 移到 “Assumption implications” 或 “Hydra dependency” 小节，避免放在 “Latest-state Settlement Semantics” 中产生 theorem-like 错觉。

### 2.3 `ValidNonInterference` 有进步，但仍不可执行

新版定义了：

- dependency graph `G_F=(V,E,λ)`；
- vertex categories：balance、reserve、membership、descriptor、receipt、operator；
- edge semantics：改变 u 可能影响 v 的 value、exit path、materialisation viability、authorisation condition；
- touched closure `cl_G_F(M)`；
- reduced signer set 必须覆盖 closure 内权利或授权策略变化涉及的 participant。

这是相对 V2 的重要进步。但仍有以下技术缺口：

1. **图方向与 closure 语义可能过宽或过窄。** 当前定义 `u -> v` 表示 changing u may change v，closure 从 A(M) 沿 outgoing paths 扩展。若边方向统一，这可捕获依赖后果。但如果某些检查需要反向依赖，例如某 right 依赖 shared reserve，改变 right 是否影响 reserve feasibility，则必须明确双向或分类型边。否则实现者可能漏掉 reserve exhaustion 影响。

2. **“outside closure unchanged” 不等于 untouched participants 安全。** 如果 closure 很大，reduced signer set 可能接近 full factory。若 closure 较小，可能漏掉全局 invariants，例如 total min-ADA reserve、operator budget、global descriptor version、membership quorum threshold。论文需要区分 local frame conditions 与 global invariants。

3. **缺少 proof witness grammar。** `W` 如何证明 outside vertices unchanged？是 Sparse Merkle multiproof、vector commitment、KZG、UTxO map diff，还是 off-chain trace replay？不同选择直接影响 Aiken/Plutus 可行性。

4. **缺少 soundness statement。** 至少应证明：若 `ValidNonInterference(F,F',M,W)` 成立且 required signers 覆盖 closure 内 participant，则任何 closure 外 participant 的 claim value、exit destination、authorisation policy、materialisation viability 均不变。

5. **动态 membership 未处理。** membership vertex 改变可能影响 signer registry、delegation、watcher authority、exit path。当前 definition 承认 membership dependency，但没有给出 transition-specific rules。

6. **proof availability 未绑定到签名。** failure table 说 header signing should bind proof availability policy，但 formal object 里并没有强制签名同时承诺 proof body availability、erasure coding、data availability committee 或 witness retrieval rule。

结论：non-interference 从“概念空白”进步为“可讨论的形式化草图”，但尚不能支撑 reduced-signature safety。V0 prototype 若保持安全，应默认 full factory authorisation，reduced signature 只能作为实验功能。

### 2.4 状态机仍不完整

`Minimal ADA-only V0 State Machine` 给出了状态元组和模式：

```text
Init, Open, Checkpointing, Materialising, ShutdownPreparing, Settled, Aborted
```

以及 transitions：initFactory、openVC、checkpoint、rebalance、localExit、materialiseBatch、prepareShutdown、cooperativeSettle、abortV0。

这比纯叙述好，但仍不是完整状态机：

- 每个 transition 只有 admissibility summary，没有输入、输出、precondition、postcondition、failure condition。
- `Checkpointing -> Open` 是瞬态还是需要 Hydra snapshot confirmed？若 checkpoint tx 提交但未 snapshot，状态如何？
- `Materialising -> Open` 的条件是 pending suffix empty 还是 local exit complete？多个 local exits 排队如何处理？
- `ShutdownPreparing` 是否禁止 rebalance、checkpoint、repair、receipt insertion？没有明确。
- `Aborted` 是 analytical state，但如果进入该状态无恢复，那么它不是协议状态而是安全失败标签。
- same-number equivocation 的 freeze/repair 没有 repair transition 规范。
- scope 间关系未定义：factory state number、virtual state number、materialisation plan index、receipt state number 是否构成总序、偏序或多 scope sequence？
- 并发规则只限制同 `(factoryId, factoryEpoch, sourceFactoryStateHash)` 的一个 active materialisation plan，但不覆盖并行 openVC/rebalance/localExit 与 pending plan 的冲突。

建议作者将状态机改写为 TLA+ / small-step operational semantics / labeled transition system，并给出 safety invariants 的 machine-checkable 或至少数学可检查版本。

### 2.5 Materialisation algorithm 是核心，但仍缺强制 admission control

材料化算法包含 canonical right ordering、descriptor-derived outputs、receipt creation、`Fits_P(k)`、max prefix、pending suffix 和 batch failure semantics。方向正确，但仍有严重未解决问题：

1. **`BudgetProfile` 是估计，不是 ledger truth。** `S_witness` 是 fixed witness-size estimate。如果 witness 数量、签名格式、script redeemer、datum、reference input 变化，估计可能失效。

2. **`Fits_P(k)` 未定义为可验证纯函数。** 它依赖 transaction bytes、execution units、datum sizes、value-bundle encoding、Hydra implementation limits。若这些由 off-chain preflight 决定，不同节点可能得出不同 k。

3. **min-ADA 在附录中列为 off-chain construction enforced。** 但若 min-ADA sufficiency 不在 validator/transaction-rule 中强制，恶意构造者可能创建不可 fanout 或无效输出。论文正文说 validator sketch check min-ADA，附录又说 off-chain construction enforced，存在边界不一致。

4. **descriptor language 未形式化。** `SettlementDescriptor` 是字段集合，不是受限 DSL 或确定性解释器。若 descriptor 规则含糊，则 materialised output hash 无法稳定复现。

5. **中途关闭仍是安全失败。** batch failure semantics 承认 pending suffix 若在 Head close 前未 materialise，V0 无 independent L1 claim。这说明算法不能独立保证安全，只能在 capacity assumption 下工作。

6. **admission control 缺失。** 论文说 `Theta_mat` 由部署测量，但没有给出在 `openVC` 时如何拒绝会超过 fanout debt bound 的新权利。

7. **receipt root growth 可能成为新的 fanout/datum debt。** 论文承认 receipt bytes/root growth 需测量，但没有 pruning、epoch rollover、checkpoint compaction 或 receipt accumulator 方案。

结论：材料化是论文最有价值贡献，也是最大技术债。必须从“算法草图”升级为“deterministic, locally computable, validator-compatible materialisation function plus admission control”。

### 2.6 ADA-only conservation model 正确但不充分

ADA-only lanes `B, M, R, O, P` 的分离是正确的，尤其强调 min-ADA reserve 不是 spendable business balance，operator budget 不能从 participant balances 隐性抽取。这是 Cardano 原生设计中的优点。

但仍需补足：

- `fee_O` 在 Hydra Head 内交易费用语义如何体现？Hydra Head 内部交易与 L1 fee、operator publication cost 的边界不同。
- `P(F)` pending materialised-output lovelace 与 `Y` 输出关系需要更严格定义，避免 batch 前后重复计入。
- Head-visible factory UTxO 的 lovelace 与 compact committed lanes 的 equality invariant 未直接给出。
- 如果 materialised outputs 在 Head 内创建后又被普通 Head transaction 花费，release receipt 与 subsequent spend 的关系如何追踪？
- 如果 min-ADA 参数变化，已有 descriptor 和 reserve 是否失效？Hydra Head 生命周期内 ledger parameter view 如何绑定？

Native asset extension 降级为 future work 是正确决定。考虑 Hydra known issues，V0 不应声称 multi-asset 安全。

---

## 3. 逻辑一致性审计

### 3.1 去掉 CKB/Morph 引用后，论文基本自洽

当前论文已不再依赖 CKB/Morph 背景来证明自身动机，而是以以下逻辑自洽展开：

```text
Hydra fanout settles UTxO set
-> Many virtual channels as Head-visible UTxOs create fanout pressure
-> Compact rights reduce fast-path churn
-> Compact rights create materialisation debt
-> V0 must materialise inside Head before fanout
-> V1 claim UTxO is future work
```

这条逻辑是 Cardano/Hydra 原生的。Hydra、Interhead、Hydrozoa 都被用作边界参照，而不是外来叙事支撑。因此，“去掉 CKB/Morph 后论文是否自洽”的答案是：**基本自洽，且比早期版本更聚焦。**

### 3.2 V1 审计指出的问题是否被修复

已明显修复或改善：

- **过度声称 V0 安全**：已降级，明确 watcher/authorised actor liveness。
- **Hydra inclusion lemma 包装成 Lerna 定理**：已明确不是 Lerna security theorem。
- **non-interference 只有字段列表**：已加入 dependency graph、closure、predicate。
- **材料化过于原则化**：已加入 batch 算法和 failure semantics。
- **native asset 范围过宽**：已明确 V0-ADA，native assets future extension。
- **失败模式隐藏**：已作为 V0 specification 一部分列出。

仍未修复：

- 没有完整 adversarial execution model；
- 没有完整状态机；
- 没有 proof system 和 validator cost；
- 没有 prototype measurements；
- 没有 delegation protocol；
- 没有 formal security theorem；
- 没有 admission control 确保 materialisation capacity invariant。

### 3.3 新引入或新版更突出的逻辑张力

#### 张力一：论文定位为 design note，但形式上使用 theorem/lemma/invariant/proposition

论文诚实说 “design note” 和 “prototype obligations”，但又大量使用 `lemma`、`proposition`、`invariant` 环境。若这些语句没有形式化模型支撑，容易产生形式化程度高于实际内容的错觉。

建议：保留 invariants，但将 lemma/proposition 改为 “Design requirement / Consequence / Observation”，除非补充完整模型和证明。

#### 张力二：V0 latest-state settlement 与 Hydra snapshot inclusion 的关系仍易误解

Design thesis T3 说 “Newer valid Lerna states supersede older Lerna states at the factory layer”。但 V0 中 supersession 只有在 Head-local rule 接受、snapshot inclusion、contestability 均成立时才有 settlement 意义。

建议将 T3 改为：

> Lerna uses monotonic latest-state evidence, but V0 settlement effect arises only after authorised Head inclusion and Hydra contestability.

#### 张力三：`validator` 与 `validator-compatible transaction rule` 混用

正文说 factory 可由 validator 或 agreed Head-local transaction rules 控制；附录要求报告哪些 checks 是 validator/transaction-rule enforced。问题是：

- Plutus/Aiken validator enforcement 可在 fanout 后 L1 语义中被理解；
- Hydra application transaction rule 或 node plugin enforcement 依赖 Head participants 一致运行该规则；
- off-chain construction enforcement 只是诚实构造者假设。

三者信任边界完全不同。当前论文虽然要求报告分类，但仍在多个地方把它们合并表述。应建立三层 enforcement taxonomy。

#### 张力四：compactness 降低 fast-path UTxO churn，但可能增加 shutdown risk

论文承认 compactness postpones work。但安全上必须证明 postponed work 不会集中到 liveness 最弱时刻。当前 `Theta_mat` 是开放函数，没有 admission control，因此 compactness 的风险与收益尚未量化。

#### 张力五：Head-local virtual participants 与 “non-custodial” 语义冲突

论文说 operator 无隐含余额控制权，但非 Head user 需要 delegation 到 authorised actor。若该 actor 能决定是否提交 checkpoint/materialisation，用户实际依赖其可用性和行为。即使 operator 不能直接盗取余额，也能通过 withholding/censorship 造成 stale settlement。论文应明确这是一种 availability custody / enforcement dependency，而不仅是 operational liveness。

---

## 4. 缺陷与风险

### 4.1 威胁模型遗漏或不足

1. **Data availability / proof availability 模型不足**  
   Header 签名不保证 proof body、touched leaves、descriptor body、receipt witnesses 可用。需要明确是否要求 sign-to-availability、witness escrow、DA committee、erasure coding、或 all signers retain proofs。

2. **Delegation threat model 不完整**  
   watcher/operator delegation 是 V0 安全核心，但论文只说 open question。缺少 delegation key scope、revocation、fee funding、misbehavior evidence、service downtime、watcher equivocation。

3. **Hydra snapshot production liveness 未建模**  
   论文假设 authorised actor can ensure snapshot availability。但在 Hydra 中 snapshot confirmation 依赖协议流程和 participant cooperation。应更精确描述何种 participant set corruption 下 snapshot 可生成。

4. **Censorship inside open Head 未给出缓解**  
   若 Head participants 拒绝包含 Lerna transaction，V0 fails。论文承认失败，但没有提供 admission-time disclosure 或 user-facing security label。

5. **Economic griefing 未建模**  
   攻击者可打开许多小 rights、制造 receipt bloat、触发 shutdown preparation、制造 same-number equivocation freeze、拒绝提供 proof body，使系统进入 full authorisation 或 liveness failure。

6. **Network timing assumptions 不明确**  
   Contestability 假设 watcher 在 deadline 前观察、构造、获得 snapshot、提交 contest。需要同步/部分同步网络模型、最大延迟、clock skew、validity interval 策略。

7. **Key compromise 和 role separation 不足**  
   signer registry 区分 Hydra party key、Cardano key、application key 是好事，但没有说明 key rotation、revocation、compromise recovery、factory_epoch rollover。

8. **Ledger parameter changes 未处理**  
   min-ADA、tx size、ex units、cost model、reference script rules 变化会影响 materialisation determinism。需要绑定 ledger parameter epoch 或 upgrade path。

9. **Privacy 基本未处理**  
   论文承认 privacy out of scope，但 access manifests、touched leaves、dependency graph 可能泄露 channel topology 和 liquidity dependencies。若目标包括 wallets/payment apps，这是实际风险。

10. **Interhead/Hydrozoa composition timing 风险**  
    论文提到 timeout alignment，但没有给出组合规则。Inner materialisation deadline、Interhead conversion deadline、Hydra contest deadline、Hydrozoa rollout deadline 的嵌套关系必须形式化。

### 4.2 边界情况

1. **same-number equivocation freeze 可被滥用**  
   攻击者签两个同编号不同 hash 的 header，使 scope freeze。若 repair 需要其签名，则可无限 grief。

2. **high-number invalid transition**  
   如果高编号 header 签名有效但 transition body 无效，是否阻塞低编号有效状态？论文说 valid state supersedes，但 proof withholding 场景下验证者可能无法判断 invalid vs unavailable。

3. **descriptor upgrade race**  
   `settlement_descriptor_hash` 在 header 和 state 中出现。若 descriptor 更新与 local exit 并发，哪个 descriptor 决定 output？需要 versioned transition rules。

4. **receipt insertion without output finality**  
   release receipt 与 materialised output 必须同一 confirmed Head transaction 原子发生。若在 off-chain plan 中先生成 receipt，后续输出失败，可能造成 right 被错误标记 released。

5. **materialised output subsequent spend**  
   right materialised 后成为普通 Head UTxO。如果它在 Head 内又被花费，fanout 时不再存在该 output。receipt 的 “replacement_output_hash” 与最终 ownership 如何关联？需要说明 Lerna right discharged 后由普通 Head ledger semantics 接管。

6. **pending suffix 的 authorization**  
   前缀 materialised 后，suffix 仍 compact。若后续 factory update 改变 reserve/descriptor/membership，旧 suffix plan 是否仍有效？

7. **batch index replay across source states**  
   concurrency rule 绑定 sourceFactoryStateHash 是对的，但 receipt id 是否足以防止同一 right 在 repair state 中重新出现？需要 active right id uniqueness invariant。

8. **zero-value / dust / min-ADA edge cases**  
   论文提到 agreed zero-value releases，但未定义零余额 right 是否需要 receipt、是否可 materialise、如何避免 min-ADA reserve 浪费。

9. **operator budget exhaustion**  
   如果 operator budget 不足以完成 materialisation，但 business balances 充足，是否可强制从 shared reserve 或 participants 按比例扣费？需要 signer policy。

10. **partial fanout interaction**  
    Hydra partial fanout 可减少单 tx 压力，但 Lerna 必须确保 materialised outputs individually fanout-safe。当前讨论正确但仍缺具体 rules。

---

## 5. 与引用工作的比较

### 5.1 Hydra Head

论文对 Hydra 的基本描述准确：Hydra Head 是 Cardano isomorphic state channel，使用 snapshots、close、contest、fanout。论文正确引用 Hydra known limitations：UTxO 数量、transaction size、execution budget、unburned tokens、partial fanout 局限。

不足之处：

- 没有足够精确地区分 Hydra Head 内 ledger validation、snapshot signing、L1 fanout validation 三个层次。
- 对 “authorised actor obtains a Hydra snapshot” 的可行性描述过抽象。
- 没有说明 Hydra 当前实现中 Head participants 如何对 application-specific validator-compatible rules 达成一致。

评价：**方向准确，但协议细节比较不足。**

### 5.2 Interhead Hydra

论文现在对 Interhead 的定位较准确：Interhead virtualises a Hydra Head across Heads，Lerna virtualises rights/channels inside a Head-like ledger。论文没有错误声称继承 Interhead 安全证明，也指出 timeout alignment 是组合风险。

不足之处：

- 对 Interhead conversion phases、collateral assumptions、overlapping participants 的安全条件没有深入比较。
- 如果 Lerna 放在 Interhead virtual Head 内，inner materialisation 与 Interhead conversion 的具体触发顺序未定义。

评价：**边界叙述准确，但比较停留在架构层。**

### 5.3 Hydrozoa

论文对 Hydrozoa 的使用明显更谨慎。它将 Hydrozoa 作为 execution/custody separation 和 deterministic rollout 的设计参照，而非竞争对象或安全依赖。并且承认 Hydrozoa pre-alpha / spec in flux。这是合理的。

不足之处：

- Hydrozoa rule-based fallback 是 outer custody fallback，而 Lerna V0 无此 fallback。论文已说明，但应更突出，避免读者以为 Lerna 借用了 Hydrozoa 的 L1 enforceability。
- “Hydrozoa-style ledger” 作为 Lerna container 的接口条件尚未形式化。

评价：**引用谨慎，类比合理，但形式化映射不足。**

### 5.4 eltoo

论文对 eltoo 的处理正确：只借用 non-punitive latest-state supersession 直觉，不声称采用 Bitcoin sighash / ANYPREVOUT / replacement transaction 模型。 “Why Not Eltoo for Cardano” 是必要且有效的章节。

不足之处：

- eltoo 的核心是底层链能执行 update/settlement transaction replacement 语义。Lerna V0 并没有等价 L1 replacement path，只是 Head-local latest state 加 Hydra contest。因此 “latest-state settlement” 一词应更谨慎。

评价：**避免了常见误用，但术语仍可降级。**

### 5.5 Perun / virtual channels / channel factories

论文承认 Perun 和 channel factories 是 intellectual lineage，且 V1 claim-UTxO 才可能接近 heavyweight adjudication。这是正确的。

不足之处较明显：

- 与 Perun 的 adjudicator model、virtual channel opening/closing protocol、dispute phases、intermediary collateral 没有技术对照。
- 与 channel factories 的 reallocation、unanimous update、unilateral exit、challenge period、mass exit 问题没有系统比较。
- 没有明确说明 Lerna V0 放弃了哪些 channel factory 安全属性，例如 non-Head user unilateral enforceability。

评价：**引用不错误，但比较不充分。** 若投稿顶会，related work 必须更深入。

---

## 6. 写作质量评估

### 6.1 优点

1. **定位诚实**  
   论文多次强调不替代 Hydra、不竞争 Hydrozoa、不是 eltoo for Cardano、V1 不可部署。这显著提高可信度。

2. **Cardano/Hydra 原生意识强**  
   min-ADA、native assets、datum、reference scripts、Hydra fanout constraints、known issues 都被纳入讨论，不是泛化 state channel 叙事。

3. **风险披露较好**  
   failure scenarios 和 limitations/open questions 写得比许多白皮书诚实。

4. **结构清晰**  
   从 positioning、system model、threat assumptions、formal objects、state machine、lifecycle、security properties 到 evaluation agenda，整体可读。

5. **V0/V1 边界明确**  
   多次声明 V0 是 ADA-only Head-local，V1 claim UTxO 是 future work。

### 6.2 缺点

1. **形式化表象强于形式化实质**  
   使用 definition、lemma、proposition、invariant，但缺少完整模型、定理语句和证明。

2. **重复较多**  
   compactness is debt、V0 no independent L1 exit、V1 future work、Hydra fanout UTxO constraints 被多次重复。可以压缩以留空间给协议细节。

3. **术语有时过强**  
   “latest-state settlement”、“security properties”、“factory-local semantics” 容易让读者以为已有 enforceable theorem。

4. **规范与路线图混杂**  
   有些内容是 V0 specification，有些是 prototype recommendation，有些是 open question。建议用 RFC 风格明确 MUST/SHOULD/MAY 和 “not in V0”。

5. **缺少图表化状态机和消息流程**  
   对协议论文来说，应有 sequence diagrams：openVC、update、checkpoint、localExit、materialiseBatch、stale close contest。

6. **缺少安全性质的精确定义**  
   “balance conservation”、“non-interference”、“safe materialisation”、“close/fanout safety” 都是目标性质，但没有 game-based 或 trace-based definitions。

---

## 7. 可改进之处，按优先级排列

### P0：必须修复，否则不应声称为安全协议

1. **把 V0 安全目标重写为 trace-based conditional theorem**  
   明确 adversarial model、network model、authorised actor condition、snapshot inclusion condition、materialisation capacity condition。定理应写成：在这些条件下，已 checkpoint/materialised 的 latest valid right 不会被 stale snapshot 覆盖。不要声称更强。

2. **建立完整 V0 state machine**  
   为每个 transition 给出 input、precondition、postcondition、state updates、failure behavior。特别补齐 repair transition、shutdown transition、materialisation pending suffix、same-number equivocation freeze。

3. **定义 enforcement taxonomy**  
   区分：
   - Plutus/Aiken validator enforced；
   - Hydra Head ledger rule enforced；
   - Hydra node/application transaction rule enforced；
   - off-chain construction enforced；
   - test-only。  
   每个安全性质必须标注其最终依赖哪一层。

4. **将 materialisation capacity 从假设变为 admission control**  
   在 `openVC`、`rebalance`、`localExit` 中维护 fanout debt bound。给出保守公式或 prototype-measured bound。不能只说 deployment measures `f`。

5. **给出 proof availability 机制**  
   签名 header 时必须绑定 proof material availability。至少需要协议规则：签名者只签署包含 proof body hash、witness retrieval commitment、availability receipt 或 full witness package 的 state。

6. **默认禁止 reduced-signature V0，除非 proof system 完成**  
   当前 non-interference 仍不能部署。论文应明确：V0 safe profile = full factory authorisation；reduced-signature profile = experimental extension requiring benchmarked `ValidNonInterference`.

### P1：大修应完成

7. **形式化 settlement descriptor**  
   定义受限 DSL 或 deterministic output derivation function：
   ```text
   deriveOutput(D, right, ledgerParams, participantRegistry) -> Output | Reject
   ```
   并说明编码、hash、versioning、upgrade rules。

8. **补充 delegation protocol 草案**  
   对非 Head virtual participant，定义 watcher delegation key、scope、revocation、fee funding、submission path、failure disclosure。

9. **给出 Hydra snapshot inclusion 细节**  
   说明 Head-local transaction 被接受、snapshot confirmed、stale close contested 的具体流程和参与方权限。避免 “can ensure snapshot” 这种过强黑箱假设。

10. **补充 receipt lifecycle**  
    定义 receipt accumulator、pruning/compaction、epoch rollover、防 bloat 策略，以及 materialised UTxO subsequent spend 后 receipt 的语义。

11. **修正 min-ADA enforcement 不一致**  
    正文 validator sketch 说检查 min-ADA，附录说 off-chain construction enforced。必须统一。若链上/Head-local rule 不检查，安全声明需降级。

12. **补充 same-number equivocation repair 规则**  
    定义 repair state 的 required signers、timeout、fallback、griefing penalty 或 full shutdown path。

13. **加强与 Perun/channel factories 比较**  
    用表格比较：participants、funding lock、virtual channel opening、update authorization、dispute path、unilateral exit、intermediary role、timeout、data availability、settlement object。

14. **给出 prototype measurement plan 的目标阈值**  
    不只列 measurements，还要说明最低可接受标准，例如 max rights per batch、receipt bytes growth、watcher response margin。

### P2：增强论文质量

15. **压缩重复叙述**  
    将 compactness debt、V0 boundary、V1 future work 的重复段落合并，释放篇幅给形式化。

16. **改名或副标题降级**  
    建议副标题改为：
    > A Design Discipline for Head-local Virtual-Channel Factories in Cardano Hydra

17. **将 theorem-like 语言降级或补证**  
    没有完整证明的 proposition/lemma 改为 observation/design requirement。

18. **增加消息流程图**  
    至少给出：
    - off-chain VC update -> checkpoint；
    - local exit -> materialise -> receipt；
    - stale close -> watcher contest；
    - shutdown preparation -> batch materialisation。

19. **明确 user-facing security labels**  
    区分：
    - Head participant user：can contest if online；
    - delegated non-Head user：assisted safety；
    - operator-only submission path：availability-dependent；
    - V1 future：not available。

20. **补充 privacy 和 griefing 分析**  
    即使 out of scope，也应列出 access manifest leakage、dependency graph leakage、receipt bloat griefing、freeze griefing。

---

## 8. 作为 Cardano/Hydra 原生论文的定位是否站得住

结论：**基本站得住，但必须降级为设计说明 / 原型纪律，而非完整安全协议。**

站得住的原因：

- 核心问题确实来自 Hydra fanout 与 Cardano eUTxO 语义；
- ADA-only V0 限制是合理且必要的；
- inner materialisation before outer fanout 是 Cardano/Hydra 原生约束；
- 与 Interhead/Hydrozoa 的层次边界比早期版本清楚；
- 不再依赖 CKB/Morph 类比，叙事自洽。

尚不站得住的地方：

- 若称为 “virtual channel factory protocol”，读者会期待单边 exit / adjudication / formal safety，而 V0 没有；
- 若称为 “latest-state settlement”，读者会期待 Lerna-level enforceability，而 V0 只有 Hydra inclusion after authorised checkpoint；
- 若称为 “non-interference proof”，读者会期待 proof system 和 soundness theorem，而当前只有设计条件；
- 若称为 “deterministic materialisation algorithm”，读者会期待 executable deterministic function 和 measured bounds，而当前仍依赖 deployment-specific estimates。

最准确定位应为：

> Lerna is a Hydra-native design discipline and prototype agenda for compact Head-local factory rights, with explicit materialisation debt and release receipts. Its V0 security is conditional on authorised Head inclusion, proof availability, and measured materialisation capacity. It does not yet provide a general self-enforcing virtual channel factory for non-Head users.

---

## 9. 最终审稿意见

**建议：弱拒 / 大修后再审。**

若目标是 workshop、design note、Cardano engineering RFC，当前版本已有讨论价值。若目标是顶级密码学或系统安全会议，必须完成上述 P0/P1 项。

一句话评价：

> 论文最强之处是诚实地识别了 Hydra 内 compact virtual rights 的 materialisation debt；最弱之处是尚未把这个 debt 变成可验证、可测量、可强制的协议机制。
