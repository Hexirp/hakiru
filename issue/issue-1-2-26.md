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

## comment 2026-11-23 6

`9696a859da5d5227edb79f08a85739a6f2d4d92a` のコミットメッセージ。

```txt
commit 9696a859da5d5227edb79f08a85739a6f2d4d92a
Author: Jasper Van der Jeugt <m@jaspervdj.be>
Date:   Wed Aug 23 19:24:20 2023 +0200

    Improve async runtime scaling (#946)
```

`#946` は "[Improve async runtime scaling](https://github.com/jaspervdj/hakyll/pull/946)" のことね。

`github copilot バグ修正` で検索した時の Google AI によると、 GitHub Copilot にバグを直させるためには、まず `@workspace /explain` を使って原因を探させて `/fix` で直させるって感じらしい。 `@workspace` は、ワークスペースにあるすべてのファイルを読み込ませるコマンドである。

えっと、この使い方はなんかよくわからないけどできないみたい。 `issue/issue-1-2-26.md` ファイルと `lib/Hakyll/Core/Runtime.hs` ファイルと `tests/Hakyll/Core/Runtime/Tests.hs` ファイルを読み込ませて、原因を分析させることにする。

```txt
Hakyll.Core.Runtime.Tests が失敗する理由を解明しなさい。
```

## comment 2026-11-23 7

Author: GitHub Copilot

### やり取りの記録

以下に、質問から回答までの確認可能な発言と作業経過を、順序を保って記録する。コマンド出力は調査結果を確認できるよう要点を残し、ファイルの内容全体を再掲する代わりに対象と確認箇所を示す。

#### 入力

```txt
Hakyll.Core.Runtime.Tests が失敗する理由を解明しなさい。
```

調査対象として指定されたファイルは次の三つ。

* `issue/issue-1-2-26.md`
* `lib/Hakyll/Core/Runtime.hs`
* `tests/Hakyll/Core/Runtime/Tests.hs`

#### 発言

調査開始時の進行説明：

> 失敗の再現条件と、テスト・実装・関連 issue 記述の食い違いを照合します。まず対象ファイルと作業ツリーの状態を確認します。

テスト構成・スケジューラの調査に移る際の進行説明：

> この実装では、テストは依存循環の「検出回数」を厳密に数え、スケジューラは `forkIO` で実行されます。次に、テストランナーの実行方式・共有テスト環境・CI の実行条件を確認し、単なるログ重複と実際の並行スケジューラ不具合を切り分けます。

回答をまとめる前の進行説明：

> 原因候補は絞れました。単にテストケース同士の並列実行ではなく、`run` 自体の非同期ワーカーが再現に関与しています。最後に、テスト実行で作業ツリーの変更が増えていないことを確認し、再現結果とコード上の根拠をまとめます。

#### 作業経過

最初に対象ファイルと作業ツリーを確認した。issue ファイルと Runtime 実装は一度の表示で読める上限を超えたため、issue の各コメントと Runtime の複数の範囲に分けて読み直した。Runtime.Tests は全体を確認した。

作業開始時の `git status --short` には `M issue/issue-1-2-26.md` が表示されていた。既存の変更を戻さず、調査のみを行った。

issue の記述から、CI build #37 で Linux の `Hakyll.Core.Runtime.Tests` に次の失敗があったことを確認した。

* 03_run: 循環エラーのログ数が `expected: 1` に対して `got: 3`
* 04_run: 循環エラーのログ数が `expected: 1` に対して `got: 4`
* 07_run: `missing 'multiple writes' errors`
* Windows の同じ build では Runtime.Tests は成功

さらに前の build #36 では 03_run と04_runのログ数がそれぞれ 9 と12になっていたという記録も確認した。

`tests/TestSuite.hs` と `tests/TestSuite/Util.hs` を確認した。テストスイート全体は `sequentialTestGroup "Hakyll" AllFinish`、Runtime.Tests 内部も `sequentialTestGroup` を使う。テスト環境は `_testsite`、`_teststore`、`_testtmp` を利用し、`cleanTestEnv` で後片付けする。

`hakyll.cabal` と `package.yaml` のテスト設定を検索した。現在の設定には `-threaded -rtsopts -with-rtsopts=-N` がある。Runtime の `build` は `rtsSupportsBoundThreads` が真なら同期版でなく `pickAndChaseAsync` を選び、その関数は `getNumCapabilities` 個のワーカーを `forkIO` で起動する。

