# issue 5

## comment 2026-10-16 1

GitHub Actions を使って CI をする。全面的に stack に依存することで設定を極めて単純にする。私 (Hexirp) の無料枠を消費し切ってしまわないようにする。 stack の resolver を固定することでストレージも節約して、きちんとキャッシュする。

そのままの設定だと、最初に GitHub に上げた時から、すぐに GitHub Actions が起動してしまうので、優先度は高い。ああ、でも、オフにすればいいのか。

## comment 2026-10-25 1

ローカル環境でビルドを通す、つまり "[issue-1-1-8.md](./issue-1-1-8.md)" が終わるのを待ってから、 GitHub Actions のことをやる。

## comment 2026-10-25 2

"[issue-1-1-9.md](./issue-1-1-9.md)" と重複しているから、こっちの方を正とする。こっちの方に具体性があるから。

## comment 2026-10-25 3

GitHub に上げること、つまり "[issue-1-2-14.md](./issue-1-2-14.md)" が前提条件である。これは既に完了している。

## comment 2026-10-26 1

`1fe87ac36e8f20b753a22e358421c7f59a8e608b` において GitHub Actions の設定を作り直した。

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

GitHub Actions のキャッシュの設計をまともにするのも初めてである。というか、前にキャッシュを扱っていた時は Travis CI を使っていたはずである。まあ、ディレクトリ構造に見立てて、より大きな分類を前において、より小さい分類を後に置くということだよね。値が違っていてもキャッシュを共有できそう度が大きい値を後の方に置くメソッドと呼ぶべきかどうか。 `matrix.os` と `runner.os` の順番に迷ったけど、 `matrix.os` はアバウトな指定だけど `runner.os` には具体的なバージョンが入るはずだし、 `matrix.os` を先において `runner.os` を後に置くことにする。

`hashFiles('stack.yaml.lock')` を最後のキーとすることについては、色々と意見が分かれていて難しかったけど、 `stack.yaml` ファイルと `package.yaml` ファイルのハッシュをキーにするのは意味を変えない程度の変更でも途切れるのがいけないというのに同意したので、ロックファイルの方を使った。

`restore-keys` についてまともに考えるのも初めてだけど、ロックファイルが違うキャッシュを使うと、既に使わない依存ライブラリのビルド成果物を引き継ぐことになってしまうと考えたので、利便性よりもキャッシュの縮小を優先して、ロックファイルが一致することを条件にする。 resolver を固定しているから、ほとんど変化しないはずだしね。

`stack.yaml` ファイルを変更して `stack.yaml.lock` ファイルを変更していない場合には少々厄介な状況になるはずだけど、ローカルでのビルドを忘れなければ大丈夫だし、それほど大変なことにはならないはず。

## comment 2026-10-26 1

