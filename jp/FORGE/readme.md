# FORGE — AI支援開発を継続可能にする Engineering System

> **Never explain your project twice.**
>
> 必要な文脈だけをAIへ渡し、  
> セッションを越えて知識と判断を引き継ぎ、  
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

販売用のデモとして作ったものではなく、**自分自身の開発環境として使いながら改善を続けています。**

---

# Why FORGE Exists

AIはコードを書く速度を大きく上げてくれます。

しかし、長期間AIと開発していると、問題はコード生成能力だけではないことが分かりました。

たとえば、

- 前回までの背景を知らない
- なぜその設計にしたのか分からない
- すでに却下した方法を再び提案する
- 今回の作業に不要なファイルまで読み始める
- セッションが変わるたびに説明をやり直す
- 「完了しました」という報告と実際の状態が一致しない

といった問題です。

FORGEを作るきっかけの一つも、AIが作業完了を報告したにもかかわらず、期待した変更が実際には反映されていなかった経験でした。

そこから、FORGEでは次の考え方を重視しています。

> **AIの自己申告を、成功の証拠にはしない。**

重要なのは、

**何が変更され、何が実行され、どの結果が確認されたか**

です。

そのためFORGEでは、AIとの開発を単発のPrompt / Responseではなく、

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

という継続的なEngineering Processとして扱います。

目標は、

> **AIが毎回プロジェクトを初めて見る状態をやめること。**

です。

---

# 1. Context & Handover

## 必要な情報だけを渡す

大きなリポジトリをAIに扱わせるとき、プロジェクト全体を毎回そのまま渡す必要はありません。

関係のないコードまで大量に渡すと、

- トークン消費が増える
- 重要な情報が埋もれる
- 作業範囲が不必要に広がる
- 誤った変更対象を選ぶ可能性が増える

という問題があります。

FORGEでは、**今の作業に必要なソースとコード領域を選び、必要な粒度でAIへ渡す**という考え方を採用しています。

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

必要に応じて、

```text
全文
関連部分
署名
構造
除外
```

のように扱う範囲を変えます。

具体的なランキング方式、スコア、重み、選択アルゴリズムは公開していません。

---

## Verified Context Audit — 2026.08.14

2026年8月14日、FORGE自身の実リポジトリを対象に、当時のProduction Context経路で監査を行いました。

| 項目 | 結果 |
|---|---:|
| Audit Scope | **1,269 sources** |
| Current Repository Sources | **1,268 sources** |
| Focused Sources | **8 sources** |
| Initial Context Reduction | **98.91%** |

```text
1,269 audited sources
        ↓
8 focused sources
```

概算トークン換算では、

```text
736,903
   ↓
8,000
```

となりました。

この**98.91%**は、2026.08.14時点の特定条件下で行った**初期コンテキスト監査の歴史的な実測値**です。

すべてのプロジェクトやターンで同じ削減率になるという意味ではなく、その後のすべてのTool Useまで98.91%削減されると主張するものでもありません。

数字そのものより、**何を測った数字なのかを明確にすること**を優先しています。

---

## セッションが変わっても、続きから始める

FORGEを作り始めた本来の目的は、AIセッション間の引き継ぎを安定させることでした。

プロジェクトにはコードだけでは分からない情報があります。

- 現在どこまで完成しているか
- 何が未完了なのか
- どんな問題が起きたか
- 何を試して失敗したか
- どの仕様が確定しているか
- なぜその設計を選んだか
- 次に何をするべきか

これらを毎回人間が説明し直すのではなく、**次のAIが利用できるプロジェクト状態として引き継ぐ**ことがFORGEの役割です。

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

> **プロジェクトを初めて見る新人ではなく、前任者から引き継ぎを受けた開発者に近い状態**

から始めることを目指しています。

---

## Knowledge & Engineering Decisions

FORGEでは、現在の状態だけでなく、**なぜその判断をしたのか**も引き継ぎます。

コードを見れば「何が実装されているか」は分かります。

しかし、

- なぜこの方式を採用したのか
- なぜ別案を採用しなかったのか
- 何を意図的に固定したのか
- 今後も守るべき制約は何か

までは、コードだけでは分からないことがあります。

こうした情報を次のAIセッションでも参照できるようにすることで、

> 前回のセッションで意図的に決めたことを、次のAIが知らずに壊す

という問題を減らします。

---

## Traceable Knowledge

長期開発では、プロジェクトの事実も変化します。

```text
OPEN
 ↓
RESOLVED
 ↓
REOPENED
```

のように、一度解決した問題が再び発生することもあります。

そのためFORGEでは、単純に古い情報を消して最新情報だけを残すのではなく、**出所・時点・状態を意識しながら変化を追跡できるKnowledge**を重視しています。

現在、より時間的な変化を扱うMemory経路は**SHADOW**で検証中です。

最新の検証時点では、

```text
5 / 15 genuine diverse turns
```

まで実運用Evidenceが蓄積されています。

