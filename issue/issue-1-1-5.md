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

"error: unknown architecture" だそうだけど。

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