Runtime のスケジューラを追った。`schedulerPop` はキューが空、処理中がなく、ブロック中の項目があると循環エラーを `schedulerErrors` に追加する。一方、その状態を「報告済み」に変更する処理はない。Runtime.Tests の case03/case04 は循環エラーに該当するログの数がちょうど1であることを検証する。

`pickAndChaseAsync` は各ワーカーが `SchedulerError` または `SchedulerFinish` を返した際に `MVar.tryPutMVar` で通知し、親側は `MVar.readMVar` で通知を一つ受け取る。起動したワーカーをすべて待ち合わせる処理は見当たらなかった。`work` ではコンパイラーエラーも `SchedulerError` として返す。

`schedulerRoute` は `atomicModifyIORef'` で route map を更新し、同じ route に異なる Identifier が現れたら `"multiple writes"` エラーを記録する。Logger の in-memory 実装も `atomicModifyIORef'` を利用する。このため、「IORef は並列に弱い」という一般論だけでは、観測した件数増加を説明できないと判断した。

Stack でテストを試した。最初のコマンドは存在しないコンポーネント名を指定して失敗した。

```powershell
stack test hakyll:test --test-arguments "-p /Hakyll.Core.Runtime.Tests/"
```

出力は `Component test does not exist in package hakyll.`。テストスイート名を指定し直した。

次のコマンドは実行できたが、結果には Runtime 以外のテストも含まれた。

```powershell
stack test hakyll:test:hakyll-test --test-arguments "-p /Hakyll.Core.Runtime.Tests/"
```

このローカル実行では、Runtime 01_run で `_teststore` に対する `Store.set: invalid argument (cannot decode byte sequence starting from 143)` が出て、生成物がなくなったため02_runも失敗した。03_run と04_runはそれぞれ循環エラー数が8件、issue1000（07_run）は成功し、スイート全体では `4 out of 51 tests failed` だった。

Tasty が失敗時に提示したパターンを引用符付きで渡す試みは `Could not parse: '/Hakyll.Core.Runtime.Tests/&&/Hakyll.Core.Runtime.Tests.03_run/' is not a valid pattern` となった。

続けて次を実行した。テストのパターン指定では case03 とともに01_run/02_runも実行され、case03 は `expected: 1 but got: 11` となった。

```powershell
stack test hakyll:test:hakyll-test --test-arguments "-p /Hakyll.Core.Runtime.Tests.03_run/"
```

同じパターンに `+RTS -N1 -RTS` を付けて実行した。01_runに前述のローカル Store エラーが残り、02_runは成功、03_runは成功した。

```powershell
stack test hakyll:test:hakyll-test --test-arguments "-p /Hakyll.Core.Runtime.Tests/&&/Hakyll.Core.Runtime.Tests.03_run/ +RTS -N1 -RTS"
```

これは循環ログ数が RTS の並列度・実行タイミングで変わる観測と一致した。ただし、ローカルではテストスイート全体が成功したわけではない。

最後に `git status --short` を確認し、調査による追加のソース変更がないことを確認した。

#### 回答

> ## 原因
> <!-- space -->
> 主因は Tasty のテスト並列実行ではなく、`run` が内部で起動する非同期ワーカーです。テストコンポーネントには `-threaded` と `-with-rtsopts=-N` が指定されており、`build の分岐`は `rtsSupportsBoundThreads` が真なら `forkIO` を使う経路を選びます。したがって、テストを `sequentialTestGroup` にしても、各 `run` 内の処理は並列になり得ます。
> <!-- space -->
> 依存循環の件はコード上ではっきり説明できます。`schedulerPop`は、キューが空で処理中の項目がなく、ブロック中の項目があると循環エラーを記録します。しかし、その状態を「エラー報告済み」に変えません。複数のワーカーが同じ状態を確認すれば、同じ循環を繰り返し記録できます。そのため、`case03 と case04`が期待する「循環エラーのログが1件」が、ワーカー数やタイミングによって複数件になります。`schedulerCycles` が循環を過剰検出しているというより、エラー報告が一度限りになっていない問題です。
> <!-- space -->
> さらに、`pickAndChaseAsync`はワーカーを `forkIO` で起動し、いずれかが `SchedulerError` または `SchedulerFinish` を通知すると戻りますが、残りのワーカーを待ち合わせていません。`run`は戻った直後にエラー一覧を読み取るため、別のワーカーの処理完了前に結果を確定する可能性があります。これは結果のタイミング依存や、`run` の終了後まで残るワーカーにつながります。
> <!-- space -->
> ただし、issue1000 の「multiple writes」欠落をこの再現だけで完全に特定できたとは言えません。`schedulerRoute`のルート記録自体は `atomicModifyIORef'` で保護されています。少なくとも、単純な `IORef` のスレッド安全性不足が原因ではありません。非同期ワーカーの終了・待ち合わせの問題は疑わしいものの、このケースには追加の切り分けが必要です。
> <!-- space -->
> ## 確認結果
> <!-- space -->
> この環境で既定の RTS 設定で実行すると、case03 / case04 の循環エラー件数がそれぞれ8件以上になりました。一方、`+RTS -N1 -RTS` では case03 は成功しました。これは並列度による変動と整合します。実行中、case01 では `_teststore` の文字コード関連エラーも発生したため、ローカル環境でテストスイート全体が成功したわけではありません。
> <!-- space -->
> 要するに、**逐次化できているのは Tasty のテストケースであり、`run` 内部のスケジューラではありません。循環エラーの報告が非冪等で、非同期ワーカーも確実に待ち合わせていないことが、確認できた失敗の中核です。**

