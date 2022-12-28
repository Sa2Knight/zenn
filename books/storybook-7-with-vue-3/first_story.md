---
title: はじめてのストーリー作成
free: true
---

`Counter` コンポーネントのストーリーを作成し、`Storybook` 上で動作を確認できるようにします。

# アンビエント宣言の追加

出鼻をくじくようですが、ストーリーファイルを作成する前に、 `MyButton.vue` のような `.vue` ファイルを `.ts` ファイルから `import` できるように、アンビエント宣言を追加します。

`src/vite-env.d.ts` の内容を以下のようにしてください。

```ts:src/vite-env.d.ts
/// <reference types="vite/client" />

declare module "*.vue" {
  import type { DefineComponent } from "vue";
  const component: DefineComponent<{}, {}, any>;
  export default component;
}
```

これによって、 `.ts` ファイルから `.vue` ファイルを `import` する際に、そのモジュールが Vue コンポーネントであることを `TypeScript` に伝えます。

:::message
この宣言によって、すべての Vue コンポーネントが空のコンポーネントと推論されてしまいますが、他の手段が見当たりませんでした。なにか良いプラクティスがあれば教えて下さい。
:::

# ストーリーファイルの作成

はじめの一歩として、 `MyButton` コンポーネントのストーリーを作成しましょう。

`src/stories/MyButton.stories.ts` のファイル名で、内容は以下のようにします。

```ts:src/stories/MyButton.stories.ts
import MyButton from "../components/MyButton.vue";
import type { Meta, StoryObj } from "@storybook/vue3";

type Story = StoryObj<typeof MyButton>;

const meta: Meta<typeof MyButton> = {
  title: "MyButton",
  component: MyButton,
};

export default meta;

export const Default: Story = {
  render: () => ({
    components: { MyButton },
    template: "<MyButton label='ボタン' />",
  }),
};
```

ストーリーファイルの中身を見る前に、まずは `Storybook` を起動して、動作を確認します。

```bash
$ yarn storybook dev
```

以下のように、`Storybook` のサンドボックス上にコンポーネントが描画されました。これではじめてのストーリーが完成しました。

![](https://storage.googleapis.com/zenn-user-upload/874d250953dc-20221225.png)
