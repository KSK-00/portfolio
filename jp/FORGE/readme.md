# FORGE — AI支援開発を継続可能にする Engineering System
> **Never explain your project twice.**
>
> 必要な文脈だけをAIへ渡し、セッションを越えて知識と判断を引き継ぎ、
> 実行結果を検証しながら、次の開発へつなげる。
FORGEは、AIにコードを書かせるためだけのツールではありません。
私自身が複数の実プロジェクトをAIと継続開発する中で感じた、

- セッションごとの文脈消失
- 不要なコード参照
- 過去の設計判断との矛盾
- AIの自己申告と実際の結果の不一致
- 長期開発で蓄積される知識の扱い

といった問題を、**AIが作業する環境側から解決するために作ったEngineering System**です。
LINE業務自動化、Threads運用自動化、Trading Platformなど、私自身が実際に開発・運用している複数のプロジェクトで使用しています。
販売用に作ったデモではありません。
**自分自身の開発環境として使い、実際の運用で見つかった問題をもとに改善を続けています。**

---

# 1. Why FORGE Exists
AIは、コードを書く速度を大きく上げてくれます。
しかし、実際のプロジェクトを長期間AIと開発していると、問題はコード生成能力だけではないことが分かりました。
たとえば、

- 前回までの背景を知らない
- なぜその設計にしたのか分からない
- すでに却下した方法を再び提案する
- 今回の作業に不要なファイルまで読み始める
- セッションが変わるたびに説明をやり直す
- 「完了しました」という報告と実際の状態が一致しない

という問題です。
FORGEを作るきっかけの一つも、AIが作業完了を報告したにもかかわらず、期待した変更が実際には反映されていなかった経験でした。
そこから、FORGEでは次の原則を置いています。

> **AIの自己申告を、成功の証拠にはしない。**
重要なのは、AIが「できた」と言ったかどうかではありません。
**何が変更され、何が実行され、どの結果が確認されたか。**
そのためFORGEでは、AIとの開発を単発のPrompt / Responseではなく、継続するEngineering Processとして扱います。

```text
Context
   ↓
Knowledge / Decisions
   ↓
AI Development
   ↓
Verification
   ↓
Handover
   ↓
Next Turn / Next Session
```

目標は単純です。

> **AIが毎回プロジェクトを初めて見る状態をやめる。**
前回までに何を作り、何を決め、何が失敗し、どこまで完成し、次に何をするべきなのか。
その状態を引き継いだうえで、必要な情報だけを使い、結果を確認しながら開発を続けられる環境を作る。
それがFORGEの基本です。

---

# 2. Context & Handover

## 必要な情報だけを渡す
大きなリポジトリをAIに扱わせるとき、プロジェクト全体を毎回そのまま渡す必要はありません。
関係のないコードまで大量に渡すと、

- トークン消費が増える
- 重要な情報が埋もれる
- 作業範囲が不必要に広がる
- 誤った変更対象を選ぶ可能性が増える

という問題があります。
FORGEでは、
**今の作業に必要なソースとコード領域を選び、必要な粒度でAIへ渡す**
という考え方を採用しています。

```text
Large Repository
       ↓
Relevant Sources
       ↓
Relevant Code Regions
       ↓
Compact Context
       ↓
AI
```

すべてのコードを同じ粒度で渡すわけではありません。
必要に応じて、全文、関連部分、署名、構造、除外といった粒度で扱います。
具体的なランキング方式、スコア、重み、選択アルゴリズムは公開していません。

## Historical Context Audit — 2026.08.14
2026年8月14日、FORGE自身の実リポジトリを対象に、当時のProduction Context経路で監査を行いました。

| 項目 | 結果 |
|---|---:|
| Audit Scope | **1,269 sources** |
| Focused Sources | **8 sources** |
| Initial Context Reduction | **98.91%** |
| Estimated Tokens | **736,903 → 8,000** |

```text
1,269 audited sources
        ↓
8 focused sources
```

この**98.91%**は、2026.08.14時点の特定条件下で行った
**初期コンテキスト監査の歴史的な実測値**です。
すべてのプロジェクト、すべてのタスク、すべてのターンで同じ削減率になるという意味ではありません。
その後のTool Useや追加参照まで98.91%削減されると主張するものでもありません。
数字を大きく見せることより、
**何を、どの条件で測った数字なのかを明確にすること**
を優先しています。

