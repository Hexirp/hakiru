# issue 2

## comment 2026-10-14 1

issue の番号の形式を確定する。 issue の番号の形式にはガンマ符号の変種を使う。通常の十進法では番号の順序とファイルの名前の辞書式順序が一致せず、アルファ符号は長くなりすぎ、デルタ符号はオーバースペックすぎる。

* `issue-1-1.md`
* `issue-1-2.md`
* …
* `issue-1-9.md`
* `issue-2-10.md`
* `issue-2-11.md`
* …
* `issue-2-99.md`
* `issue-3-100.md`
* `issue-3-101.md`
* …
* `issue-8-99999999.md`
* `issue-9-100000000.md`
* `issue-9-100000001.md`
* …
* `issue-9-999999999.md`

issue の番号の形式を遵守していることを検証するプログラムを Haskell で作る。 shebang を使って容易に実行できるようにする。それぞれのファイルの名前に対してパースする。パーサーコンビネーターのモナドを組み合わせて作る。一番目の数値のパース結果に応じて二番目の数値のパーサーが変化する。

"[Monadic profunctors for bidirectional programming](https://blog.poisson.chat/posts/2017-01-01-monadic-profunctors.html)" (Lysxia, 2007) にあるようなパーサーとプリンターをペアにしたモナドを使う。私のポスト（[リンク](https://x.com/hexirp_prixeh/status/2045762101885677996)）でも紹介しているテクニックである。

## comment 2026-10-14 2

ガンマ符号の変種を用いると可能な番号の数は `10 ^ 10` となり、デルタ符号の変種を用いると可能な番号の数は `10 ^ 10 ^ 10` となる。

デルタ符号の変種は UUID や IPv6 や SHA-256 などを 0 と 1 の数字だけを用いて埋め込めるほどのスペックを持つ。つまり 2 進法を 0 と 1 だけの数字を使う 10 進法にするのである。たとえば SHA-256 を埋め込んだ時のファイルの名前は 1 桁分のタグをつけて `issue-3-257-1{{ binary-id }}.md` となる。通常の数値として扱うと `2 ^ 256` 未満の数値は 10 進法の最大 78 桁である。

0 と 1 の数字だけを使うと長くなるので、より大きい桁で分割することで短くすることができるかどうかを考えてみる。 256 桁を綺麗に分割することを考えると 1, 2, 4, 16, 32, 64, 128, 256 というように `256` の約数を桁数として分割することになる。

```txt
10 ^  0 < 2 ^   1 < 10 ^  1
10 ^  0 < 2 ^   2 < 10 ^  1
10 ^  1 < 2 ^   4 < 10 ^  2
10 ^  2 < 2 ^   8 < 10 ^  3
10 ^  4 < 2 ^  16 < 10 ^  5
10 ^  9 < 2 ^  32 < 10 ^ 10
10 ^ 19 < 2 ^  64 < 10 ^ 20
10 ^ 38 < 2 ^ 128 < 10 ^ 39
10 ^ 77 < 2 ^ 256 < 10 ^ 78
```

0, 1, 2, …, 15 を 10 進法の 2 桁で表すと `2 ^ 256` 未満の数値は 10 進法の最大 128 桁の数値になる。 0, 1 を 10 進法の 1 桁で表すと `2 ^ 256` 未満の数値は 10 進法の最大 256 桁の数値になるため、半分の長さに短縮することができている。（二〇二六年一〇月一八日に追記する：四進法を使えば一六進法よりも低い複雑度でありながら同じ効率となる。）

ここまで 10 進法の場合について考察していたが a, b, c, d, e, f の使用を解禁することで 16 進法に変更することもできる。 16 進法を用いたガンマ符号の変種とデルタ符号の変種で表現できる数値のサイズは次の通りである。

* ガンマ符号の変種 - `2 ^ 64` - `16 ^ 16 = 2 ^ (4 * 16) = 2 ^ 64`
* デルタ符号の変種 - `2 ^ 2 ^ 66` - `16 ^ 16 ^ 16 = 2 ^ (4 * 2 ^ (4 * 16)) = 2 ^ 2 ^ (2 + 64) = 2 ^ 2 ^ 66`

ガンマ符号とデルタ符号は増加速度の違いがあるけど、今回の変種においては差が現れる前に最初の数値が 1 桁に収まらなくなるという上限に到達するので気にする必要がない。

漸近的に最適であるという理由で様々な欠点を無視してデルタ符号を使うのは流石にマニアックすぎるか。

## comment 2026-10-14 3

ガンマ符号の変種とデルタ符号の変種は、例えば `2-10-1000000000` と書いた時に 2, 10, 1000000000 と増えていくように単調増加だけど、オメガ符号の変種は違う。

## comment 2026-10-15 1

双方向パーサーは、ようするに、パーサーとプリンターのペアであるというわけで。

MediaWiki の Parsoid もそれっぼい。

```hs
type Parser a = String -> (a, String)
type Printer a = a -> String
```

`Parser` 型は covariant functor となる。 `Printer` 型は contravariant functor となる。

しかし、この二つをセットにしようとすると、にっちもさっちもいかなくなってしまう。

```hs
type PP a = (Parser a, Printer a)
```

a は invariant となってしまう。

```hs
type PP a b = (Parser b, Printer a)
```

このようにすると profunctor となる。

```hs
type Printer a = a -> Writer String ()

type PP a b = (Parser b, Printer a)
```

こうすると `PP` 型がモナドになれるようになる。（二〇二六年一〇月一九日に追記する：間違っている。）

スペースリークするから `Writer` 型を `Strict` 版にするべき？

今は `CPS` 版というのがあるらしいんだけど、その正体が全く分からない。コードを読んでも別に継続モナドは使っていないみたいだし。

"[Writerを使ってはならない](https://qiita.com/ruicc/items/28b1cd2df215ce724e26)" (ruicc, 2022) と "[Writerを使いたいけどスペースリークが怖い](https://www.ncaq.net/2022/05/19/19/10/58/)" (ncaq, 2022) に目を通したけど分かんないな。

`Writer` 型を `State` 型で表現すると存在形式の `Lens` 型と同じになる。

```hs
type PP a b = (String -> (a, String), a -> String -> ((), String))

type PP a b = (String -> (a, String), (a, String) -> String)
```

ようするに、 `String` を `(a, String)` だと解釈しているというわけだ。

`String` は、カーソル位置を保存する型などに取り替えることができる。

あれ、でも、パース失敗の表現は？

`Prism` 型になるよなあ。つまり `(String -> Either a String, Either a String -> String)` に？

どっちなんだろう？

`Prism` 型と `Lens` 型の組み合わせ？

そもそも `PP` 型において失敗は考慮されていない。

どうやって失敗の作用を追加する？　`String -> Either e (a, String)` か？　`String -> (Either e a, String)` か？

失敗して消費したまま終わるなら、それは実質的に別の種類の成功と言えるかもしれない。

`(<|>)` により組み合わせることができる。そのやり方には、かなり自由度があって、ライブラリも多様である。

`String -> [(a, String)]` とかね。

`parsec` ライブラリだと消費ありなしと成功失敗の直積で四種類があった。

Elm の公式ライブラリだと、成功失敗の二種類があって、バックトラック可能かどうかが `Bool` で保存されていた。

失敗を扱う方法を考えるためには `Parser` モナドの基本操作を考えないといけなさそうである。内包的なアプローチでは選択肢が多すぎて迷うから、外延的なアプローチから絞り込んでいく。

`Free` モナドで表現できる？

`char` 関数という一文字だけ読んでマッチするかどうか確かめる操作が基本になるのだろうか？

いや、 `char` 関数は最初の文字を取り出す `pop` 関数と一致しているかどうかの分岐に分解できる。

`pop` 関数を基本操作とするべきだろうか？

そうなるとバックトラックが表現できなくなるのでは？

失敗した時のバックトラックは、失敗した後に元通りに `push` するという規約で表現できる。

それらを `withFile` みたいなイディオムで重ねていく。

継続モナド。後で何かするというのは継続モナドで表現できる。

そもそもバックトラックという時点で継続モナドなんだわ。気づいていなかったんだけど。

`Codensity` モナドで性能よく継続と `Free` の作用を加える。

`pop` 関数は失敗しうる。空の文字列もあり得る。リストの構造に沿って `String` と `Either () (Char, String)` を同一視する。

結局は `Prism` 型と `Lens` 型の組み合わせであるというわけだ。

`pop :: m Char` は `pop :: m (Maybe Char)` と `exit :: m Void` に分解できる。

エラーには検討の余地がある。

なんであるにせよ、 `s -> (Maybe a, s)` という形でなければ、 `pop` した時に、後でエラーになった時に `push` し戻すという動きができない。

なぜ、通常の `Parser` モナドは継続がないのにバックトラックできるのだろうか？

`(<|>)` に制御されているから。

"[Monadic profunctors for bidirectional programming](https://blog.poisson.chat/posts/2017-01-01-monadic-profunctors.html)" (Lysxia, 2007) を読み進める。

```hs
type Lens s x a = (s -> a, x -> (s -> s, a))
```

こんなのが登場しているんだけど、これはいったい？

```hs
type Lens s t a b = exists x. (s -> (a, x), (b, x) -> t)
```

これが普通のフォーマットなんだけど、これに従っていない。

`Lens s (s -> s, a) a x` ということか？

存在量化された変数 `x` が普通の変数 `s` と等しいことを前提にしてしまっている。

```hs
type Lens s t a b = exists x. (s -> (a, x), b -> x -> t)
```

このように変形してみる。

`x := s` と `t := s` を前提にして……

```hs
type Lens s t a b = (s -> (a, s), b -> s -> s)
```

`(b, x)` から `t` を作れる。 `t := s` であれば `s` から `(a, x)` を作れるので `(b, x)` から `(a, x)` を作れる。 `a` が `x` に依存しなくなるのが大きな違いになっている。

でも、こうしないとモナドにならないって。（二〇二六年一〇月一九日に追記する：そんなことは元の記事には書いていない。）

"[Existential optics](https://www.reddit.com/r/haskell/comments/ujmjx0/existential_optics/)" (Reddit, 2022) や "[Existential Optics](https://www.tweag.io/blog/2022-05-05-existential-optics/)" (Marco Perone, 2022) や "[Traversals 1: van Laarhoven and Existentials](https://bartoszmilewski.com/2021/04/01/traversals-1-van-laarhoven-and-existentials/)" (Bartosz Milewski, 2021) や "[Profunctor Optics を見渡す](https://zenn.dev/yvvakimoto/articles/498531610198da)" (yvvakimoto, 2024) などを読んだ。

```hs
type Lens s t a b = exists x. (s -> (a, x), (b, x) -> t)
```

これをモナドにする方法という観点から考えよう。

まず、変性に合わせて変数の順番を入れ替えないと。

```hs
type Lens s t a b = exists x. (s -> (b, x), (a, x) -> t)
```

元の記事の Lens のモナドの実装を見て考えよう。

```hs
pure :: b -> Lens s s a b
pure x = (\s -> (x, s), \(_, s) -> s)

(<*>) :: _
```

`(<*>)` を実装するには存在変数に `(_, _)` を代入する。

indexed monad になるかも？

`Traversal` 型とかは `Affine` 型とか `PowerSeries` 型とかで構成されるらしいけど。

`Affine` 型の定義（[リンク](https://github.com/marcosh/existential-optics/blob/b4727e2ae6119d7039d7575d5eb64fc5ae74dc1f/src/Affine.hs)）と `PowerSeries` 型の定義（[リンク](https://github.com/marcosh/existential-optics/blob/b4727e2ae6119d7039d7575d5eb64fc5ae74dc1f/src/PowerSeries.hs)）を読んだ。

普通の型とは違って `(Type, Type)` カインドの変数が存在量化されることになるのね。

あと、 `PowerSeries` 型の定義にある `n` という変数って意味がないような？

うーん、 `Vect` 型の定義（[リンク](https://github.com/marcosh/existential-optics/blob/b4727e2ae6119d7039d7575d5eb64fc5ae74dc1f/src/Vect.hs)）を見ても、暗黙変数のような痕跡はない。

純粋なミスかな？
