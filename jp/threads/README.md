# Threads Automation Platform

## Meta Threads API × OpenAI API × Java / Spring Boot

> **外部API・AI・自動化・運用設計を組み合わせた、実運用を前提としたThreads自動化プラットフォーム**

---

## Overview

Meta Threads APIとOpenAI APIを組み合わせた、Threads運用自動化システムです。

単純な「AIで文章を生成して投稿するだけ」のBotではなく、

* OAuth認証
* トークン管理
* 自動投稿
* AIによるコンテンツ生成
* 投稿内容の検証
* 重複防止
* コメント監視
* AIによる返信生成
* 自動返信
* API障害時のRetry
* Circuit Breaker
* 運用メトリクス
* Telegram通知
* ヘルスチェック

までを一つの運用システムとして設計しています。

---

## System Concept

```text
┌─────────────────────────────┐
│        Scheduled Jobs       │
│   Automatic Post / Reply    │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│      Business Workflow      │
│  Post / Reply / Validation  │
│  Safety / Metrics / State   │
└───────┬─────────┬───────────┘
        │         │
        ▼         ▼
┌────────────┐ ┌────────────┐
│ Threads API│ │ OpenAI API │
└────────────┘ └────────────┘
        │         │
        └────┬────┘
             ▼
┌─────────────────────────────┐
│      Persistent Storage      │
│   Token / Post / Reply Log  │
└─────────────────────────────┘
             │
             ▼
      Telegram Monitoring
```

---

# Core Features

## 1. OAuth Authentication

Threadsアカウントとの連携をOAuthベースで実装しています。

### 対応内容

* OAuth Authorization
* State検証
* Authorization Code処理
* Short-Lived Tokenからの交換
* Long-Lived Token管理
* Token Refresh
* 有効期限管理
* 再認証状態の検知

認証情報はアプリケーションの一時的なメモリだけに依存せず、
再起動後も継続して利用できるように管理しています。

---

## 2. Scheduled Automatic Posting

定期的な自動投稿に対応しています。

```text
Schedule
   ↓
Natural Timing Control
   ↓
Content Generation
   ↓
Content Validation
   ↓
Duplicate Check
   ↓
Threads Publishing
   ↓
Result Recording
```

投稿処理では、単純にAIの生成結果をそのまま投稿しません。

投稿前にシステム側で内容を確認し、
問題がある場合は再生成または投稿を停止します。

---

## 3. AI Content Generation

OpenAI APIを利用して、Threads向けの投稿コンテンツを生成します。

AIへの入力には、投稿の目的や過去の投稿状況などのコンテキストを利用します。

また、AIの出力をそのまま信頼するのではなく、
システム側で投稿前の検証を行う構成にしています。

### Design Principle

```text
AI
 ↓
Candidate Generation
 ↓
Application-side Validation
 ↓
Publish / Reject
```

AIは「投稿内容を生成する役割」を担当し、
最終的な投稿判断はアプリケーション側で行います。

---

## 4. Duplicate Prevention

自動投稿システムでは、
AIが似た内容を繰り返し生成する問題があります。

そのため、過去の投稿履歴を参照し、
内容の重複や類似度を確認する仕組みを実装しています。

```text
New Candidate
      ↓
Normalize
      ↓
Compare with Recent Posts
      ↓
Similar?
  ├── Yes → Regenerate / Reject
  └── No  → Continue
```

これにより、同じような投稿を繰り返すリスクを低減しています。

---

# Comment Automation

## 5. Automatic Comment Monitoring

定期的に対象投稿のコメントを取得し、
未処理のコメントを確認します。

```text
Recent Posts
     ↓
Fetch Replies
     ↓
Filter
     ↓
Safety Check
     ↓
AI Analysis
     ↓
Generate Reply
     ↓
Validate
     ↓
Publish Reply
```

---

## 6. AI-Assisted Reply Generation

コメントに対する返信もOpenAI APIを利用して生成します。

ただし、すべてのコメントに自動返信するわけではありません。

以下のようなケースは処理対象から除外する設計としています。

* 自分自身のコメント
* 既に処理済みのコメント
* 不適切な内容
* 自動返信すべきでない内容
* システム側の安全条件に該当する内容

AIによる判断だけに依存せず、
アプリケーション側のフィルタリングと制御を組み合わせています。

---

# Reliability & Safety

## 7. HTTP Retry and Timeout

外部APIを利用するシステムでは、
一時的なネットワーク障害やAPIの一時エラーが発生します。

そのため、共通のHTTP通信レイヤーに以下を実装しています。

* Connection Timeout
* Read Timeout
* Retry
* Backoff
* HTTP Error Handling
* Error Response Parsing

Threads APIとAI APIの双方で、
共通の通信制御を利用できる構成にしています。

---

## 8. Circuit Breaker

連続して外部APIへの処理が失敗した場合、
自動処理を継続し続けないようにCircuit Breakerを実装しています。

```text
Normal
  ↓
Failure
  ↓
Consecutive Failures
  ↓
Circuit Open
  ↓
Automatic Processing Paused
  ↓
Manual Resume
```

外部サービス側で障害が発生している場合に、
自動処理がエラーを繰り返し続けることを防ぎます。

---

## 9. Concurrent Execution Protection

投稿処理とコメント処理が重複して実行されることを防ぐため、
処理単位で実行制御を行っています。

これにより、

* 同じ投稿処理の重複実行
* 同じコメント処理の重複実行
* スケジューラと手動実行の競合

