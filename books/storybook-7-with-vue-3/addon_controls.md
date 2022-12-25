---
title: props を動的に切り替える
---

# Controls Addon について

`Controls Addon` は、 `Essentials addons` の一つで、**コンポーネントに対するパラメータ(≒props)をGUI上で動的に切り替える**仕組みを提供します。

まずはアドオンをインストールしましょう。

```bash
$ yarn add -D @storybook/addon-controls@7.0.0-beta.14
```

インストールしたアドオンを利用するためには、 `Storybook` の設定ファイルに、 `addons` フィールドを追加します。

```ts:.storybook/main.ts
import { StorybookConfig } from "@storybook/vue3-vite";

const config: StorybookConfig = {
  framework: "@storybook/vue3-vite",
  stories: ["../src/**/*.stories.@(js|ts|mdx)"],
  addons: ["@storybook/addon-controls"], // 使用するアドオンを追加
};

export default config;
```

設定ファイル編集後は、 `Storybook` を再起動する必要があります。

```bash
$ yarn storybook dev
```

`Storybook` で　`With Initial Count` を開くと、画面下部のパネルに `Controls` タブが表示されるようになっています。

![](https://storage.googleapis.com/zenn-user-upload/2323a33dd6dc-20221225.gif)

`Controls` タブでは、 ストーリーの `args` フィールドで設定した `props` を画面から切り替えて変更を反映できます。