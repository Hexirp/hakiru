# issue 27

## comment 2026-11-20 1

GHC のオプションを `stack new` のテンプレートに合わせる。

現在の GHC のオプションは次の通りである。

```cabal
Library
  Ghc-Options:      -Wall
```

`stack new` のテンプレートにおける GHC のオプションは次の通りである。

```yaml
ghc-options:
  - -Wall
  - -Wcompat
  - -Widentities
  - -Wincomplete-record-updates
  - -Wincomplete-uni-patterns
  - -Wmissing-export-lists
  - -Wmissing-home-modules
  - -Wpartial-fields
  - -Wredundant-constraints
```

## comment 2026-11-23 1

`9a9de4cd96199419ce6d234dc13f4ed8a6704966` としてコミットした。
