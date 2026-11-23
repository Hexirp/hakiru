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

コメントを信じれば `tasty-1.5.3` だから失敗しないはずなんだけどなー。

やっぱり並列処理のバグは怖えや。

さて、こうなると、前に考察した通り `sequentialTestGroup` 関数（[リンク](https://hackage-content.haskell.org/package/tasty-1.5.3/docs/Test-Tasty.html#v:sequentialTestGroup)）を使わないといけない。

そういや LTS 22.44 だと `tasty-1.4.3` になるから `sequentialTestGroup` 関数（[リンク](https://hackage-content.haskell.org/package/tasty-1.5.3/docs/Test-Tasty.html#v:sequentialTestGroup)）も `inOrderTestGroup` 関数（[リンク](https://hackage-content.haskell.org/package/tasty-1.5.5/docs/Test-Tasty.html#v:inOrderTestGroup)）も使えないのか。

とりあえず　`tests/TestSuite.hs` を見てみるけど `sequentialTestGroup` 関数（[リンク](https://hackage-content.haskell.org/package/tasty-1.5.3/docs/Test-Tasty.html#v:sequentialTestGroup)）を使うのは最小限にしたいんだよね。そうしないと並列処理ができないから……

でも、それぞれのモジュールの中にあるテスト同士で干渉し合っているし、モジュールを跨いだテスト同士でも干渉し合っているみたいなんだよね。となると、問題が発生した、それぞれのモジュールにおいて逐次処理するようにして、 `tests/TestSuite.hs` でも逐次処理するという感じにするしかないかな？　`Hakyll.Core.Identifier.Tests` とかでは問題が発生していなかったから、並列処理のままでも大丈夫であるはず。

## comment 2026-11-23 4

`sequentialTestGroup` 関数（[リンク](https://hackage-content.haskell.org/package/tasty-1.5.3/docs/Test-Tasty.html#v:sequentialTestGroup)）に置き換えた。モジュールの内部において一つのテストケースしか定義されていない場合には、モジュールの内部における干渉が発生しないということだから、 `tests/TestGroup.hs` において逐次処理して他のモジュールとの干渉を防ぐだけで成功するようになったけど、並列処理ができないコードということで、今後において別のテストケースを追加することも考えて、 `sequentialTestGroup` 関数（[リンク](https://hackage-content.haskell.org/package/tasty-1.5.3/docs/Test-Tasty.html#v:sequentialTestGroup)）に置き換えておく。

これで一件落着と思いきや `Hakyll.Core.Runtime.Tests` において新しいパターンの失敗をするようになった。どうしてこうなるのか私の理解を超えている。

```txt
  Hakyll.Core.Runtime.Tests
    01_run:                    OK (0.12s)
    02_run:                    OK (0.01s)
    03_run:                    FAIL
      tests\Hakyll\Core\Runtime\Tests.hs:140:
      expected: 1
       but got: 9
      Use -p '/Hakyll.Core.Runtime.Tests.03_run/' to rerun this test only.
    04_run:                    FAIL
      tests\Hakyll\Core\Runtime\Tests.hs:172:
      expected: 1
       but got: 12
      Use -p '/Hakyll.Core.Runtime.Tests.04_run/' to rerun this test only.
    05_run:                    OK (0.02s)
    06_run:                    OK (0.01s)
    07_run:                    OK (0.02s)
    08_run:                    OK
```

依存関係の循環を検出する処理に対するテストで、具体的には `Hakyll.Core.Runtime` の `run` 関数をテストしているんだけど、なぜか依存関係の循環の検出の結果が一致しない。

具体的には、依存関係の循環が発生しているのは 1 つだけであるはずなのに 9 つとか 12 つとかに増えている。

もしかしたら並列処理すると `Hakyll.Core.Runtime` の `run` 関数の結果が変わるという可能性が……？

ただ単に、普通は依存関係の循環が発生しないように気を付けるし、依存関係の循環を検出できないとか誤検出してしまうとかそういうタイプのバグじゃなくて、依存関係の循環の有無を正しく検出できるけど内訳を間違えるというタイプのバグだから、今まで気付かれていなかっただけで……

非常にデバッグしづらい。やっぱり effect system は必要なんじゃない？　いや、 DI を考えるなら `ReaderT IO` パターンでもいいかもしれないけど。

## comment 2026-11-23 5

以下のログにあるように CI が失敗した。

* build #37
  * [Build os-ubuntu-latest ghc-9.8.4](./issue-1-2-26/logs_122071162680/1_Build%20os-ubuntu-latest%20ghc-9.8.4.txt)
  * [Build os-windows-latest ghc-9.8.4](./issue-1-2-26/logs_122071162680/0_Build%20os-windows-latest%20ghc-9.8.4.txt)

Linux において失敗している。

```txt
  Hakyll.Core.Runtime.Tests
    01_run:                    OK
    02_run:                    OK
    03_run:                    FAIL
      tests/Hakyll/Core/Runtime/Tests.hs:140:
      expected: 1
       but got: 3
      Use -p '/Hakyll.Core.Runtime.Tests.03_run/' to rerun this test only.
    04_run:                    FAIL
      tests/Hakyll/Core/Runtime/Tests.hs:172:
      expected: 1
       but got: 4
      Use -p '/Hakyll.Core.Runtime.Tests.04_run/' to rerun this test only.
    05_run:                    OK
    06_run:                    OK
    07_run:                    FAIL
      tests/Hakyll/Core/Runtime/Tests.hs:263:
      missing 'multiple writes' errors
      Use -p '/Hakyll.Core.Runtime.Tests.07_run/' to rerun this test only.
    08_run:                    OK
```

でも、 Windows においては成功している。

```txt
  Hakyll.Core.Runtime.Tests
    01_run:                    OK (0.12s)
    02_run:                    OK
    03_run:                    OK (0.01s)
    04_run:                    OK
    05_run:                    OK (0.01s)
    06_run:                    OK (0.01s)
    07_run:                    OK (0.01s)
    08_run:                    OK
```

しかしも、どういう失敗をしているのかも、手元の環境と違うし。マジでえぐい。

`Hakyll.Core.Runtime` のソースコードを読むしかないか……

一つ目に怪しいのは `Hakyll.Core.Runtime` の `Scheduler` 型である。

```hs
-- | A Scheduler is a pure representation of work going on, works that needs
-- to be done, and work already done.  Workers can obtain things to do
-- by interacting with the Scheduler, and execute them synchronously or
-- asynchronously.
--
-- All operations on Scheduler look like 'Scheduler -> (Scheduler, a)' and
-- should be used with atomicModifyIORef'.
data Scheduler = Scheduler
    { -- | Items to work on next.  Identifiers may appear multiple times.
      schedulerQueue     :: !(Seq Identifier)
    , -- | Items that we haven't started yet.
      schedulerTodo      :: !(Map Identifier (Compiler SomeItem))
    , -- | Currently processing
      schedulerWorking   :: !(Set Identifier)
    , -- | Finished
      schedulerDone      :: !(Set Identifier)
    , -- | Any snapshots stored.
      schedulerSnapshots :: !(Set (Identifier, Snapshot))
    , -- | Any routed files and who wrote them.  This is used to detect multiple
      -- writes to the same file, which can yield inconsistent results.
      schedulerRoutes    :: !(Map FilePath Identifier)
    , -- | Currently blocked compilers.
      schedulerBlocked   :: !(Set Identifier)
    , -- | Compilers that may resume on triggers
      schedulerTriggers  :: !(Map Identifier (Set Identifier))
    , -- | Number of starved pops; tracking this allows us to start a new
      -- number of threads again later.
      schedulerStarved   :: !Int
    , -- | Dynamic dependency info.
      schedulerFacts     :: !DependencyFacts
    , -- | Errors encountered.
      schedulerErrors    :: ![(Maybe Identifier, String)]
    }
```

ドキュメントにあるように `IORef` 型を使って持ちまわしているから、扱いをミスると並列処理でバグるはず。

二つ目に怪しいのは `Hakyll.Core.Logger` の `inMemLog` 関数である。

```hs
-- | Create a new logger that just stores all the messages, useful for writing
-- tests.
newInMem :: IO (Logger, IO [(Verbosity, String)])
newInMem = do
    ref <- IORef.newIORef []
    pure
        ( Logger
            { string = \vbty msg -> liftIO $ IORef.atomicModifyIORef' ref $
                \msgs -> ((vbty, msg) : msgs, ())
            , flush  = pure ()
            }
        , reverse <$> IORef.readIORef ref
        )
```

`Hakyll.Core.Runtime.Tests` のコードは実行が終了してから `newInMem` 関数で作成した logger の中からログを取り出すという仕組みになっているから、ここが並列処理のせいでバグれば、ログの重複が発生する可能性がある。

三つ目に怪しいのは `IORef` 型と `MVar` 型と `TVar` 型の使い分けのミスである。正直なところ、状態の保持とかログの保持とかの時にどれを使えばいいのか分からないんだけど、これをミスっていたらおかしなことになりそう。特に `IORef` 型ってマルチスレッドに対応していなかったはずだよね。

やっぱりテストの環境は可能な限り本番環境と一致させるべきだよね。そうしないと、本番の環境にだけ発生するバグがあって困る。というわけでテストコンポーネントにおいて `-threaded` が指定されなくなったタイミングを探せば、どの時期からバグが発生するようになったのか絞り込めるはずである。

`git log hakyll.cabal` コマンドで。

あった！

```txt
commit ae55652243351d9b9e58fbeef44633b582ccbb61
Author: Laurent P. René de Cotret <LaurentRDC@users.noreply.github.com>
Date:   Fri Mar 27 16:28:21 2026 -0400

    Disable threaded tests to support tasty-1.5.4 (#1097)
```

`git show ae55652243351d9b9e58fbeef44633b582ccbb61` コマンドで。

```txt
$ git show ae55652243351d9b9e58fbeef44633b582ccbb610
commit ae55652243351d9b9e58fbeef44633b582ccbb61
Author: Laurent P. René de Cotret <LaurentRDC@users.noreply.github.com>
Date:   Fri Mar 27 16:28:21 2026 -0400

    Disable threaded tests to support tasty-1.5.4 (#1097)

diff --git a/CHANGELOG.md b/CHANGELOG.md
index f6ba743..fd3882c 100644
--- a/CHANGELOG.md
+++ b/CHANGELOG.md
@@ -4,12 +4,13 @@ title: Releases

 # Releases

-## Unreleased
+## 4.15.8.0

 - Added support for djot for pandoc 3.1.12+ (#1096)
 - Added support for pandoc 3.9 (#1095)
 - Added support for declaring metadata-only dependencies (#1084).
 - Added support for AsciiDoc for pandoc 3.8.3+ (#1089).
+- Added support for tasty-1.5.4+ by disabling threaded tests.

 ## Hakyll 4.16.7.1 (2025-09-06)

diff --git a/hakyll.cabal b/hakyll.cabal
index 696db4d..dd81715 100644
--- a/hakyll.cabal
+++ b/hakyll.cabal
@@ -1,5 +1,5 @@
 Name:    hakyll
-Version: 4.16.7.1
+Version: 4.16.8.0

 Synopsis: A static website compiler library
 Description:
@@ -263,7 +263,10 @@ Test-suite hakyll-tests
   Type:             exitcode-stdio-1.0
   Hs-source-dirs:   tests
   Main-is:          TestSuite.hs
-  Ghc-options:      -Wall -threaded
+  -- Note that tests are purposefully run without
+  -- -threaded because of their side-effects. Test
+  -- failures became apparent with tasty-1.5.4
+  Ghc-options:      -Wall
   Default-language: Haskell2010

   Other-modules:
```

さて、この後の変更において原因がある……と思いたいけど、環境によっては失敗しないということを思い出した。だから、地雷のように埋まっているかもしれない。

[`db14392f8ee8070261c7e8f1f8d275dfa05b9669`](https://github.com/jaspervdj/hakyll/commit/db14392f8ee8070261c7e8f1f8d275dfa05b9669) のようなコミットも原因かもしれない。

とりあえず、仮定して調査する。

`#1097` というのは "[Disable threaded tests to support tasty-1.5.4](https://github.com/jaspervdj/hakyll/pull/1097)" ね。

`git log ae556522..HEAD` コマンドを試したけど、コミットが多すぎて最後まで辿れないな。

`git rev-list ae556522..HEAD --count` コマンドによると 206 コミットか。

`git log lib\Hakyll\Core\Runtime.hs` コマンドを使って抽出してみる。

うーん…… 2026 年の変更がない。

そもそも、 `IORef` 型を使っていたとしてもシングルスレッドなら大丈夫であるはずで、わざわざ `forkIO` 関数みたいなプリミティブを使わない限りはマルチスレッドにならないはず。となると、並列処理をしているテストコードにおいてのみ問題が発生する可能性がある。となると、 `Hakyll.Core.Runtime` モジュールの `run` 関数じゃなくて `Hakyll.Core.Logger` モジュールの `newInMem` 関数が原因か？

より古いコミットに調査を進める。

[`6e144c0c48983ef67a5d840e4b8448d7f021da28`](https://github.com/jaspervdj/hakyll/commit/6e144c0c48983ef67a5d840e4b8448d7f021da28) において追加された `multiple writes` エラーのテストがこけているんだよな。

あれ？　なんか `async` という言葉がメッセージに入っているコミットがあるな？

[`9696a859da5d5227edb79f08a85739a6f2d4d92a`](https://github.com/jaspervdj/hakyll/commit/9696a859da5d5227edb79f08a85739a6f2d4d92a) において `newInMem` 関数が追加されている。

めっちゃこのコミットが原因っぽい。

うーん？

`Hakyll.Core.Runtime` モジュールの中でも `forkIO` 関数を使っている。

```hs
pickAndChaseAsync :: ReaderT RuntimeRead IO ()
pickAndChaseAsync = do
    runtimeRead <- ask
    numThreads  <- liftIO getNumCapabilities
    let scheduler = runtimeScheduler runtimeRead
    Logger.message (runtimeLogger runtimeRead) $
        "Using async runtime with " <> show numThreads <> " threads..."
    liftIO $ do
        signal     <- MVar.newEmptyMVar

        let spawnN :: Int -> IO ()
            spawnN n = replicateM_ n $ forkIO $
                IORef.atomicModifyIORef' scheduler schedulerPop >>= go

            go :: SchedulerStep -> IO ()
            go step = case step of
                SchedulerFinish       -> void $ MVar.tryPutMVar signal ()
                SchedulerStarve       -> pure ()
                SchedulerError        -> void $ MVar.tryPutMVar signal ()
                (SchedulerWork i c n) -> do
                    spawnN n
                    step' <- runReaderT (work i c) runtimeRead
                    go step'

        spawnN numThreads
        MVar.readMVar signal
```

ようするに `getNumCapabilities` 関数（[リンク](https://hackage-content.haskell.org/package/base-4.22.0.0/docs/Control-Concurrent.html#v:getNumCapabilities)）を使って得た最大スレッド数の分だけ並列処理しているというわけだ。

`getNumCapabilities` 関数（[リンク](https://hackage-content.haskell.org/package/base-4.22.0.0/docs/Control-Concurrent.html#v:getNumCapabilities)）の値には `-with-rtsopts=-N` オプションが作用する。

となると、 `-threaded` オプションを外す前のテストコンポーネントにおいても `-with-rtsopts=-N` オプションがないがゆえに実質的にはシングルスレッドになっていた可能性があるわけだ。

そう考えると、 `-threaded` オプションを外す前においてバグが見えていなかったことが説明できる。

もうお手上げって感じだし、 GitHub Copilot に任せてしまうか？

それか、テストコードの条件を緩めるとか。
