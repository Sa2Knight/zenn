---
title: Vue I18n での多言語化に対応する
---

サービスを海外展開する場合、UI を国際化し、テキストも多言語対応することがあるでしょう。

本章では、`Vue.js` における国際化・多言語化のデファクトスタンダードといえる、[Vue I18n](https://vue-i18n.intlify.dev/) と `Storybook` を組み合わせる手法を紹介します。

# Vue I18n のセットアップ

`Vue I18n` は、名前の通り `Vue.js` アプリケーションにプラグインの形式で多言語化を実装するためのパッケージです。 まずは `Vue 3` にあわせたバージョンをインストールしましょう。

```bash
yarn add vue-i18n@9
```

`Vue I18n` の詳細については割愛しますが、本章では以下のような多言語対応をします。

- 日本語・英語の二言語をサポート
- `MyPage` コンポーネント上に表示されるすべての文言を対応(子コンポーネント含む)
- デフォルトは日本語

`src/i18n.ts` を作成し、以下のように i18n を実装します。

```ts:src/i18n.ts
import { createI18n } from "vue-i18n";

const i18n = createI18n({
  // デフォルトで日本語を使用する
  locale: "ja",
  fallbackLocale: "ja",
  messages: {
    // 日本語の定義
    ja: {
      // MyHeader コンポーネント内の文言
      header: {
        title: "Storybook 7 + Vue 3 + TypeScript サンプル",
        login: "ログイン",
        logout: "ログアウト",
        signUp: "会員登録",
      },
      // myPage コンポーネント内の文言
      page: {
        content: "コンテンツがここに表示されます。",
      },
    },
    // 英語の定義
    en: {
      header: {
        title: "Storybook 7 + Vue 3 + TypeScript Sample",
        login: "Login",
        logout: "Logout",
        signUp: "Sign Up",
      },
      page: {
        content: "Content will be displayed here\n",
      },
    },
  },
});

export default i18n;
```

:::message
通常はユーザーエージェントなどを元にユーザーの言語設定を決定し、`Vue I18n` にそれを反映するというプロセスをはさみますが、ここでは割愛してハードコードしています。
:::

`createI18n` で作成した I18n の設定は、Vue プラグインの形式になっているため、開発環境で動作確認するために `src/main.ts` も修正します。

```ts:src/main.ts
import { createApp } from "vue";
import App from "./App.vue";
import i18n from "./i18n";

createApp(App).use(i18n).mount("#app");
```

# コンポーネント側の I18n 対応

現在は `MyHeader` `MyPage` コンポーネントにハードコードしている文言を、 `$t` 関数で I18n 化します。

```html:src/components/MyHeader.vue
<template>
  <header>
    <h1>{{ $t("header.title") }}</h1>
    <div class="buttons">
      <template v-if="props.isLoggedIn">
        <MyButton
          size="small"
          :label="$t('header.logout')"
          @click="$emit('logout')"
        />
      </template>
      <template v-else>
        <MyButton
          size="small"
          :label="$t('header.login')"
          @click="$emit('login')"
        />
        <MyButton
          size="small"
          :label="$t('header.signUp')"
          @click="$emit('signUp')"
        />
      </template>
    </div>
  </header>
</template>
```

```html:src/components/MyPage.vue
<template>
  <div>
    <MyHeader
      :isLoggedIn="!!user"
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

ここまでで、開発環境上での I18n 対応が完了しました。開発サーバーを起動して動作を確認してみましょう。

```bash
$ yarn dev
```

これまでと同様の画面が表示されていますが、 `Vue I18n` の `locale` 設定を `en` に変えてみましょう。

```ts:src/i18n.ts
import { createI18n } from "vue-i18n";

const i18n = createI18n({
  // locale: "ja",
  locale: "en",
  fallbackLocale: "ja",
  messages: {
// 以下略
```

以下のように、テキストが英語に切り替わっていれば完了です。

![](https://storage.googleapis.com/zenn-user-upload/b62a0d5cd3cf-20221230.png)

開発環境上では完成したので、これを `Storybook` でも確認できるようにしていきます。

# `Storybook` 側にも `Vue I18n` を適用する

現在の状態で `Storybook` を開き、 `MyHeader` または `MyPage` コンポーネントのストーリーを開いてみましょう。

```bash
$ yarn dev storybook --port 6006
```

以下のようなエラーが発生します。

![](https://storage.googleapis.com/zenn-user-upload/c0810fdc60ac-20221230.png)

これは、 `Storybook` がストーリーを描画する際に使用している Vue インスタンスに、 `VueI18n` プラグインを注入していないため発生するエラーです。

:::message
先程 `src/main.ts` で設定したのはアプリケーションルートの Vue インスタンスであって、 `Storybook` 用のそれとは関係ないのでご注意ください。
:::

`Storybook` 用の Vue インスタンスを拡張する場合は、 `.storybook/preview.ts` にて `setup` 関数を使用します。

```ts:.storybook/preview.ts
import i18n from "../src/i18n";
import { setup } from "@storybook/vue3";

// 中略

setup((app) => {
  // app が Vue インスタンスにあたるので Vue I18n インスタンスを注入する
  // 同一の Vue インスタンスに対して setup 関数は複数回実行されるため、既に注入済みかを確認する
  if (!app.__VUE_I18N__) {
    app.use(i18n);
  }
});

// 以下略
```

これでストーリーが正しく描画できるようになりました。

ただし現状では、デフォルト設定である日本語が表示され、英語に変更する手段がありません。

# Toolbar addon のセットアップ

本章では、日本語・英語の切り替えをコードやブラウザ設定なしに、ツールバーから切り替えられるようにします。

`Storybook` におけるツールバーとは、これまで `Viewport addon` や `Background addon` で使用してきたこの部分です。

![](https://storage.googleapis.com/zenn-user-upload/225e71ba68ec-20221230.png)

ツールバーは `@storybook/addon-toolbars` を使用すれば任意にカスタマイズし、ツールバーで選択された値を参照もできるようになるため、まずはアドオンを追加します。

```bash
$ yarn add -D @storybook/addon-toolbars@7.0.0-beta.20
```

```ts:.storybook/main.ts
  addons: [
    "@storybook/addon-controls",
    "@storybook/addon-actions",
    "@storybook/addon-viewport",
    "@storybook/addon-backgrounds",
    "@storybook/addon-measure",
    "@storybook/addon-outline",
    "@storybook/addon-docs",
    "@storybook/addon-interactions",
    "@storybook/addon-toolbars", // 追加
  ],
```

# 言語切り替えメニューを追加する

`Toolbar addon` を使用したツールバーの拡張は `.storybook/preview.ts` 内の `globalTypes` で定義します。

`globalTypes` は、各ストーリーごとでなく、 `Storybook` 全体で一意の設定を定義するための key-value ストアです。

ここでは `locale` という名前の `globalType` を定義し、ツールバー上から `ja` または `en` を選択できるようにします。

```ts:.storybook/preview.ts
// 前略

export const globalTypes = {
  locale: {
    name: "Locale",
    description: "多言語設定",
    defaultValue: "ja",
    toolbar: {
      icon: "globe",
      items: ["ja", "en"],
    },
  },
};

// 以下略
```

`Storybook` を起動するとツールバーにメニューが追加され、言語を選択できるようになります。

![](https://storage.googleapis.com/zenn-user-upload/16bf778e00e9-20221230.png)

# globalType を用いて `Vue I18n` の `locale` を変更する

`globalType` を定義して、ツールバーからその設定を変更しても、 `Vue I18n` インスタンスにその変更が反映されていないため何も起こりません。

`globalType` は主に `Decorator` から参照します。`Decorator` を使って、ストーリーを描画するたびに、 `globalType` の設定値を参照し、 `Vue I18n` の設定に反映させます。

```ts:.storybook/preview.ts
// 前略

export const decorators: Decorator[] = [
  (story, context) => {
    i18n.global.locale = context.globals.locale;
    return { template: "<story />" };
  },
];

// 以下略
```

`globalType` は `Decorator` の第二引数から参照可能で、 `i18n.global.locale` にその設定を反映することで、 I18n で使用する言語が切り替わります。

![](https://storage.googleapis.com/zenn-user-upload/b84102924e15-20221230.gif)

このように、 `globalType` と `Decorator` を組み合わせることで、GUI 上からアプリケーション上の設定を柔軟に変更できるようになります。

# 関連リンク

https://vue-i18n.intlify.dev/
https://storybook.js.org/docs/7.0/vue/essentials/toolbars-and-globals