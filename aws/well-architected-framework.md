# はじめに

Well-Architedtedフレームワークは、クラウドにおけるシステム設計構築運用のベストプラクティス。W-Aフレームワークとも呼ばれる

AWS Well-Architectedフレームワーク5つの柱
* 運用上の優位性
* セキュリティ
* 信頼性
* パフォーマンス効率
* コスト最適化

# フレームワークの5本の柱

## 運用上の優秀性

システムを稼働およびモニリングする能力

### 設計原則

* 運用をコードとして実行する
* ドキュメントに注釈をつける
* 小規模な元に戻すことのできる変更を定期的に適用する
* 運用手順を定期的に改善する
* 障害を予想する
* 運用上のすべての障害から学ぶ

### ベストプラクティス

* 準備
* 運用
* 進化

### 主要なAWSサービス

準備
* AWS Configで環境がスタンダードに準拠しているか確認する
運用
* Amazon CloudWatchによるモニタリング
進化
* Amazon Elasticsearch Serviceでログデータを分析し実用的なインサイトを得る

## セキュリティ

情報、システム、アセットのセキュリティ保護機能

### 設計原則

* 強力なアイデンティティ基盤の実装
    * 最小権限の原則、AWSリソースとの通信に置いて適切な認証を実行、権限管理の一元化
* トレーサビリティの実現
    * リアルタイムで監視、アラート、監査
* 全レイヤへのセキュリティの適用
* セキュリティのベストプラクティスの自動化
* 伝送中および補完中のデータの保護
* データに人の手を入れない
* セキュリティイベントへの備え

### ベストプラクティス

* アイデンティティ管理とアクセス権限
* 発見的統制
* インフラストラクチャ保護
* データ保護
* インシデント対応

### 主要なAWSサービス

アイデンティティ管理とアクセス権限
* IAMによるアクセス管理
* MFAでユーザアクセスに保護レイヤを追加
* AWS Organizationsにより複数のAWSアカウントのポリシーを一元管理

発見的統制
* AWS CloudTrailでAWS APIコールを記録
* AWS ConfigでAWSのリソースと設定の詳細なインベントリを提供
* Amazon Guard

インフラストラクチャ保護
データ保護
インシデント対応

## 信頼性

インフラストラクチャやサービスが復旧し、需要に適したコンピューティングリソースを動的に獲得し、誤設定や一時的なネットワークの問題といった中断の影響を緩和する能力

### 設計原則

* 復旧手順をテストする
* 障害から自動的に復旧する
* 水平方向にスケールしてシステム全体の可用性を高める
* キャパシティを推測しない
* オートメーションで変更を管理する

### ベストプラクティス

3つのベストプラクティス
* 基盤
* 変更管理
* 障害の管理

### 主要なAWSサービス

Amazon CloudWatch

## パフォーマンス効率

システムの要件を満たすためにコンピューティングリソースを効率的に使用し、要求の変化とテクノロジーの進化に対してもその効率性を維持する能力

### 設計原則

* 最新テクノロジーの標準化
* 数分でグローバルに展開
* サーバレスアーキテクチャを使用
* より頻繁に実験可能
* システムを深く理解

### ベストプラクティス

* 選択
* レビュー
* モニタリング
* トレードオフ

### 主要なAWSサービス

## コスト最適化

最も低い価格でシステムを運用してビジネス価値を実現する能力

### 設計原則

* 消費モデルを導入する
* 全体的な効率を測定する
* データセンター運用のための費用を排除する
* 費用を分析し帰結させる
* アプリケーションレベルのマネージドサービスを使用して所有コストを削減する

### ベストプラクティス

* 費用認識
* 費用対効果の高いリソース
* 需要と供給を一致させる
* 長期的な最適化

### 主要なAWSサービス

# レビュープロセス

対策を必要とする重大な問題、改善可能な領域を特定することが目的