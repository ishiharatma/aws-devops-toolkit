# AWS Cost Watcher for Microsoft Teams

[![🇯🇵 日本語](https://img.shields.io/badge/%F0%9F%87%AF%F0%9F%87%B5-日本語-white)](./README-ja.md) [![🇺🇸 English](https://img.shields.io/badge/%F0%9F%87%BA%F0%9F%87%B8-English-white)](./README.md)

Microsoft TeamsチャンネルにAWSの週次コストサマリーを自動投稿するAWSコスト監視システムです。このツールはaws-devops-toolkitリポジトリの一部で、`tools/monitoring/aws-cost-watcher`に配置されています。この解決策は[こちらの記事](https://dev.classmethod.jp/articles/aws-cost-watcher-with-sfn-jsonata/)にインスパイアされ、SlackからMicrosoft Teamsに対応し、メッセージをカスタマイズしたものです。

## 機能

- 📊 **週次コスト監視**: 過去7日間のAWSコストを自動追跡
- 🚨 **閾値アラート**: 定義した閾値を超えると怒りの通知を送信
- 📈 **トップ5サービス**: コスト上位5つのAWSサービスを表示
- 🤖 **自動スケジュール**: 毎日午前10時（JST）に実行
- 👥 **Teams連携**: Microsoft Teamsチャンネルに直接フォーマットされたメッセージを投稿

## アーキテクチャ

この解決策は以下のAWSサービスを使用します：

- **AWS Step Functions**: コスト監視ワークフローのオーケストレーション
- **AWS Cost Explorer API**: コストと使用量データの取得
- **AWS Chatbot**: Microsoft Teams連携
- **Amazon SNS**: 通知の発行
- **Amazon EventBridge Scheduler**: 日次実行のトリガー

## 前提条件

- 適切な権限で設定されたAWS CLI
- 通知を受信したいMicrosoft Teamsチャンネル
- Teams App ID、Tenant ID、Channel ID

## Teams情報の取得

この解決策をデプロイするには、以下のMicrosoft Teams情報が必要です：

### 1. Team ID、Channel ID、Tenant ID

1. Microsoft Teamsを開く
2. 対象のチームに移動
3. チーム名の横の三点リーダー（...）をクリック
4. "チームへのリンクを取得"を選択
5. URLからTeam ID、Channel ID、Tenant IDを抽出

```text
https://teams.microsoft.com/l/channel/<Channel ID>/xxxx?groupId=<Team ID>&tenantId=<Tenant ID>
```

## デプロイ

1. ツールディレクトリに移動

```bash
cd tools/monitoring/aws-cost-watcher
```

2. CloudFormationスタックをデプロイ

```bash
aws cloudformation deploy \
  --template-file aws-cost-watcher-for-teams.yaml \
  --stack-name aws-cost-watcher-teams \
  --parameter-overrides \
    Project=your-project \
    Env=prod \
    TeamId=your-team-id \
    TeamsChannelID=your-channel-id \
    TeamsTenantId=your-tenant-id \
    AngryThreshold=150 \
  --capabilities CAPABILITY_NAMED_IAM
```

## パラメータ

| パラメータ | 説明 | デフォルト | 必須 |
|-----------|------|-----------|------|
| `Project` | リソース命名用のプロジェクト名 | `sample` | いいえ |
| `Env` | 環境名 | `dev` | いいえ |
| `NameSuffix` | リソース名のサフィックス | `aws-cost-watcher` | いいえ |
| `TeamId` | Microsoft Teams Team ID（GUID形式） | - | はい |
| `TeamsChannelID` | Teams Channel ID（URLエンコード形式） | - | はい |
| `TeamsTenantId` | Microsoft Teams Tenant ID（GUID形式） | - | はい |
| `AngryThreshold` | 怒り通知の週次コスト閾値（USD） | `150` | いいえ |

## メッセージ形式

システムはコスト閾値に基づいて2種類のメッセージを送信します：

### 通常メッセージ（✅😄）

コストが閾値以下の場合、以下の情報を含む穏やかな通知を受信します：
- 今週の総コスト
- コスト上位5サービス
- 集計期間
- コスト閾値情報

### 怒りメッセージ（❌🤬）

コストが閾値を超えた場合、同じ情報を含む緊急通知を受信しますが、アラート絵文字と緊急メッセージングが付きます。

## カスタマイズ

以下の要素をカスタマイズできます：

### スケジュール

CloudFormationテンプレートの`ScheduleExpression`を変更

```yaml
ScheduleExpression: "cron(00 10 * * ? *)"  # 毎日午前10時（JST）
```

### コスト閾値

デプロイ時に`AngryThreshold`パラメータを調整するか、スタックを更新します。

### メッセージ形式

メッセージ形式はJSONata式を使用してStep Functions定義で定義されています。`title`と`description`フィールドを変更して外観をカスタマイズできます。

## 監視とトラブルシューティング

### Step Functions実行の確認

1. AWS Step Functionsコンソールにアクセス
2. ステートマシンを探す：`{Project}-{Env}-aws-cost-watcher-sfn`
3. 実行履歴とログを確認

### Chatbot設定の確認

1. AWS Chatbotコンソールにアクセス
2. Microsoft Teams設定を確認
3. SNSトピックが適切に設定されていることを確認

### Cost Explorer権限

Step Functions実行ロールに`ce:GetCostAndUsage`権限があることを確認してください。

## コスト考慮事項

この解決策は最小限のコストを発生させます。

- Step Functions：1,000実行あたり約$0.025
- SNS：100万通知あたり約$0.50
- Cost Explorer API：リクエストあたり$0.01

日次実行の場合、月額コストは$1以下になります。

## セキュリティ

この解決策はAWSセキュリティのベストプラクティスに従います。

- 最小限の必要権限を持つIAMロール
- ハードコードされた認証情報なし
- 暗号化されたSNSトピック（オプション、有効化可能）

## 貢献

このツールはaws-devops-toolkitリポジトリの一部です。貢献するには：

1. aws-devops-toolkitリポジトリをフォーク
2. 機能ブランチを作成
3. `tools/monitoring/aws-cost-watcher`ディレクトリで変更を加える
4. プルリクエストを送信

## ライセンス

このプロジェクトはMITライセンスの下でライセンスされています - 詳細はLICENSEファイルを参照してください。

## 謝辞

- 元の[AWS Cost Watcher記事](https://dev.classmethod.jp/articles/aws-cost-watcher-with-sfn-jsonata/)にインスパイアされました
- Step FunctionsとJSONataの例を提供してくれたAWSコミュニティに感謝