---
title: PagefindをSolid.js・SolidStartで利用する際の落とし穴
date: 2026-01-13
tags:
  - 開発
  - フロントエンド
  - SolidJS
---

## pagefind.jsがインポートできない

pagefindについて調べると、下記のようなコードを記述する記事が見つかります。

```ts
export async function loadPagefind() {
  if (window.pagefind) return;
  
  try {
    window.pagefind = await import(
      // @ts-expect-error
      "/pagefind/pagefind.js"
    );
  } catch (_e) {
    window.pagefind = {
      search: async () => ({ results: [] }),
    };
  }
}
```

ですが、このコードを実際にSolidJS・SolidStartで使用してもエラーが発生します。

原因は、vite-analyzerが動的import文を解析してしまい、ビルド前の時点では`"/pagefind/pagefind.js"`が存在しないこと。

pagefindは`"/pagefind/pagefind.js"`を実行時（インデックス作成時）に作成します。なのでSSGを使用するようなサイトでは、インデックスするページ自体が作られるビルド後に走らせるような形になります。一方でvite-analyzerはすでに存在するものとしてインポートを解析します。そこで辻褄が合わなくなるわけです。

## 解決策

ビルド後のサイトアクセス時まで読み込まれない場所に記述する必要があります。

SolidStartでは下記の`entry-server.tsx`のhead要素内が該当します。なので下記のように記述。

```jsx title="entry-server.tsx"
...
	<link rel="icon" href="/favicon.png" />
	<Show when={import.meta.env.PROD}>
		<script src="/pagefind/pagefind.js" type="module" defer />
		<script
			type="module"
			innerHTML={`
				import * as pagefind from "/pagefind/pagefind.js";
				window.pagefind = pagefind;
			`}
			defer
		/>
	</Show>
	{assets}
</head>
...
```

ここで読み込むことでエラーが発生しなくなりました。pagefindはSSGなどビルドを要するようなものと組み合わせるのがややこしいですね。。
