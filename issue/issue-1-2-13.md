# issue 13

## comment 2026-10-23 1

Hakyll のコードの中には Hakyll 自身さえ対応していないバージョンの GHC に対応するための CPP が入っているので削除しておこう。

## comment 2026-10-24 1

`hexirp-hakyll` ライブラリを開発している時に、そういうコードに遭遇したという記憶はあるんだけど、どこだったかは思い出せない。

「現実だったかな……？」と思ったので、調査してみる。

`hexirp-hakyll` ライブラリにおいて CPP を除去したコミット（[リンク](https://github.com/Hexirp/hexirp-hakyll/commit/5ce2aa39800c6bc787e2a619d1deea6262f62281)）を参考にして、それぞれの分岐の条件になっているライブラリのバージョンを、その時期における Cabal ファイル（[リンク](https://github.com/Hexirp/hexirp-hakyll/blob/a312fd4972f9add0736a9f8335bcd51e0e163b06/hakyll.cabal)）にある依存ライブラリのバージョン範囲と照らし合わせる。

"[古いライブラリに対応するためのコードを削除する](https://github.com/Hexirp/hexirp-hakyll/issues/14)" (Hexirp, 2019-11-17) と "[古いライブラリに対応するためのコードを掃除する](https://github.com/Hexirp/hexirp-hakyll/pull/47)" (Hexirp, 2019-11-23) も参考に。

あった。

[`jaspervdj/hakyll`](https://github.com/jaspervdj/hakyll) の [v4.13.0.1](https://github.com/jaspervdj/hakyll/releases/tag/v4.13.0.1) における `Hakyll.Web.Template.Context` モジュール（[リンク](https://github.com/jaspervdj/hakyll/blob/a312fd4972f9add0736a9f8335bcd51e0e163b06/lib/Hakyll/Web/Template/Context.hs)）において以下のようなコード（[リンク](https://github.com/jaspervdj/hakyll/blob/a312fd4972f9add0736a9f8335bcd51e0e163b06/lib/Hakyll/Web/Template/Context.hs#L463-L468)）がある。

```hs
parseTimeM :: Bool -> TimeLocale -> String -> String -> Maybe UTCTime
#if MIN_VERSION_time(1,5,0)
parseTimeM = TF.parseTimeM
#else
parseTimeM _ = TF.parseTime
#endif
```

しかしながら、その時期の Cabal ファイル（[リンク](https://github.com/jaspervdj/hakyll/blob/a312fd4972f9add0736a9f8335bcd51e0e163b06/hakyll.cabal)）における `time` ライブラリ（[リンク](https://hackage.haskell.org/package/time)）のバージョン指定は `time >= 1.8 && < 1.10` となっている。

つまり、どうやっても `MIN_VERSION_time(1,5,0)` が偽になることはないのである。

[`ab9eea7029e2f666a0accce07ce5f5105f5c2155`](https://github.com/jaspervdj/hakyll/commit/ab9eea7029e2f666a0accce07ce5f5105f5c2155) で修正されている。 "[Remove unnecessary conditional compilation for parseTimeM](https://github.com/jaspervdj/hakyll/pull/781)" (L-TChen, 2020-06-08) において。

でも、過去に存在したからとはいって、今において存在しているとは限らない。

現在の Hakyll においても同じようなコードがあるかどうか。

`#if` で検索した結果を Hakiru がフォークした時点の Hakyll の Cabal ファイル（[リンク](https://github.com/jaspervdj/hakyll/blob/092b6a497326ba7b80d17249b6bbd41ca9bf5e19/hakyll.cabal)）と照らし合わせる。

あった。

`Hakyll.Core.File` モジュールにおいて以下のようなコード（[リンク](https://github.com/jaspervdj/hakyll/blob/092b6a497326ba7b80d17249b6bbd41ca9bf5e19/lib/Hakyll/Core/File.hs#L17-L21)）がある。

```hs
#if MIN_VERSION_directory(1,2,6)
import           System.Directory              (copyFileWithMetadata)
#else
import           System.Directory              (copyFile)
#endif
```

しかしながら、現在の Cabal ファイル（[リンク](https://github.com/jaspervdj/hakyll/blob/092b6a497326ba7b80d17249b6bbd41ca9bf5e19/hakyll.cabal)）における `directory` ライブラリ（[リンク](https://hackage.haskell.org/package/directory)）のバージョン指定は `directory >= 1.2.7.0 && < 1.4` となっているため、どうやっても `MIN_VERSION_directory(1,2,6)` が偽になることはない。

`Hakyll.Core.File` モジュールにおいて以下のようなコード（[リンク](https://github.com/jaspervdj/hakyll/blob/092b6a497326ba7b80d17249b6bbd41ca9bf5e19/lib/Hakyll/Core/File.hs#L46-L51)）がある。

```hs
instance Writable CopyFile where
#if MIN_VERSION_directory(1,2,6)
    write dst (Item _ (CopyFile src)) = copyFileWithMetadata src dst
#else
    write dst (Item _ (CopyFile src)) = copyFile src dst
#endif
```

`Hakyll.Core.Provider.Internal` モジュールにおいて以下のようなコード（[リンク](https://github.com/jaspervdj/hakyll/blob/092b6a497326ba7b80d17249b6bbd41ca9bf5e19/lib/Hakyll/Core/Provider/Internal.hs#L39-L43)）がある。

```hs
#if !MIN_VERSION_directory(1,2,0)
import           Data.Time              (readTime)
import           System.Locale          (defaultTimeLocale)
import           System.Time            (formatCalendarTime, toCalendarTime)
#endif
```

`Hakyll.Core.Provider.Internal` モジュールにおいて以下のようなコード（[リンク](https://github.com/jaspervdj/hakyll/blob/092b6a497326ba7b80d17249b6bbd41ca9bf5e19/lib/Hakyll/Core/Provider/Internal.hs#L194-L202)）がある。

```hs
fileModificationTime :: FilePath -> IO UTCTime
fileModificationTime fp = do
#if MIN_VERSION_directory(1,2,0)
    getModificationTime fp
#else
    ct <- toCalendarTime =<< getModificationTime fp
    let str = formatCalendarTime defaultTimeLocale "%s" ct
    return $ readTime defaultTimeLocale "%s" str
#endif
```

33 個あったすべての `#if` ディレクティブを検証した。

さて、これらについては、フォークとか抜きで修正しないと。 GitHub から簡単にプルリクエストを出せたりしないかな？

一つのファイルの編集だけなら大丈夫なんだけど、複数のファイルを一度に編集するとなると、できないっぽい。

ちょっと Hakiru を離れて Hakyll の方でやっておく。
