# LINE AI CONSULTATION AUTOMATION

> 日本の開発相談を、
> LINE上で受け付ける。
>
> **Not a chatbot.
> A consultation system built for real conversations.**

---

## このシステムについて

これは、単にOpenAI APIをLINEに接続したチャットボットではありません。

現在、私自身の日本向けソフトウェア開発・自動化案件の問い合わせ窓口として実際に運用している、**LINE AI Consultation Automation System** です。

ユーザーから届いた自然言語の相談を、

```text
LINE Message
      ↓
Webhook
      ↓
Signature Verification
      ↓
Event Deduplication
      ↓
Asynchronous Processing
      ↓
AI Analysis
      ↓
Structured JSON Validation
      ↓
Java Business Logic
      ↓
Consultation State
      ↓
Reply / Push Delivery
```

という一連のシステムとして処理します。

重要なのは、

> AIがそれらしい文章を返すこと

ではありません。

実際の運用では、

* 同じWebhookが複数回届く
* AIの応答に時間がかかる
* AI APIがタイムアウトする
* AIが不正なJSONを返す
* AIが存在しない情報を生成する
* Reply Tokenが使用できなくなる
* LINE APIが失敗する
* ユーザーが複数回にわたって相談する
* 複数のメッセージが同時に届く
* サーバーが再起動する

といった問題が発生します。

このシステムは、**正常な1回の会話だけではなく、失敗することを前提に設計しています。**

---

# The Problem

一般的なAIチャットボットは、次のような構造です。

```text
User
  ↓
LLM
  ↓
Text Response
```

しかし、実際の開発外注相談ではこれだけでは不十分です。

ユーザーが、

> LINE Botを開発したいのですが、
> どのくらい期間が必要ですか？

と聞いたとき、

単純に「要件によります」と答えるだけでは相談は進みません。

また、

> 相談だけでも大丈夫ですか？

というユーザーに対して、いきなり予算を聞くのも自然ではありません。

実際の相談では、

```text
何を作りたいのか
      ↓
何を質問されたのか
      ↓
すでに何を回答したのか
      ↓
何の情報が不足しているのか
      ↓
次に何を確認すべきか
      ↓
どの情報を確定してはいけないか
```

を考える必要があります。

そのため、このシステムではAIを単独の意思決定者として扱っていません。

---

# AI Is Not The System

このシステムの中心的な設計思想は、

> **AIにシステムの権限を与えない**

ことです。

AIは、

```text
Natural Language
      ↓
Intent
      ↓
Structured JSON
      ↓
Draft Response
```

までを担当します。

その後、

```text
Structured JSON
      ↓
Parser
      ↓
Validation
      ↓
Java Business Logic
      ↓
State / Memo / Draft
      ↓
LINE Delivery
```

をJavaアプリケーションが担当します。

AIは、

* LINE APIを直接呼び出さない
* ユーザー状態を直接変更しない
* 外部APIを直接呼び出さない
* 送信処理を直接実行しない
* 検証を回避できない

という境界を持っています。

```text
┌──────────────────────────────┐
│            AI                │
│                              │
│  Analyze                     │
│  Classify                    │
│  Generate Draft              │
│                              │
│  ❌ No API Access            │
│  ❌ No State Mutation        │
│  ❌ No Direct Delivery       │
└──────────────┬───────────────┘
               │
               ▼
        Structured JSON
               │
               ▼
┌──────────────────────────────┐
│       Java Application       │
│                              │
│  Validate                    │
│  Decide                      │
│  Update State                │
│  Deliver                     │
└──────────────────────────────┘
```

AIが正しい回答を返すことを期待するだけではなく、

**AIが間違えた場合でも、システム側で被害を限定する構造**

を目指しています。

---

# AI Hallucination Is A System Problem

AIの「幻覚」をプロンプトだけで解決しようとはしていません。

このシステムでは、情報を複数の責任領域に分離しています。

```text
policy.md
    ↓
Consultation Rules
Truthfulness
Safety
Business Policy

examples.md
    ↓
Conversation Style
Consultation Flow
Japanese Communication

profile.md
    ↓
Developer Facts
Actual Experience
Confirmed Technologies

faq.md
    ↓
Business Knowledge
Frequently Asked Questions
```

それぞれの役割は異なります。

## policy.md

最も高い優先順位を持つルールです。

* 事実性
* 禁止される主張
* 価格の扱い
* 納期の扱い
* 対応可否の扱い
* 情報不足時の対応
* 相談フロー
* 安全ルール

