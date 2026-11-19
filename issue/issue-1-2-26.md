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
