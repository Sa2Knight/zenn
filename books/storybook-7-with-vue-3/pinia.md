---
title: pinia でグローバルステートを管理する
free: false
---

本章では、[Pinia](https://pinia.vuejs.org/) を用いたグローバルステートをアプリケーションに導入し、`Storybook` 上でもそれを扱えるようにします。

なお、`Vue.js` コミュニティの推奨に基づいて `Pinia` を使用しますが、 `vuex` でも大きな違いはありません。

```bash
$ yarn add pinia
```

# カレントユーザーストアを作成する

これまで、ユーザーのログイン情報は、以下のように `MyPage` コンポーネントがステートで保持している実装でした。

```ts:src/components/MyPage.vue
type User = {
  id: number;
};

const user = ref<User | null>(null);

function login() {
  user.value = { id: 1 };
}

function logout() {
  user.value = null;
}
```

実際のアプリケーションでは、ログイン中ユーザーの情報というのはアプリケーション各所で読み書きされることが多いです。

複数のコンポーネントから同一データを参照するための仕組みは様々ありますが、ここでは `Pinia` を使用したグローバルストアを作成します。

今回は `currentUserStore` の一種のみ作成するので、ストアの定義と `Pinia` のセットアップをまとめて `src/pinia.ts` で行います。

```ts:src/pinia.ts
import { createPinia, defineStore } from "pinia";

export type User = {
  id: number;
};

/**
 * ログイン中ユーザーの読み書きを行うストア
 */
export const useCurrentUserStore = defineStore("currentUser", {
  state: () => ({
    user: null as User | null,
  }),
  getters: {
    isLoggedIn(): boolean {
      return !!this.user;
    },
  },
  actions: {
    login(user: User) {
      this.user = user;
    },
    logout() {
      this.user = null;
    },
  },
});

/**
 * Pinia のセットアップ
 */
export const pinia = createPinia();

export default pinia;
```

`createPinia` で作成した I18n の設定は、Vue プラグインの形式になっているため、開発環境で動作確認するために `src/main.ts` も修正します。

```ts:src/main.ts
import { createApp } from "vue";
import App from "./App.vue";
import i18n from "./i18n";
import pinia from "./pinia";

createApp(App).use(i18n).use(pinia).mount("#app");
```

# `MyPage` からストアを参照する

`currentUserStore` を使用して、 `MyPage` コンポーネントからこれを読み書きするように修正します。

```ts:src/components/MyPage.vue
import { useCurrentUserStore } from "../pinia";
import MyHeader from "./MyHeader.vue";

const currentUserStore = useCurrentUserStore();

function login() {
  currentUserStore.login({ id: 1 });
}

function logout() {
  currentUserStore.logout();
}

function signUp() {
  // TODO: 会員登録フォームに移動する
}
```

同様にテンプレート側も、ユーザーがログイン中かをストアのメソッドを参照するようにします。

```html:src/components/MyPage.vue
<template>
  <div>
    <MyHeader
      :isLoggedIn="currentUserStore.isLoggedIn" <- ここを修正
      @login="login"
      @logout="logout"
      @signUp="signUp"
    />
    <main>
      <div class="content-wrapper">
        <div class="content" :key="i" v-for="i in 10">
          {{ $t("page.content").repeat(20) }}
        </div>
      </div>
    </main>
  </div>
</template>
```

これで開発環境上での対応は完了しました。開発環境上ではこれまで通り、ログインボタンを押下するとヘッダーメニューが切り替わるでしょう。

# `Storybook` 側にも `Pinia` を適用する

現在の状態で `Storybook` を開き、 `MyPage` コンポーネントのストーリーを開きと、以下のようなエラーが発生します。

```bash
$ yarn dev storybook --port 6006
```

![](https://storage.googleapis.com/zenn-user-upload/e31b268d039f-20221231.png)

`Vue I18n` と同じ流れになりますが、こちらも `Storybook` 用の Vue インスタンスに対して `Pinia` プラグインを適用する必要があります。

```ts:.storybook/preview.ts
import pinia from "../src/pinia";

// 中略

setup((app) => {
  // app が Vue インスタンスにあたるので Vue I18n / Pinia インスタンスを注入する
  app.use(i18n);
  app.use(pinia);
});
```

これで `Storybook` 上でもグローバルストアが参照できるようになります。

ただし、このままだとすべてのストーリーでストアを共有しているので、あるストーリーで書き換えた状態が他のストーリーでも引き継がれてしまいます。

それが意図的であれば問題ないのですが、ストーリーの描画内容に冪等性を持たせたい場合は、以下のように `Decorator` 内で状態の初期化を行います。

```ts:.storybook/preview.ts
import pinia, { useCurrentUserStore } from "../src/pinia";

// 中略

export const decorators: Decorator[] = [
  (story, context) => {
    i18n.global.locale = context.globals.locale;
    return {
      setup() {
        const currentUserStore = useCurrentUserStore();
        currentUserStore.$reset();
      },
      template: "<story />",
    };
  },
];

// 以下略
```

これでストーリーを新たに描画するたびに、グローバルストアが初期化されるため、どの順でストーリーを開いても同じ結果を得られるようになります。

# 関連リンク

https://pinia.vuejs.org/