# issue 2

issue の番号の形式を確定する。 issue の番号の形式にはガンマ符号の変種を使う。通常の十進法では番号順と辞書順が一致せず、アルファ符号は長くなりすぎ、デルタ符号はオーバースペックすぎる。

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
