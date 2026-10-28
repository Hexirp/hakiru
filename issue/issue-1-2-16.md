# issue 16

## comment 2026-10-26 1

ここから "[issue-1-1-5.md](./issue-1-1-5.md)" の `comment 2026-10-26 1` の一部をコピーする。

[`303c00da75b18f7a33b732347ba135646694ff96`](https://github.com/Hexirp/hakiru/commit/303c00da75b18f7a33b732347ba135646694ff96) をプッシュした。これにより GitHub Actions の実行（[リンク](https://github.com/Hexirp/hakiru/actions/runs/40219334907)）が始まった。

Linux のジョブ（[リンク](https://github.com/Hexirp/hakiru/actions/runs/40219334907/job/121247278179)）と Windows のジョブ（[リンク](https://github.com/Hexirp/hakiru/actions/runs/40219334907/job/121247278331)）は問題なく終わったけど、 macOS のジョブ（(https://github.com/Hexirp/hakiru/actions/runs/40219334907/job/121247278365)）が失敗した。

該当の場所は以下の通りである。全部のログは [0_Build os-macos-latest ghc-9.8.4.txt](./issue-1-1-5/logs_109007744674/0_Build%20os-macos-latest%20ghc-9.8.4.txt) から。

```txt
2026-10-25T18:07:00.5996890Z ##[group]Run stack build --fast --system-ghc --test --no-run-tests --only-dependencies
2026-10-25T18:07:00.5997600Z [36;1mstack build --fast --system-ghc --test --no-run-tests --only-dependencies[0m
2026-10-25T18:07:00.6299820Z shell: /bin/bash -e {0}
2026-10-25T18:07:00.6300120Z ##[endgroup]
2026-10-25T18:07:29.9023270Z [1 of 3] Compiling Main             ( /Users/runner/.stack/setup-exe-src/setup-w2MFVN35.hs, /Users/runner/.stack/setup-exe-src/setup-w2MFVN35.o )
2026-10-25T18:07:30.0229180Z [2 of 3] Compiling StackSetupShim   ( /Users/runner/.stack/setup-exe-src/setup-shim-w2MFVN35.hs, /Users/runner/.stack/setup-exe-src/setup-shim-w2MFVN35.o )
2026-10-25T18:07:30.1937510Z [3 of 3] Linking /Users/runner/.stack/setup-exe-cache/aarch64-osx/tmp-Cabal-simple_w2MFVN35_3.10.3.0_ghc-9.8.4
2026-10-25T18:07:30.5991280Z ld: warning: -U option is redundant when using -undefined dynamic_lookup
2026-10-25T18:07:31.3177230Z ld: multiple errors: tapi error: malformed file
2026-10-25T18:07:31.3195150Z 
2026-10-25T18:07:31.3200000Z /Library/Developer/CommandLineTools/SDKs/MacOSX27.0.sdk/usr/lib/libSystem.B.tbd:4:20: error:
2026-10-25T18:07:31.3201040Z      error: unknown architecture
2026-10-25T18:07:31.3201500Z                        arm64e.x1-macos, arm64e.x1-maccatalyst ]
2026-10-25T18:07:31.3202080Z                        ^~~~~~~~~~~~~~~
2026-10-25T18:07:31.3202970Z      in '/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/lib/libm.tbd'; tapi error: malformed file
2026-10-25T18:07:31.3213020Z   |
2026-10-25T18:07:31.3213660Z 4 |                    arm64e.x1-macos, arm64e.x1-maccatalyst ]
2026-10-25T18:07:31.3214140Z   |                    ^
2026-10-25T18:07:31.3214450Z 
2026-10-25T18:07:31.3214900Z /Library/Developer/CommandLineTools/SDKs/MacOSX27.0.sdk/usr/lib/libiconv.2.tbd:4:20: error:
2026-10-25T18:07:31.3215520Z      error: unknown architecture
2026-10-25T18:07:31.3216020Z                        arm64e.x1-macos, arm64e.x1-maccatalyst ]
2026-10-25T18:07:31.3216650Z                        ^~~~~~~~~~~~~~~
2026-10-25T18:07:31.3237030Z      in '/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/lib/libiconv.tbd'; tapi error: malformed file
2026-10-25T18:07:31.3238360Z   |
2026-10-25T18:07:31.3239200Z 4 |                    arm64e.x1-macos, arm64e.x1-maccatalyst ]
2026-10-25T18:07:31.3240200Z   |                    ^
2026-10-25T18:07:31.3240650Z 
2026-10-25T18:07:31.3241330Z /Library/Developer/CommandLineTools/SDKs/MacOSX27.0.sdk/usr/lib/libSystem.B.tbd:4:20: error:
2026-10-25T18:07:31.3242680Z      error: unknown architecture
2026-10-25T18:07:31.3243680Z                        arm64e.x1-macos, arm64e.x1-maccatalyst ]
2026-10-25T18:07:31.3244500Z                        ^~~~~~~~~~~~~~~
2026-10-25T18:07:31.3246020Z      in '/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/lib/libdl.tbd'; tapi error: malformed file
2026-10-25T18:07:31.3247980Z   |
2026-10-25T18:07:31.3248820Z 4 |                    arm64e.x1-macos, arm64e.x1-maccatalyst ]
2026-10-25T18:07:31.3249740Z   |                    ^
2026-10-25T18:07:31.3250280Z 
2026-10-25T18:07:31.3251110Z /Library/Developer/CommandLineTools/SDKs/MacOSX27.0.sdk/usr/lib/libffi.tbd:4:20: error:
2026-10-25T18:07:31.3252270Z      error: unknown architecture
2026-10-25T18:07:31.3253210Z                        arm64e.x1-macos, arm64e.x1-maccatalyst ]
2026-10-25T18:07:31.3254190Z                        ^~~~~~~~~~~~~~~
2026-10-25T18:07:31.3255430Z      in '/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/lib/libffi.tbd'; tapi error: malformed file
2026-10-25T18:07:31.3256760Z   |
2026-10-25T18:07:31.3257500Z 4 |                    arm64e.x1-macos, arm64e.x1-maccatalyst ]
2026-10-25T18:07:31.3258670Z   |                    ^
2026-10-25T18:07:31.3268960Z 
2026-10-25T18:07:31.3278290Z /Library/Developer/CommandLineTools/SDKs/MacOSX27.0.sdk/usr/lib/libSystem.B.tbd:4:20: error:
2026-10-25T18:07:31.3280110Z      error: unknown architecture
2026-10-25T18:07:31.3296540Z                        arm64e.x1-macos, arm64e.x1-maccatalyst ]
2026-10-25T18:07:31.3299070Z                        ^~~~~~~~~~~~~~~
2026-10-25T18:07:31.3300340Z      in '/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/lib/libSystem.tbd'
2026-10-25T18:07:31.3301060Z   |
2026-10-25T18:07:31.3304290Z 4 |                    arm64e.x1-macos, arm64e.x1-maccatalyst ]
2026-10-25T18:07:31.3304950Z   |                    ^
2026-10-25T18:07:31.3305740Z clang: error: linker command failed with exit code 1 (use -v to see invocation)
2026-10-25T18:07:31.3306720Z ghc-9.8.4: `gcc' failed in phase `Linker'. (Exit code: 1)
2026-10-25T18:07:31.3801610Z 
2026-10-25T18:07:31.3803170Z Error: [S-6374]
2026-10-25T18:07:31.3803810Z        While building simple Setup.hs (scroll up to its section to see the error) using:
2026-10-25T18:07:31.3806000Z        /Users/runner/.ghcup/ghc/9.8.4/bin/ghc-9.8.4 -rtsopts -threaded -clear-package-db -global-package-db -hide-all-packages -package base -main-is StackSetupShim.mainOverride -package Cabal-3.10.3.0 /Users/runner/.stack/setup-exe-src/setup-w2MFVN35.hs /Users/runner/.stack/setup-exe-src/setup-shim-w2MFVN35.hs -o /Users/runner/.stack/setup-exe-cache/aarch64-osx/tmp-Cabal-simple_w2MFVN35_3.10.3.0_ghc-9.8.4
2026-10-25T18:07:31.3809430Z        Process exited with code: ExitFailure 1 
2026-10-25T18:07:31.4022630Z ##[error]Process completed with exit code 1.
```

macOS のビルドがよくわからない理由で落ちている。

`error: unknown architecture` だそうだけど。

たぶん古いコンパイラだから新しいアーキテクチャに対応できていない？

"[Unknown architecture when attempting to install GHCup on Red Hat linux](https://stackoverflow.com/questions/76950449/unknown-architecture-when-attempting-to-install-ghcup-on-red-hat-linux)" という似たエラーに関する質問がある。

"[platforms](https://gitlab.haskell.org/ghc/ghc/-/wikis/platforms?version_id=0fe22127d49247a5ad1762d10d25a13c0d26c422&version_number=85)" によると macOS の場合は x86-64 と AArch64 のみに対応していて ARM64 に対応していないということなのかな。

Hakyll は macOS でテストしているんだよな。どうやっているんだ？

"[main.yml](https://github.com/jaspervdj/hakyll/blob/3b62127a9d9c9c6df84acc9db0a42f02a4d8e07c/.github/workflows/main.yml#L17)" によると `macOS-14` を指定している。

ところが、 "[GitHub Actions: macOS 14 runner image retirement](https://github.blog/changelog/2026-10-01-github-actions-macos-14-runner-image-retirement/)" によると、 `macOS-14` の提供は 2026 年 11 月 2 日に終了するとのこと。

macOS でのテストは諦めるべきか？

ここまで "[issue-1-1-5.md](./issue-1-1-5.md)" の `comment 2026-10-26 1` の一部をコピーする。
