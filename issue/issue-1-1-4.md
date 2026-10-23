# issue 4

## comment 2026-10-16 1

古い依存ライブラリを更新する。何が古いのかという基準は次の通りである。

* deprecated になっている。
* GHC 8.x 以下に対応するためのコードがまだ残っている。
* コードの見た目のためにだけ用意されたモナド則を満たさないインスタンスがある。
* 新しい言語拡張で解決できるような古い書き方がある。
* 別の代替ライブラリが登場している。
* GitHub のリポジトリにおいて大量のバグ報告が溜まっている。
* `head` 関数のような部分関数や `undefined` 関数のようなエラー関数などを乱用している。
* 不完全な `case` 式などのような GHC の警告オプションにより容易に発見できる不適切なコードが残留している。

## comment 2026-10-19 1

テンプレートエンジンである `mustache` ライブラリ（[リンク](https://hackage.haskell.org/package/mustache)）と統合する。 `shake` ライブラリ（[リンク](https://hackage.haskell.org/package/shake)）のキャッシュを利用するのも良いのかもしれない。

古いライブラリの条件について。

* stack や Hadrian のような Haskell 最先端のソフトウェアにおいて使われている代替ライブラリが存在する。
* そのライブラリの欠点を指摘して新しい書き方を示しているブログ記事や Reddit の投稿などが存在する。

`path` ライブラリ（[リンク](https://hackage.haskell.org/package/path)）もどうかな？

## comment 2026-10-21 1

既に「それ」を提供するライブラリが存在するけど、依存関係の膨張を防ぐために、自分のパッケージの中で「それ」を定義する時に、適切な名前はなんであろうか？

依存関係を減らした代替ライブラリを定義する時は `microlens` ライブラリ（[リスト](https://hackage.haskell.org/package/microlens)）のように "micro" をつけることが多いけど、モジュール単位の時の慣習を知らない。

同じ名前のモジュールを作るのは紛らわしい。 `Data.List.Extra` みたいな名前にするのは、他のパッケージと被るかもしれない。

Haskell の慣習とは違うから仕方がないんだけど、ここまで出した案は「必ず `Foo` は `Foo.Bar` に依存する」と「必ず `Foo.Bar` が `Baz.Baq` に依存しているならば `Foo` は `Baz` に依存する」という規則に違反している。この二つの規則はパッケージとモジュールを統一して扱う時に必要となるんだけどな。

`Misc` の下に置く。 `Micro` の下に置く。 `Portable` の下に置く。

どれもいまいちだな。

`Prelude` とか？

## comment 2026-10-21 2

`binary` ライブラリ（[リスト](https://hackage.haskell.org/package/binary)）も `blaze-html` ライブラリ（[リスト](https://hackage.haskell.org/package/blaze-html)）も嘘モナドを使っている。嘘モナドというのは、 `do` 式を使ってモノイドによる合成を記述するために、 `(>>=)` を実装せず `(>>)` だけを実装したインスタンスのことね。「嘘モナド」というのはいま考えた名前ね。 `Writer` モナドを使えば、おそらく修正できる。

私が間違っていた。 `binary` ライブラリ（[リスト](https://hackage.haskell.org/package/binary)）はきちんと `Writer` モナドを使っていた。

`blaze-html` ライブラリ（[リスト](https://hackage.haskell.org/package/blaze-html)）も単純なモノイドとは言えない複雑な構造を使っていた。でも、よくよく見ると `bind` 関数による合成の時に `Append` 構築子を使っていて `Append x (Append y z)` と `Append (Applend x y) z` が一致しないので結合法則が成り立たない。（[リンク](https://github.com/jaspervdj/blaze-markup/blob/209c61f04c6ea6a06e8ddf033f6dda03119e56b7/src/Text/Blaze/Internal.hs#L227)）

同じ作者である `hakyll` ライブラリ（[リンク](https://hackage.haskell.org/package/hakyll)）の `Compiler` 型も、前に見た時には本当にモナドになるのかが疑わしかったし、今、見ても同じ感想である。（[リンク](https://github.com/jaspervdj/hakyll/blob/092b6a497326ba7b80d17249b6bbd41ca9bf5e19/lib/Hakyll/Core/Compiler/Internal.hs#L157-L174)）

`ApplicativeDo` 拡張みたいに `MonoidDo` 拡張があれば良いのかもしれないね。 `MonoiDo` なんてね。

## comment 2026-10-22 1

汎用コードについて。利用できるライブラリになかったり、一部だけを利用したいのに巨大な依存ライブラリが背後にあったりする時に、汎用コードだとしても、独自に定義したい時がある。

`ghcup` ライブラリ（[リンク](https://hackage.haskell.org/package/ghcup)）は `Prelude` という名前を使っているようである。

## comment 2026-10-22 2

`Profunctor` 型クラス（[リンク](https://hackage-content.haskell.org/package/profunctors-5.6.3/docs/Data-Profunctor.html#t:Profunctor)）は重要な型クラスである。 `profunctors` ライブラリ（[リンク](https://hackage.haskell.org/package/profunctors)）にある。

`hakyll` ライブラリ（[リンク](https://hackage.haskell.org/package/hakyll)）は profunctors ライブラリ（[リンク](https://hackage.haskell.org/package/profunctors)）に間接的に依存している。

`profunctors` ライブラリ（[リンク](https://hackage.haskell.org/package/profunctors)）が `Functor (Kleisli m a)` の制約を `Monad m` にしているのが嫌である。

そういうのを修正するプルリクエスト（[リンク](https://github.com/ekmett/profunctors/pull/120)）は既にあったし、メジャーバージョンが 6 になる時には修正されるらしいけど、プルリクエストがマージされる様子もないしメジャーバージョンが上がる様子もない。というか、メジャーバージョンが 6 であるコードが置いてある `main` ブランチでは 2021 年に `Monad m` から `Functor m` へと変更するコミット（[リンク](https://github.com/ekmett/profunctors/commit/4af782eb86f092f8880890facaef8c1880b3c7a2)）がされているんだけど、そこから 5 年も進んでいない。

incoherent なインスタンスをつくるのは？　かなり厄介なやつである。

`profunctors` ライブラリ（[リンク](https://hackage.haskell.org/package/profunctors)）を使わずに、独自に定義するのは？

だとしても、最初から依存しているからなあ。

`profunctors` ライブラリ（[リンク](https://hackage.haskell.org/package/profunctors)）に対する依存をなくしたい。

`profunctors` ライブラリ（[リンク](https://hackage.haskell.org/package/profunctors)）に依存しているライブラリに依存しているかどうかを検知したい。

`stack dot` コマンド（[リンク](https://docs.haskellstack.org/en/stable/commands/dot_command/)）による出力をパースするべきだろうか？　おそらく内部でグラフのデータを構築しているのに、二度手間になるよなあ。

内部のデータを取り出せないかな？

`Stack.DependencyGraph` モジュール（[リンク](https://hackage-content.haskell.org/package/stack-3.11.1/docs/Stack-DependencyGraph.html)）というのはある。

`stack dot --reach profunctors` コマンドを使えば可能になるかも？

`haskell-misc` リポジトリ（[リンク](https://github.com/Hexirp/haskell-misc)）にグラフを扱うプログラムを置いているけど、少し改造すれば使えるかもしれんな。そういや、アルゴリズムとして最適化していないんだよな。まあ、実際に使う時に困ったら考えれば良いか。
