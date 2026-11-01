# issue 18

## comment 2026-11-01 1

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

## comment 2026-11-01 2

`.gitignore` を優先して進める。新しく除外したいファイルがあるので。

以下のテキストを `stack new` から得た。

```gitignore
.stack-work/
*~
```

以下のテキストを "[`Haskell.gitignore`](https://github.com/github/gitignore/blob/6bb5af7a1b050e5add3a71b6e6365c93498822ac/Haskell.gitignore)" から得た。

```gitignore
dist
dist-*
cabal-dev
*.o
*.hi
*.hie
*.chi
*.chs.h
*.dyn_o
*.dyn_hi
.hpc
.hsenv
.cabal-sandbox/
cabal.sandbox.config
*.prof
*.aux
*.hp
*.eventlog
.stack-work/
cabal.project.local
cabal.project.local~
.HTF/
.ghc.environment.*
```

以下のテキストを現在の `.gitignore` から得た。

```gitignore
# Ignore swap files and stack/cabal output.
*.hi
*.o
*.swo
*.swp
*~
.DS_Store
TAGS
dist
tags
cabal.sandbox.config
.cabal-sandbox/
.stack-work
dist
dist-*
.ghc.environment.*

# Ignore test builds.
tests/Main
_cache
_store
_site

# Rest of the file: ignore examples
web/_cache
web/_site
web/site
web/reference

# This is generated from the CHANGELOG
web/releases.markdown
```

以下のファイルは、テストを実行する時と Hakyll を使って Hakyll のウェブサイトを生成する時のファイルなので、テストのコードと Hakyll のウェブサイトを生成するコードに手を入れる時にまでそのままにしておく。

```gitignore
# Ignore test builds.
tests/Main
_cache
_store
_site

# Rest of the file: ignore examples
web/_cache
web/_site
web/site
web/reference

# This is generated from the CHANGELOG
web/releases.markdown
```

以上のファイルを除くと、大体は "[`Haskell.gitignore`](https://github.com/github/gitignore/blob/6bb5af7a1b050e5add3a71b6e6365c93498822ac/Haskell.gitignore)" により賄える。

```gitignore
# elem (load "Haskell.gitignore")
*.hi
*.o
dist
cabal.sandbox.config
.cabal-sandbox/
.stack-work
dist
dist-*
.ghc.environment.*

# notElem (load "Haskell.gitignore")
*.swo
*.swp
*~
.DS_Store
TAGS
tags
```

ようするに、様々な種類のエディタにおける一時ファイルだな。そういうファイルをまとめたやつは "[github/gitignore](https://github.com/github/gitignore)" にないのかな？

"swp" で検索すると "[Add .swp files](https://github.com/github/gitignore/pull/1964)" がある。ようするに "[`Vim.gitignore`](https://github.com/github/gitignore/blob/6bb5af7a1b050e5add3a71b6e6365c93498822ac/Global/Vim.gitignore)" (2024-01) を使えと。これは最新の内容であって、そのコメントがあった当時に参照していたのは、おそらく "[`Vim.gitignore`](https://github.com/github/gitignore/blob/504f444dfa447e39e0205d3db784e31c2698b397/Global/Vim.gitignore)" (2026-02) の方である。

`*~` は "[`Emacs.gitignore`](https://github.com/github/gitignore/blob/6bb5af7a1b050e5add3a71b6e6365c93498822ac/Global/Emacs.gitignore)" にあった。

`.DS_Store` は "[`macOS.gitignore`](https://github.com/github/gitignore/blob/main/Global/macOS.gitignore)"

ううむ……？　そもそも、このようなローカルの環境に依存するようなものを加えるのは不適切なのでは？

でも `~*` は `stack new` の時にもあった。 `stack new` の時に使ったテンプレートは "[`new-template.hsfiles`](https://github.com/commercialhaskell/stack-templates/blob/360442dbdc3d114df5fe377306978d4f47694370/new-template.hsfiles#L130)" だね。

```gitignore
# Use https://github.com/github/gitignore/blob/6bb5af7a1b050e5add3a71b6e6365c93498822ac/Haskell.gitignore
dist
dist-*
cabal-dev
*.o
*.hi
*.hie
*.chi
*.chs.h
*.dyn_o
*.dyn_hi
.hpc
.hsenv
.cabal-sandbox/
cabal.sandbox.config
*.prof
*.aux
*.hp
*.eventlog
.stack-work/
cabal.project.local
cabal.project.local~
.HTF/
.ghc.environment.*

# Use https://github.com/commercialhaskell/stack-templates/blob/360442dbdc3d114df5fe377306978d4f47694370/new-template.hsfiles#L130
.stack-work/
*~

# Ignore test builds.
tests/Main
_cache
_store
_site

# Rest of the file: ignore examples
web/_cache
web/_site
web/site
web/reference

# This is generated from the CHANGELOG
web/releases.markdown
```

こんな感じでいいかな。
