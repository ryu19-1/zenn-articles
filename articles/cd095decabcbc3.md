---
title: "Houdini+SvelteKitで快適なGraphQL生活を"
emoji: "🎄"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["frontend", "svelte", "sveltekit", "houdini", "graphql"]
published: false
---

この記事は[LabBase テックカレンダー Advent Calendar 2023](https://qiita.com/advent-calendar/2023/labbase)の 19 日目です。

## はじめに

Houdini^[Houdini で調べると 3DCG 作成用のソフトウェアが先に出てきますね。もちろん別物です] は GraphQL クライアントとして作られた Web アプリケーションフレームワークです。今のところ React と SvelteKit で利用できます。
https://houdinigraphql.com/

## セットアップ

すでに SvelteKit を利用している場合は、以下のコマンドで簡単に Houdini を導入できます。
:::message
今回は Svelte5 `5.0.0-next.25` および SvelteKit2 を使用しています。
:::

```bash
$ npx houdini@latest init
Need to install the following packages:
houdini@1.2.34
Ok to proceed? (y) y
┌  🎩 Welcome to Houdini!
│
◇  Will you use a remote GraphQL API?
│  No
│
◇  Where is your schema located?
│  schema.graphql
│
●  Here's what we found: ✨ SvelteKit, 📦 ES Modules, 🟦 TypeScript
│
◇  Houdini's files generated ✓
│
└  🎉 Everything is ready!
```

セットアップの途中で GraphQL スキーマのパスを聞かれるので入力しましょう。今回は以下を用意しました。

```graphql:schema.graphql
type User {
  id: Int!
  name: String!
}

type Query {
  users: [User!]!
}
```

コマンド実行後`houdini.config.js`というファイルが生成されるので見てみると schema のパスが指定されています。

```js:houdini.config.js
/// <references types="houdini-svelte">

/** @type {import('houdini').ConfigFile} */
const config = {
    "schemaPath": "schema.graphql",
    "plugins": {
        "houdini-svelte": {}
    }
}

export default config
```

package.json に設定が追加されるので忘れずに`pnpm install`を実行しましょう。`pnpm dev`でローカルサーバを立ち上げられたことを確認したら OK

セットアップに関するその他の情報はこちら
https://houdinigraphql.com/guides/setting-up-your-project

## (TODO)利用方法

### Query

### Mutation

## おまけ

- Mock Service Worker を利用してスキーマ駆動開発

## おわりに

最近仕事で Svelte + GraphQL の開発をしていたので、自分への備忘録を兼ねて記事にまとめました。余談ですが Svelte5 で[Rune](https://svelte.jp/blog/runes)が使えるようになったので state 周りの書き方が大分変わりそうです。この辺りを年末にかけて深掘りしたいと思っています。
ところで今年はあまりテックブログを書けませんでしたね、アドカレ以外でも書けるよう来年は頑張ります。ではみなさん良いお年を！
