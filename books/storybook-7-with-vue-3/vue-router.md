---
title: Vue Router でのルーティング
---

本章では、 [Vue Router](https://router.vuejs.org/) を使用したルーティングをアプリケーションに組み込み、 `Storybook` 上でもルーティングを行えるようにします。

# Vue Router のセットアップ

```bash
$ yarn add vue-router@4
```

# ルーターを作成する

ここでは、 `/` と `/profile` のルートを作成します。

|ルート|コンポーネント|備考|
|---|---|---|
|`/`|`topPage`|`/profile` へのリンクあり|
|`/profile`|`profilePage`|`/` へのリンクあり|

まず、両方ルートに対応するページコンポーネントを作成します。両ページはもう一方へのリンクを持つだけのシンプルなページになってます。

```vue:src/pages/topPage.vue
<script setup lang="ts"></script>

<template>
  <div>
    <h1>Top Page</h1>
    <router-link to="/profile">Profile</router-link>
  </div>
</template>
```

```vue:src/pages/profilePage.vue
<script setup lang="ts"></script>

<template>
  <div>
    <h1>Profile Page</h1>
    <router-link to="/">Top</router-link>
  </div>
</template>
```


`src/router.ts` にて、ルート定義とインスタンス生成をまとめて実装します。

```ts:src/router.ts
import * as VueRouter from "vue-router";
import TopPage from "../src/pages/TopPage.vue";
import ProfilePage from "../src/pages/ProfilePage.vue";

const routes: VueRouter.RouteRecordRaw[] = [
  {
    path: "/",
    name: "Top",
    component: () => TopPage,
  },
  {
    path: "/profile",
    name: "Profile",
    component: () => ProfilePage,
  },
];

export const createRouter = (type: "memory" | "history") => {
  const history = type === "memory" ? VueRouter.createMemoryHistory() : VueRouter.createWebHistory();
  return VueRouter.createRouter({ history, routes });
};
```

`createRouter` 関数にて、 `History Mode` を指定できるようになっています。アプリケーション上では実際の URL を使用するのに対して、 `Storybook` では `iframe` 上にストーリーが埋め込まれる都合、URL が使用できないためです。

https://router.vuejs.org/guide/essentials/history-mode.html

よって、アプリケーションコードである `src/main.ts` からは `createWebHistory` を使うようにオプションを指定してセットアップします。

```ts:src/main.ts
import { createApp } from "vue";
import App from "./App.vue";
import i18n from "./i18n";
import pinia from "./pinia";
import { createRouter } from "./router";

createApp(App).use(i18n).use(pinia).use(createRouter("history")).mount("#app");
```

ルーターの設定に基づいて、ルートに対応するコンポーネントを描画するように `App.vue` を書き換えます。

```vue:src/App.vue
<template>
  <router-view />
</template>
```

これで開発環境を立ち上げると、 `/` と `/profile` を行き来できるようになっています。

![](https://storage.googleapis.com/zenn-user-upload/80f151429dc2-20230101.gif)

これを、`Storybook` 上でも `TopPage` `ProfilePage` をストーリー化し、同様に画面遷移できるようにしていきます。

# `Storybook` 側にも　`Vue Router` を組み込む

`Vue I18n` `Pinia` の章と同様になりますが、 `Storybook` 用の Vue インスタンスにも `VueRouter` プラグインを適用する必要があります。

アプリケーションコードと異なり、URL を直接使えないため、 `createMemoryHistory` を使ったインメモリルーターを生成します。

```ts:.storybook/pinia
import i18n from "../src/i18n";
import pinia, { useCurrentUserStore } from "../src/pinia";
import { createRouter } from "../src/router";
import { setup } from "@storybook/vue3";

// 中略

// Storybook 用にインメモリルーターを作成する
const router = createRouter("memory");

setup((app) => {
  // app が Vue インスタンスにあたるので Vue I18n / Pinia / VueRouter インスタンスを注入する
  app.use(i18n);
  app.use(pinia);
  app.use(router);
});
```

これで `Storybook` 上でも `Vue Router` が使えるようになりました。

# ページコンポーネント用のストーリーを作成する

ここでは、ページコンポーネント(`TopPage` `ProfilePage`) それぞれのストーリーを一つのファイルにまとめて実装します。

本ファイルは `src/App.vue` と同じ用に、現在のルートに対応するコンポーネントを描画するようにします。

```ts:src/storybook/Pages.stories.ts
import type { StoryObj } from "@storybook/vue3";
import { useRouter } from "vue-router";
import { ref } from "vue";

const meta = {
  title: "Pages",
  parameters: {
    backgrounds: {
      default: "light", // 背景色が白い前提で実装しているのでデフォルトで白色にする
    },
  },
};

function createPageStory(name: string): StoryObj {
  return {
    render: () => ({
      setup() {
        const pageLoaded = ref(false);
        const router = useRouter();
        router.push({ name }).then(() => {
          pageLoaded.value = true;
        });

        return { pageLoaded };
      },
      template: `<template v-if="pageLoaded"><router-view /></template>`,
    }),
  };
}

export default meta;

export const Top = createPageStory("Top");
export const Profile = createPageStory("Profile");
```

ややトリッキーですが、やっていることはストーリーの生成を `createPageStory` 関数にて動的に行っているだけです。

各ページコンポーネントのストーリーは、ルート名(`Top` など) に基づいたルートへ `router.push` で遷移します。遷移が完了したら完了フラグ(`pageLoaded`)をセットし、 `<router-view />` を描画するようにすることで、それぞれのルートに基づいたページコンポーネントを描画しています。

`Storybook` を開くと、 `Pages` という階層が追加され、そこから `Top` `Profile` のストーリーを開くことができます。また、 `Storybook` 上でも `Vue Router` を使用できるようにしたので、それぞれのストーリーからもリンクをたどることが出来ます。

![](https://storage.googleapis.com/zenn-user-upload/32649558d3ed-20230101.gif)

# 関連リンク

https://router.vuejs.org/