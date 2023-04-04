---
title: はじめてのストーリー作成
free: true
---

本章では、コンポーネントのストーリーを作成し、`Storybook` を本格的に動かしていきます。

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

export const Default: Story = {
  render: () => ({
    components: { MyButton },
    template: "<MyButton label='ボタン' />",
  }),
};

export default meta;
```

ストーリーファイルの中身を見る前に、まずは `Storybook` を起動して、動作を確認します。

```bash
$ yarn storybook dev --port 6006
```

以下のように、`Storybook` のサンドボックス上にコンポーネントが描画されました。これではじめてのストーリーが完成しました。

![](https://storage.googleapis.com/zenn-user-upload/874d250953dc-20221225.png)
