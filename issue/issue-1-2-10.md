# issue 10

## comment 2026-10-22 1

Cabal のファイル (`hakyll.cabal`) から Hpack のファイル (`package.yaml`) へと移行する。

Stack を使って別のプロジェクトを生成して、現在の Stack のテンプレートにはどのような項目が入っているのか確かめてから、それに合わせて変更する。

## comment 2026-10-24 1

以下のコマンドを使って別の場所で生成して `package.yaml` を採取した。

```sh
stack new hakiru new-template -p "author-email:https://github.com/Hexirp/hakiru" -p "author-name:Hexirp" -p "category:Web" -p "copyright:(C) Hexirp 2026" -p "github-username:Hexirp"
```
