# FORGE — AI支援開発を継続可能にする Engineering System

> **Never explain your project twice.**
>
> AIにコードを書かせるだけではなく、
> **AIがプロジェクトを理解し、判断を引き継ぎ、検証された状態で開発を続けられる環境そのものを設計しました。**

FORGEは、私自身が複数の実プロジェクトをAIと開発する中で感じた問題を解決するために作った、**AI支援開発用のEngineering System**です。

一般公開を目的に作ったデモではありません。

LINE業務自動化、Threads運用自動化、Trading Automationなど、実際に開発・運用している複数のプロジェクトを継続的に扱うために、自分自身の開発環境として設計・改善を続けています。

---

# なぜ作ったのか

AIは、コードを書く速度を大きく上げてくれます。

しかし、実際のプロジェクトを長期間AIと開発していると、別の問題が出てきます。

* 前回までの背景を忘れる
* なぜその設計にしたのかを知らない
* 必要のないファイルまで読み始める
* 過去に決めた方針と矛盾する変更をする
* セッションが変わるたびに説明をやり直す
* 「修正しました」と報告しても、実際の結果と一致するとは限らない

つまり問題は、

**AIがコードを書けるかどうかではありません。**

問題は、

> **AIがプロジェクトの状態を正しく引き継ぎ、
> 必要な情報を使い、
> 変更結果を検証しながら開発を続けられるか。**

ということでした。

そこで私は、AIそのものを変えるのではなく、

**AIが作業する環境側を設計する**

ことにしました。

それがFORGEです。

---

# FORGEが管理するもの

FORGEでは、AIとの開発を単発のチャットとして扱いません。

プロジェクトの継続的な開発フローとして扱います。

```text
Project
   │
   ▼
Context
   │
   ▼
Knowledge
   │
   ▼
Engineering Decisions
   │
   ▼
AI Development
   │
   ▼
Validation
   │
   ▼
Snapshot
   │
   ▼
Next Session
```

AIのセッションが変わっても、

**「また最初から説明する」ではなく、
「前回の続きから開発する」**

ことを目標にしています。

---

# 1. Context — 必要な情報だけを渡す

大きなプロジェクトをAIに扱わせるとき、

プロジェクト全体を毎回そのまま渡すのは効率的ではありません。

FORGEでは、実際のリポジトリを対象に、

**その作業に必要な初期コンテキストを絞り込んでAIへ引き渡す**

仕組みを持っています。

2026年8月14日に、FORGE自身の実リポジトリを対象として再検証しました。

### 実測結果

| 項目                         |                結果 |
| -------------------------- | ----------------: |
| Audit Scope                | **1,269 sources** |
| Current Repository Sources | **1,268 sources** |
| Focused Sources            |     **8 sources** |
| Initial Context Reduction  |        **98.91%** |

```text
1,269 audited sources
        │
        ▼
8 focused sources
```

概算トークン換算では、

```text
736,903
   ↓
8,000
```

まで初期コンテキストを集約しました。

> **重要**
>
> 98.91%という値は、今回検証した**初期プロンプトへの引き渡し範囲**に対する実測結果です。
> その後のすべてのTool Useや追加参照まで98.91%削減される、と主張するものではありません。

数字を大きく見せることより、

**何を測った数字なのかを明確にすること**

を優先しています。

---

# 2. Knowledge — 確定した知識を引き継ぐ

プロジェクトには、ソースコードだけでは表現しきれない情報があります。

たとえば、

* 現在どこまで完成しているのか
* 過去にどんな問題が起きたのか
* 何を試して失敗したのか
* どの仕様が確定しているのか
* 次に何をするべきなのか

といった情報です。

FORGEでは、こうした情報を単なる会話履歴として扱わず、

**次のAIセッションが利用できるプロジェクト知識**

として引き継ぎます。

ただし、AIが勝手に推測した内容をそのまま永続的な知識にはしません。

**人間が確認・承認した内容と、AIの一時的な推論を区別する**

ことを前提に設計しています。

---

# 3. Engineering Decisions — 「なぜそうしたか」を残す

コードだけを読めば、

**何が実装されているか**

はある程度わかります。

しかし、

**なぜその実装を選んだのか**

までは分からないことがあります。

実際の開発では、この「なぜ」が非常に重要です。

FORGEでは、

* 過去に採用した設計方針
* 意図的に採用しなかった方法
* 運用上守るべきルール
* 明示的に固定したアーキテクチャ判断

を次の開発セッションでも参照できるようにします。