を定義します。

## profile.md

実際の開発経験や確認された技術情報を管理します。

AIは、ここにない経歴を勝手に作りません。

## faq.md

事前に確認されたビジネス情報を管理します。

FAQにない価格・契約条件・保証などを推測しません。

## examples.md

事実情報ではありません。

これは、

* 日本語の自然な相談対応
* 相談の進め方
* 何を先に答えるか
* 複数の質問への対応
* 不要な質問の回避
* 顧客の不安を減らす表現

を学習させるためのfew-shot consultation examplesです。

---

# Prompt Assembly

プロンプトは単純に全ファイルを毎回連結するのではありません。

```text
┌────────────────────────────┐
│ 1. policy.md               │
│    Highest Authority       │
├────────────────────────────┤
│ 2. Relevant Examples       │
│    Only 2–3 selected       │
├────────────────────────────┤
│ 3. profile.md              │
│    Verified Facts          │
├────────────────────────────┤
│ 4. faq.md                  │
│    Business Knowledge      │
├────────────────────────────┤
│ 5. JSON Output Rules       │
├────────────────────────────┤
│ 6. Consultation Flow       │
├────────────────────────────┤
│ 7. Safety Rules            │
└────────────────────────────┘
```

`examples.md`は全量を毎回投入しません。

問い合わせ内容に対して、

```text
Explicit Tags
      ↓
Example Title
      ↓
Example Content
      ↓
Keyword Matching
```

の順で関連性を判断し、必要な2〜3件だけを選択します。

EmbeddingやVector Databaseを使用する必要がない範囲では、あえて導入しません。

```text
No Vector DB
No Embeddings
No RAG
No External Retrieval Service
```

シンプルな問題に、複雑なシステムを追加しない。

これも重要な設計判断です。

---

# Truthfulness Policy

AIが最も簡単に「それっぽい嘘」をつけるのは、

* 価格
* 納期
* 経験
* 対応可能性
* 外部API仕様

です。

そのため、対応可能性を次の3段階に分けています。

```text
CONFIRMED
    ↓
PROBABLE
    ↓
UNKNOWN
```

## CONFIRMED

実際の経験や情報源で確認できる場合。

## PROBABLE

関連する経験はあるが、正確な仕様をまだ確認していない場合。

## UNKNOWN

確認できる情報がない場合。

この場合、

```text
できます
```

とも、

```text
できません
```

とも断定しません。

必要な要件を確認したうえで判断します。

同様に、AIは、

```text
〇〇円でできます
1週間で完成します
必ず対応できます
```

といった確定的な発言をしません。

最終的な価格・納期・対応可否は、要件確認後に判断します。

---

# Real Consultation Flow

例えば、ユーザーが次のように相談します。

```text
LINEの自動化を作りたいのですが、
いくらくらいで可能でしょうか？
```

システムは、いきなり金額を作りません。

```text
Development Scope
      ↓
Required Functions
      ↓
Current Operation
      ↓
Budget
      ↓
Deadline
      ↓
Requirement Review
      ↓
Quotation / Proposal
```

また、ユーザーが期間について質問した場合、

```text
User:
LINE Botを開発したいのですが、
どのくらい期間が必要ですか？
```

価格だけを聞き返すのではなく、

```text
開発内容や連携する機能によって異なるため、
具体的な要件を確認したうえで
スケジュールをご案内しています。

現在考えている主な機能を教えていただけますでしょうか？
```

のように、ユーザーの質問にまず対応し、次の相談に進めます。

ユーザーが、

```text
相談だけでも大丈夫ですか？
```

と聞いた場合は、

```text
もちろんです。

まだ仕様が決まっていない段階でも問題ございません。
「こんなことを実現したい」という内容から
お気軽にご相談ください。
```

と答えるべきです。

**相談の目的は、質問を順番に消化することではありません。**

ユーザーが何を知りたいのかを理解し、
自然に必要な情報を集めることです。

---

# Consultation State

相談は1回のメッセージで終わりません。

```text
Message 1
「LINE Botを作りたい」

        ↓

Message 2
「予算は300万円ほどです」

        ↓

Message 3
「納期は3ヶ月です」

        ↓

Message 4
「予約機能と管理画面が必要です」
```

システムはこれらを別々の質問として扱いません。

```text
CONSULTATION_START
        ↓
BUDGET
        ↓
DEADLINE
        ↓
FEATURES
        ↓
REQUIREMENT REVIEW
        ↓
CONSULTATION COMPLETE
```

