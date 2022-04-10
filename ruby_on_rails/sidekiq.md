# NameError (uninitialized constant Sidekiq::Logging)

- [「現場で使える Ruby on Rails 5速習実践ガイド」](https://www.amazon.co.jp/%E7%8F%BE%E5%A0%B4%E3%81%A7%E4%BD%BF%E3%81%88%E3%82%8B-Ruby-Rails-5%E9%80%9F%E7%BF%92%E5%AE%9F%E8%B7%B5%E3%82%AC%E3%82%A4%E3%83%89-%E5%A4%A7%E5%A0%B4%E5%AF%A7%E5%AD%90/dp/4839962227)の「Chapter7-8　非同期処理や定期実行を行う（Jobスケジューリング）」を進めていたところ、```sidekiq```の実行でハマってしまいました。

# 1.開発環境
```Ruby:terminal
$ ruby -v
ruby 2.6.9p207 (2021-11-24 revision 67954) [x86_64-darwin19]
$ rails -v
Rails 5.2.7
```

# 2.はまったこと
```bundle exec sidekiq```を実行したところ、次のようなエラーが出てきた。
```Ruby:terminal
$ bundle exec sidekiq

2022-04-07T13:09:00.520Z pid=86343 tid=oxv90atm3 class=SampleJob jid=9bc76e2ad41ce269483e50e9 INFO: start
2022-04-07T13:09:00.524Z pid=86343 tid=oxv90atm3 class=SampleJob jid=9bc76e2ad41ce269483e50e9 INFO: Performing SampleJob (Job ID: 5a984a92-f202-4701-9df6-7432530baa87) from Sidekiq(default)
2022-04-07T13:09:00.527Z pid=86343 tid=oxv90atm3 class=SampleJob jid=9bc76e2ad41ce269483e50e9 ERROR: Error performing SampleJob (Job ID: 5a984a92-f202-4701-9df6-7432530baa87) from Sidekiq(default) in 2.52ms: NameError (uninitialized constant Sidekiq::Logging):
```
# 3.試したこと
- ```sidekiq```のバージョンが新しすぎるのが、原因のよう。
- ```Gemfile```に次のように記述し、```bundle```を実行
```Ruby:Gemfile
gem 'sidekiq','~> 5.0'
```
- 無事ジョブが実行された
```Ruby:terminal
$ bundle exec sidekiq

2022-04-07T13:17:24.300Z 86981 TID-ov9dgyje9 SampleJob JID-ea78daf25173baffbf567f19 INFO: start
2022-04-07T13:17:24.607Z 86981 TID-ov9dgyje9 SampleJob JID-ea78daf25173baffbf567f19 INFO: サンプルジョブを実行しました。
2022-04-07T13:17:24.607Z 86981 TID-ov9dgyje9 SampleJob JID-ea78daf25173baffbf567f19 INFO: done: 0.307 sec
```
# 4.参考
- [NameError: uninitialized constant Sidekiq::Loggingの解決方法](https://teratail.com/questions/235628)