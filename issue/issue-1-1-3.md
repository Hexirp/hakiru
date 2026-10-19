# issue 3

## comment 2026-10-15 1

コンパイラと依存ライブラリのバージョンを一つに固定する。バージョンを固定しないと、ビルドするたびに依存ライブラリのバージョンが変化して、キャッシュが膨大になるため。

まずは書き直しに集中したいので、新しいバージョンに追従するとか、複数バージョンに対応するとか、そういうことは後回しにしたい。

stackage を利用してコンパイラと依存ライブラリのバージョンを固定する。では、どのバージョンに固定するべきだろうか？

GHC の LTS は "[GHC LTS Releases](https://www.haskell.org/ghc/blog/20250702-ghc-release-schedules.html)" と "[GHC LTS Releases](https://blog.haskell.org/ghc-lts-releases/)" にあるように 9.14 からだから、まだまだ枯れていない。 "post-hoc LTS" というのもあるそうだけど。

"[GHC LTS Releases](https://discourse.haskell.org/t/ghc-lts-releases-the-glasgow-haskell-compiler/12469)" において [George Thomas](https://discourse.haskell.org/u/george.fst) さんにより 8.10, 9.2, 9.6 が "post-hoc LTS" だというコメント（[リンク](https://discourse.haskell.org/t/ghc-lts-releases-the-glasgow-haskell-compiler/12469/3)）がつけられている。なお "[Bump ghc major version to 10?](https://gitlab.haskell.org/ghc/ghc/-/work_items/26198)" という GHC 9.14 の次を GHC 10 にする提案がある。

"post-hoc LTS" を選ぶか？

Stackage Server のメインページ（[リンク](https://www.stackage.org/)）には次のリストがある。

* Stackage Nightly 2026-10-12 (ghc-9.12.4), 2 days ago
* LTS 24.63 for ghc-9.10.3, published 3 days ago
* LTS 24.11 for ghc-9.10.2, published a year ago
* LTS 23.28 for ghc-9.8.4, published a year ago
* LTS 22.44 for ghc-9.6.7, published a year ago
* LTS 22.43 for ghc-9.6.6, published a year ago
* LTS 22.27 for ghc-9.6.5, published 2 years ago
* LTS 22.20 for ghc-9.6.4, published 2 years ago
* LTS 22.6 for ghc-9.6.3, published 3 years ago
* LTS 21.25 for ghc-9.4.8, published 3 years ago

nightly は不適切である。最新の LTS は現在進行形でアップデートされていて、新しすぎる。となると LTS 23.28 か？

そういや "[haskell-misc](https://github.com/Hexirp/haskell-misc)" でも LTS 23.28 を使っていたな。（[コード](https://github.com/Hexirp/haskell-misc/blob/f0700bb2bfc5fffbec681f173ff8b14bb3867c44/stack.yaml#L20)）

これでいいか。

GHC のバージョンを固定すると、古い GHC に対応するためのコードも削ることができる。特に `CPP` 拡張を使った分岐とか。
