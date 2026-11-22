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
