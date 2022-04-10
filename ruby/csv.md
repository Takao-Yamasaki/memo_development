# RubyでCSVファイルを出力してみた

# 1.やりたいこと
- Amazonの商品URLから`商品ID`と`商品名`を抽出
- `商品名`を抽出したうえで、エンコーディングを文字列に変換
- CSVファイルで書き出し

# 2.やったこと
- AmazonのURLの構造は次のようになっている。
- そのうち、`商品名`と`商品ID`を取り出して、加工し、CSV出力したい。
```Ruby：URLの構造
https://www.amazon.co.jp/商品名(エンコード)/dp/商品ID
```
- 標準ライブラリの`uri`と`csv`を使用する
```Ruby:output_csv.rb
require 'uri'
require 'csv'

url_array = [
  'https://www.amazon.co.jp/%E3%82%B0%E3%83%AC%E3%83%B4%E3%82%A3%E3%82%AA-%E3%83%AA%E3%83%A5%E3%83%83%E3%82%AF-%E3%83%93%E3%82%B8%E3%83%8D%E3%82%B9%E3%83%AA%E3%83%A5%E3%83%83%E3%82%AF-AIN_HYT_SWN_businessryukku_00-%E3%83%96%E3%83%A9%E3%83%83%E3%82%AF/dp/B085HLVJYP',
  'https://www.amazon.co.jp/%E3%83%93%E3%82%A2%E3%83%B3%E3%82%AD-%E3%83%95%E3%83%A9%E3%83%83%E3%83%97%E3%83%90%E3%83%83%E3%82%AF%E3%83%91%E3%83%83%E3%82%AF-%E3%83%AA%E3%83%A5%E3%83%83%E3%82%AF-NBTC-37-%E6%92%A5%E6%B0%B4/dp/B00UY43FVS',
  'https://www.amazon.co.jp/MARK-RYDEN-%E3%83%90%E3%83%83%E3%82%AF%E3%83%91%E3%83%83%E3%82%AF%E9%98%B2%E6%B0%B4-%E3%83%93%E3%82%B8%E3%83%8D%E3%82%B9%E3%83%AA%E3%83%A5%E3%83%83%E3%82%AF-%E7%9B%97%E9%9B%A3%E9%98%B2%E6%AD%A2%E3%83%A9%E3%83%83%E3%83%97%E3%83%88%E3%83%83%E3%83%97%E3%83%90%E3%83%83%E3%82%B017%E3%82%A4%E3%83%B3%E3%83%81%E3%83%91%E3%82%BD%E3%82%B3%E3%83%B3%E5%AF%BE%E5%BF%9C/dp/B082VTKYY9',
  'https://www.amazon.co.jp/%E3%83%93%E3%82%B8%E3%83%8D%E3%82%B9%E3%83%AA%E3%83%A5%E3%83%83%E3%82%AF-%E3%83%90%E3%83%83%E3%82%AF%E3%83%91%E3%83%83%E3%82%AF-USB%E5%85%85%E9%9B%BB%E3%83%9D%E3%83%BC%E3%83%8815-6%E3%82%A4%E3%83%B3%E3%83%81PC-%E3%83%AA%E3%83%A5%E3%83%83%E3%82%AF%E3%82%B5%E3%83%83%E3%82%AF-%E3%83%A1%E3%83%B3%E3%82%BA%E3%83%AA%E3%83%A5%E3%83%83%E3%82%AF/dp/B08FY6NGWN',
  'https://www.amazon.co.jp/SUNOGE-%E3%83%90%E3%83%83%E3%82%AF%E3%83%91%E3%83%83%E3%82%AF-%E3%83%AA%E3%83%A5%E3%83%83%E3%82%AF%E3%82%B5%E3%83%83%E3%82%AF-%E3%83%93%E3%82%B8%E3%83%8D%E3%82%B9%E3%83%AA%E3%83%A5%E3%83%83%E3%82%AF-%E3%83%A9%E3%83%83%E3%83%97%E3%83%88%E3%83%83%E3%83%97%E3%83%90%E3%83%83%E3%82%B0/dp/B08XHS9FQP'
]

items = {}

# URLの加工
url_array.each do |url|
  # 商品IDを抽出
  item_id = url.split('/')[5]
  # 商品名を抽出し、エンコーディングを変換
  item_name = URI.decode_www_form_component(url.split('/')[3])
  # ハッシュ化する
  items[item_id] = item_name 
end

# CSVの書き出し
CSV.open('./result.csv','w') do |result|
  # ヘッダを出力
  result << ['商品ID','商品名']
  items.each do |key, value|
    # 明細行を出力
    result << [key, value]
  end
end

puts 'CSV出力が終了しました'
```
- ```map```メソッドを使うと、よりRubyのコードらしくなるかも。
```Ruby:output_csv.rb
items = {}

# URLの加工
# key: urlから商品IDを抽出したものを格納
# value: urlから商品名を抽出し、エンコーディングを変換したものを格納
# mapだと配列を返すので、to_hでハッシュ化する
items =  url_array.map { |url| [url.split('/')[5], URI.decode_www_form_component(url.split('/')[3])] }.to_h

# CSVの書き出し
CSV.open('./result.csv','w') do |result|
  # ヘッダを出力
  result << ['商品ID','商品名']
  # 明細行を出力
  items.map {|key, value| result << [key, value]} 
end

puts 'CSV出力が終了しました'
```
- 出力結果
```shell:terminal
$ cat result.csv
商品ID,商品名
B085HLVJYP,グレヴィオ-リュック-ビジネスリュック-AIN_HYT_SWN_businessryukku_00-ブラック
B00UY43FVS,ビアンキ-フラップバックパック-リュック-NBTC-37-撥水
B082VTKYY9,MARK-RYDEN-バックパック防水-ビジネスリュック-盗難防止ラップトップバッグ17インチパソコン対応
B08FY6NGWN,ビジネスリュック-バックパック-USB充電ポート15-6インチPC-リュックサック-メンズリュック
B08XHS9FQP,SUNOGE-バックパック-リュックサック-ビジネスリュック-ラップトップバッグ
```

# 3.参考
- [【Ruby入門説明書】csvについて解説](https://web-camp.io/magazine/archives/16802)
- [RubyでCSVファイルの読み込み・書き込みをする](https://uxmilk.jp/19202)
- [【Ruby】Array から Hash を作る方法７選（a.k.a. やっぱり Array#zip はかわいいよ）](https://tech.raksul.com/2018/02/06/ruby_array_to_hash/)
- [module URI(Ruby 3.1 リファレンスマニュアル )](https://docs.ruby-lang.org/ja/latest/class/URI.html)
- [instance method String#split(Ruby 3.1 リファレンスマニュアル)](https://docs.ruby-lang.org/ja/latest/method/String/i/split.html)
- [【Ruby】Array から Hash を作る方法７選（a.k.a. やっぱり Array#zip はかわいいよ） | Raksul ENGINEERING](https://tech.raksul.com/2018/02/06/ruby_array_to_hash/)
