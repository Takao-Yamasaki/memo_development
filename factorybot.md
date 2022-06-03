# FactoryBotで複数インスタンスを作成する
- 今回は、`FactoryBot`の'create_list`を使用して、大量のテストデータを生成して、ページネーションできるか検証したい。

## 実施手順

- 前提として、ファクトリに以下のような定義が必要
```
# spec/factories/posts.rb

FactoryBot.define do
  factory :post do
    body { Faker::Lorem.sentence }
    user
    after(:build) do |post|
      post.images.attach(io: File.open('spec/fixtures/dummy.jpeg'), filename: 'dummy.jpeg', content_type: 'image/jpeg')
    end
  end
end
```

- まずは、webコンテナに入る
```
$ docker-compose exec web bash
```

- `rails console`でコンソールを起動
```
# rails console
```

- コンソールでFactoryBotを使用するには、以下を実施する
```
$ require 'factory_bot_rails'
include FactoryBot::Syntax::Methods
```

- その上で、以下を実施して、テストデータを生成
```
$ posts = create_list(:post, 35)
```

- 'require 'factory_bot_rails'だけ実施すると、以下のようなエラーが表示されるので、注意が必要
```
(irb):2:in `<main>': undefined method `create_list' for main:Object (NoMethodError)
post_list = create_list(:post, 35)
            ^^^^^^^^^^^
```

## 参考
- [[Rails] Rspecでcreate_listを使って複数のインスタンスを作成する方法](https://osamudaira.com/331/)
- [Module: FactoryBot::Syntax::Methods](https://www.rubydoc.info/gems/factory_bot/FactoryBot/Syntax/Methods#create_list-instance_method)
- [【FactoryBot】create_listで複数のインスタンスの配列を作成する【RSpec】](https://qiita.com/kikikikimorimori/items/5e91fbd7e088f3b9d4a4)
