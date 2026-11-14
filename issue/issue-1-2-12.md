# issue 12

## comment 2026-10-23 1

LTS 23.28 よりも古い環境におけるサポートは打ち切る。たとえば、バージョンが 9.8.4 未満の GHC とか。

## comment 2026-11-12 1

GHC 9.6 は実質的な LTS になっているから LTS 22.44 を最古のバージョンとするのもありかも。

## comment 2026-11-14 1

"[issue-1-2-10.md](./issue-1-2-10.md)" で言及したように、 `package.yaml` において `build-depends` フィールドにおける依存ライブラリのバージョンの指定を縮める。 LTS 23.28 でビルドしていることを前提として、そこを基点として "[Haskell Package Versioning Policy](https://pvp.haskell.org/)" に従って正常にビルドできることを期待できる範囲にまで広げる。それ以上の範囲については別の resolver でビルドできたら追加する。
