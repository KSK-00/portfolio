# RoleLens — 自分の条件で読む求人情報

繰り返す求人確認を、候補・必須条件・未確認情報の確認に整理した個人プロジェクトです。HUNTERY・Daily Job Hunterは同じ製品の既存UI・通知の表記です。

## 使用の流れ

個人条件で候補を整理 → Telegramで推薦・評価現況を確認 → コードで権限を確認 → 個人Webで根拠と記録を検討。

自動応募や履歴書送信は行いません。未確認情報を確定した事実に置き換えません。

## 実際の画面

Web撮影2026.11.04、既存Telegram通知撮影2026.11.05。企業・個人・接続情報をマスクまたは除外した公開用画像です。件数・評価値は当時の表示で、採用確率・成果ではありません。

![推薦候補](https://portfolio-643.pages.dev/kr/assets/rolelens/live-recommendations.png)
![必須条件](https://portfolio-643.pages.dev/kr/assets/rolelens/live-requirements.png)
![既存通知の評価現況](https://portfolio-643.pages.dev/kr/assets/rolelens/live-telegram-evaluation.png)

[画面を見る](https://portfolio-643.pages.dev/jp/rolelens/)。管理パネルのみサンプルデータとして区別しています。

## 変更と確認範囲

- 2026.11.03整理記録: 再掲載の重複18件を削除し、2,675→2,657件へ。既存推薦131件のIDと個人記録を保持。
- 2026.11.03運用データのコピーによる再実行試験: 追加推薦通知0件。実際のTelegram送信試験ではありません。
- 同じデータの重複処理を減らし、評価待ち処理とレポート終了を確認。2026.11.03保存状態のコピーでの試験であり、定期全検索完了の検証とは区別します。

## 担当・構成・制約

課題・条件と使用の流れを定め、AI開発ツールで制作し、使用確認・配置と運用改善を担当。Python、HTML/CSS/JavaScript、Telegram、AWS、HTTPSを使用。運用ソース・個人設定は非公開です。

外部ページの形式・制限とAPI応答に影響されます。未確認条件と最終判断は人が確認します。2026.11.03の受信競合と定期全検索の点検項目は、画面確認のみで解決済みとしません。

## 起動方法

```sh
python -m employment_hunter --help
```

[한국어 기록](../../kr/rolelens/README.md) · [日本語ポートフォリオ](https://portfolio-643.pages.dev/jp/)

連絡先: [len007070@gmail.com](mailto:len007070@gmail.com)
