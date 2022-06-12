# 「らぶくら」 `- クラウドを愛してやまない、エンジニアへ -` 
## 1. サービス概要
- `好きなAWSサービス`を自動でレコメンドしてくれるサービス。
- 診断結果をTwitter上で拡散することができる。
### ターゲットユーザー
  - `AWSを愛してやまない、エンジニア`
### ユーザーが抱える課題
  - エンジニアなら、皆さんご存知のAWS。サービス数は、現在220を超えており、年々増加の一途を辿っている。そのため、熟練のエンジニアでも使用したことはおろか、知らないサービスも数多く存在するはず。
  - AWS系の技術カンファレンスの際に、`「好きなAWSサービスは？」`と聞かれる場面もあるが、EC2、RDS、S3など、`「みんなが知っているサービスは言いたくない..」`というエンジニアのプライドがある。
  - また、`AWS`や`Amazon`が接頭字に付くサービスが数多くあり、正式名称がわからなくなってしまう。
### 解決方法
  - `好きなAWSサービス`を自動でレコメンドしてくれるサービス。
  - Twitterのアイコンを画像解析にかけて、診断。
  - 診断結果をTwitter上で出力する。
### なぜこのサービスを作りたいのか？
  - AWSに興味があって、AWS系の技術カンファレンスに参加した際に、登壇者が華麗に「`好きなAWSサービス`」を回答していくのを見て、大変驚いた。AWS初心者の私は、スラスラとそんなの答えられないよ...
  - そこで、自動でAWSサービスをレコメンドしてくれるサービスがあればと思い、このサービスを作成した。

## 2. 機能要件
- ユーザー登録機能
- ログイン・ログアウト機能
  -  外部認証（SSO）
- 画像解析機能
  - `Amazon Rekognition`との連携
- 診断結果画像の作成機能
  - 画像化したい、どうしよう
- Twitterへの出力機能
  - `Twitter API`との連携
- SEO対策
  - Google Analytics/OGP設定/メタタグ設定

## 3. 非機能要件
- `AWSを愛してやまないエンジニア`のため、みんな大好き`EC2`を使用する必要がある。

## 4. 使用技術
  - 開発言語: Ruby
  - フレームワーク: Ruby on Rails
  - DB: MySQL
  - ソース管理: Git/GitHub
  - テスト: 
  - インフラ: AWS(EC2/S3/RDS)/Docker
  - 主要なライブラリ
    - `aws-sdk` (Amazon Rekognition)
    - `meta-tags` (メタタグの設定)
  - 使用ツール： Twitter/Draw.io

## 5. 今後追加したい機能
- GCPサービスの追加（やるかな..）

## 6. 参考URL
### ポートフォリオ作成
- [エンジニア転職に成功するポートフォリオ戦略!!](https://www.youtube.com/watch?v=nkdz-lLwDZI)

### ライブラリ
- [【ポートフォリオ】周りと差別化できる技術3選!!【エンジニア転職】](https://www.youtube.com/watch?v=Kl_GlmWGST8)
- [Amazon Rekognitionを新米エンジニアが触ってみた。](https://xp-cloud.jp/blog/2020/09/29/7645/)
- [Amazon RekognitionをRubyから叩いてみる](https://qiita.com/kaibadash@github/items/037a5cebc35f3925fec8)
- [AWS Service Namespace を流行らせたい、そんな想いを朝早くから話しました #jawsug_asa](https://dev.classmethod.jp/articles/ws-service-namespace-to-become-popular/)

### インフラ
- [ポートフォリオにAWSとDockerとCircleCIを組み込むための学習順序と教材について](https://www.youtube.com/watch?v=LT-dXBUnZdI)
- [(下準備編)世界一丁寧なAWS解説。EC2を利用して、RailsアプリをAWSにあげるまで](https://zenn.dev/naoki_mochizuki/books/1471ce20222227)
- [入門 Docker](https://y-ohgi.com/introduction-docker/)
- [Docker/Kubernetes 実践コンテナ開発入門](https://www.amazon.co.jp/dp/B07GP1Q3VT)
- [チュートリアル - CircleCI](https://circleci.com/docs/ja/2.0/tuto...)
- [CircleCI-Public/circleci-demo-ruby-rails](https://github.com/CircleCI-Public/ci...)
