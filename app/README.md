# Torifune — Tauri v3 app（AWAI-3502）

鳥船 — 民生用ロケットの機体グラフを、この端末で検算します。段ごとの Δv（ツィオルコフスキー）、推進剤の正味の炭素収支、段とデブリの処分計画を、repo の種データ（代表的な推定値）から計算します。兵器の運搬は表現できず、外部へは何も送信しません。

この `app/` は、repo の `.cljk` ドメインコードを ClojureScript でそのまま端末内で動かす Tauri v3 アプリです。

## 仕組み

- `scripts/build.mjs` が、repo の `.cljk`（`package.json` の `itonami.cljsSources`）を `.cljc` として写し、`src/cljs/itonami/app/entry.cljs`（この app の操作の定義）と固定した `kotoba.lang.text` と一緒に ClojureScript（1.12.145、advanced）で `dist/domain.js` にまとめます。コンパイラの警告が 1 つでもあれば build は失敗します。
- 画面（`src/ui/`）は操作ごとに EDN の入力欄と例を出し、結果を EDN で表示します。実行の記録は端末の localStorage にだけ残ります。
- 外部への通信はありません。descriptor の `operations` は空、CSP は `connect-src ipc: http://ipc.localhost` だけ、Tauri の権限は読み取り専用の `awai_host_info` だけです。Android の manifest に `INTERNET` 権限はありません。
- 入れていないもの（daemon・送受信・遠隔推論・公開など）は `entry.cljs` の冒頭に書いています。

## 使い方

```sh
cd app
npm ci
npm test            # build + 6 checks (descriptor, no network API, CSP, capabilities, release pins, every example runs)
npm run build:android   # Android debug APK, package network.itonami.torifune
```

`npm run build` には Clojure CLI（JVM）が要ります。Android には SDK・NDK 27・JDK 21・Rust の `aarch64-linux-android` が要ります。

Store への掲載は network-awai/awai-os の `android/local-apps.json` と `build.py --local-apps-dir` で行います。実機・署名済みの配布は未確認です。
