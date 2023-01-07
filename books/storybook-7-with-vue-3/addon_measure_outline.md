---
title: CSS レイアウトをデバッグしやすくする
free: true
---

ここでは残りの `essentials addons` である、 `Measure addon` と `Outline addon` についてまとめて紹介します。

これらについては百聞は一見に如かずであるため、さっそくインストールしましょう。

```bash
$ yarn add -D @storybook/addon-measure@7.0.0-beta.20 @storybook/addon-outline@7.0.0-beta.20
```

`.storybook/main.ts` に追記します。

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
    "@storybook/addon-measure", // 追加
    "@storybook/addon-outline", // こっちも
  ],
};

export default config;
```

再起動すると、画面上部のツールバーに `Measure addon` と `Outline addon` それぞれの有効化ボタンが表示され、CSS のレイアウトを可視化出来るようになります。

![](https://storage.googleapis.com/zenn-user-upload/930f1a9f3e63-20221227.gif)

# Measure addon

`Measure addon` では、 `MyPage` のような複数コンポーネントが同時に表示されるコンポーネントにおける余白を可視化出来ます。

これによって、その余白がどのコンポーネントから生まれているのか、`marin` `padding` はどの程度なのかのデバッグを行いやすくなります。

# Outline addon

`Outline addon` は、ストーリーに含まれる各アウトラインを可視化することで、UI 間の整列に狂いがないかのデバッグを行いやすくします。

# 関連リンク

https://storybook.js.org/docs/react/essentials/measure-and-outline