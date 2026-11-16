# issue 21

## comment 2026-11-08 1

Hakiru 0.1.0.0 をリリースするためには。

そもそも `Hexirp/blog` で使いたいからだったのが発端である。

コードは変えず、プロジェクトの構造だけ整理する。

`package.yaml` を整理する。

コードのサンプルを別のディレクトリに分ける。

プロジェクトの直下にあるファイルを整理する。つまり `.ghci` ファイルと `.hlint.yaml` ファイルと `CODE_OF_CONDUCT.md` ファイルと `logo.svg` ファイルと `RELEASE-CHECKLIST.md` ファイルと `test.hs` ファイルを整理する。

 `Hexirp/blog` で使いながらコードを改良していく。

 ## comment 2026-11-14 1

 `.ghci` ファイルと `.hlint.yaml` ファイルと `CODE_OF_CONDUCT.md` ファイルと `logo.svg` ファイルと `RELEASE-CHECKLIST.md` ファイルと `test.hs` ファイルについては "[issue-1-2-23.md](./issue-1-2-23.md)" に。

 ## comment 2026-11-17 1

 現在、 `hakyll.cabal` ファイルにおいては 4 つのコンポーネントがある。

* `Library`
* `Test-suite hakyll-tests`
* `Executable hakyll-init`
* `Executable hakyll-website`

`Library` コンポーネントは本体だし、 `Test-suite hakyll-tests` コンポーネントは必要である。

でも、 `Executable hakyll-init` コンポーネントと `Executable hakyll-website` コンポーネントの必要性は薄いように見える。

そこで、 `hakiru` パッケージと `hakyll-init` パッケージと `hakyll-website` パッケージに分ける。

`Executable hakyll-init` が `src` ディレクトリを占有しているせいで `Library` が `lib` ディレクトリを使うことになっているのも気に入らないし。

でも、ファイルの大規模な移動をすると `git blame` が有効ではなくなるんだよね。

まあ、古いコミットを参照して `git blame` すればいいし、 "[issue-1-1-2.md](./issue-1-1-2.md)` のために新しいパッケージを作ったりすることを考えると、いつかはファイルを移動しなければならないから、 Hakiru としての履歴を積み立ててから消してしまうよりは良いかも。
