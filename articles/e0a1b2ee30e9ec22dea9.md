---
title: "vim バッファ入門"
emoji: "📖"
type: "tech"
topics:
  - "vim"
published: true
published_at: "2020-09-19 22:54"
---

# 概要

vimを日常的に使っておきながら、**バッファの仕組みをロクに使わずに、1ファイル1プロセスで編集したまま長い時間が経過してしまった人** に贈る、バッファ機能の基本的な使い方を紹介。

`iTerm2` を始めとするターミナルツールで画面分割をすることで、複数ファイルの編集を容易に行うことも出来るが、バッファを使った複数ファイル操作に慣れることで、関連するvimの機能やvimプラグインをさらに活用することができるので、使う使わないは一旦考えずにやってみよう。

# とりあえず複数ファイル開こうや

作業用ディレクトリに以下の3つのファイルを作る。内容は `Ruby` のコードだが、コードそのものはまったく関係ないので読む必要はない。

ひとつ(`a.rb`)

```ruby
file = 'this is A'
```

ふたつ(`b.rb`)

```ruby
file = 'this is B'
```

みっつ(`c.rb`)

```ruby
file = 'this is C'
```

3つのファイルがあるディレクトリで、以下のコマンドを実行しよう。３つのファイル全てが１つのvimプロセスで読み込まれる。

```bash
$ vim *.rb
```

vimではメモリに展開したファイルの内容を`バッファ`と呼ぶ。

vimプロセス上で、どんなバッファが読み込まれているかは`:ls`コマンドで確認できる。

```vim
:ls
  1 %a   "a.rb"                         line 1
  2      "b.rb"                         line 0
  3      "c.rb"                         line 0
Press ENTER or type command to continue
```

# 画面に表示するバッファを切り替えようや

vimプロセスを起動した段階では、対象の3ファイルのうち先頭のファイル(ここでは`a.rb`)が画面上に表示されている。

```ruby
file = 'this is A'
```

これを`b.rb`や、`c.rb`に切り替えたい。そんなときに使うコマンドが、`:bprev` `:bnext` `:bfirst` `:blast` だ。

|コマンド|効果|
|-----|----|
|:bprev|1つ前のバッファに切り替え|
|:bnext|1つ後のバッファに切り替え|
|:bfirst|先頭のバッファに切り替え|
|:blast|末尾のバッファに切り替え|


`a.rb`を開いてる状態で、`b.rb`にバッファを切り替えよう。

```vim
:bnext
```

`b.rb`に切り替わった。

```ruby
file = 'this is B'
```

同様に`:bnext`を再度実行すると`c.rb`が開かれ、続けて`:bprev`を実行すると、また`b.rb`に帰ってくる。

`:bfirst`を実行すると常に先頭の`a.rb`に、`:blast`を実行すると常に末尾の`c.rb`に切り替わることは想像に難くないだろう。

# え？切り替えめんどくさくない？

**誰だってそう思う。** 少ないタイプ数で高度な処理をこなせることに定評のあるvimで、何が楽しくて`:bnext`だの長ったらしいコマンドを打ち込まなければならないのか。それもバッファを切り替えるたびに。

ということで、大抵のvimmerはそれらのコマンドにキーバインドを割り当てる。流行りは`[b`とからしいが、ここでは私が採用したキーバインドの例をあげる。もちろんキーバインドにはvimmerそれぞれに拘りがあると思うので使いやすければ何でも良い。

```vim
nnoremap <silent> <C-j> :bprev<CR>
nnoremap <silent> <C-k> :bnext<CR>
```

`<C-j>` と `<C-k>` は、標準で割と空いてるのに、ホームポジション的に死ぬほど使いやすいので、他に頻出の操作を割あることがない場合はこれが良いだろう。

これでバッファの切替がメタクソに早くなった。`:bfirst`と`:blast`は、個人的にはそんな使わない気がするので見送り。使うのであれば必要に応じて他のキーバインドを探す。

# バッファたくさん開いたときはどうするのさ

なんやかんやでバッファリストが以下のようになったよしよう。

```vim
:ls
  1 %a   "a.rb"                         line 1
  2      "b.rb"                         line 0
  3      "c.rb"                         line 0
  4      "d.rb"                         line 0
  5      "e.rb"                         line 0
  6      "f.rb"                         line 0
  7      "g.rb"                         line 0
  8      "h.rb"                         line 0
  9      "i.rb"                         line 0
 10      "j.rb"                         line 0
 11      "k.rb"                         line 0
Press ENTER or type command to continue
```

**おいおい、俺は`g.rb`を開きたいんだが、まさか`:bnext`を6回叩かせる気かい？**
(と言っても`<C-k>`にバインドしていれば`6<C-k>`で済むが)

そんなときは`:ls`で出てきたバッファリストに目を通そう。左側にバッファごとに数字(バッファ番号)が振られているのがわかるだろう？

`:b N`コマンドでは、バッファ番号を指定してバッファを開くことができる。よって、`g.rb`を開きたい場合はこうだ。

```vim
:b 7
```

一発でバッファ番号７が割り当てられた`g.rb`に移動できる。ステキ！

さらに、ファイル名がわかっていて短いなら以下のほうが早い。

```vim
:b g.rb
```