まだ既定のMemory経路ではありません。

新しい仕組みを作ったからすぐ置き換えるのではなく、十分なEvidenceを確認してから次の判断を行います。

---

## Knowledge Evolution と Code Self-Improvement

FORGEでは、この二つを分けて扱います。

**Knowledge Evolution**は、プロジェクトの記憶・状態・判断を整理し、次のセッションへ正しく引き継ぐためのものです。

**Code Self-Improvement**は、実際の運用Evidenceから改善候補を見つけ、検証へ渡すためのものです。

```text
Knowledge Evolution
Project Change
      ↓
Knowledge / Handover Update
      ↓
Next Session
```

```text
Code Self-Improvement
Operational Evidence
      ↓
Improvement Candidate
      ↓
Independent Verification
```

つまり、

> **正しく覚え続けること**

と、

> **根拠をもとに改善すること**

は別の役割です。

後半では、このEvidence・Self Improvement・Independent Verificationについて説明します。

---

# 2. One Real Turn

FORGEでは、AIとの1回のやり取りを単なるチャットとして扱いません。

ひとつの論理的なターンを、

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

として追跡します。

実運用では、

```text
Real Turn
   ↓
Real File
   ↓
Real Symbol
```

まで結び付いた記録が生成されることを確認しています。

実際のファイルパス、シンボル名、内部保存形式は公開していません。

また、

```text
Context Participation
        ≠
Root Cause
```

という区別も重要にしています。

あるファイルがそのターンの文脈に含まれていたからといって、それだけで問題の原因だったとは判断しません。

**何が使われたかを追跡することと、何が原因だったかを断定することは別です。**

この区別が、後半のEvidence-to-Code AttributionやSelf Improvementにもつながっています。
# 3. Evidence — 報告ではなく、結果で確認する

FORGEでは、AIが

> 「修正しました」  
> 「テストは問題ありません」  
> 「改善しました」

と報告しただけでは、成功とは扱いません。

重要なのは、**外部から確認できるEvidenceがあるか**です。

---

## Historical Context Audit

2026年8月14日、FORGE自身の実リポジトリを対象にContext Auditを行いました。

```text
1,269 audited sources
        ↓
8 focused sources
```

概算トークン換算:

```text
736,903
   ↓
8,000
```

初期Context Reduction:

```text
98.91%
```

これは**2026.08.14時点の歴史的な実測値**です。

現在のすべてのターンで同じ削減率になるという意味ではありません。

---

## Exact Prompt Handoff

Context側で生成されたFocused Promptが、実際のAI Provider実行境界まで維持されていることも検証しました。

```text
Generated Focused Prompt
          ↓
Provider Boundary
          ↓
EXACT MATCH
```

当時の検証結果:

```text
selectedFileCount = 8
tests = 1
failures = 0
errors = 0
```

つまり、

**「Context側で8件を選んだ」だけではなく、その結果が実際のAI実行境界まで届いたこと**

を確認しています。

---

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

---

## Beyond Unit Tests

単体テストだけでなく、使い捨ての隔離環境で実Repository操作を伴う検証も行っています。

```text
Candidate
   ↓
Isolated Environment
   ↓
Real Repository Operations
   ↓
Verification
```

実際のファイルパス、内部手順、保護ロジックは公開していません。

---

# 4. Self Improvement & Independent Verification

FORGEには、実際の運用Evidenceから改善候補を見つける仕組みがあります。

ただし、

> AIが「もっと良くできそう」と考えた

という理由だけでコードを変更することはありません。

公開上の流れは、

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

です。

Evidenceが不十分であれば、**DEFER**します。

実際にFORGE自身を対象としたDiscoveryでも、改善を進めるだけの根拠が無かった場面ではDEFERを選択しました。

---

## Proposer ≠ Judge

Self Improvementで最も重要な原則の一つです。

```text
Proposer ≠ Judge
```

変更を提案する側と、その変更を採用するか判断する側を分けています。

同じ側が、

```text
変更を作る
   ↓
判定基準を決める
   ↓
自分で合格する
```

という構造では、独立した検証になりません。

そのため、Candidateはまず本体から隔離された状態で検証されます。

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

内部の判定順序、Threshold、具体的な保護対象は公開していません。

---

## 「壊れていない」と「良くなった」は別

Testが通っただけでは、

> 改善した

とは判断しません。

性能改善なら、実際に性能が改善したEvidenceが必要です。

不具合修正なら、元の問題が再現でき、変更後に解消したことを確認する必要があります。

つまり、

**壊れていないこと**

と、

**実際に改善したこと**

を分けて扱います。

---

## Architecture Guardが実際に止めた変更

この境界は、設計上の説明だけではありません。

Self Improvement機能の実装中、生成側コードが本来越えてはいけない検証側のArchitecture Boundaryへ依存したことがありました。

その変更は、人間が後から気付く前に**既存のArchitecture GuardによってBuild時に検出されました。**

