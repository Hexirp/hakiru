# issue 2

issue ディレクトリの元に `issue-*.md` というファイルを配置して issue を管理する。 issue のファイルの名前において issue の番号を表示する時には、 issue の番号の順序と issue のファイルの名前の辞書式順序を一致させるために、デルタ符号の変種を用いる。

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

0 と 1 の数字だけを使うと長くなるので、より大きい桁で分割することで短くすることができるかどうかを考えてみる。 256 桁を綺麗に分割することを考えると 1, 2, 4, 16, 32, 64, 128, 256 というように 256 の約数を桁数として分割することになる。

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

スペースリークするから `Writer` 型を `Strict` 版（[リンク](https://hackage-content.haskell.org/package/transformers-0.6.3.0/docs/Control-Monad-Trans-Writer-Strict.html)）にするべき？

今は `CPS` 版（[リンク](https://hackage-content.haskell.org/package/transformers-0.6.3.0/docs/Control-Monad-Trans-Writer-CPS.html)）というのがあるらしいんだけど、その正体が全く分からない。コードを読んでも別に継続モナドは使っていないみたいだし。

"[Writerを使ってはならない](https://qiita.com/ruicc/items/28b1cd2df215ce724e26)" (ruicc, 2022) と "[Writerを使いたいけどスペースリークが怖い](https://www.ncaq.net/2022/05/19/19/10/58/)" (ncaq, 2022) に目を通したけど分かんないな。

`Writer` 型を `State` 型で表現すると存在形式の lens と同じになる。

```hs
type PP a b = (String -> (a, String), a -> String -> ((), String))

type PP a b = (String -> (a, String), (a, String) -> String)
```

ようするに、 `String` を `(a, String)` だと解釈しているというわけだ。

`String` は、カーソル位置を保存する型などに取り替えることができる。

あれ、でも、パース失敗の表現は？

prism になるよなあ。つまり `(String -> Either a String, Either a String -> String)` に？

どっちなんだろう？

prism と lens の組み合わせ？

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

失敗した時のバックトラックは、失敗した後に元通りに push するという規約で表現できる。

それらを `withFile` みたいなイディオムで重ねていく。

継続モナド。後で何かするというのは継続モナドで表現できる。

そもそもバックトラックという時点で継続モナドなんだわ。気づいていなかったんだけど。

`Codensity` モナド（[リンク](https://hackage-content.haskell.org/package/kan-extensions-5.2.8/docs/Control-Monad-Codensity.html)）で性能よく継続と `Free` の作用を加える。

`pop` 関数は失敗しうる。空の文字列もあり得る。リストの構造に沿って `String` と `Either () (Char, String)` を同一視する。

結局は prism と lens の組み合わせであるというわけだ。

`pop :: m Char` は `pop :: m (Maybe Char)` と `exit :: m Void` に分解できる。

エラーには検討の余地がある。

なんであるにせよ、 `s -> (Maybe a, s)` という形でなければ、 pop した時に、後でエラーになった時に push し戻すという動きができない。

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

元の記事の lens のモナドの実装を見て考えよう。

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

## comment 2026-10-16 1

`Parser` モナドのバックトラックを `Alternative` 型クラスではなく継続モナドにより実装する。

`String -> [(a, String)]` のような構造は、パースエラーの記録を残せないという時点で、大きな欠陥を持っている。

パースエラーは、よく起こり、そのまま終了したり、バックトラックしたりする。

パースエラーの記録がリスト状に連なっていて、リストの最後が「成功」だったら成功していることで「失敗」だったら失敗していることになるように。

あまり「成功」と「失敗」を特別扱いしない。

`Writer` モナドを混ぜることで実現できる？

`forall r. s -> (a -> s -> m r) -> m r -> m r` において `callCC` 関数を実装できるのか？

リストモナドは合成が難しい。変換子にならない。

ストリームで代替しないと。

## comment 2026-10-16 2

```hs
type Parser s a b = (State s b, a -> Writer s b)
```

プリンターに該当する部分はクライスリ圏としてモナドになる。

```hs
type Parser s a b = (State s b, a -> Writer s ())
```

これは `Writer s` と `Proxy` の合成が交換可能ではないため（つまり `Proxy` がモナド変換子ではないため）モナドになれない。

```hs
type F a = (G a, H a)
```

`G` と `H` がモナドである時に `F` がモナドになれるかどうか。

"[Monadic profunctors for bidirectional programming](https://www.reddit.com/r/haskell/comments/8kkmqq/monadic_profunctors_for_bidirectional_programming/)" (Reddit, 2018) を読んだ。

実例が Reddit のコメントにある。

[chshersh](https://www.reddit.com/user/chshersh/) さんが "[chpatrick/codec](https://github.com/chpatrick/codec)" を紹介する投稿（[リンク](https://www.reddit.com/r/haskell/comments/8kkmqq/comment/dzt8pqc/)）をしている。

[kowainik](https://www.reddit.com/user/kowainik/) さんが "[kowainik/tomland](https://github.com/kowainik/tomland)" を紹介する投稿（[リンク](https://www.reddit.com/r/haskell/comments/8kkmqq/comment/dza8xkv/)）をしている。

"[kowainik/tomland](https://github.com/kowainik/tomland)" の実装におけるパーサーの部分を読む。（[リンク](https://github.com/kowainik/tomland/blob/4d71d4a6f8026f9d4b4c909799b2493e6aa60e7b/src/Toml/Codec/Types.hs#L154-L156)）

`s -> (Maybe a, s)` タイプなのね。

「やはりか」という気持ちと自分で辿り着けなくて悔しいという気持ちが同居している。

push するバックトラックとか、状態を残すことが前提のやつは考えていたけど。

lens のままにするには `Maybe` 型を使うしかないからなあ。

あと、状態を残さないとプリンターの時にうまく書き戻せないというか、逆関数にならないっぽいのもあったな。

いや、ちょっと待って。

これって、プリンターの方じゃん？

`TomlEnv` 型の方がパーサーだよ。

勘違いしていた。

これ（[リンク](https://github.com/kowainik/tomland/blob/4d71d4a6f8026f9d4b4c909799b2493e6aa60e7b/src/Toml/Codec/Types.hs#L50)）やね。

なんかよくわからん。

"[chpatrick/codec](https://github.com/chpatrick/codec)" の方を読む。

定義（[リンク](https://github.com/chpatrick/codec/blob/0faf37a1cefb21dc46cc2857e538dd983bb28652/src/Control/Monad/Codec.hs#L17-L20)）を見つけた。

```hs
data CodecFor r w c a = Codec
  { codecIn :: r a
  , codecOut :: c -> w a
  } deriving (Functor)
```

なんて単純なんだ。

`Monad` 型クラスのインスタンスの定義（[リンク](https://github.com/chpatrick/codec/blob/0faf37a1cefb21dc46cc2857e538dd983bb28652/src/Control/Monad/Codec.hs#L34-L39)）を見つけた。

```hs
instance (Monad r, Monad w) => Monad (CodecFor r w c) where
  return = pure
  m >>= f = Codec
    { codecIn = codecIn m >>= \x -> codecIn (f x)
    , codecOut = \c -> codecOut m c >>= \x -> codecOut (f x) c
    }
```

なんて単純なんだ……！

```hs
type Codec r w a b = (a -> w b, r b)
```

順番という観点から見ると、プリンターを前に置いた方が良いかな？

プリンターの実装の方が簡単というのもあるし。

```hs
type F a = (G a, H a)
```

このモナドの件だけど `(:*:)` という型（[リンク](https://hackage-content.haskell.org/package/ghc-internal-9.1401.0/docs/src/GHC.Internal.Generics.html#line-976)）がしっかりあった。

issue の番号の符号化の件だけど。

`issue-10-1000000000` を超えると辞書式順序になるのは終わりだと思っていたけど `issue-99-1000000000` とすれば続けられることに気付いた。 `999...` というアルファ符号の変種を使う。

いや、もっと詰め込める。

`issue-91-1000000000` とすればいい。

`9` を `n / 9` だけ並べる。その後に `n % 9` をつける。

いや、一つだけずれているから、もう少し考えないといけない。

`` n `mod` 9 == 0 `` のとき `9` の数を一つ減らして余りを `9` とする。

桁数が `(9 / 10) * n` となる。

ガンマ符号の変種だけではなくデルタ符号の変種にも同じことができる。アルファ符号の変種にも。

このようにして延長した場合、ガンマ版よりデルタ版の方が短くなる最小の `n` はいくつだろうか？

とんでもなく大きくなりそうなのは分かる。

これでわずかにだけあった極めて巨大な数値を表現できなくなる上限があるという問題が解決されたので、ガンマ版が最良ということにしてもよさそう。

ライブラリとするなら `issue` をつけていない `3-123` のように切り出そう。

Lean 4 で証明したいんだけど、そうすると環境構築が膨大になるかもしれない。

文字列としてのみならず、数列としても辞書式順序になってほしい。

`"1-1" < "10-10" < "11-100" < ... < "2-10000000000"` みたいなものは、文字列として辞書式順序になっていてもダメである。

ようするに、正の自然数から正の自然数のリストに変換する関数である。

`PositiveInteger` 型を新しく定義する。お馴染みの `newtype` 宣言による制約。

`f : PositiveInteger -> [PositiveInteger]` が本体というわけで。

`x < y` ならば `f x < f y` なのは当然で。

`` (<) `on` (intercalate "-" . showPositiveInteger) `` においても単調増加になる。

`encode : Proxy a -> PositiveInteger -> [PositiveInteger]` を型クラスにする。本体に型が現れないので `Proxy` 型を使用する。

`PositiveInteger -> a` と `a -> [PositiveInteger]` に分けようと思ったけど、めんどうくさいよね。

でも、やっぱり分けた方が良いかな？

さらに追加の条件として `last (encode _ n) == n` もつける。

`NonEmpty` 型の方が良いな。

アルファ版を実装する時には `replicateWithTail` が欲しくなる。

```hs
replicateWithTail :: Int -> a -> [a] -> [a]
replicateWithTail 0 x s = s
replicateWithTail n x s = replicateWithTail (n - 1) x (x : s)
```

リスト系の関数が裏で `build` 関数を使って fusion しているのはすごいよね。ただ、私は `foldl` ライブラリみたいなアプローチの方が好みなのだけど。

名前はどうしよう？

"easy readable lexicographic encoding" を "ERLE" に。

"very easy readable lexicographic encoding" を "VERLE" に。

"prefix" はつけるか？

prefix code ではなさそう。

"universal" をつけるか？

universal coding ではなさそう。

"decimal" をつけるか？

他の底にも応用できる。

"very easy readable lexicographic coding" を "VERLC" に。

"coding" より "encoding" の方がいいな。

"very easy human-readable and human-writable lexicographic encoding" を "VEHRAHWLE" に。

"very easy human-readable and very easy human-writable lexicographic encoding" を "VEHRAVEHWLE" に。

"writable" と "lexicographic" の間に母音が欲しいな。

"very easy human-readable and human-writable uniquely lexicographic encoding" を "VEHRAHWULE" に。

"wu" の並びは不都合。

"very easy human-readable and human-writable Elias lexicographic encoding" を "VEHRAHWELE" に。

"very easy human-readable and human-writable integer lexicographic encoding" を "VEHRAHWILE" に。

"very easy human-readable and human-writable integer sequence lexicographic encoding" を "VEHRAHWICLE" に。

"very easy human-readable and human-writable integer sequence encoding" を "VEHRAHWICE" に。

"very easy human-readable and human-writable Elias-like lexicographic encoding" を "VEHRAHWELE" に。

"very easy human-readable-and-writable Elias-like lexicographic encoding" を "VEHRAWELE" に。

もう "Vehrawele" でいいや。複合語を分解するかどうかに恣意性があるのは、ご愛敬で。

`1, 2, 3, 9, 90, 91, 92, ...` という風に `0` も使える。

`n / 10` 個の `9` と `n % 10` となって極めて理解しやすくなった。

アルファ符号の変種を実装してみるか？

```hs
alpha :: PositiveInteger -> PositiveInteger
alpha n = case n - 10 of { Nothing -> n; Just n' -> 9 ++ alpha n' }
```

ハイパー数学である。

```hs
alpha :: PositiveInteger -> PositiveInteger -> PositiveInteger
alpha n s = case n - 10 of { Nothing -> 10 * s + n; Just n' -> alpha n' (10 * s + 9) }
```

`(-) :: PositiveInteger -> PositiveInteger -> Maybe PositiveInteger` であるわけだけど `n == 10` の時にうまくいかない。

```hs
alpha :: Word -> Word -> Word
alpha n s = if n < 10 then 10 * s + n else alpha (n - 10) (10 * s + 9)
```

`Word` を使ってみる。

```hs
alpha :: Word -> [Word8]
alpha n = case n - 10 of { Nothing -> [n]; Just n' -> 9 : alpha n' }
```

リストを使ってみる。

```hs
alpha :: Word -> NonEmpty Word8
alpha n = case n - 10 of { Nothing -> singleton n; Just n' -> 9 <| alpha n' }
```

`NonEmpty` 型を使ってみる。

除算を使った方が早いかも？

`f n = g (n / 10) + h (n % 10)` と仮定する。

```txt
g 0 = 0
g 1 = 90
g 2 = 990
g 3 = 9990
```

`g` の計算結果は上記の通りである。

```txt
g n = (10 ^ n - 1) × 10
g n = 10 ^ (n + 1) - 10
```

よって、上記の式で表すことができる。

```txt
f n = 10 ^ (n / 10 + 1) - 10 + n % 10
```

定義をまとめる。

"[第37回　書き換え規則を使って不要な計算や中間データを除去](https://xtech.nikkei.com/it/article/COLUMN/20100112/343099/)" (shelarcy, 2010) を読んだ。

うーん？

リストを使うと `a ^ (n * 2) = (a ^ n) ^ 2` を使った最適化ができない。

```txt
f n = (10 ^ (n / 10) - 1) * 10 + n % 10
```

こういうことで。

見た目がすっきりして、負にならないことも分かりやすい。

あ、途中で現れる数値の大きさが小さいという効果もあるのか。

## comment 2026-10-19 1

ガンマ版の方がデルタ版より良いと考えてきたけど、改めてノートを読み直すと、なんか 2 文字増えるだけで済むんならデルタ版でいいんじゃないかと思えてきた。

ただ、本当に asymptotically optimal になるのかな？

ウィキペディアのオメガ符号の記事では Kraft-McMillan inequality を使って証明するというスケッチがあるけど、どうして証明できていることになるのか分からないんだよなあ。

そもそも prefix code になるのかな？　`2-13-1000000000000` と書くのではなく `2131000000000000` というようにハイフンをなくした場合において。

ウィキペディアのデルタ符号の記事では `00001` みたいなアルファ符号を使っているんだけど、私が定義したデルタ版では `11110` みたいな反転したバージョンを使っているんだよね。

アルファ符号の定義を取り換えた時に prefix code になるのかどうかの質問文を Microsoft Copilot に入力してみたけど、その返答が明らかに矛盾していて、その論理が通るなら `0001` みたいなアルファ符号を使ったバージョンまでも prefix code にならないはずなのに、ゼロから始まる版は prefix code であり 1 から始まる版は prefix code ではないというような返答が返ってきた。

これはおかしいと否定するために prefix code の定義を再確認したけど、やはり私が定義した符号化は prefix code になる。

ついでに、一般の n 進法について asymptotically optimal universal coding を作れるのかを尋ねている質問（[リンク](https://cs.stackexchange.com/questions/141440/asymptotically-optimal-universal-code-in-other-bases)）を見つけた。

ついでに、符号の開始位置が不明である場合に読み取りを間違えたまま連鎖することを防ぐことができる性質を "self-synchronizing" と呼ぶことも知った。ウィキペディアの記事（[リンク](https://en.wikipedia.org/wiki/Self-synchronizing_code)）を読んだ。

Microsoft Copilot の返答の出力は、 prefix code と self-synchronizing code を混同して、後者でないことをもって前者であることを否定していた。

私の入力は "prefix code" にだけ言及していて、 "self-synchronizing code" については名称についても（たとえ名称と結びついていないとしても）性質についても現在においても過去においても全く言及していない。そもそも符号化に関係した質問をするのも初めてである。

ただ、 "self-synchronizing code" という言葉を知らなくても、そうであるかどうかということは気になっていたので、そのことを明確に否定できたのは良かった。私自身も prefix code と self-synchronizing code を混同していて、木構造として考えると prefix code になるはずなのに、途中から読み込もうとしたら失敗するから非 prefix code なんじゃないかという考えもあって、もやもやしていたし。

asymptotically optimal であるというのはどういうことなのか再確認したけど、よく分からない。

とりあえず、なんかの比率が極限において 1 になることらしい？

元の長さを `x` として符号化した後の長さを `y` とする。

ガンマ符号は `y / x` がだいたい `(x + x) / x` だから極限において 2 になる。

デルタ符号は `y / x` がだいたい `(x + log x + log x) / x` だから極限において 1 になる。

こういうことなのかな？

私が定義したデルタ版は asymptotically optimal だけど universal coding ではない。最初の桁が 0 にならないので。 universal coding にするためには、いわゆる Excel 進法である bijective numeration を使わないといけない。 2 進法なら、ケチ表現と同じ原理により最初の桁が 1 であることが確定しているので省略できるけど、より底が大きいと省略できなくなるから。 asymptotically optimal であるのは、元の長さと符号化した後の長さの比が 1 に近づくから。

asymptotically optimal universal coding は self-synchronizing code になれるのだろうか？

Fibonacci coding は self-synchronizing code だけど asymptotically optimal ではない。

Elias delta coding は asymptotically optimal だけど self-synchronizing code ではない。

なんか両立しないように思えてくるんだけど。

Google AI は「両立できる」と出力しているけど、これの出力は信用できないからなあ。

## comment 2026-10-19 2

lens をモナドにする。

indexed monad においては `m i o a` において `i` が反変であり `o` が共変である必要がある。

```hs
type Lens s t a b = forall x. Lens (s -> (x, b), (x, a) -> t)

fmap :: (b -> b') -> Lens s t a b -> Lens s t a b'
fmap f (Lens (ge, se)) = Lens (fmap f . ge, se)

-- ge は get の略である。 se は set の略である。

pure :: b -> Lens s s a b
pure be = Lens (\se -> (se, be), \(se, _) -> se)

-- ついでに別の変数の末尾にも e をつける。

(<*>) :: Lens t u a (b -> b') -> Lens s t a b -> Lens s u a b'
Lens (ge0, se0) (ge1, se1) = Lens (assoc . fmap se0 . se1) _
```

うーん？

"[Glassery](https://oleg.fi/gists/posts/2017-04-18-glassery.html)" (Oleg Grenrus, 2017) を読んだ。

存在量化された変数を明示的に扱える言語で、たとえば Rocq か Lean 4 かでやらないと分かりにくいな。

まあ、それでも手で書けば良い。

いや、これだと lens の合成になる。

`a <*> b` と書いた時に `b` の状態を `a` に引き継がないといけないけど `b` の中で存在量化された変数 `x` と `t` が一致しなければならない。

モナドは作れない。

```hs
type Lens a b c d e f = (a -> (b, c), (d, e) -> f)
```

もうこんな感じにするか？

```hs
type Lens s a b = (s -> (s, a), (s, b) -> s)
```

微妙だな……

```hs
type Codec pr ps a b = (a -> pr b, ps b)
```

汎用性を持っている。

エラーについて `NonEmpty` 型を使うことを想定していたけど、バックトラックすることを考えると木構造の方が良いかもしれない。

パースに成功した場合も複数含めることができるため、多値となる。

```hs
data Result e a = Failure e | Success a | Backtrack (Result e a) (Result e a)
```

バックトラックの情報を含めるためには、継続に対して `Backtrack (Failure e)` を適用することになる。

モナドの `bind` 関数の時は、それぞれの `Success` である葉に対して関数を適用して「接ぎ木」することになる。

ああ、でも、そうなると、状態について微妙なのでは？

```hs
data Result e s a = Failure e | Success a s | Backtrack (Result e a) (Result e a)
```

こんな感じかな？

継続モナドを混ぜにくいけど、継続モナドがないと結合法則が成り立たない。

`s -> [Either e (a, s)]` でいいんちゃう？

`StateT s (ExceptT e [])` という感じで。

そもそも継続モナドでのバックトラックはどうやるのか詳しく覚えていないんだよな。再確認するか？

リストモナドをやめてストリームモナドにした方が、その性能が良くなるんじゃない？

```hs
data TreeF a r = Leaf a | Node (Bool -> r)
```

これを使えば？

## comment 2026-10-20 1

わずか 2 文字を追加するだけで漸近的に最適になるのは魅力的なので、やっぱりデルタ版にする。

`CPS` 版（[リンク](https://hackage-content.haskell.org/package/transformers-0.6.3.0/docs/Control-Monad-Trans-Writer-CPS.html)）というのは、ただ単に `seq` 関数を噛ませる箇所が違うだけであるようである。

`forall r. s -> (a -> s -> m r) -> m r -> m r` において `callCC` 関数を実装できるのか考えるよりも `ContT` 型を追加して明示的に継続の作用を追加した方が良さそうである。とはいえ `m` が `Applicative` であるという条件を追加したりすれば `shift` 関数を実装できるかもしれない。

self-synchronizing code には区切りが必要であるというわけだけど、どれかのパターンを区切りのために割り当ててしまうと、たとえば 4 種類のパターンのうち 1 つのパターンを区切りに割り当てると、残りの 3 つのパターンだけを使って表現する必要があるので、符号の長さが `log 4 / log 3` 倍になってしまい、 asymptotically optimal にならなくなるのでは。つまり、両立は不可能なのでは。

名前は、最初に考えた "very easy readable lexicographic encoding" の略である "verle" でいいんじゃないかな？

## comment 2026-10-20 2

バックトラックの作用を持つモナドはリストモナドと継続モナドの二種類がある。リストモナドは深さ優先探索のみを行うけど、継続モナドは幅優先探索などを自由自在に行える。

まずは `s -> [Either e (s, a)]` または `StateT s (ExceptT e [])` というようにリストモナドを使って実装してから、機能面や性能面などにおいて難点があったら継続モナドに切り替えるのはどうだろうか？

見出しのようにコンビネータの合成の列の先頭に失敗してバックトラックするかどうかとバックトラックの理由はなんであるかを設定するのは継続モナドがないとできないのでは？　いや、 `Writer` モナドも積めばできるか？

`StateT s (ExceptT e (WriterT w []))` ということだよね。

ああ、でも、成功してバックトラックした時には何も出力せずに通過するというのはできないかも。

正順に進行している時にログを残すのは `Writer` モナドでもできるけど、逆順に進行している時にログを残すのは継続モナドが必要である。

`forall r. s -> (s -> a -> m (w, r)) -> m (w, r) -> m (w, r)` なら可能になるかも？

いや、普通の `Writer` モナドと同じになるかも？

`Codensity (StateT s (ExceptT e []))` または `Codensity (StateT s (ExceptT e Tree))` ならなんとかなるかも？

なにはともあれ実装してみてからだな。

## comment 2026-10-20 3

prefix code をパースする時にはバックトラックは発生しない。

リストモナドは外してもいい。ログはライターモナドを使う。

ツリー状のログのためには、継続モナドを使って「それ以降のログ」を包むような形で `Tree` 型のログを取るか、 `Tree` 型を諦めてインデントを増やす関数とインデントを減らす関数で本体を包み込むしかない。後者から `Tree` 型のログを生成するためには改めてログをパースしないといけない。

`Tree` 型ではなく `Forest` 型か。そうでないとモノイドにならない。

ログの単位は `Matched '0' "Number"` のようにする。後者の引数は型の名前となる。マッチしない場合も。複数の文字が並んだ数値である場合も。継続モナドを使う場合、消費する前の文字列と消費した後の文字列を並べることができる。

一つの文字にだけマッチするコンビネータを並列してバックトラックするよりも、複数の種類の文字にマッチするコンビネータを使って、文字を pop するのを 1 回だけにした方が良い。

## comment 2026-10-21 1

継続モナドでバックトラックを実装するとして、 `(<|>)` を使えば十分なんじゃないかと。

一応、どんな時にも消費前の状態を取得できるというメリットがある。

`s -> Maybe (s, a)` でも `s -> (Maybe a, s)` でも `(<|>)` を実装してバックトラックできるんじゃないかと思っていたけど、後者は `MonadPlus` 型クラスの則を満たさない可能性もあるか。

`logict` ライブラリ（[リンク](https://hackage.haskell.org/package/logict)）は継続渡しスタイルにおいてバックトラックができるようになるという実例である？

`forall r. s -> (w -> m r) -> (w -> s -> a -> m r) -> m r` として、普通に `w` に `mempty` を渡すのが正順の時で、逆順の時は……キャッチできない？

うーん？

`Codensity` モナドを入れると `w` が正の位置にも負の位置にも現れるようになる。入力と出力であり、正順処理と逆順処理である。

`LogicT` モナドが継続を操る能力を持つのは `m r` が正の位置にも負の位置にも現れるからなのでは？

`Codensity (StateT s (MaybeT (WriterT (Forest String) m)))` という感じかな？

`MaybeT (WriterT w m)` が `MonadPlus` になれない可能性もある。

いや、デフォルトのインスタンスで普通になれるか？

デフォルトのインスタンスで普通になれるから `DerivingVia` 言語拡張でいけるわ。

## comment 2026-10-22 1

"very easy writable lexicographic prefix encoding" を "vewlpe" に。

良いかも？

ガンマ版とデルタ版の区別は "vewlpe gamma" と "vewlpe delta" で。

## comment 2026-10-22 2

もしかして vewlpe delta よりも vewlpe omega の方が短い？

デルタ版は以下の通りである。

* `1-1-1`
* `1-1-2`
* …
* `1-1-9`
* `1-2-10`
* `1-2-11`
* …
* `1-2-99`
* `1-3-100`
* `1-3-101`
* …
* `1-9-999999999`
* `2-10-1000000000`
* `2-10-1000000001`
* …
* `2-10-9999999999`
* `2-11-10000000000`

オメガ版は以下の通りである。

* `1-1`
* `1-2`
* …
* `1-9`
* `2-2-10`
* `2-2-11`
* …
* `2-2-99`
* `2-3-100`
* `2-3-101`
* …
* `2-9-999999999`
* `3-2-10-1000000000`
* `3-2-10-1000000001`
* …
* `3-2-10-9999999999`
* `3-2-11-10000000000`

1 桁の範囲においては短くなるけど 10 桁の範囲においては長くなる。

オメガ版の方に切り替えるまでの利点にはならないな。

アルファ版をパースする時には 0 を認めた方が都合の良いところがある。

正規表現だと `9*[0-9]` だからね。

なんで `log 0` は未定義なのに 0 の桁数は 0 になるんだろう？

`[0-9]*` だと解釈した時に 0 は空の文字列 `ε` に対応するから？

`0 < 10 ^ 0` だから？

`a` の桁数は `a < 10 ^ n` を満たす最小の自然数 `n` である。

高速に計算するには？

```hs
lengthWithState :: Word -> Word -> Word -> Word
lengthWithState base n x y = if n < y then x else lengthWithState base n (x + 1) (y * base)

length :: Word -> Word -> Word
length base n = lengthWithState base n 0 1
```

Lean 4 の `log` 関数（[リンク](https://leanprover-community.github.io/mathlib4_docs/Mathlib/Data/Nat/Log.html#Nat.log)）は、なんか複雑な再帰をしている。いや、ただ単に指数を 2 倍にしていって天井を定めた後に二分探索しているだけである。

同じようにすれば良い。

安心のために証明も欲しいな。

0 を含む自然数のパースの仕方は？

PEG だと `0|[1-9][0-9]*` という風にバックトラックするしかない。

モナドの作用があると最初の文字が 0 であるか 1, 2, ..., 9 であるかで分岐することができて、バックトラックが不要になる。
