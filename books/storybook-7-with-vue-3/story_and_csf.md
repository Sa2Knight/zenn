---
title: 「ストーリー」と CSF
---

前章にて、`Counter.stories.ts` という、`Storybook` で読み込むためのファイルを作成しました。

`.stories.js` のようなファイル形式を **CSF (Component Story Format)** と呼びます。以下は前章で作成した、`CSF` で書かれたストーリーファイルです。

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

`Storybook` における「ストーリー」とは、コンポーネントを `Storybook` 上で描画する最小単位であり、指定されたレンダリング方法に基づいてコンポーネントをレンダリングします。

`CSF` は、ストーリー及びそのメタデータを [ESM](https://developer.mozilla.org/ja/docs/Web/JavaScript/Guide/Modules) を用いて定義します。

`CSF` 自体は純粋な JavaScript モジュールであることから、 `Storybook` と疎結合であるため、 `Storybook` 以外からも活用できるのがポイントです。

:::message
`Storybook 7` では `CSF 3` という最新のバージョンを使用できるようになり、`CSF 2` 以前とは記法が異なるのでご注意ください。
:::
