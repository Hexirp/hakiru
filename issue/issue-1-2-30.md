# issue 30

## comment 2026-11-23 1

"[issue-1-2-21.md](./issue-1-2-21.md)" において `hakyll` パッケージと `hakyll-init` パッケージと `hakyll-website` パッケージに分けるということについて言及した。

標準のスタイルに近づけたい。ライブラリのソースコードのディレクトリを `lib` ではなく `src` にしたい。テストのソースコードのディレクトリを `tests` ではなく `test` にしたい。 `hakyll-website` に至っては、ほぼ無関係であるように思える。

`Paths_hakyll` モジュールを `hakyll-init` パッケージから利用できなくなるけど、そこは `hakyll` パッケージで適切にエクスポートすればいい。

`hakyll-init` パッケージと `hakyll-website` パッケージは将来において削除したい。その前準備である。

`hakyll` パッケージだけを切り離して、その名前を `hakyll` から `hakiru` に変更したい。その前準備である。
