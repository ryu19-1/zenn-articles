---
title: "Rustで真面目にフロントエンド開発ができるのか考えてみた"
emoji: "🦀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Rust","フロントエンド開発","dioxus"]
published: false
---

幸運なことに、私は最近バックエンドはほぼ全てRustで開発しています。一方でフロントエンドはReactで開発しているのですが、もし同じくRustで書けたらRust信者の私としてはとても嬉しくないか？と日々思っていました。実はこれは夢ではなく、RustのプログラムをWebAssembly(WASM)にコンパイルすることでWebブラウザでもJavaScriptから呼び出すことが出来ることからアプリケーション全体をRustで構築することが可能なのです。そしてそれをサポートするフレームワークもOSSでたくさん出てきています。

Rustでのフロントエンド開発のためのフレームワークについて[こちら](https://github.com/flosse/rust-web-framework-comparison)で比較を行っているのですが、GitHubのRepositoryのStar数やダウンロード数が一番多いyewに対して、[Dioxus](https://dioxuslabs.com/)というフレームワークは直近でのActivity数が優っており、発展途上ながら勢いを感じます。

今回はこのDioxusを使って実用的なフロントエンドのアプリケーションが作れるのか検証してみます。

:::message
執筆時現在（2023年1月6日）の最新のDioxusのバージョンは0.3.1です。
:::

## Dioxusとは？
[Dioxus](https://dioxuslabs.com/)はReactチックな書き方でRustで開発出来るライブラリであり、トップページにある「One codebase, every platform」の通りWebアプリだけでなく、デスクトップアプリやモバイルアプリの開発にも対応しています。

## セットアップ

:::message
[公式のチュートリアル](https://dioxuslabs.com/reference/platforms/web.html)は、おそらくDioxusのバージョンが2系時に書かれており、3系だと動かなかったので一部修正しています。
:::
RustとCargo、およびCargo.tomlを編集するのに便利なcrateの[cargo-edit](https://crates.io/crates/cargo-edit)はインストール済みとします。

まずWASMをコンパイルするためにターゲットアーキテクチャを追加します。
```
$ rustup target add wasm32-unknown-unknown
```
次にRust向けWASMアプリケーション用バンドラーの[Trunk](https://trunkrs.dev/)をインストールします。
```
$ cargo install trunk
```
新しいプロジェクトの作成
```
$ cargo new dioxus-tutorial
$ cd dioxus-tutorial
```
dioxus, dioxus-webを追加（3系からdioxusのcrateから独立した？）
```
$ cargo add dioxus --features html
$ cargo add dioxus-web
```
ルートディレクトリ直下にindex.htmlを追加
```html:index.html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
  </head>
  <body>
    <div id="main"> </div>
  </body>
</html>
```
RustでReactっぽく書く
```rust:src/main.rs
use dioxus::prelude::*;

fn main() {
    dioxus_web::launch(app);
}

fn app(cx: Scope) -> Element {
    cx.render(rsx! (
        div { "Hello, world!" }
    ))
}
```
開発用サーバで起動するとブラウザで確認出来ます。
```
$ trunk serve
```

## 真面目にアプリ作れるの？

## デモアプリ

https://github.com/ryu19-1/dioxus-tutorial

## まとめ

---
## 書きたいこと
- Rustでフロントエンド開発したい！WasmのおかげでフロントエンドでもRust動かせるよ！
- コンポーネント、状態管理、Styleの管理などモダンなフロントエンド開発で必須な要素をどこまでカバーできるか
- 近年注目のRustのGUIアプリケーションフレームワーク"Dioxus"で試してみた
- 2023年にdioxusのv3がリリースされたのでメジャーな変更を追う
