# RubyでJSONを扱ってみよう

# 1.やりたいこと
Slack上でのやり取りをJSON形式で出力してきて、その内容を分析したい。
具体的には、次の３つを行うプログラムを作成したい。

- 発信が多かったチャンネルベスト3を特定したい。
    - ```high_motivation_user_aggregator.rb```
- スタンプを多く押したユーザーベスト3を特定したい。
    - ```kind_user_aggregator.rb```
- 一番スタンプを獲得したメッセージを特定したい。
    - ```poupular_message_aggregator.rb```

また、Slackから出力したJSONデータの構成については、次の記事を参照すること。
[Slack からエクスポートしたデータの読み方](https://slack.com/intl/ja-jp/help/articles/220556107-Slack-%E3%81%8B%E3%82%89%E3%82%A8%E3%82%AF%E3%82%B9%E3%83%9D%E3%83%BC%E3%83%88%E3%81%97%E3%81%9F%E3%83%87%E3%83%BC%E3%82%BF%E3%81%AE%E8%AA%AD%E3%81%BF%E6%96%B9)

なお、JSONデータをRubyで読み込む方法については、記載していない。
```load```メソッドで記載されている部分が、JSONの読み込み実行部分。
# 2.やったこと
## 2.1 発信が多かったチャンネルベスト3を特定したい。
```Ruby:high_motivation_user_aggregator.rb
def exec
    result = []
    
    @channel_names.each do |channel_name|
      message_count = 0

      data = self.load(channel_name)
      data["messages"].map {|m| message_count += 1 if m["type"] == "message" }
      hash = {}
      hash[:channel_name] = channel_name
      hash[:message_count] = message_count
      result << hash
    end

    result.max(3) { |a, b| a[:message_count] <=> b[:message_count] }
  end
```
- 空白の配列を宣言せず、mapメソッドを使った。こっちのほうがスマートかもしれない。
```Ruby:high_motivation_user_aggregator.rb
def exec
    messages_count_data = @channel_names.map do |channel_name|
      {
        channel_name: channel_name,
        message_count: self.load(channel_name)["messages"].each.count {|m| m["type"] == "message"}
      }
    end

    messages_count_data.max(3) { |a, b| a[:message_count] <=> b[:message_count] }
  end
```
## 2.2 スタンプを多く押したユーザーベスト3を特定したい。
```Ruby:kind_user_aggregator.rb
def exec
    result = []
    users_data = []

    @channel_names.each do |channel_name|
      data = self.load(channel_name)
      data["messages"].map {|m| m["reactions"].nil? ? next : (users_data << m["reactions"].map {|r| r["users"] })}
    end
    # users_dataを平坦化
    users_data.flatten!
    # uniqで重複した要素を除く
    users_data.uniq.each do |u|
      hash = {}
      hash[:user_id] = u
      hash[:reaction_count] = users_data.count(u)
      result << hash
    end
    
    result.max(3){ |a, b| a[:reaction_count] <=> b[:reaction_count] }
  end
```
- こっちのほうがスマート
```Ruby:kind_user_aggregator.rb
def exec
    messages_data = @channel_names.map { |channel_name| self.load(channel_name)["messages"] }.flatten
    users_data = messages_data.map { |m| m["reactions"] }.compact.flatten.map { |r| r["users"] }.flatten
    reactions_data = users_data.uniq.map { |u| { user_id: u, reaction_count: users_data.count(u) } }
    reactions_data.max(3){ |a, b| a[:reaction_count] <=> b[:reaction_count] }
end
```
## 2.3 一番スタンプを獲得したメッセージを特定したい。

```Ruby:poupular_message_aggregator.rb

def exec
    result = []

    @channel_names.each do |channel_name|
      data = self.load(channel_name)
      data["messages"].each do |m|
        hash = {}
        hash[:text] =  m["text"]
        m["reactions"].nil? ? next : (hash[:reaction_count] =  m["reactions"].map { |r| r["count"] }.sum)
        result << hash
      end
    end
    result.max {|a, b| a[:reaction_count] <=> b[:reaction_count] }
  end
```
- こっちのほうがスマート
```Ruby:poupular_message_aggregator.rb
def exec
    messages_data = @channel_names.map {|channel_name| self.load(channel_name)["messages"]}.flatten
    reaction_count_data = messages_data.map { |m| m["reactions"].nil? ? next : { text: m["text"], 
    reaction_count: m["reactions"].map{|r| r["count"]}.sum } }.compact!
    reaction_count_data.max {|a, b| a[:reaction_count] <=> b[:reaction_count] }
end
```
# 3. 出力結果
``` Ruby:terminal
$ ruby main.rb
[{:channel_name=>"times_taishiro.json", :message_count=>301}, {:channel_name=>"times_miketa.json", :message_count=>197}, {:channel_name=>"sparta_course.json", :message_count=>183}]
[{:user_id=>"UEAENGTNY", :reaction_count=>80}, {:user_id=>"U011L4JS3L3", :reaction_count=>74}, {:user_id=>"U01L7QF8WNR", :reaction_count=>49}]
{:text=>"<!here>\n突然すみません！\n個人的なことで、大変恐縮なんですが先月末にweb系自社+受託開発の企業に内定を頂いたので報告させてもらいます！\n\n<@US0NKHJS3> さんのご厚意で、面接を受けさせて頂いたのがきっかけでした！\nなんで、お気付きの方もいるかと思いますが\n<@US0NKHJS3> , <@U011L4JS3L3> さんと同じ会社です！笑\n\nスパルタコースに来てから、すごく楽しく勉強できたので、結果的に僕はフルタイムで働きながら、内定を頂くことができました！\nなんで、今働きながら勉強してる方は本当に大変だと思いますが頑張って下さい！\n時間はかかりますが、スパルタコースにいたら、絶対実力は付きますし転職も出来ると思います！\n\n1人でやっていたら絶対無理だったと思うので、だいそんさんを初め、スパルタコースの皆さんのお陰です。本当に感謝してます！\n\nまた、面接の機会を下さっただいさん本当にありがとうございました！\n\nやっとスタートラインなので、これからも基礎を大切にして頑張って行こうと思います。\nなんでこれからもよろしくお願いしますー！", :reaction_count=>98}
```

# 4. 感想
- Rubyでよりスマートに書くためには、```map```メソッドをいかに使いこなすかが肝だと思われる。

# 5. 参考
- [RubyでJSONを読み込む - No Solution for Life](https://masuyama13.hatenablog.com/entry/2020/06/15/183142)
- [sortメソッドで配列とHashの中身を並び替える方法](https://pikawaka.com/ruby/sort)
- [三項演算子とは？？？ - Qiita](https://qiita.com/wacker8818/items/37d78222461232ef89f2)
- [Array#flatten (Ruby 3.1 リファレンスマニュアル)](https://docs.ruby-lang.org/ja/latest/method/Array/i/flatten.html)
- [Array#max (Ruby 3.1 リファレンスマニュアル)](https://docs.ruby-lang.org/ja/latest/method/Array/i/max.html)
- [Slack からエクスポートしたデータの読み方](https://slack.com/intl/ja-jp/help/articles/220556107-Slack-%E3%81%8B%E3%82%89%E3%82%A8%E3%82%AF%E3%82%B9%E3%83%9D%E3%83%BC%E3%83%88%E3%81%97%E3%81%9F%E3%83%87%E3%83%BC%E3%82%BF%E3%81%AE%E8%AA%AD%E3%81%BF%E6%96%B9)