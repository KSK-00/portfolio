# Trading Automation Platform

> Production · Live System

---

# Why This Project Exists

このプロジェクトは、
ポートフォリオのために作ったものではありません。

最初は、
自分自身が実際に運用するためのシステムとして開発を始めました。

投資に興味を持ったことをきっかけに、
システムトレードを学び、

ルールを設計し、

実行を自動化し、

運用を継続しながら改善を繰り返してきました。

その過程で私の興味は、
「自動売買」そのものではなく、

**長期間、安定して運用できるバックエンドシステムを設計すること**

へと変わっていきました。

このリポジトリは、
トレーディングシステムを紹介するためのものではありません。

実際に運用しながら、
設計・改善を積み重ねてきた
エンジニアリングプロセスそのものをまとめたプロジェクトです。

---

# From Personal Tool to Engineering

このシステムは、
誰かから依頼されて開発したものではありません。

実際に自分で使用し、

運用し、

問題を見つけ、

改善を繰り返してきたシステムです。

実運用では、
コードを書くことよりも、

APIの変化。

ネットワーク障害。

例外処理。

状態管理。

ログ分析。

継続的な改善。

こうした運用上の課題と向き合う時間の方が
はるかに長くなります。

その経験を通して、

「実装」

ではなく、

**運用を前提とした設計**

を最も重視するようになりました。

現在開発している

- Threads AI Automation
- LINE AI Assistant
- Business Automation Systems

も、
このプロジェクトで得た設計思想を基盤として構築しています。

---

# Design Philosophy

私が作りたかったのは、
自動売買システムではありません。

本当に作りたかったものは、

**変化する環境でも、
安心して運用し続けられるシステム**です。

イベントを受信し、

状態を管理し、

安全に処理を実行し、

結果を検証し、

継続して改善する。

この一連のライフサイクル全体を
一つの設計対象として考えています。

その考え方は、
このプロジェクトだけではなく、
現在開発しているすべてのシステムに共通しています。
---

# Engineering Challenges

実際に運用を始めると、
実装よりも運用の方が難しいことに気付きました。

システムは、
注文を送信した時点で終わるわけではありません。

外部APIは遅延することがあります。

通信は失敗することがあります。

同じイベントが重複して届くこともあります。

市場は常に変化し、

システムは停止せずに動き続ける必要があります。

重要なのは、

一度成功することではなく、

**変化の中でも安定して運用できること**でした。

---

# Architecture Overview

そのため、
本プロジェクトでは
イベント単位ではなく、

**システム全体のライフサイクル**

を設計対象としています。

```text
Market Signal
      │
      ▼
Signal Reception
      │
      ▼
Rule Evaluation
      │
      ▼
Order Execution
      │
      ▼
Execution Verification
      │
      ▼
State Management
      │
      ▼
Monitoring
      │
      ▼
Continuous Improvement
```

このアーキテクチャでは、

イベントを受信して終わりではありません。

処理結果を検証し、

状態を更新し、

運用状況を監視し、

次の改善へ反映するところまでを
一つの流れとして設計しています。

---

# Engineering Decisions

システム全体では、
長期間の運用を前提として
以下の設計方針を採用しています。

| Challenge | Design Decision | Operational Benefit |
|------------|-----------------|---------------------|
| External API latency | Asynchronous Processing | Stable communication |
| Duplicate events | Explicit State Management | Reliable lifecycle tracking |
| Complex responsibilities | Layered Architecture | Easier maintenance |
| Continuous operation | Monitoring & Logging | Operational visibility |
| Future expansion | Loose Coupling | Flexible scalability |

これらは
個別機能のための設計ではありません。

システム全体を
長期間運用するための設計判断です。

---

# Core Engineering Principles

本プロジェクトでは、
実装よりも設計を重視しています。

そのため、
開発全体を通して
以下の原則を一貫して採用しています。

- Reliability over Complexity
- Maintainability over Short-term Optimization
- Explicit State Management
- Separation of Responsibilities
- Continuous Verification
- Continuous Improvement

これらは単なるコーディングルールではありません。

設計・実装・運用までを通して
意思決定の基準となる
エンジニアリング原則です。

---

# Beyond Trading

このプロジェクトは、
トレーディングシステムとして始まりました。

しかし、
運用を続ける中で分かったことがあります。

イベントを受信する。

状態を管理する。

外部サービスと連携する。

処理を検証する。

継続して改善する。

この一連の流れは、

トレーディングだけではありません。

業務自動化。

AIシステム。

SNS運用。

通知システム。

予約管理。

イベント駆動で動作する多くのシステムに共通しています。

そのため、
このプロジェクトで構築した設計思想は、

現在開発している

- Threads AI Automation
- LINE AI Assistant
- Business Automation Systems

にも共通して適用されています。

---

# Technology Stack

| Category | Technologies |
|-----------|--------------|
| Language | Java |
| Framework | Spring Boot |
| Architecture | Layered Architecture |
| Communication | REST API / Webhook |
| Infrastructure | AWS EC2 / Linux |
| Database | PostgreSQL |
| Build Tool | Gradle |
| Version Control | Git / GitHub |
| AI & External APIs | OpenAI API / Telegram API / Exchange APIs |

技術は目的ではありません。

課題に応じて、
最適な構成を選択するための手段です。

私が重視しているのは、

新しい技術ではなく、

**長く運用できる設計**です。

---

# Key Takeaways

このプロジェクトを通して学んだことは、

高度なアルゴリズムでも、

複雑なコードでもありません。

本当に重要だったのは、

実際に運用し、

課題を発見し、

改善を積み重ねることでした。

ソフトウェアは、

完成するものではなく、

運用を通して成長していくものだと考えています。

---

# Related Projects

同じ設計思想を基盤として、
以下のシステムも開発しています。

- Threads AI Automation
- LINE AI Assistant
- Business Automation Systems

それぞれ異なる課題を扱っていますが、

共通しているのは、

**現場で長く運用できるバックエンドシステムを設計すること**です。

---

# Closing

このリポジトリは、

自動売買システムを紹介するためのものではありません。

一つの課題に向き合い、

設計し、

実装し、

運用し、

改善してきた

エンジニアリングプロセスをまとめた記録です。

これからも、

業務を理解し、

変化に対応し、

長く運用できるシステムを設計し続けます。

---

> **Software is not finished when it works.**
>
> **It becomes valuable when it continues to work.**
