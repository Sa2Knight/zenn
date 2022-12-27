---
title: ビューポートを変更する
---

# Viewport Addon について

`Viewport Addon` は、 `Essentials Addons` の一つで、**ストーリーを描画するビューポートを切り替える**仕組みを提供します。

モバイルファーストな時代である現代において、コンポーネントが画面サイズによって形を変えるレスポンシブな構成になることはよくあります。

そんな中で、一つのコンポーネントをある端末見た時は正常に描画されているが、別な画面サイズの端末で見ると崩れてしまっているということも多いでしょう。

`Viewport Addon` を活用することによって、様々な画面サイズでコンポーネントを描画したり、画面サイズ別のストーリーを定義できます。

## MyPage コンポーネントのストーリーを作成

本章ではまず、ビューポートに応じてコンテンツの段組みが変わる `MyPage` コンポーネントのストーリーを作成します。

`MyPage` コンポーネントは `props` / `emits` を特に持たないため、以下のようなシンプルなストーリーから初めましょう。

```ts:src/stories/MyPage.stories.ts
import MyPage from "../components/MyPage.vue";
import type { Meta, StoryObj } from "@storybook/vue3";

type Story = StoryObj<typeof MyPage>;

const meta: Meta<typeof MyPage> = {
  title: "MyPage",
  component: MyPage,
  render: () => ({
    components: { MyPage },
    template: "<MyPage />",
  }),
};

export default meta;

export const Default: Story = {};
```

以下のように、 `MyHeader` とコンテンツを含んだ `MyPage` コンポーネントが確認できます。

![](https://storage.googleapis.com/zenn-user-upload/fe58af6bc015-20221226.png)

# アドオンのインストール

これまで同様、アドオンをインストールし、`.storybook/main.ts` に追加します。

```bash
$ yarn add -D @storybook/addon-viewport@7.0.0-beta.14
```

```ts:.storybook/main.ts
import { StorybookConfig } from "@storybook/vue3-vite";

const config: StorybookConfig = {
  framework: "@storybook/vue3-vite",
  stories: ["../src/**/*.stories.@(js|ts)"],
  addons: [
    "@storybook/addon-controls",
    "@storybook/addon-actions",
    "@storybook/addon-viewport", // 追加
  ],
};

export default config;
```

`Storybook` を開くと、画面上部ツールバーに Viewport を切り替えメニューが追加されています。

Viewport を切り替えると、それに応じたサイズでストーリーが再描画されます。

![](https://storage.googleapis.com/zenn-user-upload/bb5ca33ca0e8-20221226.gif)

# Viewport のプリセットを変更する

デフォルトでは、Viewport は以下から選択できます。

- Small mobile (320 x 568)
- Large mobile (414 x 896)
- Tablet (834 x 1112)

多くの場合はこの３種類があれば概ね充分ですが、各種端末別の画面サイズで検証したいケースも考えられます。

そういった場合は、 `INITIAL_VIEWPORTS` というもう一つ用意されたプリセットを使用できます。

`INITIAL_VIEWPORTS` は、 `@storybook/addon-viewport` パッケージから公開されているので、 `import` して使用します。

```ts:src/stories/MyPage.ts
import { INITIAL_VIEWPORTS } from "@storybook/addon-viewport";
```

使用するプリセットの指定は `parameters` というフィールドを使用します。 `parameters` は本書で初めて登場しますが、後半の章で詳しく扱うため、ここではアドオンに対する設定を注入するフィールドだと考えてください。

```ts:src/stories/MyPage.ts
// 前略

const meta: Meta<typeof MyPage> = {
  title: "MyPage",
  component: MyPage,
  render: () => ({
    components: { MyPage },
    template: "<MyPage />",
  }),
  // parameters で Viewport のプリセットを設定
  parameters: {
    viewport: {
      viewports: INITIAL_VIEWPORTS,
    },
  },
};

// 以下略
```

これで `iPhone12` や `Pixel` など、実際の主要端末に基づいたビューポートが選択できるようになりました。

![](https://storage.googleapis.com/zenn-user-upload/9f9a754941a2-20221226.png)

# カスタム Viewport を使用する

開発するアプリケーションによっては、レスポンシブデザインのブレークポイントが決まっている場合もあるでしょう。

例えば筆者が開発したアプリケーションでは、レイアウトは以下のルールで定義していました。

- 幅 992px 以上を PC レイアウトとする
- 幅 991px 以下を Mobile レイアウトとする
- 最低幅 375px でもレイアウトが崩れないようにする

この場合、レイアウト崩れが発生していないかを確認するために、以下のパターンのみ確認できれば十分そうです。

- 992px: PC レイアウトの最低幅
- 375px: Mobile レイアウトの最低幅

:::message
ビューポートが広すぎることでレイアウトが崩れるということも無くはないですが、今回はレイアウトが広がる分にはコンテンツが広がるだけで無問題とします。
:::

`Storybook` を用いてこれを確認しやすくするために、カスタム ViewPort を作成します。

```ts:src/stories/MyPage.ts
// 前略

const meta: Meta<typeof MyPage> = {
  title: "MyPage",
  component: MyPage,
  render: () => ({
    components: { MyPage },
    template: "<MyPage />",
  }),
  // PC, Mobile それぞれのビューポートを作成 (高さは指定しない)
  parameters: {
    viewport: {
      viewports: {
        pc: {
          name: "Min PC Layout",
          styles: {
            width: "992px",
            height: "100%",
          },
        },
        mobile: {
          name: "Min Mobile Layout",
          styles: {
            width: "375px",
            height: "100%",
          },
        },
      },
    },
  },
};

// 以下略
```

Viewport をカスタマイズできました。

![](https://storage.googleapis.com/zenn-user-upload/6d4e5b7b67a6-20221226.png)

# ストーリーごとに Viewport を差し替える

ツールバーから Viewport を切り替えるのも良いですが、一々ツールバーを操作するのも億劫です。

使用するビューポートごとにストーリーを分けることで、各ビューポートでもレイアウトを確認できます。

以下では、`MyPage` コンポーネントのストーリーを `ForPC` `ForMobile` に分割し、それぞれに `parameters` フィールドを追加してデフォルトの Viewport を設定しています。

```ts:src/stories/MyPage.stories.ts
// 前略
// ここで指定している defaultViewport は、前回定義したカスタム Viewport です

export const ForPc: Story = {
  parameters: {
    viewport: {
      defaultViewport: "pc",
    },
  },
};

export const ForMobile: Story = {
  parameters: {
    viewport: {
      defaultViewport: "mobile",
    },
  },
};
```

ストーリーを切り替えるとビューポートも自動で切り替わるようになりました。

![](https://storage.googleapis.com/zenn-user-upload/db22f0342537-20221226.gif)

# 関連リンク

https://storybook.js.org/docs/7.0/vue/essentials/viewport