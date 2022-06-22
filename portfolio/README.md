# 「らぶくら」 `- クラウドを愛してやまない、エンジニアへ -`(あとから考える) 
## 1. サービス概要
- QAに答えていくことで、ユーザーの嗜好に合わせて、`好きなAWSサービス`を自動でレコメンドしてくれるサービス。
- 診断結果をTwitter上で拡散（シェア）することができる。

### 類似サービス
- [水見式念能力診断 | リアル脱出ゲーム × HUNTER×HUNTER「グリードアイランド遊園地からの脱出」](https://realdgame.jp/hunter2020/special/shindan/)
- [診断メーカー | 水見式](https://shindanmaker.com/767275)

### ターゲットユーザー
  - `AWS関連のイベントによく登壇するインフラエンジニア`

### ユーザーが抱える課題
  - エンジニアなら、皆さんご存知のAWS。サービス数は、現在220を超えており、年々増加の一途を辿っている。そのため、熟練のインフラエンジニアでも使用したことはおろか、知らないサービスも数多く存在するはず。
  - AWS系のイベントによく登壇するインフラエンジニアだと、`「好きなAWSサービスは？」`と問われる場面が多くあるが、EC2、RDS、S3など、`「みんなが知っているサービスは言いたくない..」`というエンジニアとしてのプライドがあると考えられる。
  - また、数多くのイベントに登壇しなきゃいけないので、いつも同じ`好きなAWSサービス`を言うわけにはいかないし、それでは聴衆を飽きさせてしまう。
### 解決方法
  - 「QAに答えていくことで、ユーザーの嗜好に合わせて「`好きなAWSサービス`」を自動でレコメンドしてくれるサービス」があれば、上記課題を解決することができる。
    - 事前に用意された質問に答えていく。
    - 診断結果をTwitter上で出力する。
### なぜこのサービスを作りたいのか？
  - AWSに興味があって、AWSのカンファレンスに参加した際に、登壇者が華麗に「`好きなAWSサービス`」を回答していくのを見て、大変驚いた。AWS初心者の私は、基本的なサービスしか知らないし、ましてや他のサービスなんてスラスラと答えられない。きっと数多くのAWSのイベントに登壇しているであろうインフラエンジニアも、数あるAWSサービスから「`好きなAWSサービス`」を選び出すのに苦慮しているだろう。
  - そこで、自動でAWSサービスをレコメンドしてくれるサービスがあればと思い、このサービスを作成した。

## 2. 機能要件
### MVP
- 質問・回答機能
- Twitterへの出力機能
  - `Twitter API`との連携
- SEO対策
  - Google Analytics/OGP設定/メタタグ設定
### 将来的に実装したい機能
- ユーザー登録機能
- ログイン・ログアウト機能
  -  外部認証（SSO）
- 管理画面の実装
  - 質問内容について、自由にカスタマイズすることができる。
## 3. 非機能要件
- `AWSを愛してやまないインフラエンジニア`のため、みんな大好き`EC2`上にホストする必要がある。

## 4. 使用技術
  - 開発言語: Ruby
  - フレームワーク: Ruby on Rails
  - DB: MySQL
  - ソース管理: Git/GitHub
  - テスト: 
  - インフラ: AWS(EC2/S3/RDS)/Docker
  - 主要なライブラリ
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
