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

## 参考
- [Rubocop導入＋手順とコマンドの備忘録](https://qiita.com/kaito_program/items/c7d21f2e90cf9b7f7cf2)
- [RuboCopとERB LintをRailsに導入する](https://zenn.dev/junara/articles/efa04ce1ab02c9)
