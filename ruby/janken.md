# Rubyでじゃんけんゲームを作ってみた

# 1.やりたいこと
- Rubyでじゃんけんゲームを作りたい。
- [「プロを目指す人のためのRuby入門」](https://www.amazon.co.jp/%E3%83%97%E3%83%AD%E3%82%92%E7%9B%AE%E6%8C%87%E3%81%99%E4%BA%BA%E3%81%AE%E3%81%9F%E3%82%81%E3%81%AERuby%E5%85%A5%E9%96%80-%E8%A8%80%E8%AA%9E%E4%BB%95%E6%A7%98%E3%81%8B%E3%82%89%E3%83%86%E3%82%B9%E3%83%88%E9%A7%86%E5%8B%95%E9%96%8B%E7%99%BA%E3%83%BB%E3%83%87%E3%83%90%E3%83%83%E3%82%B0%E6%8A%80%E6%B3%95%E3%81%BE%E3%81%A7-Software-Design-plus%E3%82%B7%E3%83%AA%E3%83%BC%E3%82%BA/dp/4774193976)で、クラスの概念を学習したばかりなので、クラスを使いながら書いてみた。
# 2.やったこと
- コードを書いてみたものの、１つの処理が長いので、もう少し処理を分けて記述したほうが見やすいかもしれない。
- Rubyの配列やハッシュ、メソッドをうまく使いこなせれば、もっとスマートな書き方ができそう。
```Ruby:main.rb
class RockPaperScissors
  def fight(game_count)
    win = 0
    lose = 0
    one_more =0

    game_count.times do |n|
      # あいこだった場合、one_more = 1となっている
      if one_more == 1
        puts "あいこで...(press g or c or p)" 
        one_more = 0
      else
        puts "じゃんけん…(press g or c or p)"
      end

      shoots = { 'g' => 'グー', 'c' => 'チョキ', 'p' => 'パー' }
      
      my_shoot = gets.chomp!
      # CPUはランダムでピックアップ
      cpu_shoot = ['g', 'c' ,'p'].sample
      
      puts "CPU…#{shoots[cpu_shoot]}"
      puts "あなた…#{shoots[my_shoot]}"

      if my_shoot == 'g' &&  cpu_shoot == 'c' || my_shoot == 'c' && cpu_shoot == 'p' || my_shoot == 'p' && cpu_shoot == 'g' 
        win += 1
        puts '勝ち!'
      elsif my_shoot == 'g' && cpu_shoot == 'p' || my_shoot == 'c' && cpu_shoot == 'g' || my_shoot == 'p' && cpu_shoot == 'c'
        lose += 1
        puts '負け!'
      else
        # あいこだった場合、one_moreフラグを立てる
        one_more = 1
        # 繰り返し処理をやり直す
        redo unless one_more == 0
      end
      puts "#{win}勝#{lose}敗"
    end

    puts "結果"
    if win > lose
      puts "#{win}勝#{lose}敗であなたの勝ち"
    else
      puts "#{win}勝#{lose}敗であなたの負け"
    end

  end
end

rock_paper_scissors = RockPaperScissors.new

puts "何本勝負？(press 1 or 3 or 5)"   
# 押されたキーを取得
game_count = gets.chomp!.to_i

if game_count != 1 && game_count != 3 && game_count != 5
  return puts 'もう一度実行し直してください'
else
  puts "#{game_count}本勝負を選びました"
end

rock_paper_scissors.fight(game_count)
```
```Ruby:ターミナルでの実行結果
❯ ruby main.rb
main.rb: warning: argument of top-level return is ignored
何本勝負？(press 1 or 3 or 5)
3
3本勝負を選びました
じゃんけん…(press g or c or p)
g
CPU…グー
あなた…グー
あいこで...(press g or c or p)
g
CPU…パー
あなた…グー
負け!
0勝1敗
じゃんけん…(press g or c or p)
p
CPU…グー
あなた…パー
勝ち!
1勝1敗
じゃんけん…(press g or c or p)
g
CPU…チョキ
あなた…グー
勝ち!
2勝1敗
結果
2勝1敗であなたの勝ち
```
