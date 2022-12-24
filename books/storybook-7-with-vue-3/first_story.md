---
title: はじめてのストーリー作成
---

`Counter` コンポーネントのストーリーを作成し、`Storybook` 上で動作を確認できるようにします。

# アンビエント宣言の追加

出鼻をくじくようですが、ストーリーファイルを作成する前に、 `Counter.vue` のような `.vue` ファイルを `.ts` ファイルから `import` できるように、アンビエント宣言を追加します。

`src/vite-env.d.ts` の内容を以下のようにしてください。

```ts:src/vite-env.d.ts
/// <reference types="vite/client" />

declare module "*.vue" {
  import type { DefineComponent } from "vue";
  const component: DefineComponent<{}, {}, any>;
  export default component;
}
```

これによって、 `.ts` ファイルから `.vue` ファイルを `import` する際に、そのモジュールが Vue コンポーネントであることを TypeScript に伝えます。

:::message
この宣言によって、すべての Vue コンポーネントが空のコンポーネントと推論されてしまいますが、他の手段が見当たりませんでした。なにか良いプラクティスがあれば教えて下さい。
:::

# ストーリーファイルの作成

`src/stories/Counter.stories.ts` のファイル名で、以下のようなストーリーを作成しましょう。

```ts:src/stories/Counter.stories.ts
import Counter from "../components/Counter.vue";
import type { Meta, StoryObj } from "@storybook/vue3";

type Story = StoryObj<typeof Counter>;

const meta: Meta<typeof Counter> = {
  title: "Counter",
  component: Counter,
};

export default meta;

export const Default: Story = {
  render: () => ({
    components: { Counter },
    template: '<Counter v-bind="args" />',
  }),
};
```

ストーリーファイルの中身を見る前に、まずは `Storybook` を起動して、動作を確認します。

```bash
$ yarn storybook dev
```

以下のように、`Storybook` のサンドボックス上にコンポーネントが描画されました。 `-` `+` ボタンを押下するとカウンターが増減することも確認できます。

![](https://storage.googleapis.com/zenn-user-upload/eaa705b85efc-20221224.png)