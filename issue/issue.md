# issue

## issue 1

Use: [issue-1-1-1.md](./issue-1-1-1.md)

Status: Open

Depends: [issue-1-2-21.md](./issue-1-2-21.md)

"Hakyll" を "Hakiru" に置き換える。

## issue 2

Use: [issue-1-1-2.md](./issue-1-1-2.md)

Status: Open

issue のファイルの名前には以下の形式を用いる。

* `issue-1-1-1.md`
* `issue-1-1-2.md`
* `issue-1-1-3.md`
* …
* `issue-1-1-9.md`
* `issue-1-2-10.md`
* `issue-1-2-11.md`
* …
* `issue-1-2-99.md`
* `issue-1-3-100.md`
* `issue-1-3-101.md`
* …
* `issue-1-9-999999999.md`
* `issue-2-10-1000000000.md`
* `issue-2-10-1000000001.md`
* …
* `issue-2-10-9999999999.md`
* `issue-2-11-10000000000.md`
* `issue-2-11-10000000001.md`
* …
* `issue-2-99-{{ 10 ^ 99 - 1 }}.md`
* `issue-3-100-{{ 10 ^ 99 }}.md`
* `issue-3-100-{{ 10 ^ 99 + 1 }}.md`
* …
* `issue-3-100-{{ 10 ^ 100 - 1 }}.md`
* `issue-3-101-{{ 10 ^ 100 }}.md`
* `issue-3-101-{{ 10 ^ 100 + 1 }}.md`
* …
* `issue-3-999-{{ 10 ^ 999 - 1 }}.md`
* `issue-4-1000-{{ 10 ^ 999 }}.md`
* `issue-4-1000-{{ 10 ^ 999 + 1 }}.md`
* …
* `issue-8-99999999-{{ 10 ^ 99999999 - 1 }}.md`
* `issue-90-100000000-{{ 10 ^ 99999999 }}.md`
* `issue-90-100000000-{{ 10 ^ 99999999 + 1 }}.md`
* …
* `issue-90-999999999-{{ 10 ^ 999999999 - 1 }}.md`
* `issue-91-1000000000-{{ 10 ^ 999999999 }}.md`
* `issue-91-1000000000-{{ 10 ^ 999999999 + 1 }}.md`
* …
* `issue-98-99999999999999999-{{ 10 ^ 99999999999999999 - 1 }}.md`
* `issue-990-100000000000000000-{{ 10 ^ 99999999999999999 }}.md`
* `issue-990-100000000000000000-{{ 10 ^ 99999999999999999 + 1 }}.md`

## issue 3

Use: [issue-1-1-3.md](./issue-1-1-3.md)

Status: Resolved

コンパイラのバージョンと依存ライブラリのバージョンを固定する。

## issue 4

Use: [issue-1-1-4.md](./issue-1-1-4.md)

Status: Open

依存ライブラリを更新する。

## issue 5

Use: [issue-1-1-3.md](./issue-1-1-3.md)

Status: Resolved

Depends: [issue-1-2-16.md](./issue-1-2-16.md)

GitHub Actions を使って CI を実行する。

## issue 6

Use: [issue-1-1-3.md](./issue-1-1-3.md)

Status: Resolved

issue のファイルは以下の構造を持つ。見出しは木構造を作る。

* issue のファイルは次の構造を持つ。
  * "issue" と issue の番号を並べた見出し。たとえば "issue 6" というようになる。
  * 要約。
  * 複数の part の列挙、または、複数の comment の列挙。
* part は次の構造を持つ。
  * "part" と part の番号を並べた見出し。たとえば "part 2.3.1" というようになる。
  * 要約。
  * 複数の part の列挙、または、複数の comment の列挙。
* comment は次の構造を持つ。
  * "comment" と comment の日付と comment の番号を並べた見出し。たとえば "comment 2026-11-14 3" というようになる。
  * 本文。

## issue 7

Use: [issue-1-1-7.md](./issue-1-1-7.md)

Status: Duplicated ([issue-1-2-10.md](./issue-1-2-10.md))

## issue 8

Use: [issue-1-1-8.md](./issue-1-1-8.md)

Status: Open

ローカルの環境でビルドとテストを通す。

## issue 9

Use: [issue-1-1-9.md](./issue-1-1-9.md)

Status: Duplicated ([issue-1-1-5.md](./issue-1-1-5.md))

## issue 10