これによって、

> 前回のAIが決めたことを、
> 次のAIが知らずに壊す

という問題を減らします。

---

# 4. Validation — AIの自己申告ではなく、結果で確認する

FORGEを作るきっかけの一つになったのは、

AIの報告と実際の状態が一致しないケースでした。

AIが、

> 「完了しました」

と答えたとしても、

それだけを成功判定には使いません。

実際の開発では、

* Build
* Automated Test
* 実際の生成物
* 変更結果
* Providerへの引き渡し状態

など、**外部から確認できる結果**を重視します。

---

# Verified Evidence

FORGEでは、公開可能な範囲について実リポジトリ上で再検証を行いました。

## Automated Test Suite

Fresh Clean Buildを実行し、JUnit XMLを個別集計しました。

| 項目       |      結果 |
| -------- | ------: |
| Tests    | **892** |
| Failures |   **0** |
| Errors   |   **0** |

```text
892 tests
0 failures
0 errors
```

295件のJUnit XML Reportを集計した結果です。

---

## Exact Prompt Handoff

Context Engine側で生成されたFocused Promptと、

実際にProvider実行境界へ渡されたPromptが一致していることを検証しました。

```text
Generated Focused Prompt
          │
          ▼
Provider Boundary
          │
          ▼
EXACT MATCH
```

検証結果:

```text
selectedFileCount = 8

tests    = 1
failures = 0
errors   = 0
```

これは、

> 「Context Engineが8件を選んだ」

だけではなく、

> **「実際にAI Providerへ渡される境界まで、その結果が維持されている」**

ことを確認するための検証です。

---

# One Real Development Flow

FORGEを使った開発は、概念上のデモだけではありません。

実際の作業では、次のような流れになります。

```text
1. Project Open
        │
        ▼
2. Previous State Restore
        │
        ▼
3. Context Selection
        │
        ▼
4. Knowledge Load
        │
        ▼
5. Decision History Load
        │
        ▼
6. AI Development
        │
        ▼
7. Build / Test / Validation
        │
        ▼
8. Snapshot
        │
        ▼
9. Next Session
```

次のAIセッションは、

プロジェクトを初めて見る新人としてではなく、

**前任者から引き継ぎを受けた開発者に近い状態**

から始めることを目指しています。

---

# 「作って終わり」にしないための仕組み

FORGEで重要視しているのは、

AIの生成速度そのものではありません。

### Project Continuity

セッションが変わっても、プロジェクトの状態を引き継ぐ。

### Focused Context

必要な情報を必要なタイミングで扱う。

### Engineering Knowledge

確定した知識と一時的な推論を混同しない。

### Decision History

過去の設計理由を次の変更へ引き継ぐ。

### Validation

AI自身の報告だけで完了判定をしない。

### Snapshot

その時点の状態を次の作業へ残す。

これらを組み合わせて、

**AIとの開発そのものを、継続可能なEngineering Processとして扱う**

ことがFORGEの中心です。

---

# 実際のプロジェクトで使用

FORGEは、公開用に作った単独デモではありません。

私自身が開発・運用している複数のシステムを扱うために使用しています。

## LINE Automation

日本のお客様からの問い合わせを入口として、

* AI一次対応
* 要件ヒアリング
* 状態管理
* Google Calendar予約
* Google Sheets連携
* 通知
* Failure / Fallback

までを一つの業務フローとして扱う自動化システムです。

---

## Threads Automation

日本語ユーザー向けThreadsアカウントを対象に、

* Content Planning
* AI Generation
* Validation
* Duplicate Control
* Scheduled Publishing
* Metrics
* Comment Reply

などを継続運用するシステムです。

---

## Trading Automation Platform

TradingViewからのイベントを起点に、

* Webhook Reception
* Broker API Integration
* Order Execution
* Verification
* Telegram Monitoring
* AWS Operation

までを統合した、自身で実運用しているシステムです。

---

これらはそれぞれ別のプロジェクトですが、

開発を続けるための基盤として、

**同じFORGEの考え方と開発フローを再利用しています。**

---

# Current Implementation Status

FORGEは完成を装ったプロジェクトではありません。

実装済みの部分と、これから改善する部分を分けて扱っています。

## Implemented

* プロジェクト単位のContext管理
* Project Knowledgeの継承
* Engineering Decisionの継承
* AIセッション間の継続
* Focused Context生成
* Provider BoundaryまでのHandoff検証
* Build / TestベースのValidation
* Snapshotを使った状態継承
* 複数実プロジェクトでの利用