現在の会話状態や過去の回答を考慮し、

```text
すでに聞いたこと
```

を何度も聞かないことを重視しています。

---

# Production Failure Handling

このシステムは、正常系だけを想定していません。

## Duplicate Webhook

同じイベントが複数回届く可能性があります。

```text
Same Event
    ↓
Event ID
    ↓
Deduplication
    ↓
Process Once
```

## Slow AI

AIの応答が遅れる可能性があります。

```text
Webhook
    ↓
Async Dispatch
    ↓
AI Processing
```

Webhook受信とAI処理を分離します。

## AI Timeout

AIが利用できない場合でも、アプリケーション全体が停止しないようにします。

```text
AI Request
    │
    ├── Success
    │      ↓
    │   Validate
    │      ↓
    │   Process
    │
    └── Timeout / Error
           ↓
       Safe Fallback
```

## Invalid AI Output

AIが不正なJSONや想定外の値を返した場合、

```text
AI Output
    ↓
Parser
    ↓
Validation
    ↓
Business Logic
```

という境界を通過しなければ処理されません。

## Reply Token Failure

LINEのReply Tokenには制限があります。

そのため、

```text
Message
   ↓
Processing
   │
   ├── Reply Available
   │        ↓
   │      Reply
   │
   └── Reply Failed / Expired
            ↓
       Push Fallback
```

という構造を持ちます。

返信に失敗したからといって、可能な限りユーザーを無反応のままにしません。

---

# User-Level Concurrency

複数のメッセージが同時に届いた場合、

```text
Message A
Message B
Message C
```

を無秩序に並列処理すると、

```text
State A
State B
State C
```

の競合が発生する可能性があります。

そのため、ユーザー単位で処理を直列化します。

```text
User A
  ├── Message 1
  ├── Message 2
  └── Message 3

User B
  ├── Message 1
  └── Message 2
```

ユーザーAとユーザーBは並列に処理できます。

しかし、同じユーザーの会話は順序を維持します。

---

# Architecture

```text
┌────────────────────────────┐
│       LINE Webhook         │
└──────────────┬─────────────┘
               │
               ▼
┌────────────────────────────┐
│    Signature Verification  │
└──────────────┬─────────────┘
               │
               ▼
┌────────────────────────────┐
│     Event Deduplication    │
└──────────────┬─────────────┘
               │
               ▼
┌────────────────────────────┐
│     Async Event Dispatch   │
│     Queue / Backpressure   │
└──────────────┬─────────────┘
               │
               ▼
┌────────────────────────────┐
│    User-Level Serialization│
└──────────────┬─────────────┘
               │
               ▼
┌────────────────────────────┐
│       AI Analysis          │
│    OpenAI API Integration  │
└──────────────┬─────────────┘
               │
               ▼
┌────────────────────────────┐
│    Structured JSON         │
│       Validation           │
└──────────────┬─────────────┘
               │
               ▼
┌────────────────────────────┐
│     Java Business Logic    │
│                            │
│  Intent                    │
│  Consultation State        │
│  Draft / Memo              │
└──────────────┬─────────────┘
               │
               ▼
┌────────────────────────────┐
│     LINE Delivery          │
│     Reply / Push           │
└──────────────┬─────────────┘
               │
               ▼
┌────────────────────────────┐
│       Trace & Logs         │
└────────────────────────────┘
```

---

# Built For Failure

| Problem                 | Handling                      |
| ----------------------- | ----------------------------- |
| Duplicate webhook       | Event ID deduplication        |
| Invalid request         | Signature verification        |
| Slow AI                 | Asynchronous processing       |
| AI timeout              | Safe fallback                 |
| Invalid AI output       | Structured JSON validation    |
| Multiple messages       | User-level serialization      |
| Reply Token unavailable | Push fallback                 |
| LINE API failure        | Delivery fallback             |
| Repeated questions      | Conversation state / history  |
| Hallucinated facts      | Policy + knowledge boundaries |
| Unknown capability      | No unsupported claim          |
| Token / API errors      | Explicit error handling       |

---

# What I Actually Built

このプロジェクトで重要なのは、

```text
LINE APIを呼べた
```

ことではありません。

本当に難しかったのは、

```text
AIをどこまで信用し、
どこから先を信用しないか
```

を決めることでした。

AIに全部を任せれば、最初は簡単です。

```text
User
  ↓
GPT
  ↓
Reply
```

