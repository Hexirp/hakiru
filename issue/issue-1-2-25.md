# issue 25

## comment 2026-11-19 1

`build-depends` フィールドで `^>=` 記法を使う。

"[issue-1-2-12.md](./issue-1-2-12.md)" で書いたように、たとえば `random` パッケージの `1.2.1.3` バージョンでビルドできているならば、 [Haskell Package Versioning Policy](https://pvp.haskell.org/) を信じれば、 `random >= 1.2.1.3 && 1.3` の範囲でビルドできるはずであり、このように `build-depends` フィールドにおいて指定することにした。

しかし、こうなると `^>=` 記法を使った方が楽である。

だが、 `^>=` 記法を使わない理由もあって、たとえば `random` パッケージの `1.2.1.3` バージョンと `1.3.1` バージョンでビルドできているときに `random >= 1.2.1.3 < 1.4` というように指定したいからであった。

でも、 `build-depends` フィールドのドキュメント（[リンク](https://cabal.readthedocs.io/en/3.18/cabal-package-description-file.html#pkg-field-build-depends)）においては `network ^>= 2.6.3.6 || ^>= 2.7.0.2 || ^>= 2.8.0.0 || ^>= 3.0.1.0` というようなサンプルがある。

たしかに [Haskell Package Versioning Policy](https://pvp.haskell.org/) を守るならば正解ではある。

しかし、ところどころに隙間があって、現在において選んでいる可能な限り依存ライブラリの再ビルドを避けるために枯れた resolver を使う方針では、とりわけ隙間が大きくなってしまうという不安がある。

でも……

やはり [Haskell Package Versioning Policy](https://pvp.haskell.org/) を守るならば、その責任を負うのは依存ライブラリとなる。そうでなければ、こちら側の責任となる。

`^>=` 記法を使うことにするか。

"[Cookbook for common build-failures § Best practice for managing meta-data](https://github.com/haskell-infra/hackage-trustees/blob/98e67ce2f5febf3ea7d47eee9d411daf1e423485/cookbook.md#best-practice-for-managing-meta-data)" でも [Haskell Package Versioning Policy](https://pvp.haskell.org/) に従って不都合があったら revision を更新するようにと書いてあった。