## セッションが変わっても、続きから始める
FORGEを作り始めた本来の目的は、AIセッション間の引き継ぎを安定させることでした。
プロジェクトには、コードだけでは分からない情報があります。

- 現在どこまで完成しているか
- 何が未完了なのか
- どんな問題が起きたか
- 何を試して失敗したか
- どの仕様が確定しているか
- なぜその設計を選んだか
- 次に何をするべきか

これらを毎回人間が説明し直すのではなく、
**次のAIが利用できるプロジェクト状態として引き継ぐ**
ことがFORGEの役割です。

```text
AI Development
      ↓
Meaningful Change
      ↓
Review
      ↓
Handover / Status / History
      ↓
Next Session
```

意味のある変更が無ければ、記録を増やすためだけに更新することは目的にしていません。
次のセッションは、

> **プロジェクトを初めて見る新人ではなく、
> 前任者から引き継ぎを受けた開発者に近い状態**
から始めることを目指しています。

## Knowledge & Engineering Decisions
FORGEでは、現在の状態だけでなく、
**なぜその判断をしたのか**も引き継ぎます。
コードを見れば、「何が実装されているか」は分かります。
しかし、

- なぜこの方式を採用したのか
- なぜ別案を採用しなかったのか
- 何を意図的に固定したのか
- 今後も守るべき制約は何か

までは、コードだけでは分からないことがあります。
こうした情報を次のAIセッションでも参照できるようにすることで、

> 前回のセッションで意図的に決めたことを、
> 次のAIが知らずに壊す
という問題を減らします。

## Traceable Knowledge
長期開発では、プロジェクトの事実も変化します。
一度解決した問題が再発したり、過去の判断が新しいEvidenceによって見直されたりすることがあります。
FORGEでは、単純に古い情報を消して最新情報だけを残すのではなく、
**出所・時点・状態を意識しながら変化を追跡できるKnowledge**
を重視しています。
現在、時間的な変化をより正確に扱うMemory経路は
**SHADOW**で検証中です。
最新の検証時点では、
**5 / 15 genuine diverse turns**
まで実運用Evidenceが蓄積されています。
まだ既定のMemory経路ではありません。
新しい仕組みを作ったからすぐ置き換えるのではなく、十分なEvidenceを確認してから次の判断を行います。

## Knowledge Evolution と Code Self-Improvement
FORGEでは、この二つを分けて扱います。
**Knowledge Evolution**は、
プロジェクトの記憶・状態・判断を整理し、次のセッションへ正しく引き継ぐためのものです。
**Code Self-Improvement**は、
実際の運用Evidenceから改善候補を見つけ、独立した検証へ渡すためのものです。
つまり、

> **正しく覚え続けること**
と、

> **根拠をもとに改善すること**
は別の役割です。

---

# 3. One Real Turn & Evidence
FORGEでは、AIとの1回のやり取りを単なるチャットとして扱いません。
ひとつの論理的なターンを、開発フロー全体を通して追跡します。

```text
Project State
   ↓
Context Selection
   ↓
AI Development
   ↓
Verification
   ↓
Handover Update
   ↓
Turn Evidence
```

実運用では、
**Real Turn → Real File → Real Symbol**
まで結び付いた記録が生成されることを確認しています。
実際のファイルパス、シンボル名、内部保存形式は公開していません。
また、あるファイルがそのターンの文脈に含まれていたからといって、それだけで問題の原因だったとは判断しません。

> **Context Participation ≠ Root Cause**
何が使われたかを追跡することと、何が原因だったかを断定することは別です。

## Exact Prompt Handoff
Context側で生成されたFocused Promptが、実際のAI Provider実行境界まで維持されていることも検証しました。

```text
Generated Focused Prompt
          ↓
Provider Boundary
          ↓
EXACT MATCH
```

2026.08.14の検証時点では、

- selectedFileCount = **8**
- tests = **1**
- failures = **0**
- errors = **0**

でした。
つまり、
**「Context側で8件を選んだ」だけではなく、
その結果が実際のAI実行境界まで届いたこと**
を確認しています。

## Current Full Regression
最新のFull Regressionでは、

| 項目 | 結果 |
|---|---:|
| Total Tests | **1,339** |
| Passed | **1,338** |
| Known Pre-Existing Failure | **1** |

既知の1件は、直近の実装変更とは無関係であることを確認している既存のBenchmark条件によるものです。
**0 failuresだったことにはしません。**
FORGEでは、数字をきれいに見せることより、実際に確認できた状態をそのまま残すことを優先しています。

