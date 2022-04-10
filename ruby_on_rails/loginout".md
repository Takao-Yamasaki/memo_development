# Rails5】No route matches [GET] "/loginout"

# 1. はじめに
- [「現場で使える Ruby on Rails 5速習実践ガイド」](https://www.amazon.co.jp/%E7%8F%BE%E5%A0%B4%E3%81%A7%E4%BD%BF%E3%81%88%E3%82%8B-Ruby-Rails-5%E9%80%9F%E7%BF%92%E5%AE%9F%E8%B7%B5%E3%82%AC%E3%82%A4%E3%83%89-%E5%A4%A7%E5%A0%B4%E5%AF%A7%E5%AD%90/dp/4839962227)を進めています。
- 開発環境については、次のとおりです。
```
$ ruby -v
ruby 2.6.9p207 (2021-11-24 revision 67954) [x86_64-darwin19]
$ rails -v
Rails 5.2.7
```
```Ruby:Gemfile
gem 'turbolinks', '~> 2.5.3'
```
# 2. はまったこと
- 一覧画面からログアウトしようとしたところ、```No route matches [GET] "/loginout"```と```Routing Error```が表示された。
- ChromeのディベロッパーツールのConsoleも確認したところ、こちらでもエラーを確認
```GET http://localhost:3000/loginout 404 (Not Found)```
# 3. 試したこと
- まずは、現在のコードや設定を確認する。
- こちらは```method: :delete```になっている。
```slim:app/views/layouts/application.html.slim
li.nav-item= link_to 'ログアウト', loginout_path, method: :delete, class: 'nav-link'
```
- ```rails-ujs```はちゃんと入っている。
    - このライブラリが```method:delete```によるDELETEリクエストの発行を担っている。
    - ```jquery```と```jquery_ujs```を入れないといけないのかと思ったが、```rails-ujs```と役割が重複するので、入れなかった。
- rails6系は、記載場所（/app/javascript/packs/application.js）と記載方法そのものが、違うようなので注意。
```Ruby:app/assets/javascripts/application.js
// = require rails-ujs
// = require activestorage
// = require turbolinks
// = require_tree .
```
- 一応、こちらを```link_to```から```button_to```に変更すれば、応急処置的には動作する。
- ただ、ちゃんと```link_to```で動かしたい。
```slim:app/views/layouts/application.html.slim
li.nav-item= button_to 'ログアウト', loginout_path, method: :delete, class: 'nav-link'
```

- 単純に、```= javascript_include_tag 'application', 'data-turbolinks-track': 'reload'```がコメントアウトになっていたので、```rails-ujs```が動かなかったようです。
```slim:app/views/layouts/application.html.slim
    /= javascript_include_tag 'application', 'data-turbolinks-track': 'reload'
```
# 4.参考
- [No route matches [GET] "/logout"の解決](https://teratail.com/questions/291155)
- [railsでlink_toメソッドが効かない](https://takayuki-inoue.hatenablog.com/entry/2018/01/12/105846)
- [（Rails6）link_to....method: :delete設定してもNo route matches [GET]エラーが発生します](https://qiita.com/takuo_maeda/items/d237fdb33411a20b9ec4)