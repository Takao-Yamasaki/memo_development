# Ruby on Railsでrails db:migrateできない

# 1. はじめに
- [「現場で使える Ruby on Rails 5速習実践ガイド」](https://www.amazon.co.jp/%E7%8F%BE%E5%A0%B4%E3%81%A7%E4%BD%BF%E3%81%88%E3%82%8B-Ruby-Rails-5%E9%80%9F%E7%BF%92%E5%AE%9F%E8%B7%B5%E3%82%AC%E3%82%A4%E3%83%89-%E5%A4%A7%E5%A0%B4%E5%AF%A7%E5%AD%90/dp/4839962227)を２年程前に購入して、全く手を付けて行かなかった。
- 今回エンジニア転職をするに当たって、再度Ruby on Railsを勉強することになったので、学習を初めてみた。

# 2. はまったこと
- RubyとRuby on Railsの環境構築を行った上で、「Chapter2-6：Railsに触れてみよう」の「2-6 1:実際にアプリケーションを動かしてみよう」を試してみたところ、```cannot load such file```と表示された。
```ruby:terminal
$ rails db:create
bin/rails:3:in `require_relative': cannot load such file -- /Users/takao.yamasaki/Documents/ruby_quick_learning/scaffold_app/config/boot (LoadError)
	from bin/rails:3:in `<main>'
```
# 3. 試したこと
- Rubyのバージョンを確認してみる。
```Ruby:terminal
$ ruby -v
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin19]
```
- Railsのバージョンを確認してみる
```Ruby:terminal
＄ rails -v
Rails 5.2.6
```
- どうやら、[configフォルダにboot.rbファイルが作られないため、rails sが出来ない。](https://teratail.com/questions/323505)によるとRailsのバージョンがRubyのバージョンに対応していないよう。
- インストール可能なrubyのバージョンを確認してみる。
```Ruby:terminal
$ rbenv install -l
2.6.9
2.7.5
3.0.3
3.1.1
jruby-9.3.3.0
mruby-3.0.0
rbx-5.0
truffleruby-22.0.0.2
truffleruby+graalvm-22.0.0.2

Only latest stable releases for each Ruby implementation are shown.
Use 'rbenv install --list-all / -L' to show all local versions.
```
- Rubyをインストール
```Ruby:terminal
rbenv install 2.6.9
Downloading openssl-1.1.1l.tar.gz...
-> https://dqw8nmjcqpjn7.cloudfront.net/0b7a3e5e59c34827fe0c3a74b7ec8baef302b98fa80088d7f9153aa16fa76bd1
Installing openssl-1.1.1l...
Installed openssl-1.1.1l to /Users/takao.yamasaki/.rbenv/versions/2.6.9

Downloading ruby-2.6.9.tar.bz2...
-> https://cache.ruby-lang.org/pub/ruby/2.6/ruby-2.6.9.tar.bz2
Installing ruby-2.6.9...
ruby-build: using readline from homebrew
Installed ruby-2.6.9 to /Users/takao.yamasaki/.rbenv/versions/2.6.9
```
- ローカルのRubyのバージョンを`2.6.9`に変更
```Ruby:terminal
$ rbenv local 2.6.9
```
- Rubyのバージョンを確認する。
```Ruby:terminal
ruby -v
ruby 2.6.9p207 (2021-11-24 revision 67954) [x86_64-darwin19]
```
- 再度、railsをインストールする。
```Ruby:terminal
$ gem install rails -v 5.2.6
```
- アプリケーションを新規作成する。
```Ruby:terminal
$ rails new scaffold_app
```

- データベースを新規作成してみる。どうやら、成功したよう。
```Ruby:terminal
$ rails db:create
Created database 'db/development.sqlite3'
Created database 'db/test.sqlite3'
```
# 4. 参考
- [「現場で使える Ruby on Rails 5速習実践ガイド」](https://www.amazon.co.jp/%E7%8F%BE%E5%A0%B4%E3%81%A7%E4%BD%BF%E3%81%88%E3%82%8B-Ruby-Rails-5%E9%80%9F%E7%BF%92%E5%AE%9F%E8%B7%B5%E3%82%AC%E3%82%A4%E3%83%89-%E5%A4%A7%E5%A0%B4%E5%AF%A7%E5%AD%90/dp/4839962227)
- [configフォルダにboot.rbファイルが作られないため、rails sが出来ない。](https://teratail.com/questions/323505)