## Beyond Unit Tests
検証は、単体テストだけで完結しません。
使い捨ての隔離環境で、実際のRepository操作を伴う検証も行っています。
Candidateが本体へ反映される前に、隔離された環境で動作と結果を確認します。
具体的な内部手順、保護ロジック、保存方式は公開していません。

---

# 4. Self Improvement & Independent Verification
FORGEには、実際の運用Evidenceから改善候補を見つける仕組みがあります。
ただし、

> AIが「もっと良くできそう」と考えた
という理由だけでコードを変更することはありません。
公開上の流れは次の通りです。

```text
OBSERVE
   ↓
ATTRIBUTE
   ↓
DISCOVER
   ↓
PROPOSE
   ↓
VERIFY
   ↓
ACCEPT / REJECT / DEFER
```

Evidenceが不十分であれば、
**DEFER**します。
実際にFORGE自身を対象としたDiscoveryでも、改善を進めるだけの根拠が無かった場面ではDEFERを選択しました。
何かを必ず変えることより、
**根拠が無いときに変えないこと**
を重視しています。

## Proposer ≠ Judge
Self Improvementで最も重要な原則の一つです。

> **Proposer ≠ Judge**
変更を提案する側と、その変更を採用するか判断する側を分けています。
同じ側が、
1. 変更を作る
2. 判定基準を決める
3. 自分で合格する
という構造では、独立した検証になりません。
そのためCandidateは、まず本体から隔離された状態で検証されます。
公開上の流れは、

```text
Candidate
   ↓
Isolated Verification
   ↓
Integrity / Safety
   ↓
Compile / Test
   ↓
Measured Evidence
   ↓
ACCEPT / REJECT / RETRY
   ↓
Safe Local Promotion
```

です。
内部の判定順序、Threshold、具体的な保護対象、Retry条件は公開していません。

## 「壊れていない」と「良くなった」は別
Testが通っただけでは、「改善した」とは判断しません。
性能改善なら、実際に性能が改善したEvidenceが必要です。
不具合修正なら、元の問題が再現でき、変更後に解消したことを確認する必要があります。
つまり、
**壊れていないこと**
と、
**実際に良くなったこと**
を分けて扱います。

## Architecture Guardが実際に止めた変更
この境界は、README上の設計原則だけではありません。
Self Improvement機能の実装中、生成側コードが本来越えてはいけない検証側のArchitecture Boundaryへ依存したことがありました。
その変更は、人間が後から目視で気付く前に、
**既存のArchitecture GuardによってBuild時に検出されました。**
結果として、その変更はそのまま採用されず、境界を維持する構造へ修正されました。

> **ルールは、文書だけではなく、実際にBuildで働く形にする。**
FORGEで重視している考え方の一つです。

## Safe Local Promotion
独立した検証を通過したCandidateだけが、Local側へ昇格します。
自律的な処理から、Remote Repositoryへ勝手にPushすることはありません。
**Verified Candidate → Local Promotion**
**Autonomous Remote Push → OFF**

---

# 5. Operational Status & Real Use
FORGEでは、
**実装したこと**
と、
**Productionの既定動作として使っていること**
を分けて扱っています。
現在の公開上の状態は次の通りです。

| Capability | Status |
|---|---|
| Context Diet | **PRODUCTION** |
| Living / Semantic Handover | **PRODUCTION** |
| Turn Tracking | **PRODUCTION** |
| Turn Provenance | **PRODUCTION** |
| Evidence-to-Code Attribution | **FLAGGED / limited rollout** |
| Temporal Memory | **SHADOW — 5 / 15 genuine diverse turns** |
| Independent Verification | **PRODUCTION** |
| Improvement Discovery | **PROVEN / GATED** |
| Automatic Self-Improvement Execution | **OFF / GATED** |
| Temporal Default Rollout | **PENDING EVIDENCE** |

**SHADOW**は既存動作を変えず裏側で評価中、
**FLAGGED**は限定的な範囲で評価中、
**GATED**は実装済みでも自動実行を制限している状態です。
新しい仕組みを作っただけで、すぐにProduction扱いにはしません。

