---
title: Vue I18n での多言語化に対応する
---

```bash
yarn add vue-i18n@9
```

```ts:src/i18n.ts
import { createI18n } from "vue-i18n";

const i18n = createI18n({
  locale: "ja",
  fallbackLocale: "ja",
  messages: {
    ja: {
      header: {
        title: "Storybook 7 + Vue 3 + TypeScript サンプル",
        login: "ログイン",
        logout: "ログアウト",
        signUp: "会員登録",
      },
      page: {
        content: "コンテンツがここに表示されます。",
      },
    },
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

```ts:src/main.ts
import { createApp } from "vue";
import i18n from "./i18n";
import MyPage from "./components/MyPage.vue";

createApp(MyPage).use(i18n).mount("#app");
```

```html:src/components/header.vue
<template>
  <header>
    <h1>{{ $t("header.title") }}</h1>
    <div class="buttons">
      <template v-if="props.user.id">
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

```ts:.storybook/preview.ts
import i18n from "../src/i18n";
import { setup } from "@storybook/vue3";

setup((app) => {
  app.use(i18n);
});
```

```bash
$ yarn add -D @storybook/addon-toolbars@7.0.0-beta.14
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

```ts:.storybook/preview.ts
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
```

```ts:.storybook/preview.ts
export const decorators: Decorator[] = [
  (story, context) => {
    i18n.global.locale = context.globals.locale;
    return { template: "<story />" };
  },
];
```

# 関連リンク

https://vue-i18n.intlify.dev/
https://storybook.js.org/docs/7.0/vue/essentials/toolbars-and-globals