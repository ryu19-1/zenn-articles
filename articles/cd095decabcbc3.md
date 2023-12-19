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

### サンプルコード

https://github.com/ryu19-1/svelte-houdini-demo

## セットアップ

すでに SvelteKit を利用している場合は、以下のコマンドで簡単に Houdini を導入できます。
:::message
今回は Svelte5 `5.0.0-next.25` および SvelteKit2 `2.0.2` を使用しています。
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

input UserInput {
	name: String!
}

type AddUserResponse {
	success: Boolean!
}

type Mutation {
	addUser(user: UserInput!): AddUserResponse!
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

:::message alert
SvelteKit のバージョン 2 系を使うと、`pnpm dev`でサーバ起動時に以下のようなエラーに遭遇しました。^[SvelteKit の 2 系は破壊的な変更が入っていそうですので使う際は慎重に]
`SyntaxError: The requested module '@sveltejs/kit/vite' does not provide an export named 'vitePreprocess'`

このエラーを回避するために`vitePreprocess`を`@sveltejs/vite-plugin-svelte`からインポートするように変更しています。

```js:svelte.config.js
import adapter from '@sveltejs/adapter-auto';
import { vitePreprocess } from '@sveltejs/vite-plugin-svelte';

/** @type {import('@sveltejs/kit').Config} */
const config = {
	// Consult https://kit.svelte.dev/docs/integrations#preprocessors
	// for more information about preprocessors
	preprocess: vitePreprocess(),

	kit: {
		adapter: adapter(),
		alias: {
			$houdini: './$houdini',
		}
	}
};

export default config;
```

:::

`pnpm dev`を実行することで`/$houdini`というフォルダが自動生成されます。こちらをインポートすることで GraphQL を利用できます。

セットアップに関するその他の情報はこちら
https://houdinigraphql.com/guides/setting-up-your-project

## 利用方法

### Query

Query の呼び出し方は様々あり、大きく分けてページ読み込みに同期してローディングが行われる方法と手動で呼び出す方法があります。また、GraphQL のクエリをインラインで埋め込む方法もありますが、業務レベルだと Svelte コンポーネントと TS のロジック、GraphQL のクエリ文は切り離して扱う場合が多いと思いますので、その例を紹介します。

https://houdinigraphql.com/api/query

まずクエリを用意して

```graphql:query.gql
query GetUsers {
	users {
		id
		name
	}
}
```

それを load 関数の中で呼び出します。コードの中の`GetUsersStore`は Houdini が自動生成したストアになります。

```ts:+page.ts
import { GetUsersStore } from '$houdini';
import type { PageLoad } from './users/$types';
import { browser } from '$app/environment';

export const load: PageLoad = async (event) => {
	if (browser) {
		const store = new GetUsersStore();
		const result = await store.fetch({ event });
		if (result.data?.users) {
			return {
				users: result.data.users
			};
		} else {
			throw new Error(result.errors?.join('\n'));
		}
	}
};
```

Svelte コンポーネントで以下のように呼び出すことが出来ます。

```
<script lang="ts">
	import type { PageData } from './users/$types';

	export let data: PageData;
</script>

{#if data.users}
	<h1>ユーザ一覧</h1>

	{#each data.users as user}
		<div class="user-content">
			<span>ID: {user.id}</span>
			<span>ユーザ名: {user.name}</span>
		</div>
	{/each}
{/if}

<style>
	.user-content {
		display: flex;
		gap: 16px;
	}
</style>
```

### Mutation

Mutation も同様にファイルを用意して

```graphql:mutation.gql
mutation AddUser($user: UserInput!) {
	addUser(user: $user) {
		id
	}
}
```

このように`+page.svelte`の中で呼び出すと良いでしょう。

```
<script lang="ts">
	import { AddUserStore } from '$houdini';
	import type { PageData } from './$types';

	export let data: PageData;

	const handleClick = async () => {
		const store = new AddUserStore();
		const result = await store.mutate({
			user: {
				name: 'addUser'
			}
		});
		if (result.data?.addUser) alert('ユーザを追加しました');
	};
</script>

{#if data.users}
	<h1>ユーザ一覧</h1>

	{#each data.users as user}
		<div class="user-content">
			<span>ID: {user.id}</span>
			<span>ユーザ名: {user.name}</span>
		</div>
	{/each}

	<button on:click={handleClick}>ユーザー追加</button>
{/if}

<style>
	.user-content {
		display: flex;
		gap: 16px;
	}
</style>
```

また今回は取り上げませんが、データの Cache, Pagenation などの設定も行うことが出来ます。

https://houdinigraphql.com/api/mutation

## おまけ

GraphQL の API を利用したフロントエンド開発がしたいけれど、まだ API が出来ていない場合は Mock Service Worker を利用してスキーマから API をモック化してしまうと楽です。

https://mswjs.io/

### 導入方法

```bash
pnpm add @graphql-codegen/cli @graphql-codegen/typescript @graphql-codegen/typescript-operations @graphql-codegen/typescript-msw graphql-codegen-typescript-mock-data msw
```

`codegen.ts`ファイルを作成

```ts:codegen.ts
import type { CodegenConfig } from '@graphql-codegen/cli';

const config: CodegenConfig = {
	schema: './schema.graphql',
	generates: {
		'./src/mocks/generated/graphql.ts': {
			plugins: [
				'typescript',
				'typescript-operations',
				'@graphql-codegen/typescript-msw',
				{
					'graphql-codegen-typescript-mock-data': {
						prefix: 'mock'
					}
				}
			]
		}
	}
};

export default config;
```

`package.json`の`scripts`に`"generate": "graphql-codegen"`を追加後、`pnpm generate`を実行することで指定したディレクトリに GraphQL の型が出来る。

次に`src/mocks/handlers.ts`にモックの中身を書く。

```ts:src/mocks/handlers.ts
import { graphql, HttpResponse } from 'msw';
import { mockUser } from './generated/graphql';

export const handlers = [
	graphql.query('GetUsers', () => {
		return HttpResponse.json({
			data: {
				users: [...new Array(5).keys()].map((id) =>
					mockUser({
						id: id + 1,
						name: `User${id + 1}`
					})
				)
			}
		});
	})
];
```

モックを使用するためのコードを用意します。SvelteKit では SSR をデフォルトで使用しているのでクライアント側とサーバ側でそれぞれ必要です。

```ts:src/mocks/browser.ts
import { setupWorker } from 'msw/browser';
import { handlers } from './handlers';

export const worker = setupWorker(...handlers);
```

```ts:src/mocks/node.ts
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);
```

```ts:src/hooks/client.ts
import { dev } from '$app/environment';

if (dev) {
	const { worker } = await import('./mocks/browser');

	await worker.start({ onUnhandledRequest: 'bypass' });
}
```

```ts:src/hooks.server.ts
import { dev } from '$app/environment';

if (dev) {
	const { server } = await import('./mocks/node');
	server.listen();
}
```

最後に以下のコマンドで`static/mockServiceWorker.js`を生成します。

```bash
npx msw init static/ --save
```

参考にした記事
https://zenn.dev/k_sato/articles/40f028d138c68a

## おわりに

最近仕事で Svelte + GraphQL の開発をしていたので、自分への備忘録を兼ねて記事にまとめました。余談ですが Svelte5 で[Rune](https://svelte.jp/blog/runes)が使えるようになったので state 周りの書き方が大分変わりそうです。次はこの辺りを深掘りしたいと思っています。
ところで今年はあまりテックブログを書けませんでしたね、アドカレ以外でも書けるよう来年は頑張ります。ではみなさん良いお年を！
