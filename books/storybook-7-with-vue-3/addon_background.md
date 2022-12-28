---
title: 背景色を変更する
free: true
---

`Backgrounds addon` は、`essentials addons` の一つで、 **コンポーネントを描画する際の画面の背景色を変更する** アドオンです。

`Storybook 7` のデフォルトの背景色は黒ですが、コンポーネントによっては背景色が白である前提でスタイルされている場合があります。

また、コンポーネントを実際に描画するであろう画面の背景色を設定することで、背景とコンポーネントの色の調和を確認したくなることもあるでしょう。

そういったケースでは、 `Backgrounds addon` を用いて、背景色を柔軟に変更することで、コンポーネント自体の背景色を際立たせることができます。

:::message
本アドオンは、前章の `Viewport addon` と使用方法が概ね一緒のため、詳細説明やデモは割愛しています。
:::

# アドオンのインストール

例によって、アドオンをインストールし、`.storybook/main.ts` に追加します。

```bash
$ yarn add -D @storybook/addon-backgrounds@7.0.0-beta.14
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
    "@storybook/addon-backgrounds", // 追加
  ],
};

export default config;
```

`Storybook` を再起動すると、画面上部のツールバーに、背景色を変更するメニューが追加されました。

デフォルトでは白と黒から選べます。 (ここで選べる黒は、デフォルトの黒よりは明るい色です。)

![](https://storage.googleapis.com/zenn-user-upload/fad19bf45dad-20221226.gif)

# カスタム背景色を使用する

アプリケーション側でテーマカラーが決まっている場合、それに準じた背景色を用意したくなるでしょう。

Viewport と同様に、 `parameters` フィールドを用いて背景色のリストを定義できます。

```ts
const parameters = {
  backgrounds: {
    default: "twitter",
    values: [
      {
        name: "twitter",
        value: "#00aced",
      },
      {
        name: "facebook",
        value: "#3b5998",
      },
    ],
  },
},
```

![](https://storage.googleapis.com/zenn-user-upload/5a5e94d1b04a-20221226.png)

# ストーリーごとにデフォルトの背景色を変更する

Viewport と同様のため、説明割愛。

```ts
export const Default: Story = {
  args: {
    label: "ボタン",
    variant: "primary",
    size: "medium",
  },
  // ストーリーごとに parameters を設定
  parameters: {
    backgrounds: {
      default: "twitter",
    },
  },
};
```

# 関連リンク

https://storybook.js.org/docs/7.0/vue/essentials/backgrounds