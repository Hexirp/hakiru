# issue 23

## comment 2026-11-14 1

プロジェクトのルートディレクトリにあるファイルを整理する。

つまり、以下のファイルを整理する。

* `.ghci`
* `.hlint.yaml`
* `CODE_OF_CONDUCT.md`
* `logo.svg`
* `RELEASE-CHECKLIST.md`
* `test.hs`

"[issue-1-2-10.md](./issue-1-2-10.md)" と "[issue-1-2-22.md](./issue-1-2-22.md)" のために `hakyll.cabal` ファイルを編集しているとき、気になっていたんだけど、 `test.hs` ファイルを使っている形跡がない。

`other-modules` フィールドのドキュメント（[リンク](https://cabal.readthedocs.io/en/3.18/cabal-package-description-file.html#pkg-field-other-modules)）によると、パッケージで使っているモジュールは `other-modules` フィールド（[リンク](https://cabal.readthedocs.io/en/3.18/cabal-package-description-file.html#pkg-field-other-modules)）と `library:exposed-modules` フィールド（[リンク](https://cabal.readthedocs.io/en/3.18/cabal-package-description-file.html#pkg-field-library-exposed-modules)）と `executable:main-is` フィールド（[リンク](https://cabal.readthedocs.io/en/3.18/cabal-package-description-file.html#pkg-field-executable-main-is)）のどれかに入っている必要があると。

`Library` セクションは `lib` ディレクトリを使っている。

`Test-suite hakyll-tests` セクションは `tests` ディレクトリを使っている。 `main-is` フィールドは `tests/TestSuite.hs` ファイルを指定している。

`Executable hakyll-init` セクションは `src` ディレクトリを使っている。 `main-is` フィールドは `src/Init.hs` ファイルを指定している。

`Executable hakyll-website` セクションは `web` ディレクトリを使っている。 `main-is` フィールドは `web/site.hs` ファイルを指定している。

どこにも `test.hs` ファイルを指定しているところがないのである。

`test.hs` ファイルの内容を読んでみよう。

```hs
main :: IO ()
main = hakyll $ do
    fileHashes <- preprocess (mkFileHashes "images")
    undefined
```

こういう内容になっていた。 `undefined` 関数を使っていて、絶対に失敗するようになっている。まさか、まだ未完成のソースコード？

……なんか見てはいけないものを見てしまった気分である。

[@jaspervdj](https://github.com/jaspervdj) さんが [`030864a921d38405778ddeb1286e8922955c8ab4`](https://github.com/Hexirp/hakiru/commit/030864a921d38405778ddeb1286e8922955c8ab4) において追加しているけど、その意図は不明である。
