# issue 28

## comment 2026-11-21 1

GitHub Actions のジョブを実行するブランチを制限する。 "[issue-1-2-17.md](./issue-1-2-17.md)" を解く時に `stale-*` というブランチを使ったけど、こういうブランチの時にはジョブを飛ばしたい。

現在の設定は次の通りである。

```yaml
on:
  push:
```

`on` 構文（[リンク](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#on)）を使っている。

`on.push.branches` 構文（[リンク](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#onpushbranchestagsbranches-ignoretags-ignore)）を使えば、特定のブランチのみにおいて実行することができる。

```yaml
on:
  push:
    branches:
      - 'main'
      - 'main-*'
      - 'work'
      - 'work-*'
      - 'issue'
      - 'issue-*'
```

こんな感じかな？

"[Workflow syntax for GitHub Actions § Filter pattern cheat sheet](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#filter-pattern-cheat-sheet)" によれば、アスタリスクは特殊文字なのでエスケープするためにシングルクォーテーションで囲む必要がある。

## comment 2026-11-21 2

`2bec1009230939a3448e885c4c7b2c948b0e32f3` としてコミットした。
