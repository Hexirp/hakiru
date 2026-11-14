# issue 24

## comment 2026-11-14 1

現在の `cabal` ファイルにおいては `other-modules` フィールドにかなりの数のモジュールが含まれている。

```cabal
  Other-Modules:
    Data.List.Extended
    Data.Yaml.Extended
    Hakyll.Check
    Hakyll.Core.Compiler.Require
    Hakyll.Core.Identifier.Pattern.Internal
    Hakyll.Core.Item.SomeItem
    Hakyll.Core.Provider.Internal
    Hakyll.Core.Provider.MetadataCache
    Hakyll.Core.Util.Parser
    Hakyll.Preview.Poll
    Hakyll.Preview.Server
    Hakyll.Web.Pandoc.Binary
    Paths_hakyll
```

私は、 Hackage においてコードの内容を読む時に困った経験があるため、このように internal なモジュールを公開しないというのを好んでいない。

このため、これらを `Paths_hakyll` を除いて `exposed-modules` フィールドに移す。

## comment 2026-11-14 2

`60e27384ebe73585445a7e474e393ced282c0e5f` としてコミットした。
