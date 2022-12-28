---
title: emit をロギングする
free: true
---

# Actions Addon について

`Actions Addon` は、 `Essentials addons` の一つで、**コンポーネントが発火したイベント(≒emit)をロギングする**仕組みを提供します。

前章の `Controls Addon` では、 `props` に基づくコンポーネントの振る舞いを確認できましたが、こちらはコンポーネント側からのイベントを確認ができます。

# MyHeader コンポーネントのストーリーを作成

これまでは `MyButton` コンポーネントのストーリーを使用しましたが、ここでは `MyHeader` コンポーネントのストーリーを新たに作成します。

`MyButton` コンポーネントで学んだ仕組みのみ使用するので、おさらいも兼ねて、まとめて作成しちゃいます。

```ts:src/stories/MyHeader.stories.ts
import MyHeader from "../components/MyHeader.vue";
import type { Meta, StoryObj } from "@storybook/vue3";

type Story = StoryObj<typeof MyHeader>;

const meta: Meta<typeof MyHeader> = {
  title: "MyHeader",
  component: MyHeader,
  render: (args) => ({
    components: { MyHeader },
    setup() {
      return { args };
    },
    template: "<MyHeader v-bind='args' />",
  }),
};

export default meta;

export const Login: Story = {
  args: {
    user: {
      id: 1,
    },
  },
};

export const Logout: Story = {
  args: {
    user: {},
  },
};
```

`MyHeader` はユーザー情報を受け取り、その有無で表示するボタンを切り替えます。`Login` `Logout` の２種類を用意することで、これを確認しやすくします。

![](https://storage.googleapis.com/zenn-user-upload/9b706c030ab3-20221226.gif)

# アドオンのインストール

`Actions Addon` の話に戻ります。まずはアドオンをインストールし、`.storybook/main.ts` に追加します。

```bash
$ yarn add -D @storybook/addon-actions@7.0.0-beta.14
```

```ts:.storybook/main.ts
import { StorybookConfig } from "@storybook/vue3-vite";

const config: StorybookConfig = {
  framework: "@storybook/vue3-vite",
  stories: ["../src/**/*.stories.@(js|ts)"],
  addons: ["@storybook/addon-controls", "@storybook/addon-actions"], // 追加
};

export default config;
```

# アクションの定義

`Storybook` における `action` は、コンポーネントに対するイベントハンドラーで、`argTypes` に定義することで、`Action` タブにイベントのログを表示します。

アクション名は `onXXX` という命名にすることで、自動的に `XXX` に対応するイベントのハンドラとして定義されます。

`MyHeader` コンポーネントは、 `login` `logout` `signUp` の３種類のイベントを発火するので、それぞれ `onLogin` `onLogout` `onSignUp` を定義します。

```ts:src/stories/MyHeader.stories.ts
// 前略

const meta: Meta<typeof MyHeader> = {
  title: "MyHeader",
  component: MyHeader,
  render: (args) => ({
    components: { MyHeader },
    setup() {
      return { args };
    },
    template: "<MyHeader v-bind='args' />",
  }),
  // ここに追加
  argTypes: {
    onLogin: { action: "onLogin" },
    onLogout: { action: "onLogout" },
    onSignUp: { action: "onSignUp" },
  },
};

// 以下略
```

`Storybook` を起動すると、画面下部に `Actions` タブが追加され、発生したイベントのログが表示されるようになります。

今回はイベンパラメータがありませんでしたが、通常はイベント名に加えてパラメータを確認もできます。

![](https://storage.googleapis.com/zenn-user-upload/8b744bfb7752-20221226.gif)

# 関連リンク

https://storybook.js.org/docs/7.0/vue/essentials/actions