などのリスクを抑えています。

---

# Token & Storage

## 10. Persistent Token Management

OAuthで取得したトークンは、
アプリケーションの再起動後も利用できるように管理しています。

### Management

* Token
* User ID
* Saved Time
* Expiration Time
* Re-authentication Status

保存処理では、
途中で不完全な状態になるリスクを考慮した書き込みを行っています。

また、トークンの有効期限を監視し、
必要に応じて更新または再認証を促します。

---

## 11. Operation History

運用に必要な情報を記録しています。

* 投稿履歴
* 投稿ID
* 投稿内容
* コメント処理履歴
* 成功時刻
* 失敗時刻
* 最新のエラー情報

これにより、
「何が実行されたか」を後から確認できるようにしています。

---

# Monitoring

## 12. Telegram Operation Notifications

システムの重要なイベントをTelegramに通知します。

### Notification Examples

* 投稿成功
* 投稿失敗
* APIエラー
* Token更新
* 再認証要求
* Circuit Breaker発動
* 日次レポート

自動化システムでは、
処理が成功したことよりも、
「失敗したときに気づけること」が重要です。

そのため、異常時の通知を運用設計の一部として組み込んでいます。

---

## 13. Daily Metrics

日次で運用状況を確認できるようにしています。

### Metrics

* 投稿成功数
* 投稿失敗数
* 自動返信成功数
* Retry回数
* OpenAI APIの平均応答時間
* Threads APIの平均応答時間
* Token更新状況
* 最新エラー

システムを動かすだけでなく、
運用状態を確認できることを重視しています。

---

# Health Check

## 14. Operational Health Check

現在のシステム状態を確認できるヘルスチェックを用意しています。

確認対象には以下が含まれます。

* Threads連携状態
* Token状態
* Token有効期限
* Circuit Breaker状態
* 最終成功時刻
* 最終失敗時刻
* 最新の投稿状態
* 運用ログの存在

異常が発生した場合に、
単純な「アプリケーションが起動しているか」だけでなく、
外部連携を含めた運用状態を確認できるようにしています。

---

# Architecture

## 15. Responsibility Separation

システムは責任ごとに分離しています。

```text
Module
  │
  ▼
Business Service
  │
  ├── External API
  ├── AI Integration
  ├── Storage
  ├── Configuration
  └── Metrics
```

### Responsibilities

#### Application Entry Point

* Scheduler
* REST Control
* OAuth Flow
* Health Check

#### Business Layer

* Posting Workflow
* Reply Workflow
* Validation
* Duplicate Prevention
* Circuit Breaker
* Token Refresh Decision
* Metrics Aggregation

#### API Layer

* Threads API Communication
* OAuth Token Exchange
* Publishing
* Reply Retrieval

#### AI Layer

* Prompt Management
* Content Generation
* Reply Generation

#### Storage Layer

* Token Persistence
* Post History
* Reply History

#### Configuration Layer

* Runtime Configuration
* Timeouts
* Retry Settings
* Validation Rules
* Safety Limits

各責任を分離することで、
一つの機能変更がシステム全体に与える影響を小さくしています。

---

# Engineering Principles

このプロジェクトでは、単純な機能実装だけでなく、
実際の運用を想定した以下の点を重視しています。

### External API Failure

外部APIは常に正常に動作するとは限りません。

そのため、

* Timeout
* Retry
* Backoff
* Circuit Breaker
* Error Notification

を組み合わせています。

---

### AI Output Is Not Automatically Trusted

AIが生成した内容を、
そのまま外部サービスに送信する構成にはしていません。

```text
AI Output
    ↓
Application Validation
    ↓
Business Decision
    ↓
External API
```

AIとアプリケーションの責任を分離することで、
予期しない生成結果がそのまま外部に公開されるリスクを抑えています。

---

### Operations Are Part of the Product

自動化システムは、
「動く」だけでは不十分です。

重要なのは、

* 失敗したときに分かる
* 復旧できる
* 何が起きたか確認できる
* 再起動後も必要な状態を維持できる
* 外部サービスの障害に耐えられる

ことです。

このプロジェクトでは、
実際に継続運用することを前提に設計しています。

---

# Technology Stack

```text
Backend
  Java
  Spring Boot
  Gradle

External APIs
  Meta Threads API
  OpenAI API
  Telegram Bot API

Infrastructure
  AWS EC2
  Linux
  Nginx

Engineering
  OAuth 2.0
  REST API
  Webhook / API Integration
  Retry / Backoff
  Circuit Breaker
  Structured Logging
  Metrics
  Automated Testing
```

---

# Project Status

現在も実際の運用を想定しながら継続的に改善しています。

主な改善方針:

* API仕様変更への対応
* AI生成品質の改善
* エラー処理の改善
* 運用メトリクスの改善
* 保守性の向上
* テストカバレッジの向上

---

# What This Project Demonstrates

このプロジェクトで重視しているのは、
単にAI APIを呼び出すことではありません。

```text
External API
      +
AI
      +
Backend
      +
Reliability
      +
Monitoring
      +
Operations
```

これらを組み合わせて、
実際に継続運用できる自動化システムとして設計することです。

外部APIとAIを利用した自動化システム、
既存サービスとのAPI連携、
運用を前提としたバックエンド開発などについてご相談いただけます。

---

## Note

このプロジェクトでは、
認証情報、APIキー、内部プロンプト、
個別の運用設定および一部の実装詳細は公開していません。
