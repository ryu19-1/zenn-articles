---
title: "Rustで真面目にフロントエンド開発ができるのか考えてみた"
emoji: "🦀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Rust", "フロントエンド開発", "dioxus"]
published: true
publication_name: "labbase"
---

幸運なことに、私は最近バックエンドはほぼ全て Rust で開発しています。一方でフロントエンドは React で開発しているのですが、もし同じく Rust で書けたら Rust 信者の私としてはとても嬉しくないか？と日々思っていました。実はこれは夢ではなく、Rust のプログラムを WebAssembly(WASM)にコンパイルすることで Web ブラウザでも JavaScript から呼び出すことが出来ることからアプリケーション全体を Rust で構築することが可能なのです。そしてそれをサポートするフレームワークも OSS でたくさん出てきています。

Rust でのフロントエンド開発のためのフレームワークについて[こちら](https://github.com/flosse/rust-web-framework-comparison)で比較を行っているのですが、GitHub の Repository の Star 数やダウンロード数が一番多い yew に対して、[Dioxus](https://dioxuslabs.com/)というフレームワークは直近での Activity 数が優っており、発展途上ながら勢いを感じます。

今回はこの Dioxus を使って実用的なフロントエンドのアプリケーションが作れるのか検証してみます。

:::message
執筆時現在（2023 年 1 月 6 日）の最新の Dioxus のバージョンは 0.3.1 です。
:::

## Dioxus とは？

[Dioxus](https://dioxuslabs.com/)は React チックな書き方で Rust で開発出来るライブラリであり、トップページにある「One codebase, every platform」の通り Web アプリだけでなく、デスクトップアプリやモバイルアプリの開発にも対応しています。

## セットアップ

:::message
[公式のチュートリアル](https://dioxuslabs.com/reference/platforms/web.html)は、おそらく Dioxus のバージョンが 2 系時に書かれており、3 系だと動かなかったので一部修正しています。
:::
Rust と Cargo、および Cargo.toml を編集するのに便利な crate の[cargo-edit](https://crates.io/crates/cargo-edit)はインストール済みとします。

まず WASM をコンパイルするためにターゲットアーキテクチャを追加します。

```
$ rustup target add wasm32-unknown-unknown
```

次に Rust 向け WASM アプリケーション用バンドラーの[Trunk](https://trunkrs.dev/)をインストールします。

```
$ cargo install trunk
```

新しいプロジェクトの作成

```
$ cargo new dioxus-tutorial
$ cd dioxus-tutorial
```

dioxus, dioxus-web を追加（3 系から dioxus の crate から独立した？）

```
$ cargo add dioxus --features html
$ cargo add dioxus-web
```

ルートディレクトリ直下に index.html を追加

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

main.rs にこのように React っぽく書けます。rsx!マクロを使うことで HTML 要素を定義出来るみたいです。

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

開発用サーバを起動するとブラウザで確認出来ます。

```
$ trunk serve
```

## デモアプリ

- デモのコードはこちら。

https://github.com/ryu19-1/dioxus-tutorial

フロントエンド開発での重要な要素をいくつかを抜粋して紹介します。

### Routing

dioxus_router を導入して以下のように直感的に書けます。

```rust:src/main.rs
#![allow(non_snake_case)]

use components::organisms::footer::Footer;
use components::organisms::header::Header;
use containers::*;
use dioxus::prelude::*;
use dioxus_router::{Route, Router};

pub mod components;
pub mod composition_root;
pub mod containers;
pub mod hooks;

fn main() {
    dioxus_web::launch(app);
}

fn app(cx: Scope) -> Element {
    cx.render(rsx! (
        Header(cx),
        Router {
            Route { to: "/home", home::Home {} }
            Route { to: "", not_found::NotFound {} }
        },
        Footer(cx)
    ))
}

```

### Components

[Tailwind CSS](https://tailwindcss.com/)を導入することが可能なので、以下のようにスタイルを追加できます。スタイル部分は別ファイルに切り出した方が取りまわしやすいかもしれないですね。

```rust:header.rs
use dioxus::prelude::*;

pub fn Header(cx: Scope) -> Element {
    cx.render(rsx! {
        div {
            class: "py-16 flex h-32 w-full absolute top-0 border",
            div{ class: "m-auto","ヘッダーです" }
        }
    })
}

```

### Async

use_future を使うことで非同期な処理を呼び出すことが出来ます。ここでは hooks として用意した use_recruits()の中でバックエンドの API を呼び出してデータを取得するようなイメージで書いています。

```rust:home.rs
use crate::hooks::use_recruits::use_recruits;
use dioxus::prelude::*;

pub fn Home(cx: Scope) -> Element {
    let result = use_future(&cx, (), |_| async move { use_recruits().await });

    cx.render(rsx! {
        div{ class: "flex flex-col mt-32",
            h1 { "トップ画面" }
            h2 { "募集一覧" }
            match result.value() {
                Some(Ok(d)) => cx.render(rsx! {
                    d.iter().map(|v| rsx!{ div{v.to_string()}})
                }),
                _ => cx.render(rsx! {
                    div{"No recruit found."}
                }),
            }
        }
    })
}

```

---

## まとめ

思ったより React っぽく書けることに驚きました。また取り上げてはいませんが SSR にも対応しているらしいので、Rust に興味があるフロントエンドエンジニアは触ってみると面白いかもです。match 文や Result 型が便利なので自分は比較的とっつきやすいと感じました。Tailwind の記法にまだ慣れていないのでフロント UI 周りはちょっと書きにくさを感じました。
