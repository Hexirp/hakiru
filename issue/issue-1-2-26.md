# issue 26

## comment 2026-11-19 1

並列処理するとテストが失敗するからといってテストコンポーネントのオプションに `-threaded` オプションがない件について。

```cabal
Test-suite hakyll-tests
  Type:             exitcode-stdio-1.0
  Hs-source-dirs:   tests
  Main-is:          TestSuite.hs
  -- Note that tests are purposefully run without
  -- -threaded because of their side-effects. Test
  -- failures became apparent with tasty-1.5.4
  Ghc-options:      -Wall
  Default-language: Haskell2010
```

`stack new` コマンドのテンプレートに合わせたいから `-threaded` オプションを追加したい。

並列処理が無理なら、テストコードの中において逐次処理することを強制すればいいんじゃない？

`tasty` ライブラリ（[リンク](https://hackage-content.haskell.org/package/tasty-1.5.5)）を使っているから、たぶん `dependentTestGroup` 関数（[リンク](https://hackage-content.haskell.org/package/tasty-1.5.5/docs/Test-Tasty.html#v:dependentTestGroup)）が使える。

## comment 2026-11-20 1

`stack new` のテンプレートにおいては、次のようになっている。

```yaml
ghc-options:
  - -threaded
  - -rtsopts
  - -with-rtsopts=-N
```

現在において resolver は LTS 23.28 を使っていて、その時の `tasty` ライブラリは `1.5.3` バージョンだから `sequentialTestGroup` 関数（[リンク](https://hackage-content.haskell.org/package/tasty-1.5.3/docs/Test-Tasty.html#v:sequentialTestGroup)）を使わないといけない。

あと、 `tasty` ライブラリの `1.5.5` バージョンにおいても `inOrderTestGroup` 関数（[リンク](https://hackage-content.haskell.org/package/tasty-1.5.5/docs/Test-Tasty.html#v:inOrderTestGroup)）の方が適切っぽい。

## comment 2026-11-23 1

`tasty-1.5.4` 以降で失敗すると書いてある。でも、今の LTS 23.28 リソルバーだと `tasty-1.5.3` を使っているから、問題はないのでは？

"[issue-1-2-29.md](./issue-1-2-29.md)" において困るかもしれないけど、そうするのはずいぶんと先になるだろうし、今はとりあえず合わせておけばいいのでは？

## comment 2026-11-23 2

`bafc4d1b5fe6e5e23c2a148f3f3c1501f7c587d9` としてコミットした。

## comment 2026-11-23 3

以下のログにあるように CI が失敗した。

* build #36
  * [Build os-ubuntu-latest ghc-9.8.4](./issue-1-2-26/logs_121889878726/1_Build%20os-ubuntu-latest%20ghc-9.8.4.txt)
  * [Build os-windows-latest ghc-9.8.4](./issue-1-2-26/logs_121889878726/0_Build%20os-windows-latest%20ghc-9.8.4.txt)

いきなりわっとたくさんのエラーがでてきて、ちょっとパニックになるけど、少しずつ整理しよう。

まず、モジュール単位で数えると、 Linux において失敗したのは以下のテストである。

* `Hakyll.Core.Provider.Tests`
* `Hakyll.Core.Routes.Tests`
* `Hakyll.Core.Rules.Tests`
* `Hakyll.Core.Runtime.Tests`
* `Hakyll.Core.Store.Tests`
* `Hakyll.Core.UnixFilter.Tests`
* `Hakyll.Web.Pandoc.Biblio.Tests`
* `Hakyll.Web.Tags`
* `Hakyll.Web.Template.Context.Tests`
* `Hakyll.Web.Template.Tests`
* `Hakyll.Web.Feed.Tests`

これらのすべてが `_teststore` ディレクトリを起因にしている。

Windows において失敗したのは以下のテストである。

* `Hakyll.Core.Routes.Tests`
* `Hakyll.Core.Runtime.Tests`
* `Hakyll.Core.Store.Tests`
* `Hakyll.Web.Pandoc.Biblio.Tests`
* `Hakyll.Web.Template.Tests`
* `Hakyll.Web.Feed.Tests`

特に `Hakyll.Core.Store.Tests` のテストの結果は文字化けしてしまっている。

手元の環境だと何事もなく成功したんだけどなー。

やっぱり並列処理のバグは怖えや。

さて、こうなると、前に考察した通り `sequentialTestGroup` 関数（[リンク](https://hackage-content.haskell.org/package/tasty-1.5.3/docs/Test-Tasty.html#v:sequentialTestGroup)）を使わないといけない。

そういや LTS 22.44 だと `tasty-1.4.3` になるから `sequentialTestGroup` 関数（[リンク](https://hackage-content.haskell.org/package/tasty-1.5.3/docs/Test-Tasty.html#v:sequentialTestGroup)）も `inOrderTestGroup` 関数（[リンク](https://hackage-content.haskell.org/package/tasty-1.5.5/docs/Test-Tasty.html#v:inOrderTestGroup)）も使えないのか。

とりあえず　`tests/TestSuite.hs` を見てみるけど `sequentialTestGroup` 関数（[リンク](https://hackage-content.haskell.org/package/tasty-1.5.3/docs/Test-Tasty.html#v:sequentialTestGroup)）を使うのは最小限にしたいんだよね。そうしないと並列処理ができないから……

でも、それぞれのモジュールの中にあるテスト同士で干渉し合っているし、モジュールを跨いだテスト同士でも干渉し合っているみたいなんだよね。となると、問題が発生した、それぞれのモジュールにおいて逐次処理するようにして、 `tests/TestSuite.hs` でも逐次処理するという感じにするしかないかな？　`Hakyll.Core.Identifier.Tests` とかでは問題が発生していなかったから、並列処理のままでも大丈夫であるはず。