## comment 2026-11-23 8

GitHub Copilot の性能は期待以上であった。その一方で経緯と結果を他人に共有できる形で記録する時に難がある。上記のような出力をさせるまでに時間がかかったし、これでも満足していない。もっと「何々というファイルを調べた」とか「何々と検索した」とかが並ぶシステマチックなログのような出力が欲しかったんだけど。次のように入力したんだけど、こうするまでに 6 回はやり直した。

```txt
私の入力も途中経過のステップも含めて、上記までの経緯を順番を保ってマークアップも含めて一字一句まで可能な限り詳細かつ忠実に再現して issue-1-2-26.md の末尾に出力しなさい。
```

というか "Copy All" からコピーしてもリンクが保存されないのに困っていたんだけど、特定のファイルの内容を参照しているから、外部にコピーしたらリンク切れになるからリンクがなくなるということ？

さて、どうやって解決するべきか？　並列処理のための分岐を変更して、必ず逐次処理するようにするという方法もある。あと、 GitHub Copilot に修正させるという方法もある。

私に理解できる範囲でやるなら、逐次処理するようにする方向かな。

あるいは失敗を許容するテスト。

今すぐは修正できないけど、テストの内容を変えて対応したり、内部実装をいじったり、 GitHub Copilot に修正させたりしても、あとあと困りそうだから、失敗を許容するテストにするか。

いや、調べたけど、すぐにできる方法がないみたい。

循環を検出したやつについて「エラー報告済み」に変えていないせいで重複報告が発生しているっていうけど、最初からそういう仕組みがないのか、あるいはあるけどやり忘れているだけなのか？

`pickAndChaseAsync` 関数は、すべてのスレッドを待っていなくて、最初に戻ってきた値を使うというけど、デッドロックになったスレッドに影響されずに終わらせることができるというメリットがあるのでは？

とりあえず一番目の質問について GitHub Copilot に尋ねるか。

## comment 2026-11-23 9

下記のようになった。

