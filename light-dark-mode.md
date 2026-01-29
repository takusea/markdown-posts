---
title: 【2026最新】Webでテーマをライト・ダーク・システムで切り替えられるようにする
date: 2026-01-28
tags:
  - 開発
  - フロントエンド
  - CSS
---

現状、Webでライト/ダークなどテーマを設定するには、CSS・JSを用いたいくつかの方法があります。今回はその中でも一番シンプルに、テーマをライト・ダーク・システムで切り替えられるようにする方法を紹介します。

## prefers-color-scheme（従来の方法）

比較的前から存在する方法として、メディア特性の`prefers-color-scheme`を用いてテーマを検出し、色を設定し分けることができました。

```css
.hamu {
  background-color: #fff;
  color: #000;
}

@media (prefers-color-scheme: dark) {
  .hamu {
    background-color: #000;
    color: #fff;
  }
}
```

こちらの方法でもいいですが、セレクタなど同じような記述を2度書く必要があり少々面倒です。

## light-dark()（新しい記述）

近年では`light-dark()`でより単純に記述できるようになりました。

```css
.hamu {
  background-color: light-dark(#fff, #000);
  color: light-dark(#000, #fff);
}
```

ただ、2024年に利用可能になったという比較的新しめな状況なので、互換性には注意。

https://developer.mozilla.org/ja/docs/Web/CSS/Reference/Values/color_value/light-dark

## color-scheme（テーマ指定）

`color-scheme`は何のテーマに対応しているかを示すためのプロパティです。上記の`light-dark()`を動作させるためにも必要です。

```css
:root {
  /* ユーザーの配色設定に合わせる */
  color-scheme: light dark;
  /* ライトテーマのみ */
  color-scheme: only light;
  /* ダークテーマのみ */
  color-scheme: only dark;
}
```

片テーマ対応の際の`only`は必須ではなく、ブラウザで自動ダークテーマなどの機能がある場合、それを無効化する効果があるようです。

このプロパティをJavaScriptから変更することで、テーマ切り替えを実現できます。

### Javascriptから書き換える（直接）

```js
function changeTheme(theme) {
  const root = document.documentElement;

  switch (theme) {
    case "light":
      root.style.colorScheme = "light";
      break;
    case "dark":
      root.style.colorScheme = "dark";
      break;
    case "system":
      root.style.colorScheme = "light dark";
      break;
  }
}
```

### Javascriptから書き換える（data属性越し）

もしくはclassやdata属性を変更することで動作させる方法も考えられます。styleは優先度が強かったりと扱いが難しいので極力使わず、こちら側のような方法が望ましく思います。

```css
:root {
  color-scheme: light dark;
}

:root[data-theme="light"] {
  color-scheme: only light;
}

:root[data-theme="dark"] {
  color-scheme: only dark;
}
```

```js
function changeTheme(theme) {
  const root = document.documentElement;

  if (theme) {
    root.dataset.theme = theme;
  } else {
    delete root.dataset.theme;
  }
}
```

## おわり

以前は`light-dark()`もなく現在のテーマ管理もJavaScript上で行わなければならないなど、ややこしい部分もありました。現状ではJavaScript側は`color-scheme`の切り替えで、あとはCSS側の記述でどうにかできるようになったので、簡潔に記述できるようになりました。いい時代です。