[`303c00da75b18f7a33b732347ba135646694ff96`](https://github.com/Hexirp/hakiru/commit/303c00da75b18f7a33b732347ba135646694ff96) をプッシュした。これにより GitHub Actions の実行（[リンク](https://github.com/Hexirp/hakiru/actions/runs/40219334907)）が始まった。

Linux のジョブ（[リンク](https://github.com/Hexirp/hakiru/actions/runs/40219334907/job/121247278179)）と Windows のジョブ（[リンク](https://github.com/Hexirp/hakiru/actions/runs/40219334907/job/121247278331)）は問題なく終わったけど、 macOS のジョブ（(https://github.com/Hexirp/hakiru/actions/runs/40219334907/job/121247278365)）が失敗した。

該当の場所は以下の通りである。全部のログは [0_Build os-macos-latest ghc-9.8.4.txt](./issue-1-1-5/logs_109007744674/0_Build%20os-macos-latest%20ghc-9.8.4.txt) から。

```txt
2026-10-25T18:07:00.5996890Z ##[group]Run stack build --fast --system-ghc --test --no-run-tests --only-dependencies
2026-10-25T18:07:00.5997600Z [36;1mstack build --fast --system-ghc --test --no-run-tests --only-dependencies[0m
2026-10-25T18:07:00.6299820Z shell: /bin/bash -e {0}
2026-10-25T18:07:00.6300120Z ##[endgroup]
2026-10-25T18:07:29.9023270Z [1 of 3] Compiling Main             ( /Users/runner/.stack/setup-exe-src/setup-w2MFVN35.hs, /Users/runner/.stack/setup-exe-src/setup-w2MFVN35.o )
2026-10-25T18:07:30.0229180Z [2 of 3] Compiling StackSetupShim   ( /Users/runner/.stack/setup-exe-src/setup-shim-w2MFVN35.hs, /Users/runner/.stack/setup-exe-src/setup-shim-w2MFVN35.o )
2026-10-25T18:07:30.1937510Z [3 of 3] Linking /Users/runner/.stack/setup-exe-cache/aarch64-osx/tmp-Cabal-simple_w2MFVN35_3.10.3.0_ghc-9.8.4
2026-10-25T18:07:30.5991280Z ld: warning: -U option is redundant when using -undefined dynamic_lookup
2026-10-25T18:07:31.3177230Z ld: multiple errors: tapi error: malformed file
2026-10-25T18:07:31.3195150Z 
2026-10-25T18:07:31.3200000Z /Library/Developer/CommandLineTools/SDKs/MacOSX27.0.sdk/usr/lib/libSystem.B.tbd:4:20: error:
2026-10-25T18:07:31.3201040Z      error: unknown architecture
2026-10-25T18:07:31.3201500Z                        arm64e.x1-macos, arm64e.x1-maccatalyst ]
2026-10-25T18:07:31.3202080Z                        ^~~~~~~~~~~~~~~
2026-10-25T18:07:31.3202970Z      in '/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/lib/libm.tbd'; tapi error: malformed file
2026-10-25T18:07:31.3213020Z   |
2026-10-25T18:07:31.3213660Z 4 |                    arm64e.x1-macos, arm64e.x1-maccatalyst ]
2026-10-25T18:07:31.3214140Z   |                    ^
2026-10-25T18:07:31.3214450Z 
2026-10-25T18:07:31.3214900Z /Library/Developer/CommandLineTools/SDKs/MacOSX27.0.sdk/usr/lib/libiconv.2.tbd:4:20: error:
2026-10-25T18:07:31.3215520Z      error: unknown architecture
2026-10-25T18:07:31.3216020Z                        arm64e.x1-macos, arm64e.x1-maccatalyst ]
2026-10-25T18:07:31.3216650Z                        ^~~~~~~~~~~~~~~
2026-10-25T18:07:31.3237030Z      in '/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/lib/libiconv.tbd'; tapi error: malformed file
2026-10-25T18:07:31.3238360Z   |
2026-10-25T18:07:31.3239200Z 4 |                    arm64e.x1-macos, arm64e.x1-maccatalyst ]
2026-10-25T18:07:31.3240200Z   |                    ^
2026-10-25T18:07:31.3240650Z 
2026-10-25T18:07:31.3241330Z /Library/Developer/CommandLineTools/SDKs/MacOSX27.0.sdk/usr/lib/libSystem.B.tbd:4:20: error:
2026-10-25T18:07:31.3242680Z      error: unknown architecture
2026-10-25T18:07:31.3243680Z                        arm64e.x1-macos, arm64e.x1-maccatalyst ]
2026-10-25T18:07:31.3244500Z                        ^~~~~~~~~~~~~~~
2026-10-25T18:07:31.3246020Z      in '/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/lib/libdl.tbd'; tapi error: malformed file
2026-10-25T18:07:31.3247980Z   |
2026-10-25T18:07:31.3248820Z 4 |                    arm64e.x1-macos, arm64e.x1-maccatalyst ]
2026-10-25T18:07:31.3249740Z   |                    ^
2026-10-25T18:07:31.3250280Z 
2026-10-25T18:07:31.3251110Z /Library/Developer/CommandLineTools/SDKs/MacOSX27.0.sdk/usr/lib/libffi.tbd:4:20: error:
2026-10-25T18:07:31.3252270Z      error: unknown architecture
2026-10-25T18:07:31.3253210Z                        arm64e.x1-macos, arm64e.x1-maccatalyst ]
2026-10-25T18:07:31.3254190Z                        ^~~~~~~~~~~~~~~
2026-10-25T18:07:31.3255430Z      in '/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/lib/libffi.tbd'; tapi error: malformed file
2026-10-25T18:07:31.3256760Z   |
2026-10-25T18:07:31.3257500Z 4 |                    arm64e.x1-macos, arm64e.x1-maccatalyst ]
2026-10-25T18:07:31.3258670Z   |                    ^
2026-10-25T18:07:31.3268960Z 
2026-10-25T18:07:31.3278290Z /Library/Developer/CommandLineTools/SDKs/MacOSX27.0.sdk/usr/lib/libSystem.B.tbd:4:20: error:
2026-10-25T18:07:31.3280110Z      error: unknown architecture
2026-10-25T18:07:31.3296540Z                        arm64e.x1-macos, arm64e.x1-maccatalyst ]
2026-10-25T18:07:31.3299070Z                        ^~~~~~~~~~~~~~~
2026-10-25T18:07:31.3300340Z      in '/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/lib/libSystem.tbd'
2026-10-25T18:07:31.3301060Z   |
2026-10-25T18:07:31.3304290Z 4 |                    arm64e.x1-macos, arm64e.x1-maccatalyst ]
2026-10-25T18:07:31.3304950Z   |                    ^
2026-10-25T18:07:31.3305740Z clang: error: linker command failed with exit code 1 (use -v to see invocation)
2026-10-25T18:07:31.3306720Z ghc-9.8.4: `gcc' failed in phase `Linker'. (Exit code: 1)
2026-10-25T18:07:31.3801610Z 
2026-10-25T18:07:31.3803170Z Error: [S-6374]
2026-10-25T18:07:31.3803810Z        While building simple Setup.hs (scroll up to its section to see the error) using:
2026-10-25T18:07:31.3806000Z        /Users/runner/.ghcup/ghc/9.8.4/bin/ghc-9.8.4 -rtsopts -threaded -clear-package-db -global-package-db -hide-all-packages -package base -main-is StackSetupShim.mainOverride -package Cabal-3.10.3.0 /Users/runner/.stack/setup-exe-src/setup-w2MFVN35.hs /Users/runner/.stack/setup-exe-src/setup-shim-w2MFVN35.hs -o /Users/runner/.stack/setup-exe-cache/aarch64-osx/tmp-Cabal-simple_w2MFVN35_3.10.3.0_ghc-9.8.4
2026-10-25T18:07:31.3809430Z        Process exited with code: ExitFailure 1 
2026-10-25T18:07:31.4022630Z ##[error]Process completed with exit code 1.
```

GitHub Actions のログが閲覧できる期限があるので、手元に保存することにしたわけだけど、これらをリポジトリに置くかどうか迷った。

でも、 Linux のジョブのログと Windows のジョブのログが極めて長くて、ダウンロードした `zip` ファイルの時点で 305,854 バイトで、圧縮しないと合計 2,099,107 バイトとなっている。

Google AI によると GitHub のリポジトリのサイズは 1 GiB あたりまでにするのが良いらしくて、 GitHub のリポジトリにおける単独のファイルのサイズの上限は 100 MiB となっているらしい。

きちんとしたドキュメントを読むと、 "[Repository limits](https://docs.github.com/en/repositories/creating-and-managing-repositories/repository-limits)" によると、 GitHub のリポジトリのサイズは 10 GB が上限で、単一のファイルのサイズは 1 MB までにするのを推奨していて 100 MB が上限だそうである。

圧縮しないと単一のファイルだけで上限を超えてしまっているし、圧縮したサイズもコードのファイルをまとめたサイズに匹敵しているのはまずいし、圧縮した後のバイナリファイルを置きたくないということで、 macOS のジョブのログだけコミットして、他のログは `ignore` ディレクトリに置くことにする。

以前、「 `runner.os` には具体的なバージョンが入るはず」と書いたけど、 `Linux` のような内容となっていて、これは間違っていた。

`runner.os` は `Linux` か `Windows` か `macOS` かのどれかだそうである。

キャッシュのステップの位置だけど、なんかビルドした後にキャッシュを探してダウンロードするという挙動になってしまっている。

ビルドを始める前の位置にキャッシュのステップを置くべきだった。きちんとキャッシュのアップロードは後で行われるから。

"[actions/cache](https://github.com/actions/cache)" の公式ドキュメントを読もう。

普通に "[README.md § Creating a cache key](https://github.com/actions/cache/blob/3edfce9056124e459a23f683a21433670d47daca/README.md#creating-a-cache-key)" に書いてある。

`runner.os` を先頭に置くこと、そしてロックファイルをハッシュすること。きちんと公式ドキュメントを読むべきだったか。

"[Dependency caching reference § Using contexts to create cache keys](https://docs.github.com/en/actions/reference/workflows-and-actions/dependency-caching#using-contexts-to-create-cache-keys)" にもロックファイルをハッシュすることにより依存関係が変化した時にキャッシュのキーを変えることができることが書いてある。

Stack を使用した場合のサンプル（[リンク](https://github.com/actions/cache/blob/3edfce9056124e459a23f683a21433670d47daca/examples.md#haskell---stack)）もあるわけだけど、おそらくロックファイルが存在しなかった時期のやつで、ちょっと古い。

更新するにも、ちょっと Haskell コミュニティの意志を集約する必要があるかも？

それよりもやばいのが、 macOS のビルドがよくわからない理由で落ちていること。

`error: unknown architecture` だそうだけど。

たぶん古いコンパイラだから新しいアーキテクチャに対応できていない？

"[Unknown architecture when attempting to install GHCup on Red Hat linux](https://stackoverflow.com/questions/76950449/unknown-architecture-when-attempting-to-install-ghcup-on-red-hat-linux)" という似たエラーに関する質問がある。

"[platforms](https://gitlab.haskell.org/ghc/ghc/-/wikis/platforms?version_id=0fe22127d49247a5ad1762d10d25a13c0d26c422&version_number=85)" によると macOS の場合は x86-64 と AArch64 のみに対応していて ARM64 に対応していないということなのかな。

Hakyll は macOS でテストしているんだよな。どうやっているんだ？

"[main.yml](https://github.com/jaspervdj/hakyll/blob/3b62127a9d9c9c6df84acc9db0a42f02a4d8e07c/.github/workflows/main.yml#L17)" によると `macOS-14` を指定している。

ところが、 "[GitHub Actions: macOS 14 runner image retirement](https://github.blog/changelog/2026-10-01-github-actions-macos-14-runner-image-retirement/)" によると、 `macOS-14` の提供は 2026 年 11 月 2 日に終了するとのこと。

macOS でのテストは諦めるべきか？

依存関係が変わる場合に Stack のロックファイルのハッシュはあまり役に立たない。結局 resolver を固定しているだけで、どの依存ライブラリを使うかは変わるから。

やはり "[Document how to cache stack](https://github.com/haskell-actions/setup/issues/86)" と "[Blessed recipe how to use stack on github actions, in particular caching?](https://github.com/commercialhaskell/stack/issues/5754)" で紹介されていた `stack ls dependencies` か？

`runner.os`, `matrix.os`, `matrix.ghc`, `stack.yaml.lock`, `stack ls --system-ghc dependencies json` の順番にする。

ちょっとだけ依存ライブラリが変わった後に、キャッシュを作るのは全部やり直しというのは大変である。

一番最初のところに手書きの番号をつけて、キャッシュをリセットしたい時には増やすとか？

`restore-keys` においては間違った前方一致を防ぐためにハイフンを最後につけないといけない。

手元で失敗するのを再現するために git の改行の設定も変えないと。

## comment 2026-10-28 1

Stack のリポジトリを探していたら Travis CI の設定のサンプル（[リンク](https://github.com/commercialhaskell/stack/blob/11a148f7b871e90ef96daab0b3d933d51449e5b0/doc/resources/travis-simple.yml)）を見つけた。

懐かしい。

初めての CI は Travis CI を使っていたんだよね。いつも `--only-dependencies` フラグを使って依存ライブラリだけ先にビルドするようにしているのも、このサンプルを参考にしたのが染みついているからなんだよね。

## comment 2026-10-28 2

間違ってローカルの環境で `stack ls dependencies` に `--system-ghc` フラグをつけてしまったら、なんか Hackage の Cabal ファイルを全部ダウンロードし始めた。

`--system-ghc` フラグをつけるのは、 GHCup とは無関係に GHC が入っている GitHub Actions 環境だけなのよね。

ローカルの環境だと GHCup を使って Stack をダウンロードしているので `--system-ghc` フラグをつけるとおかしくなる。

`--external` フラグもつけていたけど、デフォルトでオンであるみたい。

大量のファイルをダウンロードしたみたいから、パソコンの容量を節約するために、削除を試みる。

まずは場所探しである。場所がログに出力されている。とりあえず、 1 GB 以上のファイルが入っている `hackage` ディレクトリの名前を変えたけど、何も影響がないみたい。

`pantry.sqlite3` ファイルも 1 GB ほどに肥大化していたから今度は `pantry` ディレクトリの名前も変えてみる。今度は再生成されたけど、その内容は小さい。

やはり、 Hackage にある全ライブラリの情報が `pantry` ディレクトリの中に登録されてしまったらしい？

`hackage.fpcomplete.com` がまだ Hackage のミラーとして Stack から参照されている。

`--system-ghc` フラグを使っていなくても `stack ls dependencies json` を使うとなんか長いダウンロード処理が挟まる。

でも、 1,672 KB 程度で止まった。

## comment 2026-10-28 3

GHCup を通して使うんだから GitHub Actions でも `--system-ghc` フラグはいらなくなっているのでは？　"[Stack integration](https://www.haskell.org/ghcup/guide/stack/)" によると Stack のバージョン 2.9.1 から GHCup との連携が追加されている。

## comment 2026-10-28 4

コミットのメッセージ。

[`79ad444d59503e7e6986a3354c963a31871ef0e6`](https://github.com/Hexirp/hakiru/commit/79ad444d59503e7e6986a3354c963a31871ef0e6) のメッセージはあまり良くなかった。キャッシュに関連する問題に焦点を絞っていることを書いていなかった。わざと macOS の問題に対応していないことを書いていなかった。

`git push --force` で書き換えるか。

[`fd1769cc62d9819803f639e5b442791f2d7d4aed`](https://github.com/Hexirp/hakiru/commit/fd1769cc62d9819803f639e5b442791f2d7d4aed) に置き換えた。

## comment 2026-10-29 1

さらに凡ミスをしていたので [`ea6b9083da4d4213930537e8affb89346f0f201d`](https://github.com/Hexirp/hakiru/commit/ea6b9083da4d4213930537e8affb89346f0f201d) に書き換えた。

## comment 2026-10-30 1

さて、繰り返し修正するたびにジョブが走っていたわけだけど、どれも問題を起こしている。

* [build #2](https://github.com/Hexirp/hakiru/actions/runs/40552975074)
  * [Build os-ubuntu-latest ghc-9.8.4](https://github.com/Hexirp/hakiru/actions/runs/40552975074/job/122361370343)
  * [Build os-macos-latest ghc-9.8.4](https://github.com/Hexirp/hakiru/actions/runs/40552975074/job/122361370953)
  * [Build os-windows-latest ghc-9.8.4](https://github.com/Hexirp/hakiru/actions/runs/40552975074/job/122361371242)
* [build #3](https://github.com/Hexirp/hakiru/actions/runs/40697634591)
  * [Build os-ubuntu-latest ghc-9.8.4](https://github.com/Hexirp/hakiru/actions/runs/40697634591/job/122848546926)
  * [Build os-macos-latest ghc-9.8.4](https://github.com/Hexirp/hakiru/actions/runs/40697634591/job/122848546691)
  * [Build os-windows-latest ghc-9.8.4](https://github.com/Hexirp/hakiru/actions/runs/40697634591/job/122848546129)
* [build #4](https://github.com/Hexirp/hakiru/actions/runs/40726439891)
  * [Build os-ubuntu-latest ghc-9.8.4](https://github.com/Hexirp/hakiru/actions/runs/40726439891/job/122950884072)
  * [Build os-macos-latest ghc-9.8.4](https://github.com/Hexirp/hakiru/actions/runs/40726439891/job/122950884585)
  * [Build os-windows-latest ghc-9.8.4](https://github.com/Hexirp/hakiru/actions/runs/40726439891/job/122950884504)

これらのログは 90 日間で消えるので、以下のように保存してある。

* build #2
  * [Build os-ubuntu-latest ghc-9.8.4](./issue-1-1-5/logs_109917615857/2_Build%20os-ubuntu-latest%20ghc-9.8.4.txt)
  * [Build os-macos-latest ghc-9.8.4](./issue-1-1-5/logs_109917615857/1_Build%20os-macos-latest%20ghc-9.8.4.txt)
  * [Build os-windows-latest ghc-9.8.4](./issue-1-1-5/logs_109917615857/0_Build%20os-windows-latest%20ghc-9.8.4.txt)
* build #3
  * [Build os-ubuntu-latest ghc-9.8.4](./issue-1-1-5/logs_110311119027/0_Build%20os-ubuntu-latest%20ghc-9.8.4.txt)
  * [Build os-macos-latest ghc-9.8.4](./issue-1-1-5/logs_110311119027/1_Build%20os-macos-latest%20ghc-9.8.4.txt)
  * [Build os-windows-latest ghc-9.8.4](./issue-1-1-5/logs_110311119027/2_Build%20os-windows-latest%20ghc-9.8.4.txt)
* build #4
  * [Build os-ubuntu-latest ghc-9.8.4](./issue-1-1-5/logs_110392183366/2_Build%20os-ubuntu-latest%20ghc-9.8.4.txt)
  * [Build os-windows-latest ghc-9.8.4](./issue-1-1-5/logs_110392183366/1_Build%20os-windows-latest%20ghc-9.8.4.txt)

最後のビルドにおいて macOS だけ欠けているのは 3 回の `Due to capacity constraints, jobs targeting macOS arm64 runners may experience longer queue times.` から `The job was not acquired by Runner of type hosted even after multiple attempts` となって `Internal server error. Correlation ID: 0e359e02-a74d-4e1a-9d87-a4ea4a9b8084` となって落ちているから。

エラーの内容は、ようするに `stack ls dependencies json` コマンドに `--system-ghc` フラグをつけられないということである。手元で実行していた時は `--system-ghc` フラグをつけていなかったから気付かなかった。

そもそも `--system-ghc` フラグが Stack と GHCup の連携において不要となったはずであるので、他の場所の `--system-ghc` フラグも外してみるのが良いかもしれない。

## comment 2026-10-30 2

push した。

以下の通りになった。

* [build #5](https://github.com/Hexirp/hakiru/actions/runs/40910985150)
  * [Build os-ubuntu-latest ghc-9.8.4](https://github.com/Hexirp/hakiru/actions/runs/40910985150/job/123578562635)
  * [Build os-macos-latest-large ghc-9.8.4](https://github.com/Hexirp/hakiru/actions/runs/40910985150/job/123578562760)
  * [Build os-windows-latest ghc-9.8.4](https://github.com/Hexirp/hakiru/actions/runs/40910985150/job/123578562207)

保存したログは以下の通り。

* build #5
  * [Build os-ubuntu-latest ghc-9.8.4](./issue-1-1-5/logs_110898142089/0_Build%20os-ubuntu-latest%20ghc-9.8.4.txt)
  * [Build os-windows-latest ghc-9.8.4](./issue-1-1-5/logs_110898142089/2_Build%20os-windows-latest%20ghc-9.8.4.txt)

`macos-latest-large` のジョブは `The job was not started because recent account payments have failed or your spending limit needs to be increased. Please check the 'Billing & plans' section in your settings` と出た。これはもうダメかな。 macOS は外そう。

`stack ls dependencies json` コマンドは動いたけど、別の GHC をインストールしてしまっている。

つけないと GHC をインストールしてしまうのに、つけることができない。矛盾している。

そもそも `--system-ghc` フラグは "[Document how to cache stack](https://github.com/haskell-actions/setup/issues/86)" の方で勧められていたんだよな。それが急に不可能になったというわけではないよね？

読み返そう。 TravisCardwell さん（[リンク](https://github.com/TravisCardwell)）のコメント（[リンク](https://github.com/haskell-actions/setup/issues/86#issuecomment-2213273002)）を引用させていただく。

````md
When listing dependencies, `--system-ghc` is necessary to prevent Stack from installing GHC.  It looks like the following argument order works:

```
stack --system-ghc ls dependencies json
```
````

つまり、 `--system-ghc` の位置が違う、そういうこと？

## comment 2026-10-30 3

"[Blessed recipe how to use stack on github actions, in particular caching?](https://github.com/commercialhaskell/stack/issues/5754)" にある ulidtko さん（[リンク](https://github.com/ulidtko)）のコメント（[リンク](https://github.com/commercialhaskell/stack/issues/5754#issuecomment-1696156869)）を参考にして変更しよう。

`config.yaml` までキャッシュするのはいけないとか、マニュアルリセットのための変数を入れるとか、既に 1 つのハイフンが区切りに使われているので 2 つのハイフンで区切っておくとか、参考になる。

## comment 2026-10-30 4

push した。

以下の通りになった。

* [build #6](https://github.com/Hexirp/hakiru/actions/runs/40917187453)
  * [Build os-ubuntu-latest ghc-9.8.4](https://github.com/Hexirp/hakiru/actions/runs/40917187453/job/123600437913)
  * [Build os-windows-latest ghc-9.8.4](https://github.com/Hexirp/hakiru/actions/runs/40917187453/job/123600437568)

保存したログは以下の通り。

* build #5
  * [Build os-ubuntu-latest ghc-9.8.4](./issue-1-1-5/logs_110915492883/0_Build%20os-ubuntu-latest%20ghc-9.8.4.txt)
  * [Build os-windows-latest ghc-9.8.4](./issue-1-1-5/logs_110915492883/1_Build%20os-windows-latest%20ghc-9.8.4.txt)

きちんと成功している！

Windows で 7-Zip と msys2 をインストールしているんだけど、これはキャッシュできるのかな……？

あと、 `stack ls dependencies` には `$STACK_ROOT/pantry` の更新が必要だから、キャッシュをするために `stack ls dependencies` が必要で、 `stack ls dependencies` を実行すると `$STACK_ROOT/pantry` が動いて、 `$STACK_ROOT/pantry` が動く前にキャッシュからとってこないといけないというデッドロックがある。

Cargo における `Cargo.lock` みたいな機能があったらいいんだけど。 `stack.yaml.lock` に追加するのは、個別のプロジェクトに対する機能ではないことから、意味論としては微妙ぎみだけど。 Hackage における revision について固定していないというのもあるけど、これは Stackage の resolver を使う限りにおいてはコードの変化がなくメタデータが変わっていないことから問題ない。メタデータが変わると依存関係も変わる可能性があるけど、 resolver を使っているなら依存関係は変わらないから。

* "[Hackage: The Haskell Package Repository](https://hackage.haskell.org/)"
* "[Uploading packages § Package versioning and curation](https://hackage.haskell.org/upload#versioning_and_curation)"
* "[Hackage Metadata Revisions — What They Are, How They Work](https://github.com/haskell-infra/hackage-trustees/blob/98e67ce2f5febf3ea7d47eee9d411daf1e423485/revisions-information.md)"
* "[Cookbook for common build-failures § Best practice for managing meta-data](https://github.com/haskell-infra/hackage-trustees/blob/98e67ce2f5febf3ea7d47eee9d411daf1e423485/cookbook.md#best-practice-for-managing-meta-data)"

revision について調べている。ようするに、依存ライブラリのバージョンの上限を変えたい時はメタデータを編集しろってことね。新しいバージョンをアップロードすると、それぞれのバージョンにおける重複ファイルが大量になるから。かといって、広めの上限を指定すると "[Haskell Package Versioning Policy](https://pvp.haskell.org/)" に違反する。

というか Hackage におけるバージョニングは "[Haskell Package Versioning Policy](https://pvp.haskell.org/)" のみだと規定されているのね。それにしてはセマンティック・バージョニングに従っているように見えるパッケージをちらほら見かけたような記憶があるけど……？

"[Blessed recipe how to use stack on github actions, in particular caching?](https://github.com/commercialhaskell/stack/issues/5754)" にある ulidtko さん（[リンク](https://github.com/ulidtko)）のコメント（[リンク](https://github.com/commercialhaskell/stack/issues/5754#issuecomment-1696156869)）について。

`STACK_ROOT` 環境変数について調べていたら "[stack ignores STACK_ROOT and --stack-root](https://github.com/commercialhaskell/stack/issues/5664) を見つけたんだけど Windows の時だけ `local-programs-path` が `$STACK_ROOT/programs` ではなく `$LOCALAPPDATA\Programs\stack` になっているのは歴史的経緯らしい。このせいで Stack のキャッシュを D ドライブに保存しようとした時に困ったんだけど。

`$STACK_ROOT` を丸ごとキャッシュするのではなく `$STACK_ROOT/pantry` と `$STACK_ROOT/snapshots` と `$STACK_ROOT/programs` を別々にキャッシュするべきだと。 `$STACK_ROOT/pantry` は Hackage にある Cabal ファイルをキャッシュするもので。 `$STACK_ROOT/snapshots` は実際のライブラリのコードをキャッシュするもので。 `$STACK_ROOT/programs` は GHC などをキャッシュするもので。

そういや `stack ls --system-ghc dependencies --external txt > dependencies.txt` を実行した時に Hackage にある全 Cabal ファイルがダウンロードされたのは `stack ls` だけを実行した時の不具合であるのかもしれない。いや、違うか？　よくわからないな。

`$STACK_ROOT/pantry` と `$STACK_ROOT/snapshots` と `$STACK_ROOT/programs` はそれぞれ再利用できる範囲が異なるため別々のキーにする。

`hashFiles('stack.yaml')` と `hashFiles('package.yaml')` ではなく `hashFiles('stack.yaml.lock')` をキーにするべきだと。まあ、これらについては依存ライブラリを無視すればそうであるよね。既に使わない依存ライブラリが `$STACK_ROOT/snapshots` に残るリスクと `hashFiles('package.yaml')` が少し変わるだけで全部の依存ライブラリを再ビルドするリスクを比較して、前者を選ぶべきだと。

やっぱり `stack ls dependencies` はやめようかな。あるいは、諦めようかな。既に使わない依存ライブラリが `$STACK_ROOT/snapshots` に残るリスクを低く見積もる。

サンプルのコードもついている。

GHCup を使って GHC をインストールしている場合、それもキャッシュして、キャッシュがある場合には GHCup のセットアップをしない。ここもキャッシュできないかなと思っていたので、できるというのは嬉しいポイントである。

`$STACK_ROOT/pantry` のキャッシュの時には `stack update` で `$STACK_ROOT/pantry` を更新している。ああ、なるほど？　`stack ls --system-ghc dependencies --external txt > dependencies.txt` に起こったことは、もしかして、これなの？

肥大化した `$STACK_ROOT/pantry` は `$STACK_ROOT/pantry-old` に移動させたまま調査が終わるまで残していたから、そろそろ消そうかなと思っていたんだけど、いったん戻してみるか。

よし、この状態で `stack update` を実行したら更新は少しだけだった。やはりこれが関係しているのか。そして、肥大化しているのは困るけど、残しておくか。

サンプルのコードでは `$STACK_ROOT/stack.sqlite3` と `STACK_ROOT/snapshots` を同時にキャッシュしている。 `stack.sqlite3.pantry-write-lock` は、おそらく必要ない。調べてみたら、なんか Stack に固有の仕組みで、ファイルを排他的に開いたり閉じたりするのをロック機構に転用しているみたい？

サンプルのコードでは `./.stack-work/` もキャッシュしているけど、まあ、今のところはやる予定がない。
