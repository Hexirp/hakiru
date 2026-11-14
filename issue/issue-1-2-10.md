# issue 10

## comment 2026-10-22 1

Cabal のファイル (`hakyll.cabal`) から Hpack のファイル (`package.yaml`) へと移行する。

Stack を使って別のプロジェクトを生成して、現在の Stack のテンプレートにはどのような項目が入っているのか確かめてから、それに合わせて変更する。

## comment 2026-10-24 1

以下のコマンドを使って別の場所で生成して `package.yaml` ファイルを採取した。

```sh
stack new hakiru new-template -p "author-email:https://github.com/Hexirp/hakiru" -p "author-name:Hexirp" -p "category:Web" -p "copyright:(C) Hexirp 2026" -p "github-username:Hexirp"
```

## comment 2026-11-01 1

"[issue-1-1-7.md](./issue-1-1-7.md)" と重複しているから、こっちの方を正とする。こっちの方に具体性があるから。

## comment 2026-11-01 2

"[issue-1-2-18.md](./issue-1-2-18.md)" において `package.yaml` ファイル以外も含む全体について管理している。

`package.yaml` ファイルに関する作業は重くなると予想できるため、ここで特別に管理する。

## comment 2026-11-12 1

以下のコマンドで生成した `package.yaml` ファイルを引っ張ってきたけど、 `spec-version` はどうするべきだろうか？

```sh
stack new hakiru new-template -p "author-email:https://github.com/Hexirp/hakiru" -p "author-name:Hexirp" -p "category:Web" -p "copyright:(C) Hexirp 2026" -p "github-username:Hexirp"
```

[`README.md`](https://github.com/sol/hpack/blob/2db1ac370df9aab08508fc3913394db0bf44ad3b/README.md) にはあんまり情報が載っていない。最新の方法で `Paths_*` モジュールを使いたいなら `spec-version: 0.36.0` を使えとだけ。

一応 "[2. Package description](https://docs.haskellstack.org/en/stable/tutorial/package_description/)" の "Must package components have distinct source file directories?" においては `spec-version: 0.36.0` を使っている。

`data-dir` フィールド周りを指定しようとしているけど、 `data-files` フィールドにおいて `data` ディレクトリの下ではなく `example` ディレクトリの下にあるファイルが指定されているんだけど、これってどういうことなんだろう？

```cabal
Data-Dir:      data

Data-files:
  example/posts/2015-11-28-carpe-diem.markdown
  example/posts/2015-10-07-rosa-rosa-rosam.markdown
  example/posts/2015-12-07-tu-quoque.markdown
  example/posts/2015-08-12-spqr.markdown
  example/site.hs
  example/images/haskell-logo.png
  example/templates/post-list.html
  example/templates/default.html
  example/templates/archive.html
  example/templates/post.html
  example/css/default.css
  example/index.html
  example/about.rst
  example/contact.markdown

Extra-source-files:
  CHANGELOG.md
  tests/data/biblio/chicago.csl
  tests/data/biblio/cites-meijer-pandoc-2.0.0plus.golden
  tests/data/biblio/cites-meijer-pandoc-3.0.0plus.golden
  tests/data/biblio/cites-meijer-pandoc-3.1.8plus.golden
  tests/data/biblio/cites-multiple-pandoc-2.0.0plus.golden
  tests/data/biblio/cites-multiple-pandoc-3.0.0plus.golden
  tests/data/biblio/cites-multiple-pandoc-3.1.8plus.golden
  tests/data/biblio/cites-multiple.markdown
  tests/data/biblio/default.html
  tests/data/biblio/page.markdown
  tests/data/biblio/refs.bib
  tests/data/biblio/refs.yaml
  tests/data/biblio/refs2.yaml
  tests/data/embed.html
  tests/data/example.md
  tests/data/example.md.metadata
  tests/data/images/favicon.ico
  tests/data/just-meta.html
  tests/data/just-meta.html.out
  tests/data/partial-helper.html
  tests/data/partial.html
  tests/data/partial.html.out
  tests/data/posts/2010-08-26-birthday.md
  tests/data/posts/2018-09-26.md
  tests/data/posts/2019/05/10/tomorrow.md
  tests/data/russian.md
  tests/data/strip.html
  tests/data/strip.html.out
  tests/data/template-empty.html
  tests/data/template.html
  tests/data/template.html.out
  data/templates/atom-item.xml
  data/templates/atom.xml
  data/templates/rss-item.xml
  data/templates/rss.xml
  data/templates/feed.json
  data/templates/feed-item.json
```

## comment 2026-11-14 1

`data` ディレクトリの下に `example/about.rst` があるということね。つまり `data/example/about.rst` である。見落としていた。

Cabal の公式ドキュメントを読もう。

`data-files` フィールド（[リンク](https://cabal.readthedocs.io/en/3.18/cabal-package-description-file.html#pkg-field-data-files)）はメインプログラムの実行時に使うファイルを入れるところで。

`data-dir` フィールド（[リンク](https://cabal.readthedocs.io/en/3.18/cabal-package-description-file.html#pkg-field-data-dir)）は `data-files` フィールドで参照するディレクトリの位置で。デフォルトではソースが入っているディレクトリになる。

`extra-source-files` フィールド（[リンク](https://cabal.readthedocs.io/en/3.18/cabal-package-description-file.html#pkg-field-data-dir)）は、 `cabal sdist` の時に入れたいファイルを、つまり一緒に配布したいファイルを入れるところで。 `CHANGELOG.md` とかを入れたりメインプログラムでは使わないけどテストプログラムに使うファイルを入れるところで。

今は `package.yaml` ファイルを編集しているけど、その前に `hakyll.cabal` ファイルを編集した方がいいこともあるような気がしてきた。

たとえば `build-depends` フィールドを編集して、依存ライブラリのバージョンの指定を LTS 23.28 に合わせて縮めるとか。 "[issue-1-2-12.md](./issue-1-2-12.md)" だね。

たとえば、ビルドフラグを編集して全部デフォルトで `True` にするとか。ビルドフラグはあまり良くないと思っているので。ありとあらゆるビルドフラグの組み合わせの全部でコンパイルできることを保証するのは難しいし。とはいっても、最初から全部消すとコードの方も変えないといけないし、全部 `True` にしておく。そうすれば `stack.yaml` ファイルの方でビルドフラグを指定する必要もなくなるし。

## comment 2026-11-14 2

Update: `cabal` ファイルの `build-depends` フィールドを更新する。

LTS 23.28 のみを resolver に指定してビルドしているため、それよりも古いバージョンや新しいメジャーバージョンなどについてビルド可能性を保証することはできない。

そのため、ビルド可能性を保証できる範囲にまで依存ライブラリのバージョンの指定を縮める。

`5b3d545018f68ccf356e7ad34262ba24d8de2481` としてコミットした。

## comment 2026-11-14 3

`cabal` ファイルに手を入れて分かったけど、 `build-depends` フィールドの重複が多いのは変更するときに大変である。やはり `package.yaml` ファイルへの移行は急務である。

## comment 2026-11-14 4

ビルドフラグに手を入れるのは、ようするに `package.yaml` に移行する前に `cabal` ファイルの内容を減らして移行しやすくするということである。
