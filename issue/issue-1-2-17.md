# issue 17

## comment 2026-10-29 1

`git push --force` を使って書き換えた履歴が残っている。

* [`79ad444d59503e7e6986a3354c963a31871ef0e6`](https://github.com/Hexirp/hakiru/commit/79ad444d59503e7e6986a3354c963a31871ef0e6)
* [`fd1769cc62d9819803f639e5b442791f2d7d4aed`](https://github.com/Hexirp/hakiru/commit/fd1769cc62d9819803f639e5b442791f2d7d4aed)

`git gc` で消えるとなると、ドキュメントから参照できなくなるし、これらを `stale-1-1-1` ブランチとか `stale-1-1-2` ブラントとかとして保存するのはどうだろう？

`stale-1-1-1` ブランチとかを push する時には GitHub Actions をオフにする。その後、 push するたびに有効無効を切り替えるのも大変なので、ワークフローのファイルにジョブが始まらないように設定しておく。

GitHub Actions といえば `hakyll-master` ブランチも GitHub Actions をオフにしなければならない。

いや、 `hakyll-master` ブランチにはワークフローのファイルが既に存在しているんだから `main` ブランチのワークフローのファイルの設定で無効にしても意味がない。 Google AI によると Repository Rulesets を使えるらしい。

いや、 `hakyll-master` ブランチにおけるワークフローのファイルの設定は `master` ブランチにおいてのみ実行するという設定になっているから大丈夫なのか。

`main` ブランチにおけるワークフローのファイルの設定でもジョブを実行するブランチをホワイトリストで絞ろうかな。