```txt
User: Hakyll.Core.Runtime.Tests が失敗する理由を解明しなさい。

Copilot: 失敗の再現条件と、テスト・実装・関連 issue 記述の食い違いを照合します。まず対象ファイルと作業ツリーの状態を確認します。

Read 
Completed with input: {
  "path": "$PROJECT_ROOT\\issue\\issue-1-2-26.md"
}

Read 
Completed with input: {
  "path": "$PROJECT_ROOT\\lib\\Hakyll\\Core\\Runtime.hs"
}

Read 
Completed with input: {
  "path": "$PROJECT_ROOT\\tests\\Hakyll\\Core\\Runtime\\Tests.hs"
}

Ran terminal command: git status --short; Get-ChildItem -Name

Read , lines 1 to 140
Completed with input: {
  "path": "$PROJECT_ROOT\\issue\\issue-1-2-26.md",
  "view_range": [
    1,
    140
  ]
}

Read , lines 141 to 280
Completed with input: {
  "path": "$PROJECT_ROOT\\issue\\issue-1-2-26.md",
  "view_range": [
    141,
    280
  ]
}

Read , lines 281 to 412
Completed with input: {
  "path": "$PROJECT_ROOT\\issue\\issue-1-2-26.md",
  "view_range": [
    281,
    412
  ]
}

Read , lines 1 to 180
Completed with input: {
  "path": "$PROJECT_ROOT\\lib\\Hakyll\\Core\\Runtime.hs",
  "view_range": [
    1,
    180
  ]
}

Read , lines 181 to 360
Completed with input: {
  "path": "$PROJECT_ROOT\\lib\\Hakyll\\Core\\Runtime.hs",
  "view_range": [
    181,
    360
  ]
}

Read , lines 361 to 540
Completed with input: {
  "path": "$PROJECT_ROOT\\lib\\Hakyll\\Core\\Runtime.hs",
  "view_range": [
    361,
    540
  ]
}

この実装では、テストは依存循環の「検出回数」を厳密に数え、スケジューラは `forkIO` で実行されます。次に、テストランナーの実行方式・共有テスト環境・CI の実行条件を確認し、単なるログ重複と実際の並行スケジューラ不具合を切り分けます。

Search for `testConfiguration|cleanTestEnv|sequentialTestGroup|testGroup|defaultMain|with...`

Search for `SchedulerError|SchedulerStarve|schedulerPop|schedulerBlock|schedulerUnblock|s...`

Read 
Completed with input: {
  "path": "$PROJECT_ROOT\\tests\\TestSuite.hs"
}

Read 
Completed with input: {
  "path": "$PROJECT_ROOT\\tests\\TestGroup.hs"
}
Tool execution failed: Path does not exist

Read , lines 240 to 335
Completed with input: {
  "path": "$PROJECT_ROOT\\hakyll.cabal",
  "view_range": [
    240,
    335
  ]
}

Read 
Completed with input: {
  "path": "$PROJECT_ROOT\\tests\\TestSuite\\Util.hs"
}

Ran terminal command: stack test hakyll:test --test-arguments "-p /Hakyll.Core.Runtime.Tests/"
Tool execution failed with exit code 1

Ran terminal command: stack test hakyll:test:hakyll-test --test-arguments "-p /Hakyll.Core.Runtime.Tests/"
Tool execution failed with exit code 1

Ran terminal command: stack test hakyll:test:hakyll-test --test-arguments "-p '/Hakyll.Core.Runtime.Tests/&&/Hakyll.Core.Runtime.Tests.03_run/'"
Tool execution failed with exit code 1

Ran terminal command: stack test hakyll:test:hakyll-test --test-arguments "-p /Hakyll.Core.Runtime.Tests.03_run/"
Tool execution failed with exit code 1

Ran terminal command: stack test hakyll:test:hakyll-test --test-arguments "-p /Hakyll.Core.Runtime.Tests/&&/Hakyll.Core.Runtime.Tests.03_run/ +RTS -N1 -RTS"
Tool execution failed with exit code 1

原因候補は絞れました。単にテストケース同士の並列実行ではなく、`run` 自体の非同期ワーカーが再現に関与しています。最後に、テスト実行で作業ツリーの変更が増えていないことを確認し、再現結果とコード上の根拠をまとめます。

Ran terminal command: git status --short

## 原因

主因は Tasty のテスト並列実行ではなく、`run` が内部で起動する非同期ワーカーです。テストコンポーネントには `-threaded` と `-with-rtsopts=-N` が指定されており、`build の分岐`は `rtsSupportsBoundThreads` が真なら `forkIO` を使う経路を選びます。したがって、テストを `sequentialTestGroup` にしても、各 `run` 内の処理は並列になり得ます。

依存循環の件はコード上ではっきり説明できます。`schedulerPop`は、キューが空で処理中の項目がなく、ブロック中の項目があると循環エラーを記録します。しかし、その状態を「エラー報告済み」に変えません。複数のワーカーが同じ状態を確認すれば、同じ循環を繰り返し記録できます。そのため、`case03 と case04`が期待する「循環エラーのログが1件」が、ワーカー数やタイミングによって複数件になります。`schedulerCycles` が循環を過剰検出しているというより、エラー報告が一度限りになっていない問題です。

さらに、`pickAndChaseAsync`はワーカーを `forkIO` で起動し、いずれかが `SchedulerError` または `SchedulerFinish` を通知すると戻りますが、残りのワーカーを待ち合わせていません。`run`は戻った直後にエラー一覧を読み取るため、別のワーカーの処理完了前に結果を確定する可能性があります。これは結果のタイミング依存や、`run` の終了後まで残るワーカーにつながります。

ただし、issue1000 の「multiple writes」欠落をこの再現だけで完全に特定できたとは言えません。`schedulerRoute`のルート記録自体は `atomicModifyIORef'` で保護されています。少なくとも、単純な `IORef` のスレッド安全性不足が原因ではありません。非同期ワーカーの終了・待ち合わせの問題は疑わしいものの、このケースには追加の切り分けが必要です。

