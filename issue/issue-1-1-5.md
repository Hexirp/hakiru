# issue 5

## comment 2026-10-16 1

GitHub Actions を使って CI をする。全面的に stack に依存することで設定を極めて単純にする。私 (Hexirp) の無料枠を消費し切ってしまわないようにする。 stack の resolver を固定することでストレージも節約して、きちんとキャッシュする。

そのままの設定だと、最初に GitHub に上げた時から、すぐに GitHub Actions が起動してしまうので、優先度は高い。ああ、でも、オフにすればいいのか。

## comment 2026-10-25 1

ローカル環境でビルドを通す、つまり "[issue-1-1-8.md](issue-1-1-8.md)" が終わるのを待ってから、 GitHub Actions のことをやる。

## comment 2026-10-25 2

"[issue-1-1-9.md](issue-1-1-9.md)" と重複しているから、こっちの方を正とする。こっちの方に具体性があるから。

## comment 2026-10-25 3

GitHub に上げること、つまり "[issue-1-2-14.md](issue-1-2-14.md)" が前提条件である。これは既に完了している。

## comment 2026-10-26 1

1fe87ac36e8f20b753a22e358421c7f59a8e608b において GitHub Actions の設定を作り直した。

そのために以下を読んだ。

* "[action.yml](https://github.com/haskell-actions/setup/blob/0f8e8c99d88aeb3fbfd523f1ef2c6f762d10d64d/action.yml)"
* "[examples.md](https://github.com/haskell-actions/setup/blob/0f8e8c99d88aeb3fbfd523f1ef2c6f762d10d64d/docs/examples.md)"
* "[GitHub Actions で Haskell プロジェクトの stack/cabal 混在 CI](https://debug-ito.hatenablog.com/entry/2021/02/28/121631)"
* "[Haskell で GitHub Actions する](https://matsubara0507.github.io/posts/2019-12-02-haskell-with-gh-actions.html)"
* "[Document how to cache stack](https://github.com/haskell-actions/setup/issues/86)"
* "[Blessed recipe how to use stack on github actions, in particular caching?](https://github.com/commercialhaskell/stack/issues/5754)"
* "[Github cache action](https://discourse.haskell.org/t/github-cache-action/9926)"

[haskell-actions/setup](https://github.com/haskell-actions/setup) の使い方とかを調べながら、既製品を使わないで一から自分で作ったのは初めてである。

GitHub Actions の設定の構造とかは前に調べていたはずなのに忘れていた。まあ、まともに扱ったのはたぶん 1 年ぶりだから仕方がないか。 `strategy` とか `fail-fast` とか `matrix` とかの使い方は以前に調べていたはずだけど、 `id` は初見かな。

他のワークフローを参照する時には commit id を固定している。タグだと別のコミットにずらして任意のコードを実行させる入口になってしまうので。

`stack` コマンドのオプションとしての `--fast` は初見だった。 `--system-ghc` を使わないといけないことは、まあ `ghcup` コマンドを使うんだから、そうだと思っていた。 `--only-dependencies` の名前がうろ覚えで `--dependencies-only` に書き間違えた。

前の Haskell のプロジェクトでは Stack を `curl` からインストールして GHC まで全部やらせていたから [haskell-actions/setup](https://github.com/haskell-actions/setup) を使うのは初めてだな。

GitHub Actions のキャッシュの設計をまともにするのも初めてである。というか、前にキャッシュを扱っていた時は Travis CI を使っていたはずである。まあ、ディレクトリ構造に見立てて、より大きな分類を前において、より小さい分類を後におくということだよね。値が違っていてもキャッシュを共有できそう度が大きい値を後の方におくメソッドと呼ぶべきかどうか。 `matrix.os` と `runner.os` の順番に迷ったけど、 `matrix.os` はアバウトな指定だけど `runner.os` には具体的なバージョンが入るはずだし、 `matrix.os` を先において `runner.os` を後におくことにする。

`hashFiles('stack.yaml.lock')` を最後のキーとすることについては、色々と意見が分かれていて難しかったけど、 `stack.yaml` ファイルと `package.yaml` ファイルのハッシュをキーにするのは意味を変えない程度の変更でも途切れるのがいけないというのに同意したので、ロックファイルの方を使った。

`restore-keys` についてまともに考えるのも初めてだけど、ロックファイルが違うキャッシュを使うと、既に使わない依存ライブラリのビルド成果物を引き継ぐことになってしまうと考えたので、利便性よりもキャッシュの縮小を優先して、ロックファイルが一致することを条件にする。 resolver を固定しているから、ほとんど変化しないはずだしね。

`stack.yaml` ファイルを変更して `stack.yaml.lock` ファイルを変更していない場合には少々厄介な状況になるはずだけど、ローカルでのビルドを忘れなければ大丈夫だし、それほど大変なことにはならないはず。
