# 各種テストの実施手順

## RSpecの実施手順
- RSpecの実施
  - 実施後、テストがうまく通らなかった箇所が表示されるので、表示を確認して、都度修正する。
```
$ bundle exec rspec
```

## ERBlintの実施手順
- ERBlintを実施
```
$ bundle exec erblint .
```
- ERBlintで自動的に修正してもらう
```
$ bundle exec erblint --lint-all
```

## rubocopの実施手順
- rubocopを実施
```
$ bundle exec rubocop
```
- rubocopで自動的に修正してもらう
```
$ bundle exec rubocop -a
```
### rubocopの警告を無効化する方法
- 次のようなエラーがでた。uniqunessを使うと出る警告らしい。
```
app/models/like.rb:25:3: C: Rails/UniqueValidationWithoutIndex: Uniqueness validation should have a unique index on the database column.
  validates :user_id, uniqueness: { scope: post_id }
  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
```
- 警告を無視するために、次のように追記をする。
```
validates :user_id, uniqueness: { scope: post_id } # rubocop:disable all
```


## 参考
- [Rubocop導入＋手順とコマンドの備忘録](https://qiita.com/kaito_program/items/c7d21f2e90cf9b7f7cf2)
- [RuboCopとERB LintをRailsに導入する](https://zenn.dev/junara/articles/efa04ce1ab02c9)
- [RuboCopの警告をコメントで無効化する方法](https://qiita.com/tbpgr/items/a9000c5c6fa92a46c206)
