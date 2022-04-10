# Rubyでバブルソート問題をやってみた

# 1.バブルソートとは？
- 大小バラバラに並んでいるデータを昇順（小さい順）、降順（大きい順）に並べる方法のひとつ。
- 具体的には、隣り合う要素の大小比較を繰り返して、並び替えを行う。

# 2.やりたいこと
- 次のような結果を得たい。
    - 実は、sortメソットを使用すると、一発でソートすることも可能だが、今回は、これを使用せずに記述していきたい。
```Ruby:
# ソート前
numbers = [10, 8, 3, 5, 2, 4, 11, 18, 20, 33]
# ソート後
numbers = [2, 3, 4, 5, 8, 10, 11, 18, 20, 33]
```

# 3.やったこと
- メインの処理はこのように記載してみた。
```Ruby:buble_sort.rb
# bubble_sortメソッドの作成
def bubble_sort(numbers)
  # 要素数の回数分繰り返す
  numbers.size.times do |i|
    # １つの要素に対して、次の要素と大小比較を繰り返す
    # 例えば、numbers[0] = 10が次の要素の値より小さくなるまで入替え処理を繰り返し、内側の繰り返し処理を抜ける
    (numbers.size - 1).times do |j|
      # numbers[j]がnumbers[j+1]より大きければ、numbers[j]とnumbers[j+1]の値を入れ替える
      numbers[j], numbers[j + 1] = numbers[j + 1], numbers[j] if numbers[j] > numbers[j + 1]
    end
  end
  numbers
end

# bubble_sortメソッドの呼び出し
p bubble_sort([10, 8, 3, 5, 2, 4, 11, 18, 20, 33])
```
- エラーが出ないかどうかテストも書いてみた。

```Ruby:bubble_sort_test.rb
require 'minitest/autorun'
require_relative 'bubble_sort'

class BubbleSortTest < Minitest::Test
  def test_bubble_sort
    assert_equal [2, 3, 4, 5, 8, 10, 11, 18, 20, 33], bubble_sort([10, 8, 3, 5, 2, 4, 11, 18, 20, 33])
  end
end
```
# 4.参考
- [可視化するとまるで泡？のような並び替えアルゴリズム【バブルソート】 - YouTube](https://www.youtube.com/watch?v=IHFBb0wYBR0)
- [Rubyでバブルソート書いてみた](https://qiita.com/RyumaRyama/items/6753bf32d435f74690be)
