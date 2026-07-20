# LINE AI CONSULTATION AUTOMATION

> 日本の開発相談を、
> LINE上で受け付ける。

AIが相談内容を理解し、
相談の状態を保持し、
必要な情報を確認しながら、
ユーザーへ応答する。

**Built for real conversations,
not just demo messages.**

---

## このシステムについて

これは、単なるLINEチャットボットではありません。

現在、私自身の日本向けソフトウェア開発・自動化案件の問い合わせ窓口として運用している、**AI-powered consultation automation system** です。

ユーザーから届いた開発相談を受け取り、

```text
Message
   ↓
Intent
   ↓
Consultation State
   ↓
Business Policy
   ↓
Response

という流れで処理します。

AIに自由に答えさせるのではなく、

何について相談しているのか
どの段階まで情報が揃っているのか
次に何を確認すべきか
どのような回答が適切か

をシステム全体で判断します。

What this system does

このシステムは、実際の開発外注相談をLINE上で受け付けるために使用されています。

たとえば、ユーザーが次のように問い合わせます。

LINEの自動化を作りたいのですが、
いくらくらいで可能でしょうか？

システムは相談内容を理解し、
必要な情報を順番に確認します。

Real consultation flow
User
  │
  ▼
開発内容の相談
  │
  ▼
予算の確認
  │
  ▼
納期の確認
  │
  ▼
必要な機能の確認
  │
  ▼
要件整理
  │
  ▼
見積・提案
Example

User

LINEの自動化を作りたいのですが、
いくらくらいで可能でしょうか？

AI

開発内容や機能によって費用が大きく異なるため、
一律の金額をお伝えすることは難しいです。

まずはご予算の目安を
教えていただけますでしょうか？

User

300万円ほどです。

AI

ご予算を300万円ほどと承知しました。

ご希望の納期はいつ頃でしょうか？
Budget

↓

Deadline

↓

Required Features

↓

Requirement Review

↓

Quotation

このように、単発の質問に答えるだけではなく、相談の進行状態を保持しながら、次に必要な情報を確認していくことを目的としています。

System Architecture
┌────────────────────────────┐
│       LINE Webhook          │
└──────────────┬─────────────┘
               │
               ▼
┌────────────────────────────┐
│   Signature Verification    │
└──────────────┬─────────────┘
               │
               ▼
┌────────────────────────────┐
│     Event Deduplication    │
└──────────────┬─────────────┘
               │
               ▼
┌────────────────────────────┐
│      Async Processing       │
│      Queue / Budget         │
└──────────────┬─────────────┘
               │
               ▼
┌────────────────────────────┐
│       AI Classification     │
│       & Interpretation      │
└──────────────┬─────────────┘
               │
               ▼
┌────────────────────────────┐
│     Consultation State      │
│      Management             │
└──────────────┬─────────────┘
               │
               ▼
┌────────────────────────────┐
│     Reply / Push Delivery   │
└──────────────┬─────────────┘
               │
               ▼
┌────────────────────────────┐
│       Trace & Logs          │
└────────────────────────────┘

システムの重要な部分は、AIを呼び出して回答を生成することだけではありません。

Webhookを受け取り、
正しく処理し、
必要な情報を保持し、
ユーザーに確実に応答すること。

そこまでを一つのシステムとして設計しています。

Built for failure

The important part is not only the happy path.

実際のメッセージングシステムでは、正常なケースだけを考えることはできません。

例えば、

同じWebhookが複数回届く
AIの応答に時間がかかる
AI APIがタイムアウトする
処理が混雑する
Reply Tokenが使用できなくなる
Reply APIが失敗する
サーバーが再起動する

といった状況が発生します。

このシステムでは、こうした失敗を前提に処理フローを設計しています。

Problem	Handling
Duplicate webhook	Event ID deduplication
Invalid request	Signature verification
Slow processing	Asynchronous queue
AI timeout	Fallback response
Limited reply budget	Reply budget management
Reply Token unavailable	PUSH policy
Reply API failure	PUSH fallback
Repeated events	Deduplication
Multiple messages	User-level state management
Invalid AI output	Validation before delivery
Reply / Push Policy

LINEのReply Tokenには制限があります。

そのため、システムは単純に

Message
  ↓
AI
  ↓
Reply

とするのではなく、処理状況に応じて返信方法を判断します。

                    ┌──────────────┐
                    │  Reply OK    │
                    └──────┬───────┘
                           │
                           ▼
                         Reply


Message
   │
   ▼
Processing
   │
   ├── Reply available ───────► Reply
   │
   └── Reply failed/expired ──► Push fallback

Replyが失敗した場合でも、可能な場合はPushによるフォールバックを行います。

返信に失敗したから、そのままユーザーが無反応になる。

という状態をできるだけ避けるための設計です。

Consultation State

相談は一回のメッセージで完結するとは限りません。

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

システムは、各メッセージを独立した質問として扱うのではなく、同じ相談の続きとして状態を保持します。

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

これにより、一般的なAIチャットではなく、業務フローに沿った相談受付として動作します。

Validation & Testing

このシステムは、単に「AIが返事をした」ことだけをテストしていません。

実際の運用で起こり得る異常系を含めて検証しています。

Current verification
189
AUTOMATED TESTS
62
REAL OPENAI API CALLS
100%
SIGNATURE VERIFICATION
24/7
PRODUCTION OPERATION
Test coverage includes
duplicate webhook delivery
invalid signature rejection
event deduplication
asynchronous processing
queue pressure
AI timeout
AI unavailable fallback
reply budget management
Reply Token expiration
Reply API failure
Push fallback
consultation state transitions
early consultation completion
concurrent messages
Japanese text integrity
JSON validation
real HTTP request paths
real LINE Webhook verification

189件という数字だけを重要視しているわけではありません。

重要なのは、

AIが遅れた場合でも

LINEのReplyが失敗した場合でも

同じイベントが二度届いた場合でも

相談が複数回にわたって続いた場合でも

システムがどう動くかを検証していることです。

Production Environment

現在、実際のサーバー環境で小規模なProduction Serviceとして運用しています。

LINE Messaging API
        │
        ▼
     Webhook
        │
        ▼
 Ubuntu Server
        │
        ├── Caddy
        │     └── HTTPS Reverse Proxy
        │
        ├── Spring Boot
        │
        └── systemd
              └── Service Management
Environment
Ubuntu Server
Java 21
Spring Boot
Caddy
systemd
OpenAI API
LINE Messaging API

現在は単一インスタンス構成で運用しています。

相談状態や重複イベント管理は、現在の運用規模に合わせた構成で管理しています。

将来的により大規模な運用が必要になった場合は、外部ストレージや複数インスタンス構成へ拡張できます。

Operational Flow

実際の処理は、次のように進みます。

1. LINE User sends a message
             │
             ▼
2. LINE Webhook receives the event
             │
             ▼
3. Signature is verified
             │
             ▼
4. Duplicate event is rejected
             │
             ▼
5. Webhook returns immediately
             │
             ▼
6. Event enters asynchronous processing
             │
             ▼
7. AI analyzes the consultation
             │
             ▼
8. Consultation state is updated
             │
             ▼
9. Response policy is determined
             │
             ▼
10. Reply or Push is delivered
             │
             ▼
11. Trace is recorded

Webhook受信とAI処理を分離することで、外部AIの応答速度にシステム全体が直接依存しない構成にしています。

Why this system exists

AIをAPIに接続するだけなら、システムは簡単に作れます。

しかし、実際の業務で使う場合に重要なのは、

AIを呼び出せるか

ではなく、

失敗したときに何が起きるか

です。

このプロジェクトでは、

外部API
Webhook
非同期処理
AI
状態管理
返信ポリシー
エラー処理
ログ
サーバー運用

を一つの実際に動くシステムとして組み合わせています。

What can be built on this foundation

このシステムの構造は、開発相談だけに限定されません。

同じ基盤を利用して、以下のようなLINEベースの業務自動化へ拡張できます。

開発案件の問い合わせ受付
予約受付
顧客サポート
リードのヒアリング
CRM連携
社内業務アシスタント
LINEベースのワークフロー自動化
外部API連携システム
AIを利用した業務受付システム

The current system is not a generic chatbot.

It is a reusable foundation for
business-specific LINE automation.

Tech Stack
Backend
Java 21
Spring Boot
Gradle
Integration
LINE Messaging API
OpenAI API
Webhook
REST API
JSON
Infrastructure
Ubuntu Server
Caddy
systemd
HTTPS
Project Philosophy

このシステムで最も重視しているのは、機能の多さではありません。

Receive
   ↓
Understand
   ↓
Maintain State
   ↓
Process Safely
   ↓
Respond

この一連の流れを、実際の運用環境で壊れにくくすることです。

Built for real conversations.
Built for failure.
Built to respond.
Contact

日本向けのソフトウェア開発・自動化案件を受け付けています。

LINE Bot / LINE Messaging API
AI-powered business automation
Webhook / API integration
Backend systems
External service integration
Business workflow automation

実際の要件に合わせて、
必要なシステム構成から設計・開発・運用まで対応します。

License

This repository is provided as a portfolio and technical reference.

Sensitive production credentials, internal business policies,
private operational data, and deployment secrets are not included.