## Current Limitations
Automatic Self-Improvementは常時ONではありません。
Temporal MemoryもまだDEFAULTではなく、SHADOWで実運用Evidenceを蓄積しています。
FORGEは、
**無制限なRecursive Self-Modification**を目的にしていません。
通常の改善Candidateが、自分自身を評価する検証ルールまで自由に変更できる状態にはしていません。
自律性を最大化することより、
**検証可能性と安全な境界を維持すること**
を優先しています。

## Real Projects
FORGEは公開用の単独Demoではありません。
私自身が開発・運用している複数の実プロジェクトで、開発基盤として使用しています。

### LINE Automation
日本向けの問い合わせ対応を入口として、AI一次対応、要件整理、状態管理、Google Calendar / Sheets連携、通知、Failure / Fallbackなどを一つの業務フローとして扱うシステムです。

### Threads Automation
日本語ユーザー向けThreads運用で、Content Planning、AI Generation、Validation、Duplicate Control、Scheduled Publishing、Metrics、Comment Reply、運用結果を使った改善などを扱っています。

### Trading Platform
Market Data、Strategy Evaluation、Backtesting、Broker API Integration、Validation、Monitoring、Notification、AWS Operationなどを扱うシステムです。
これらは別々のプロジェクトですが、
**Context / Memory / Decision / Verification / Handover**
という長期開発上の問題は共通しています。
FORGEは、プロジェクトごとにAI開発環境を作り直すのではなく、
**同じEngineering Processを再利用するための基盤**
として使っています。

---

# 6. What This Demonstrates / Public Boundary / Contact
FORGEで示したいのは、

> 「AIを使えます」
ということではありません。
AI APIを呼んだり、AIにコードを書かせたりするだけなら、それ自体は特別なことではありません。
示したいのは、

> **AIを含む複雑な処理を、
> 継続して扱えるEngineering Systemとして設計・実装・検証できること。**
です。
この経験は、たとえば次のような開発に活かせます。

- LINEを使った問い合わせ・予約自動化
- 社内業務の自動化
- 複数SaaS / API間の連携
- Webhookを起点にした自動処理
- AIを組み込んだ業務システム
- Google Calendar / Sheets / Gmailとの連携
- 通知・監視システム
- バックエンドAPI開発
- AWS上で継続稼働する小〜中規模システム

FORGEそのものを販売しているわけではありません。
要件が完全に整理されていない段階でも、
**「今、人が何をしているのか」から整理し、
どこまで自動化するのが現実的か**
を考えることができます。

## Public / Private Boundary
この公開Repositoryでは、

> **FORGEの価値を確認できるEvidenceは公開し、
> その価値を生み出すEngineering Assetは公開しない。**
という方針を取っています。

### Public
- 解決している問題
- High-Level Architecture
- 開発フロー
- Operational Status
- Context Audit / Regression結果
- Provider Boundaryの検証結果
- Self Improvementの基本原則
- Independent Verificationの考え方
- 実プロジェクトでの使用例

### Private
- Context Selection Algorithm
- Ranking / Scoring / Weighting
- Token Allocation Logic
- Prompt Template / Provider Payload
- Knowledge / Evidence内部Schema
- Candidate Generation Logic
- Verification / Safetyの具体的なRule
- Threshold / Retry Policy
- Storage Structure
- Class / Method / Package構造
- FORGE本体のSource Code
- 非公開Log / Credential / Secret

これは成果を大きく見せるためではありません。
**検証可能なEvidenceと、
保護すべきEngineering Assetを分離するための判断**
です。
この公開Repositoryには、Portfolio Website、公開用Asset、公開可能なREADMEのみを含めています。

## Portfolio
**FORGE Portfolio**
https://portfolio-643.pages.dev/jp/FORGE/
**Main Portfolio**
https://portfolio-643.pages.dev/jp/

## Contact
日本向けのシステム開発・業務自動化について、ご相談を受け付けています。
「何をどう自動化すればいいか、まだ整理できていない」
という段階でも問題ありません。
現在の作業内容や利用中のサービスを整理したうえで、現実的なシステム化の範囲から検討します。
**LINE**
https://line.me/R/ti/p/@885nddqv
**Email**
[len007070@gmail.com](mailto:len007070@gmail.com)
**Threads**
https://www.threads.com/@len_dev9

---
> **Never explain your project twice.**
>
> 必要な文脈を渡し、判断を引き継ぎ、結果を検証し、次の改善へつなげる。
>
> **Built from real engineering practice.**
>
> **No fabricated metrics. No exposed internal logic.**
>
> **動かすだけではなく、動き続ける仕組みをつくる。**
