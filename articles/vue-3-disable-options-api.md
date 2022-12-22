---
title: "Vue 3 で Options API を無効化するという選択肢"
emoji: "😊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["javascript", "vue"]
published: false
---

# 概要

本記事では、Vue 3 で Option API を使用するためのフラグである `__VUE_OPTIONS_API__` を無効化した場合の挙動やバンドルサイズの違いについてまとめています。

# TL;DR

- Vue アプリケーションの **バンドルサイズを 5.49kB 削減** できた
- gzip なら 2.14 kB の程度の削減
- 最初から Composition API 1本と決めているプロジェクトなら無効化する価値はあるが、これのために既存コードを書き換えるほどでなさそう

# バージョン情報

- `vue` 3.2.45
- `vite` 4.0.2
- `@vitejs/plugin-vue` 4.0.0
- `rollup` 3.8.0

# Options API と Composition API

Vue 3 のコンポーネントスタイルには、 `Options API` と `Compositions API` の2種類があります。

前者は Vue 2 時点での基本スタイルで、オブジェクトにコンポーネントの挙動を示す各フィールドを定義します。

後者は Vue 3 から登場した新スタイルで、リアクティブに関する API を使用し、手続き的にコンポーネントを定義します。 `<script setup>` というマクロ記法と合わせることで、 `Svelte` に近いテンプレートバインディングを自然に行うことができます。

```js
// Options API
export default {
  data() {
    return {
      count: 0
    }
  },
  methods: {
    increment() {
      this.count++
    }
  },
  computed: {
    doubleCount() {
      return this.count *2
    }
  }
}
```

```js
// Composition API
import { ref, computed, onMounted } from 'vue'

const count = ref(0)
count doubleCount = computed(() => count * 2)

function increment() {
  count.value++
}
```

`Options API` と `Composition API` はコンポーネントによって使い分けても良いし、一つのコンポーネントで同時に使用することもできます。

関連リンク

https://vuejs.org/guide/introduction.html#api-styles

# `__VUE_OPTIONS_API__`

`__VUE_OPTIONS_API__` (以下 `機能フラグ`) は、Vue 3.0.0-rc.3 から追加された、バンドラ向けのグローバル変数です。

機能フラグは [`webpack`](https://webpack.js.org/) や [`Rollup`](https://rollupjs.org/guide/en/) のようなバンドラから設定可能で、デフォルトでは `true` ですが、これを `false` にすることで `Options API` の機能を完全に無効化することができます。

`Options API` を無効化することで、それに関するコードがツリーシェイキングの対象となり、最終的なバンドルサイズを削減することができます。

関連リンク

https://github.com/vuejs/core/blob/main/packages/vue/README.md#with-a-bundler

# `create-vite` で動作確認用プロジェクトを作成

ここでは [`create-vite`](https://github.com/vitejs/vite/tree/main/packages/create-vite) を用いてスキャフォルドされたミニマムプロジェクトを用いて、機能フラグの設定による挙動の違いを確認します。

```bash
$ yarn create vite vue-3-disable-option-api --template vue
$ cd vue-3-disable-option-api
$ yarn install
```

[`Vite`](https://ja.vitejs.dev/) は開発用サーバーではバンドラが作成されませんが、`Rollup` を用いたバンドルのビルドが可能なため、以下のように動作確認をします。

```bash
$ yarn build
$ yarn preview
```

リアクティブなカウンターが表示されるサンプルアプリケーションが起動しました。

![](https://storage.googleapis.com/zenn-user-upload/1e8d6b6a0560-20221222.png)

# 機能フラグを設定する

`vite` の場合、`vite.config.js` からグローバル変数の注入ができるため、ここで `__VUE_OPTIONS_API__` を設定します。

```js:vite.config.js
import { defineConfig } from "vite";
import vue from "@vitejs/plugin-vue";

export default defineConfig({
  plugins: [vue()],
  define: {
    __VUE_OPTIONS_API__: true,
  },
});
```

この状態でビルドをすると、バンドル(`dist/assets/index-f59c7c2f.js`) は 54.21kB であることがログからわかります。

```bash
$ yarn build
yarn run v1.22.19
$ vite build
vite v4.0.3 building for production...
✓ 16 modules transformed.
dist/index.html                  0.45 kB
dist/assets/vue-5532db34.svg     0.50 kB
dist/assets/index-351bd726.css   1.29 kB │ gzip:  0.66 kB
dist/assets/index-f59c7c2f.js   54.21 kB │ gzip: 21.89 kB
```

続けて、機能フラグを無効にしてみます。

```js:vite.config.js
export default defineConfig({
  plugins: [vue()],
  define: {
    __VUE_OPTIONS_API__: false, // 無効化
  },
});
```

再度ビルドすると、バンドルサイズが 48.72 kB となり、 **5.49 kB 削減できた** ことがわかります。

# 機能フラグが無効の場合の挙動

`create-vite` でスキャフォルドされたコンポーネントは以下のように `Composition API` で書かれているため、機能フラグの有無に関わらず動作しました。

```js
<script setup>
import { ref } from 'vue'

defineProps({
  msg: String,
})

const count = ref(0)
</script>
```

これを `Options API` を使うように書き換えます。

```js
<script>
export default {
  data: () => ({
    count: 0,
  }),
  props: {
    msg: String,
  },
};
</script>
```

この状態でも、機能フラグが有効な場合は動作しますが、無効にするとカウンターの部分が動作動作しなくなる(≒テンプレートにバインドされなくなる)ことがわかります。

![](https://storage.googleapis.com/zenn-user-upload/19cb48d5701b-20221222.png)

しかし、コンソールエラーが出ているわけでもクリックするとクラッシュするわけでもありません。

これがどういうことか、 Vue のコードを追って確認してみましょう。

# Vue は機能グラグをどう見ているか

軽く調べてみると、 `__VUE_OPTIONS_API__` の設定は、バンドル前のコードでは `__FEATURE_OPTIONS_API__` という変数に埋め込まれているようです。

https://github.com/vuejs/core/blob/fe77e2bddaa5930ad37a43fe8e6254ddb0f9c2d7/rollup.config.mjs#L279

`__FEATURE_OPTIONS_API__` を使用している箇所を一つ見てみましょう。以下は Vue インスタンのパブリックメソッドのマッピングです。

https://github.com/vuejs/core/blob/fe77e2bddaa5930ad37a43fe8e6254ddb0f9c2d7/packages/runtime-core/src/componentPublicInstance.ts#L251

`$watch` に対応するメソッドが `instanceWatch` で、 `Options API` における `this.$watch` に該当します。

https://github.com/vuejs/core/blob/fe77e2bddaa5930ad37a43fe8e6254ddb0f9c2d7/packages/runtime-core/src/apiWatch.ts#L397-L426

この関数は前述の `componentPublicInstance` モジュールからしか参照されないため、機能フラグが無効な場合は

```js
$watch: i => (false ? instanceWatch.bind(i) : NOOP)
```

に展開され、 `instanceWatch` が到達不能コードとなり、ツリーシェイキングが成立します。

こういったコード分岐が Vue に仕込まれているおかげで、機能フラグの設定によってバンドルサイズが変わることがわかりました。

また、条件分岐先にある `NOOP` は以下のように何もしない関数として定義されています。

https://github.com/vuejs/core/blob/fe77e2bddaa5930ad37a43fe8e6254ddb0f9c2d7/packages/shared/src/index.ts#L22

そのため、機能フラグを無効化した状態で `Options API` を使用しても、空の関数が呼び出されるだけに書き換わるため、 `Uncaught TypeError` のようなコンソールエラーが発生しなかったというわけですね。