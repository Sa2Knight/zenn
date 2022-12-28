---
title: 「ストーリー」と CSF
free: true
---

前章にて、`MyButton.stories.ts` という、`Storybook` で読み込むためのファイルを作成しました。

`.stories.js` のようなファイル形式を **CSF (Component Story Format)** と呼びます。以下は前章で作成した、`CSF` で書かれたストーリーファイルです。

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

`Storybook` における「ストーリー」とは、コンポーネントを `Storybook` 上で描画する最小単位であり、指定されたレンダリング方法に基づいてコンポーネントをレンダリングします。

`CSF` は、ストーリー及びそのメタデータを [ESM](https://developer.mozilla.org/ja/docs/Web/JavaScript/Guide/Modules) を用いて定義します。

`CSF` 自体は純粋な JavaScript モジュールであることから、 `Storybook` と疎結合であるため、 `Storybook` 以外からも活用できるのがポイントです。

:::message
`Storybook 7` では `CSF 3` という最新のバージョンを使用できるようになり、`CSF 2` 以前とは記法が異なるのでご注意ください。
:::