ちなみにファイル名の入力は`Tab`でちゃんと補完されるので、そんなにバッファが多くないなら`:b`からのTab何度かが一番早いかも。

# ファイル編集中にバッファ切り替えられないぞオイ

`a.rb` を適当に編集して保存せずに

```ruby
file = 'this is A!!!!!!!!'
```

おっと、`b.rb`もついでに編集しないとな、`:bnext`で切り替えっと

```vim
E37: No write since last change (add ! to override)
```

**は？(激怒)**

どうやらvimでは、バッファを保存せずに閉じると警告が出るようだ。よくある「保存してないですがよろしいですか？」的なヤツ。

役立つときは役立つ警告だけど、サクサクバッファを切り替えたいときにこれが邪魔になりかねない。そういう場合は`.vimrc`に以下を追記することで、バッファを保存せずに切り替えることを許可できる。

```vim
:set hidden
```

これでよりサクサクバッファを切り替えられるようになる。

ただし、保存漏れによる事故も起こりやすくなるので、保存を忘れないような工夫が大事。

# 不要になったバッファを消そう

`c.rb`を削除しよう。削除すると言ってもファイルを削除するわけじゃく、バッファを削除する(≒編集を終了する)には、以下の`:bd N`を使う。Nにはバッファ番号を指定する。
(現在開いているバッファを消すだけなら単に`:bd`でも可)

`c.rb`のバッファ番号が3であるとき

```vim
:bd 3
```

`:ls`で見ると、`c.rb`が消えているのがわかる。

```vim
:ls
  1  h   "a.rb"                         line 2
  2 %a   "b.rb"                         line 1
  4  h   "d.rb"                         line 1
  5  h   "e.rb"                         line 1
  6  h   "f.rb"                         line 1
  7  h   "g.rb"                         line 1
  8  h   "h.rb"                         line 1
  9  h   "i.rb"                         line 1
 10  h   "j.rb"                         line 1
 11  h   "k.rb"                         line 1
Press ENTER or type command to continue
```

**が、実は`c.rb`のバッファは消えていないのだ。**`:bd`はバッファリストから消すコマンドである。(正確にはバッファに関連したオプション、変数、マッピングなどが消える)

事実、バッファリストから消したバッファも含めてた真のバッファリストを表示する`:ls!`を実行すると`c.rb`がしぶとく生き残っていることがわかる。

```vim
:ls!
  1  h   "a.rb"                         line 2
  2 %a   "b.rb"                         line 1
  3u#    "c.rb"                         line 1
  4  h   "d.rb"                         line 1
  5  h   "e.rb"                         line 1
  6  h   "f.rb"                         line 1
  7  h   "g.rb"                         line 1
  8  h   "h.rb"                         line 1
  9  h   "i.rb"                         line 1
 10  h   "j.rb"                         line 1
 11  h   "k.rb"                         line 1
Press ENTER or type command to continue
```

あとかたもなく消したいなら`:bw [N]`を使おう。

```vim
:bw 3
```

`:ls!`を使っても表示されないことがわかる。

```vim
:ls!
  1  h   "a.rb"                         line 2
  2 %a   "b.rb"                         line 1
  4  h   "d.rb"                         line 1
  5  h   "e.rb"                         line 1
  6  h   "f.rb"                         line 1
  7  h   "g.rb"                         line 1
  8  h   "h.rb"                         line 1
  9  h   "i.rb"                         line 1
 10  h   "j.rb"                         line 1
 11  h   "k.rb"                         line 1
Press ENTER or type command to continue
```

**言うてバッファ削除ってそんな使う？**

# バッファにファイルを追加したい

`a.rb`しか開いていない状態で

```vim
:ls
  1 %a   "a.rb"                         line 1
Press ENTER or type command to continue
```

`b.rb`を開きたい場合は`:e [パス]`コマンドを実行しよう。

```vim
:e b.rb
```

`b.rb`がバッファに読み込まれ、かつアクティブな状態になっている。

```vim
:ls
  1 #h   "a.rb"                         line 1
  2 %a   "b.rb"                         line 1
Press ENTER or type command to continue
```

この辺は無理せず**NERDTree**のようなファイルツリープラグインを入れたほうが幸せになるぞ！

# 参考
- [実践Vim 思考のスピードで編集しよう!](https://www.amazon.co.jp/%E5%AE%9F%E8%B7%B5Vim-%E6%80%9D%E8%80%83%E3%81%AE%E3%82%B9%E3%83%94%E3%83%BC%E3%83%89%E3%81%A7%E7%B7%A8%E9%9B%86%E3%81%97%E3%82%88%E3%81%86-Drew-Neil/dp/4048916599)
- [vimのnormal modeでctrlキーを使うキーマップ一覧 - Qiita](https://qiita.com/34ro/items/e20fa0831d78566981d3)
- [vimを瞬時に最強エディタに変えるbコマンド - Qiita](
https://qiita.com/qtamaki/items/4da4ead3f2f9a525591a)
- [vim 簡単ガイド - Qiita](https://qiita.com/FBH9999/items/a8ec99bed08592a9d69a)
- [vim-plugin NERDTree で開発効率をアップする！ - Qiita](https://qiita.com/zwirky/items/0209579a635b4f9c95ee)
