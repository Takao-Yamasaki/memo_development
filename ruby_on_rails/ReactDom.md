# 【Rails5】Warning: ReactDOM.render is no longer supported in React 18.

# 1.はじめに
- [「現場で使える Ruby on Rails 5速習実践ガイド」](https://www.amazon.co.jp/%E7%8F%BE%E5%A0%B4%E3%81%A7%E4%BD%BF%E3%81%88%E3%82%8B-Ruby-Rails-5%E9%80%9F%E7%BF%92%E5%AE%9F%E8%B7%B5%E3%82%AC%E3%82%A4%E3%83%89-%E5%A4%A7%E5%A0%B4%E5%AF%A7%E5%AD%90/dp/4839962227)を読み進めて、Reactを導入しようとした。
- 開発環境は、次のとおり。
```Ruby:terminal
$ ruby -v
ruby 2.6.9p207 (2021-11-24 revision 67954) [x86_64-darwin19]
$ rails -v
Rails 5.2.7
```
# 2.はまったこと
- Reactを導入して、試したところ、ブラウザに```hello```が表示されない。
- ChromeのディベロッパーツールでConsole画面を確認したところ、次のようなエラーが表示された。
- ReactDOMが、React 18では対応していないので、React 17に変更しろの意かな？

```Ruby:ChromeのConsole画面
Warning: ReactDOM.render is no longer supported in React 18. Use createRoot instead. Until you switch to the new API, your app will behave as if it's running React 17. Learn more: https://reactjs.org/link/switch-to-createroot
```
# 3.試したこと
- 関連するコードを確認。問題なさそう。
```Ruby:app/javascript/taskleaf/hello.js
import React from 'react';
import ReactDOM from 'react-dom';

document.addEventListener('DOMContentLoaded', () => {
  ReactDOM.render(
    React.createElement('div',null, 'Hello World'),
    document.body.appendChild(document.createElement('div')),
  );
});
```
```Ruby:app/javascript/packs/application.js
import 'taskleaf/hello';
console.log('Hello World from Webpacker')
```
```Ruby:app/views/layouts/application.html.slim
= javascript_pack_tag 'application'
```
- 現在のnodeのバージョンを確認
- どうもnodeが最新なので、動作しない模様
```Ruby:terminal
$ node -v
v17.8.0
```
- 現在のnodeをアンインストール
```Ruby:terminal
$ brew uninstall --force node
Uninstalling node... (1,947 files, 46.3MB)
```
- nodebrewをインストール
```Ruby:terminal
$ brew install nodebrew
```
- そうすると、このように表示された
```Ruby:terminal
You need to manually run setup_dirs to create directories required by nodebrew:
  /usr/local/opt/nodebrew/bin/nodebrew setup_dirs

Add path:
  export PATH=$HOME/.nodebrew/current/bin:$PATH

To use Homebrew's directories rather than ~/.nodebrew add to your profile:
  export NODEBREW_ROOT=/usr/local/var/nodebrew
```
- お目当ての12系を探す
```Ruby:terminal
$ nodebrew ls-remote
v12.22.10 v12.22.11 v12.22.12
```
- 12系をインストール
```Ruby:terminal
$ nodebrew install-binary v12.22.12
Fetching: https://nodejs.org/dist/v12.22.12/node-v12.22.12-darwin-x64.tar.gz
Warning: Failed to create the file
Warning: /Users/takao.yamasaki/.nodebrew/src/v12.22.12/node-v12.22.12-darwin-x6
Warning: 4.tar.gz: No such file or directory
                                                                                                                                                                                                        0.0%
curl: (23) Failed writing body (0 != 1124)
download failed: https://nodejs.org/dist/v12.22.12/node-v12.22.12-darwin-x64.tar.gz
```

```Ruby:terminal
$ nodebrew setup
Fetching nodebrew...
Installed nodebrew in /usr/local/var/nodebrew

========================================
Export a path to nodebrew:

export PATH=/usr/local/var/nodebrew/current/bin:$PATH
========================================
```
```~/.bash_profile```にパスを追記
```
$ echo "export PATH=\$HOME/.nodebrew/current/bin:\$PATH" >> ~/.bash_profile
```
- パスを通す
```
$ source ~/.bash_profile
```
- ```v12.22.12```を使用する
```
$ sudo nodebrew use v12.22.12
use v12.22.12
```
- バージョンの確認。たしかに、12系になった。
```
$ node -v
v12.22.12
```
- サーバーを再起動すると、```Hello World```が表示された。
![スクリーンショット 2022-04-09 22.07.31.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1159639/02032b2f-fcaa-caef-12df-7e155f123967.png)


# 4.参考
- [Node.jsの管理をHomebrewからnodebrewに変える](https://qiita.com/takeshi81/items/805f504503cd93151ca6)
- [Homebrewからnodebrewをインストールして、Node.jsをインストールするまで](https://qiita.com/limonene/items/a10c2755dd2784357c43)
- [MacにNode.jsをインストール](https://qiita.com/kyosuke5_20/items/c5f68fc9d89b84c0df09)
- [node.jsのバージョンを変更する](https://qiita.com/k3ntar0/items/322e668468716641aa5c)