結果として、その変更はそのまま採用されず、境界を維持する構造へ修正されました。

> **ルールは、文書だけではなく、実際にBuildで働く形にする。**

FORGEで重視している考え方の一つです。

---

## Safe Local Promotion

独立した検証を通過したCandidateだけがLocal側へ昇格します。

```text
Verified Candidate
        ↓
Local Promotion

Autonomous Remote Push
        ✕
```

自律的な処理が勝手にRemote RepositoryへPushすることはありません。

---

# 5. Operational Status

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
**FLAGGED**は限定的な適用範囲で評価中、  
**GATED**は実装済みでも自動実行を制限している状態です。

新しい仕組みを作っただけで、すぐにProduction扱いにはしません。

---

## Current Limitations

現在、Automatic Self-Improvementは常時ONではありません。

```text
Automatic Self-Improvement
OFF / GATED
```

Temporal MemoryもまだDEFAULTではなく、SHADOWで実運用Evidenceを蓄積しています。

また、FORGEは**無制限なRecursive Self-Modification**を目的にしていません。

通常の改善Candidateが、自分自身を評価する検証ルールまで自由に変更できる状態にはしていません。

自律性を最大化することより、

**検証可能性と安全な境界を維持すること**

を優先しています。

---

# 6. Real Projects

FORGEは公開用の単独Demoではありません。

私自身が開発・運用している複数の実プロジェクトで、開発基盤として使用しています。

## LINE Automation

日本向けの問い合わせ対応を入口として、

- AI一次対応
- 要件整理
- 状態管理
- Google Calendar連携
- Google Sheets連携
- 通知
- Failure / Fallback

などを一つの業務フローとして扱うシステムです。

## Threads Automation

日本語ユーザー向けThreads運用で、

- Content Planning
- AI Generation
- Validation
- Duplicate Control
- Scheduled Publishing
- Metrics
- Comment Reply
- 運用結果を使った改善

などを扱っています。

## Trading Platform

市場データや外部APIを扱い、

- Market Data
- Strategy Evaluation
- Backtesting
- Broker API Integration
- Validation
- Monitoring
- Notification
- AWS Operation

などを扱うシステムです。

これらは別々のプロジェクトですが、

```text
Context
Memory
Decision
Verification
Handover
```

という長期開発上の問題は共通しています。

FORGEは、プロジェクトごとにAI開発環境を作り直すのではなく、**同じEngineering Processを再利用するための基盤**として使っています。

---

# 7. What This Demonstrates

FORGEで示したいのは、

> 「AIを使えます」

ということではありません。

AI APIを呼んだり、AIにコードを書かせたりするだけなら、それ自体は特別なことではありません。

示したいのは、

> **AIを含む複雑な処理を、継続して扱えるEngineering Systemとして設計・実装・検証できること。**

です。

私が扱っているのは、たとえば、

- 業務自動化
- Backend API
- AI連携
- Webhook
- External API Integration
- Google Services連携
- 状態管理
- Validation
- Monitoring
- AWS上での継続運用

などです。

FORGEそのものを販売しているわけではありません。

この設計・開発経験は、

- LINEを使った問い合わせ・予約自動化
- 社内業務の自動化
- 複数SaaS / API間の連携
- Webhookを起点にした自動処理
- AIを組み込んだ業務システム
- 通知・監視システム
- バックエンドAPI開発
- AWS上で継続稼働する小〜中規模システム

などに活かせます。

要件が完全に整理されていない段階でも、

**「今、人が何をしているのか」から整理し、どこまで自動化するのが現実的か**

を考えることができます。

---

# 8. Public / Private Boundary

この公開Repositoryでは、

> **FORGEの価値を確認できるEvidenceは公開し、  
> その価値を生み出すEngineering Assetは公開しない。**

という方針を取っています。

## Public

- 解決している問題
- High-Level Architecture
- 開発フロー
- Operational Status
- Context Audit
- Regression結果
- Provider Boundaryの検証結果
- Self Improvementの基本原則
- Independent Verificationの考え方
- 実プロジェクトでの使用例

## Private

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

これは成果を大きく見せるためではなく、

**検証可能なEvidenceと、保護すべきEngineering Assetを分離するための判断**です。

この公開Repositoryには、Portfolio Website、公開用Asset、公開可能なREADMEのみを含めています。

---

# Portfolio

### FORGE Portfolio

https://portfolio-643.pages.dev/jp/FORGE/

### Main Portfolio

https://portfolio-643.pages.dev/jp/

---

# Contact

日本向けのシステム開発・業務自動化について、ご相談を受け付けています。

「何をどう自動化すればいいか、まだ整理できていない」

という段階でも問題ありません。

現在の作業内容や利用中のサービスを整理したうえで、現実的なシステム化の範囲から検討します。

### LINE

https://line.me/R/ti/p/@885nddqv

### Email

[len007070@gmail.com](mailto:len007070@gmail.com)

### Threads

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