## Current Focus

* AIが変更してよい範囲の制御強化
* Validationフローの標準化
* 人間によるKnowledge Approval体験の改善
* Context Selectionの継続監査

「完成した」と言うためにRoadmapを隠すのではなく、

**現在どこまでできていて、次に何を改善しているのか**

を明確にしています。

---

# FORGEを作って得たもの

このプロジェクトから得た最大の知見は、

AI開発に必要なのは、より強いPromptだけではない

ということでした。

AIを実際の開発へ組み込むには、

* コンテキスト
* 状態
* 知識
* 設計判断
* 変更
* 検証
* 復元

を含む**システム全体の設計**が必要になります。

これはFORGEだけに限った話ではありません。

外部APIをつなぐ業務自動化でも、

AIを使う社内システムでも、

長期間運用するバックエンドでも、

重要なのは、

> **一度動かすことではなく、
> 継続して扱える仕組みにすること。**

だと考えています。

---

# このプロジェクトでお見せしたいこと

私がFORGEでお見せしたいのは、

「AIを使えます」

ということではありません。

AI APIを呼ぶだけなら、それほど難しいことではありません。

お見せしたいのは、

> **AIを含む複雑な処理を、
> 実際に運用できるシステムとして設計できること。**

そして、

> **必要になった仕組みを、自分で設計し、実装し、検証し、
> 実際の開発環境として使い続けられること。**

です。

私はバックエンドを中心に、

* 業務自動化
* API連携
* Webhook
* AI連携
* 外部サービス統合
* 状態管理
* Validation
* Monitoring
* Cloud Operation

などを組み合わせたシステム開発を行っています。

---

# こんなご相談と相性があります

FORGEそのものを販売しているわけではありません。

このプロジェクトで使っている設計・開発経験は、たとえば次のような案件に活かせます。

* LINEを使った問い合わせ・予約自動化
* 社内業務の自動化
* 複数SaaS / API間の連携
* Webhookを起点にした自動処理
* AIを組み込んだ業務システム
* Google Calendar / Sheets / Gmailなどとの連携
* 定型業務の自動実行
* 通知・監視システム
* バックエンドAPI開発
* AWS上で継続稼働する小〜中規模システム
* 既存業務を整理したうえでの自動化設計

要件が完全に整理されていない段階でも、

**「今、人が何をしているのか」から整理して、
どこまで自動化するのが現実的か**

を一緒に考えることができます。

---

# Public / Private Boundary

このリポジトリでは、

**FORGEの価値を確認できる証拠は公開し、
その価値を生み出す内部実装は公開しない**

という方針を取っています。

## Public

* 解決している問題
* High-level workflow
* 実装範囲
* 実測結果
* Test結果
* Context Reduction結果
* Provider Boundaryの検証結果
* 実プロジェクトでの使用事例

## Private

次の内容は公開していません。

* Context Selection Algorithm
* Ranking / Scoring Logic
* Weighting
* Token Allocation Logic
* Prompt Template
* Provider Payload
* Knowledge内部Schema
* Engineering Decision内部構造
* Snapshot Format
* Validation Rule
* Internal API
* Class / Method / Package構造
* Storage Structure
* FORGE本体のSource Code
* 非公開Log
* Credential / Secret

これは技術を隠して実績を大きく見せるためではありません。

**検証可能なEvidenceと、保護すべきEngineering Assetを分離する**

ための判断です。

---

# Repository Policy

この公開リポジトリに含まれるのは、

* Portfolio Website
* 公開用Assets
* 公開可能な説明資料

のみです。

FORGE本体のソースコードや内部運用情報は含めていません。

---

# Portfolio

実際の画面、Evidence、Development Flowはこちらで確認できます。

**FORGE Portfolio**

https://portfolio-643.pages.dev/jp/FORGE/

**Main Portfolio**

https://portfolio-643.pages.dev/jp/

---

# Contact

日本向けのシステム開発・業務自動化についてご相談を受け付けています。

「こういうことを自動化できるのか分からない」

という段階でも問題ありません。

現在の作業内容や使っているサービスを整理したうえで、実現方法を検討します。

### LINE

https://line.me/R/ti/p/@885nddqv

### Email

[len007070@gmail.com](mailto:len007070@gmail.com)

### Threads

https://www.threads.com/@len_dev9

---

> **Built from real engineering practice.**
>
> **No fabricated metrics. No exposed internal logic.**
>
> **動かすだけではなく、動き続ける仕組みをつくる。**