## 確認結果

この環境で既定の RTS 設定で実行すると、case03 / case04 の循環エラー件数がそれぞれ8件以上になりました。一方、`+RTS -N1 -RTS` では case03 は成功しました。これは並列度による変動と整合します。実行中、case01 では `_teststore` の文字コード関連エラーも発生したため、ローカル環境でテストスイート全体が成功したわけではありません。

要するに、**逐次化できているのは Tasty のテストケースであり、`run` 内部のスケジューラではありません。循環エラーの報告が非冪等で、非同期ワーカーも確実に待ち合わせていないことが、確認できた失敗の中核です。**

User: `schedulerPop` 関数の処理において、循環エラーを報告した後に「エラー報告済み」フラグを付与しないというが、これは最初からそのような仕組みがないということであるか？　それとも、そのような仕組みがあるが利用していないということであるか？

Copilot: 質問を issue の追記内容と現行実装の両方に照らして確認します。

Read , lines 480 to 549
Completed with input: {
  "path": "$PROJECT_ROOT\\issue\\issue-1-2-26.md",
  "view_range": [
    480,
    549
  ]
}

Search for `data Scheduler|schedulerErrors|schedulerPop|schedulerError|schedulerDone|sche...`

Ran terminal command: git status --short

現行実装では、**「エラー報告済み」を表す専用の仕組み自体がありません**。既存の仕組みを使い忘れている、ということではありません。

`Scheduler`には、処理済み項目を記録する `schedulerDone` や、エラーを蓄積する `schedulerErrors` はあります。しかし、循環を報告した状態を記録するフィールドはありません。`schedulerPop`は循環条件に当たると `schedulerError` でエラーを追加しますが、`schedulerErrors` に同じエラーがあるかを確認して報告を抑止する処理もありません。

つまり、`SchedulerError` はその呼び出しで返す結果であり、以後の報告を止める永続的なフラグではありません。なお、`schedulerErrors` を報告済み判定に転用する設計は可能ですが、現在はその用途では使われていません。
```

## comment 2026-11-23 10

修正させてみた。

```txt
User: `schedulerPop` 関数において循環エラーを報告する際に `schedulerErrors` フィールドにおいて同様のエラーが存在するか確認して既に存在する場合はエラー報告をスキップするように変更しなさい。

Copilot: 循環エラーの既存テストを回帰確認に使い、重複判定を `Scheduler` 状態更新の中で行う形にします。まず現在の該当処理と未コミット変更を確認します。

Read , lines 132 to 245
Completed with input: {
  "path": "$PROJECT_ROOT\\lib\\Hakyll\\Core\\Runtime.hs",
  "view_range": [
    132,
    245
  ]
}

Read , lines 110 to 185
Completed with input: {
  "path": "$PROJECT_ROOT\\tests\\Hakyll\\Core\\Runtime\\Tests.hs",
  "view_range": [
    110,
    185
  ]
}

Ran terminal command: git status --short; git diff -- lib/Hakyll/Core/Runtime.hs tests/Hakyll/Core/Runtime/Tests.hs

Edit 

重複判定を追加しました。同じ `(Nothing, msg)` が既に記録されていれば状態を変更せず `SchedulerError` を返し、初回だけエラーを蓄積する実装です。既存の case03 / case04 を複数 capability で検証します。

Ran terminal command: stack test hakyll:test:hakyll-test --test-arguments "-p /Hakyll.Core.Runtime.Tests.03_run/ +RTS -N4 -RTS"
Tool execution failed with exit code 1

Ran terminal command: stack test hakyll:test:hakyll-test --test-arguments "-p /Hakyll.Core.Runtime.Tests.04_run/ +RTS -N4 -RTS"
Tool execution failed with exit code 1

複数 capability の実行で case03 と case04 自体はともに成功しました。選択したケースの前提として同じテストスイートの case01 も実行され、Windows の Store 文字コードエラーで失敗するため、最後に差分を確認して結果を分けて報告します。

Ran terminal command: git diff --check; git diff -- lib/Hakyll/Core/Runtime.hs

Ran terminal command: git status --short

`schedulerPop`を変更し、循環エラーの `(Nothing, msg)` が `schedulerErrors` に既にあれば、エラーを追加せずに `SchedulerError` を返すようにしました。エラーの重複記録は防ぎつつ、ワーカーにはエラー状態を引き続き通知します。

