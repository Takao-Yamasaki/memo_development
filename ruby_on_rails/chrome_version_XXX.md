# 【selenium】session not created: This version of ChromeDriver only supports Chrome version XXX

# 1. 環境
```Ruby:terminal
$ ruby -v
ruby 2.6.9p207 (2021-11-24 revision 67954) [x86_64-darwin19]
$ rails -v
Rails 5.2.7
```
```Ruby:Gemfile
gem 'rspec-rails', '~> 3.7'
gem 'factory_bot_rails', '~> 4.11'
```
- System Specを実行するドライバとして、```Headless Chrome```を使うため、```selenium_chrome_headless```をドライバーとして設定。

# 2. はまったこと
- [「現場で使える Ruby on Rails 5速習実践ガイド」](https://www.amazon.co.jp/%E7%8F%BE%E5%A0%B4%E3%81%A7%E4%BD%BF%E3%81%88%E3%82%8B-Ruby-Rails-5%E9%80%9F%E7%BF%92%E5%AE%9F%E8%B7%B5%E3%82%AC%E3%82%A4%E3%83%89-%E5%A4%A7%E5%A0%B4%E5%AF%A7%E5%AD%90/dp/4839962227)を読み進めていくなかで、初めてSpecを実行したところ、以下のようなエラーが表示された。
```Ruby:terminal
 $ bundle exec rspec spec/system/tasks_spec.rb

 1.1) Failure/Error: visit login_path

          Selenium::WebDriver::Error::SessionNotCreatedError:
            session not created: This version of ChromeDriver only supports Chrome version 100
            Current browser version is 99.0.4844.84 with binary path /Applications/Google Chrome.app/Contents/MacOS/Google Chrome
```

# 3. 原因
- 現在使用しているChromeのバージョンが対応していないためと考えられる。
- 今回の場合、現在のバージョンとしては、```99.0.4844.84```を使用しているが、```100```を使用する必要があるとのこと。

# 4. 試したこと
- 対応したchromeドライバーをダウンロード。
    - 今回の場合、```Supports Chrome version 100```と表示されている```ChromeDriver 100.0.4896.60```をダウンロードする。
    - http://chromedriver.chromium.org/downloads

- ダウンロードすると、デフォルトだと```Downloads```直下にダウンロードされるので、それを``` /usr/local/bin/```直下に移動する。
```Ruby:terminal
$ mv ~/Downloads/chromedriver /usr/local/bin/
```
- ```chromedriver```がちゃんと移動できているか確認する。
```Ruby:terminal
$ which chromedriver
/usr/local/bin/chromedriver
```
- 再度実行してみる。無事実行できた。
```Ruby:terminal
$ bundle exec rspec spec/system/tasks_spec.rb
2022-04-03 16:07:45 WARN Selenium [DEPRECATION] [:driver_path] Selenium::WebDriver::Chrome#driver_path= is deprecated. Use Selenium::WebDriver::Chrome::Service#driver_path= instead.
Capybara starting Puma...
* Version 3.12.6 , codename: Llamas in Pajamas
* Min threads: 0, max threads: 4
* Listening on tcp://127.0.0.1:54592
2022-04-03 16:07:46 WARN Selenium [DEPRECATION] [:browser_options] :options as a parameter for driver initialization is deprecated. Use :capabilities with an Array of value capabilities/options if necessary instead.
.

Finished in 7.99 seconds (files took 2.99 seconds to load)
1 example, 0 failures
```


# 5. 参考
- [[selenium]chromedriverのバージョンエラーが出たときの対処法 - Qiita](https://qiita.com/iHacat/items/9c5c186f0d146bc98784)