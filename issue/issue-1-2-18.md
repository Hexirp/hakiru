# issue 18

## comment 2026-11-01

`stack new` が作成するプロジェクトに構成を近づける。

ようするに、以下のコマンドを使って生成したファイルに近づけるということである。

```sh
stack new hakiru new-template -p "author-email:https://github.com/Hexirp/hakiru" -p "author-name:Hexirp" -p "category:Web" -p "copyright:(C) Hexirp 2026" -p "github-username:Hexirp"
```

以下のファイルが対象である。

* `.gitignore`
* `CHANGELOG.md`
* `LICENSE`
* `package.yaml`
* `README.md`
* `Setup.hs`
* `stack.yaml`

`package.yaml` ファイルについては、おそらく、この中で最も重いタスクである。

`package.yaml` ファイルについては "[issue-1-2-10.md](./issue-1-2-10.md)" において管理している。

"[issue-1-2-10.md](./issue-1-2-10.md)" に依存している。
