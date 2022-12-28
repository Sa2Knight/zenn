---
title: ドキュメントを生成する
free: true
---

# Docs addon について

`Docs addon` は、`essentials addons` の一つで、 `Storybook` 上で、 **コンポーネントのドキュメントを提供する** アドオンです。

ドキュメントには **ストーリーから自動生成** できるものや、 `MDX` を用いてカスタマイズする手法まであります。

# アドオンのインストール

`essentials addons` シリーズはこれで最後になります。

```bash
yarn add -D @storybook/addon-docs@7.0.0-beta.14
```

```ts:.storybook/main.ts
import { StorybookConfig } from "@storybook/vue3-vite";

const config: StorybookConfig = {
  framework: "@storybook/vue3-vite",
  stories: ["../src/**/*.stories.@(js|ts)"],
  addons: [
    "@storybook/addon-controls",
    "@storybook/addon-actions",
    "@storybook/addon-viewport",
    "@storybook/addon-backgrounds",
    "@storybook/addon-measure",
    "@storybook/addon-outline",
    "@storybook/addon-docs", // 追加
  ],
};
```

export default config;

# ドキュメントを自動生成する

本章では `MyButton` コンポーネントのドキュメントを作成します。

`Docs addon` では、 `default export` するメタデータに、 `tags: ["autodocs"]` を付与することで、そのコンポーネントのドキュメントを自動生成できます。

```ts:src/stories/MyButton.stories.ts
// 前略

const meta: Meta<typeof MyButton> = {
  title: "MyButton",
  component: MyButton,
  render: (args) => ({
    components: { MyButton },
    setup() {
      return { args };
    },
    template: "<MyButton v-bind='args' />",
  }),
  tags: ["autodocs"], // ここを追加
  // 以下略
}

// 以下略
```

`Storybook` を確認すると、 `MyButton` コンポーネントに `document` というストーリーが追加され、ドキュメントページを表示できます。

![](https://storage.googleapis.com/zenn-user-upload/ac7ddb6dc1d7-20221227.png)

ドキュメントにはストーリーファイルからインタフェースが推論され、 `props` や `events` の情報が表示されています。

ドキュメントはコードだけでなく、コード中のコメントも抜き出してくれます。 `MyButton` コンポーネントに以下のようにコメントを追加してみましょう。

```vue:src/components/MyButton.vue
<script setup lang="ts">
const props = defineProps({
  /**
   * ボタンに表示するテキスト
   */
  label: {
    type: String,
    required: true,
  },
  /**
   * ボタンのカラーテーマ
   */
  variant: {
    type: String,
    default: "primary",
    validator: (value: string) => {
      return ["primary", "secondary"].includes(value);
    },
  },
  /**
   * ボタンのサイズ
   */
  size: {
    type: String,
    default: "medium",
    validator: (value: string) => {
      return ["small", "medium", "large"].includes(value);
    },
  },
});

const emits = defineEmits([
  /**
   * ボタンがクリックされた
   */
  "click",
]);
</script>

// 以下略
```

`vue-docgen-api` によってコメントも抜き出されて、以下のようにドキュメントに反映されます。

![](https://storage.googleapis.com/zenn-user-upload/4dd4c091dd41-20221227.png)

# MDX でドキュメントを作成する

ドキュメントを自動生成できるのは便利ですが、複雑なコンポーネントの場合はより詳細なドキュメントが必要になることがあります。

`Storybook` では、`MDX` という、 `Markdown` にコンポーネントを埋め込めるようにしたフォーマットを用いたドキュメント作成に対応しています。

本書では `MDX` の詳細には踏み込みませんが、 `MDX` を用いたドキュメント作成の一例を紹介します。

## .mdx ファイルを読み込ませる

`MDX` を用いたドキュメントを作成する場合、 `Storybook` に `.mdx` ファイルを読み込ませる必要があるため、 `.storybook/main.ts` を以下のように修正します。

```ts:.storybook/main.ts
import { StorybookConfig } from "@storybook/vue3-vite";

const config: StorybookConfig = {
  framework: "@storybook/vue3-vite",
    stories: ["../src/**/*.stories.@(js|ts)", "../src/**/*.mdx"], // .mdx を追加
  addons: [
    "@storybook/addon-controls",
    "@storybook/addon-actions",
    "@storybook/addon-viewport",
    "@storybook/addon-backgrounds",
    "@storybook/addon-measure",
    "@storybook/addon-outline",
    "@storybook/addon-docs",
  ],
};

export default config;
```

## MyHeader コンポーネントのドキュメントを作成

`Storybook` における `MDX` では、 `markdown` でドキュメントを書きながら任意のストーリーを埋め込むことができます。

以下では `MyHeader` コンポーネントのドキュメントを、ストーリーを埋め込みつつ作成しています。

```js:src/stories/MyPage.mdx
import { Canvas, Meta, Story, ArgsTable } from '@storybook/blocks';
import MyHeader from '../components/MyHeader.vue';
import * as MyHeaderStories from './MyHeader.stories';

<Meta of={MyHeaderStories} />

# MyHeader

ページタイトルやナビゲーションボタンを表示するヘッダーコンポーネントです。

現在のユーザーを `props` で受け取り、各種ボタン押下時のイベントを発火します。

<ArgsTable of={MyHeader} />

## ログイン済み

ログアウトボタンのみ表示されます。

<Canvas>
  <Story of={MyHeaderStories.Login} />
</Canvas>

## 未ログイン

ログインボタンと会員登録ボタンが表示されます。

<Canvas>
  <Story of={MyHeaderStories.Logout} />
</Canvas>
```

以下のようにそれらしいドキュメントページが出来上がりました。

![](https://storage.googleapis.com/zenn-user-upload/b971febd8e77-20221227.png)

特筆すべきは、 `Story` コンポーネントを用いて、これまでに作成したストーリーをドキュメントに埋め込めることでしょう。

`ArgsTable` のように、 `Storybook` 内で使用されているコンポーネントを `@storybook/blocks` から `import` できます。

`vue-docgen-api` とこれらの組み合わせによって、コンポーネントのリッチなドキュメントを簡単に作成できるようになりました。

# 関連リンク

https://storybook.js.org/docs/7.0/vue/writing-docs/docs-page
https://storybook.js.org/docs/7.0/vue/writing-docs/mdx
https://storybook.js.org/docs/7.0/vue/writing-docs/doc-block-argstable