しかし、実際に運用すると、

* AIが知らない経歴を作る
* 価格を勝手に確定する
* 納期を勝手に約束する
* ユーザーの質問を無視する
* 同じ質問を繰り返す
* 事実と会話スタイルを混同する
* 不正な出力を返す
* API障害時に応答できない

といった問題が発生します。

そのため、プロンプトを長くするだけではなく、

```text
Policy
   ↓
Knowledge
   ↓
Examples
   ↓
Structured Output
   ↓
Parser
   ↓
Validation
   ↓
Business Logic
   ↓
Delivery
```

という複数の境界に分けました。

**AIに「間違えないで」とお願いするのではなく、間違えてもシステム全体が壊れないようにする。**

これがこのプロジェクトの中心的な設計思想です。

---

# Testing

このシステムでは、単に、

> AIが返事をした

だけをテストしていません。

```text
Webhook
    ↓
Signature
    ↓
Deduplication
    ↓
Async Processing
    ↓
AI
    ↓
JSON Parsing
    ↓
Intent
    ↓
Consultation State
    ↓
Business Logic
    ↓
LINE Delivery
```

の各段階を検証しています。

検証対象には、

* Webhook signature verification
* Duplicate event handling
* Event deduplication
* Async processing
* Queue pressure
* User-level serialization
* AI timeout
* AI unavailable fallback
* Invalid JSON
* Unknown intent
* Consultation state transitions
* Multi-turn conversations
* Early consultation completion
* Japanese text integrity
* Reply Token expiration
* Reply API failure
* Push fallback
* Real HTTP request paths

が含まれます。

また、実際のOpenAI API呼び出しによる検証も行っています。

テストで重要なのは、テスト数そのものではありません。

```text
AIが遅れた場合
LINEのReplyが失敗した場合
同じイベントが二度届いた場合
複数のメッセージが同時に届いた場合
AIが不正な出力を返した場合
```

に、

> システムがどう動くか

を確認することです。

---

# Production Experience

このシステムは、チュートリアルやサンプルコードとして作ったものではありません。

実際の日本向け開発外注相談の窓口として、

```text
LINE
  ↓
AWS
  ↓
Spring Boot
  ↓
OpenAI API
  ↓
LINE Messaging API
```

を実際に接続して運用しています。

運用中に見つかった問題を、

```text
発見
  ↓
原因分析
  ↓
最小限の修正
  ↓
テスト
  ↓
実際の実行検証
  ↓
再改善
```

というサイクルで改善しています。

---

# Developer Background

私は、

* Java / Spring Boot
* REST API
* Webhook
* 外部API連携
* OpenAI API
* LINE Messaging API
* Meta Threads API
* TradingView Webhook
* AWS EC2
* Linux
* Nginx
* Telegram Bot API

などを使用したシステムを実際に構築・運用しています。

特に、

```text
外部サービス
      +
自分のアプリケーション
      +
AI
      +
実際の運用
```

を接続したシステムに関心があります。

実際に、

* TradingViewからのシグナル受信
* 証券会社・取引所APIとの連携
* 自動実行
* Telegram監視
* Threads API自動化
* OpenAI APIを利用したコンテンツ生成
* LINE上でのAI開発相談受付

などのシステムを自ら構築し、運用しています。

単純なサンプルを作ることではなく、

> 実際に動かし、壊れた部分を見つけ、修正し、再び運用する

という開発を続けています。

---

# What I Can Help Build

ご相談可能な例：

* LINE Bot
* LINE業務自動化
* AIチャット・問い合わせ対応
* OpenAI API連携
* Webhookベースの自動化
* 外部API連携
* OAuth / Token管理
* 既存システムとの連携
* 業務自動化
* 管理・通知システム
* 既存システムの改善
* AIを利用した業務フロー
* APIを組み合わせたバックエンドシステム

まだ具体的な仕様が決まっていない段階でも問題ありません。

```text
「こういう作業を自動化したい」
「こういうシステムを作りたい」
「今の作業を減らしたい」
```

という段階からご相談いただけます。

---

# Contact

開発や自動化についてご相談がございましたら、お気軽にご連絡ください。

特に、

```text
LINE
AI
API連携
Webhook
業務自動化
既存システム改善
```

に関するご相談を歓迎しています。

---

> **Built for real conversations.**
>
> **Built for failure.**
>
> **Continuously improved through real operation.**
>
> **AI is powerful.**
>
> **But the system around it matters more.**