Use: [issue-1-2-10.md](./issue-1-2-10.md)

Status: Open

Depends: [issue-1-2-12.md](./issue-1-2-12.md), [issue-1-2-22.md](./issue-1-2-22.md), [issue-1-2-24.md](./issue-1-2-24.md)

Cabal から Hpack に移行する。 `*.cabal` ファイルから `package.yaml` ファイルに移行する。

## issue 11

Use: [issue-1-2-11.md](./issue-1-2-11.md)

Status: Open

`GHC2024` エディションに移行する。

## issue 12

Use: [issue-1-2-12.md](./issue-1-2-12.md)

Status: Resolved

古いバージョンの依存ライブラリに対するサポートを打ち切る。

## issue 13

Use: [issue-1-2-13.md](./issue-1-2-13.md)

Status: Open

Hakyll が対応していない古いバージョンの依存ライブラリに対応する `CPP` 言語拡張のプリプロセッサが残留しているため、それらを削除する。

## issue 14

Use: [issue-1-2-14.md](./issue-1-2-14.md)

Status: Resolved

GitHub に公開する。

## issue 15

Use: [issue-1-2-15.md](./issue-1-2-15.md)

Status: Resolved

Visual Studio Code の Haskell 拡張を通して Haskell Language Server を使えるようにする。

## issue 16

Use: [issue-1-2-16.md](./issue-1-2-16.md)

Status: Closed

GitHub Actions の MacOS の環境においてビルドするジョブを作る。

## issue 17

Use: [issue-1-2-17.md](./issue-1-2-17.md)

Status: Resolved

書き換える前の Git 履歴を別のブランチとして残す。

## issue 18

Use: [issue-1-2-18.md](./issue-1-2-18.md)

Status: Open

Depends: [issue-1-2-10.md](./issue-1-2-10.md)

プロジェクトの構成を `stack new` で作成したプロジェクトの構成に近づける。具体的には以下のファイルが対象である。

* `.gitignore`
* `CHANGELOG.md`
* `LICENSE`
* `package.yaml`
* `README.md`
* `Setup.hs`
* `stack.yaml`

## issue 19

Use: [issue-1-2-19.md](./issue-1-2-19.md)

Status: Open

最新のバージョンの依存ライブラリを使ってビルドできるかどうかを確かめる GitHub Actions のジョブを作る。

## issue 20

Use: [issue-1-2-20.md](./issue-1-2-20.md)

Status: Open

`preview` サブコマンドを deprecated にしておくのは正しいのかどうかを検討する。

## issue 21

Use: [issue-1-2-21.md](./issue-1-2-21.md)

Status: Open

Depends: [issue-1-2-10.md](./issue-1-2-10.md), [issue-1-2-18.md](./issue-1-2-18.md), [issue-1-2-23.md](./issue-1-2-23.md)

Hakiru 0.1.0.0 をリリースするために必要である作業を終わらせる。

## issue 22

Use: [issue-1-2-22.md](./issue-1-2-22.md)

Status: Resolved

ライブラリのフラグを削除する。

## issue 23

Use: [issue-1-2-23.md](./issue-1-2-23.md)

Status: Open

プロジェクトのルートディレクトリにあるファイルを整理する。具体的には以下のファイルを整理する。

* `.ghci`
* `.hlint.yaml`
* `CODE_OF_CONDUCT.md`
* `logo.svg`
* `RELEASE-CHECKLIST.md`
* `test.hs`

## issue 24

Use: [issue-1-2-24.md](./issue-1-2-24.md)

Status: Resolved

ライブラリのモジュールを全て公開する。

## issue 25

Use: [issue-1-2-25.md](./issue-1-2-25.md)

Status: Resolved

依存ライブラリのバージョンを指定する時にキャレット演算子を使う。

## issue 26

Use: [issue-1-2-26.md](./issue-1-2-26.md)

Status: Open

テストスイートのコンポーネントにおいて `-threaded` オプションを有効にする。

## issue 27

Use: [issue-1-2-27.md](./issue-1-2-27.md)

Status: Open

GHC のオプションの選択を `stack new` のテンプレートに合わせる。

## issue 28

Use: [issue-1-2-28.md](./issue-1-2-28.md)

Status: Resolved

GitHub Actions のジョブを実行するブランチを制限する。

## issue 29

Use: [issue-1-2-29.md](./issue-1-2-29.md)

Status: Open

複数の GHC のバージョンに対応する。
