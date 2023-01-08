---
title: 「ストーリー」と CSF
free: true
---

前章にて、`MyButton.stories.ts` という、`Storybook` で使用するファイルを作成しました。

`.stories.js` のようなファイル形式を **CSF (Component Story Format)** と呼びます。

:::message
厳密には `CSF` を使わずともストーリーの定義は可能ですが、非推奨で廃止予定のため、本書ではストーリーファイル = `CSF` とします。
:::

以下は前章で作成した、`CSF` で書かれたストーリーファイルです。

```ts:src/stories/MyButton.stories.ts
import MyButton from "../components/MyButton.vue";
import type { Meta, StoryObj } from "@storybook/vue3";

type Story = StoryObj<typeof MyButton>;

const meta: Meta<typeof MyButton> = {
  title: "MyButton",
  component: MyButton,
};

export const Default: Story = {
  render: () => ({
    components: { MyButton },
    template: "<MyButton label='ボタン' />",
  }),
};

export default meta;
```

`Storybook` における「ストーリー」とは、コンポーネントを `Storybook` 上で描画する最小単位であり、指定されたレンダリング方法に基づいてコンポーネントをレンダリングします。

**ストーリー = コンポーネントを描画するコンポーネント + メタデータ** と考えても良いでしょう。

`CSF` では、ストーリー及びそのメタデータを [ESM](https://developer.mozilla.org/ja/docs/Web/JavaScript/Guide/Modules) を用いて定義します。`CSF` 自体は純粋な JavaScript モジュールであることから、 `Storybook` と疎結合であるため、 `Storybook` 以外からも活用できるのがポイントです。

:::message
`Storybook 7` では `CSF 3` という最新のバージョンを使用できるようになり、本書でも最新版を使用しています。`CSF 2` 以前とは記法が異なるのでご注意ください。
:::
