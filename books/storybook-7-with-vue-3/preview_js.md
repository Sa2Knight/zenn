---
title: 全ストーリー共通設定を定義する
---

# preview.js について

これまで、 `Viewport addon` や `Background addon` などのアドオンに対する設定を、各ストーリーごとに `export default` 内で定義していました。

しかし、多くの設定はストーリーごとでなく、全ストーリー共通にしたいものです。
(あるいは共通のデフォルト設定を定義し、必要に応じてストーリーごとに書き換えるなど)

そのようなストーリーを跨いだ共通定義は、 `.storybook/preview.js` に記述します。

:::message
`Storybook` には `Manager` ブロックと `Preview` ブロックがあり、 `preview.js` は後者に対する設定を記述するファイルです。

`Preview` ブロックは `iframe` でストーリーを埋め込む役割を持ちますが、本書ではアーキテクチャに深く言及しません。
:::

:::message
説明用に `preview.js` と呼称していますが、本書では `TypeScript` を使用しているため、実際のファイル名は `preview.ts` になります。
:::

# Storybook で使用できる設定値

`preview.js` で `export` することで設定できるオブジェクトは以下の３種類です。

```ts:.storybook/preview.ts
export const parameters = {

}

export const decorators = {

}

export const globalTypes = {

}
```

`parameters` のみ、本書で既に登場していますが、改めて紹介します。

|設定名|用途|
|---|---|
|parameters|ストーリーごとに割り当てられる key-value 形式のメタデータ|
|globalTypes|Storybook 全体で一意の設定となる key-value 形式のメタデータ|
|decorators|ストーリーをラップするラッパーコンポーネント|

`parameters` は `globalTypes` と異なり、コンポーネントごと、あるいはストーリーごとに上書き可能かです。

`decorators` `globalTypes` の使い方については以降の章で改めて行いますので、ここでは `preview.js` にてまとめて設定可能であることを理解してもらえれば大丈夫です。

# Viewport addon の設定をまとめて行う

例として `Viewport addon` の設定をしてみましょう。

本書では `MyPage.stories.ts` でのみ、以下のように Viewport の設定をメタデータ及び各ストーリーに定義していました。(再掲)

```ts:src/stories/MyPage.stories.ts
import MyPage from "../components/MyPage.vue";
import type { Meta, StoryObj } from "@storybook/vue3";

import { INITIAL_VIEWPORTS } from "@storybook/addon-viewport";

type Story = StoryObj<typeof MyPage>;

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

export default meta;

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

ビューポートの影響を受けるコンポーネントがこれだけなら良いのですが、実際はコンポーネントごとにそれぞれレスポンシブであることがあるため、まとめて定義したほうが便利です。

上記のビューポートに関する設定を `preview.js` に移動します。

```ts:.storybook/preview.ts
/**
 * Storybook 全体での parameters のデフォルト値を定義する
 */
export const parameters = {
  // Viewport addon で使用される設定
  viewport: {
    // プリセットをアプリケーションの仕様に合わせて定義
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
    // すべてのストーリーでデフォルト PC ビューを使用する
    defaultViewport: "pc",
  },
};
```

`MyPage.stories.ts` 側は、デフォルト設定を上書きする場合のみ追記すれば良くなります。

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

export const ForPc: Story = {};

export const ForMobile: Story = {
  // このストーリーのみデフォルトビューポートを変更する
  parameters: {
    viewport: {
      defaultViewport: "mobile",
    },
  },
};
```

他のコンポーネントでも、デフォルトのビューポート (`Min PC Layout`) が使用されるようになりました。

![](https://storage.googleapis.com/zenn-user-upload/8facafc9e602-20221227.png)