`-N4` で実行したところ、回帰対象の case03・case04 は両方成功しました。ただし、テストスイートの先行 case01 は Windows の Store 文字コードエラーで失敗しており、スイート全体の成功は確認できていません。`git diff --check` の警告は、既存の変更がある issue ファイル内の末尾空白によるものです。
```

テストが通るようになった。

下記の変更を提案してきた。

```diff
diff --git a/lib/Hakyll/Core/Runtime.hs b/lib/Hakyll/Core/Runtime.hs
index 07296f2..fa9911d 100644
--- a/lib/Hakyll/Core/Runtime.hs
+++ b/lib/Hakyll/Core/Runtime.hs
@@ -234,8 +234,11 @@ schedulerPop scheduler@Scheduler {..} = case Seq.viewl schedulerQueue of
                         intercalate ", " (show <$> Set.toList schedulerBlocked)
                     | otherwise = "Dependency cycles: " <>
                         intercalate "; "
-                            (map (intercalate " -> " . map show) cycles) in
-            SchedulerError <$ schedulerError Nothing msg scheduler
+                            (map (intercalate " -> " . map show) cycles)
+                errorKey = (Nothing, msg) in
+            if errorKey `elem` schedulerErrors
+                then (scheduler, SchedulerError)
+                else SchedulerError <$ schedulerError Nothing msg scheduler
         | otherwise -> (scheduler, SchedulerFinish)
     x Seq.:< xs
         | x `Set.member` schedulerDone ->
```

すぐわかる欠点は `in` の位置がおかしい他にはない。

でも、よくよく見るとなんか捨てていて怪しくない？

いや、 `schedulerError` 関数を適用して新しい `scheduler` を作るのがデフォルトで、それを飛ばしているだけである。

正しい。

タプルの一番目の値を `Nothing` にするのも怪しいとは思っていたけど、デフォルトにおいて `schedulerError` 関数に与えているのが `Nothing` と `msg` だから正しい。

いやはや、 GitHub Copilot をコード補完じゃなくてバグ修正のために使ったのは初めてだけど、こんなに高性能だとは。

## comment 2026-11-23 11

`a35101ba636c18f5973cf2cc7fb16d2d902fac1a` としてコミットした。

なんか CI が通った。 `writable` の重複のエラーまでなくなっているんだけど。まあ、並列処理のエラーだからなあ。

リストに対する `elem` 関数は遅いから `Set` 型に差し替える必要もあるかも？

なぜ自力で気付けなかったのか？

一つ目。 `Dependency cycles: ...` というエラーを生成している大本が `schedulerPop` 関数であることに気付いていたのに別の場所に目を向けていた。一番近い位置から、一番基本的な所から、確かめていくべきだった。並列処理を念頭に置きすぎていて、並列処理がなさそうだからと飛ばしていた。どのように使われるのかという大枠に目を向けていなかった。数学において正負を取り違えた時と同じパターン。

二つ目。 `Dependency cycles: ...` というエラーを生成する処理が重複すればエラーが増えるという並列処理の基本を忘れていた。なにかが増えているという時点で、重複して実行されていることを疑うべきだった。直前に直面していた問題が `_teststore` ディレクトリの処理が競合していることが原因だったから、今回の問題も何等かの処理が競合しているのではないかと決めつけてしまっていた。そのため、処理が競合する部分を探してしまっていた。実行されるたびにエラーが生成されているという単純な答えに気付けなかった。

三つ目。デバッグコードを挿入することを躊躇してしまっていた。デバッグコードを挿入してデバッグすることを面倒くさいと取り掛かる前に止めてしまっていた。エラーを生成している部分にデバッグコードを挿入していれば、複数回実行されていることに気付けるはずだった。さらに、 `schedulerErrors` フィールドに目を向けていれば、いや、 `scheduler` 全体を表示するようなデバッグコードを挿入していれば、何が起こっているのかすぐに気付けるはずだった。デバッグコードを挿入してデバッグする作業は、「コミットとして残せば履歴が汚くなる」と「コミットとして残さなければ、どれだけ時間がかかって、どれだけの作業が必要だったのか、記録に残らない」の二択だからなあ。

しかし、 GitHub Copilot のログの残し方は、やっぱり GitHub Copilot 自身にやらせるよりも "Copy" の方が良いな。そっちの方がシンプルにより多くの履歴が残る。機密情報が含まれていないかどうかの確認が必要だけど……この確認こそ GitHub Copilot にやらせた方が良いのでは？
