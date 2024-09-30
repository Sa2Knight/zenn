---
title: props を動的に切り替える
free: true
---

# Controls Addon について

`Controls Addon` は、 `Essentials addons` の一つで、**コンポーネントに対するパラメータ(≒props)をGUI上で動的に切り替える**仕組みを提供します。

まずはアドオンをインストールしましょう。

```bash
$ yarn add -D @storybook/addon-controls@7.0.2
```

インストールしたアドオンを利用するためには、 `Storybook` の設定ファイルに、 `addons` フィールドを追加します。

```ts:.storybook/main.ts
import { StorybookConfig } from "@storybook/vue3-vite";

const config: StorybookConfig = {
  framework: "@storybook/vue3-vite",
  stories: ["../src/**/*.stories.@(js|ts)"],
  addons: ["@storybook/addon-controls"], // 使用するアドオンを追加
};

export default config;
```

設定ファイル編集後は、 `Storybook` を再起動する必要があります。

```bash
$ yarn storybook dev --port 6006
```

`Storybook` を開くと、画面下部のパネルに `Controls` タブが表示されるようになっています。

`Controls` タブでは `args` で設定した `props` を GUI 上で書き換えることができ、その変更がリアクティブに反映されます。

![](https://storage.googleapis.com/zenn-user-upload/689503931bf5-20221226.gif)

# Controls addon の仕組みを知る

先程の例で、`args` で設定された `label` を動的に書き換えられることがわかりました。

よく見ると、 `label` 以外に `variant` や `size` といった、他の `props` についても変更できるようになっています。 `args` で指定していないのに、`Storybook` はどうやって他の `props` を認識しているのでしょうか。

その答えは、 `@storybook/vue3-vite` が使用している、`vue-docgen-api` というパッケージにあります。

https://github.com/vue-styleguidist/vue-styleguidist/tree/dev/packages/vue-docgen-api

このパッケージは `Vue` コンポーネントから、 `props` などのインタフェース情報を抽出する機能を持っています。 `@storybook/vue3-vite` では `Vite` でのビルド時にインタフェース情報をオブジェクトに追加する処理を行うことで、`Storybook` にそれを伝えています。

https://github.com/storybookjs/storybook/blob/ca358694c34c61283da1c685078f104773e0d4e2/code/frameworks/vue3-vite/src/preset.ts#L20

試しに、`MyButton.stories.ts` 内で以下のログ出力をしてみましょう。

```ts
console.log(JSON.stringify(MyButton.__docgenInfo, null, 2));
```

ストーリーを開くと以下のようなログが確認できます。

```json
{
  "exportName": "default",
  "displayName": "MyButton",
  "description": "",
  "tags": {},
  "props": [
    {
      "name": "label",
      "type": {
        "name": "string"
      },
      "required": true
    },
    {
      "name": "variant",
      "type": {
        "name": "string"
      },
      "defaultValue": {
        "func": false,
        "value": "\"primary\""
      }
    },
    {
      "name": "size",
      "type": {
        "name": "string"
      },
      "defaultValue": {
        "func": false,
        "value": "\"medium\""
      }
    }
  ],
  "events": [
    {
      "name": "click"
    }
  ]
  ```

# props の型を定義する

`MyButton` コンポーネントの `props` には、 `label` のほかに、`variant` `size` があります。

`Login` `SignUp` ストーリーを一つにまとめつつ、これらの `props` についても書き換えられるようにしましょう。

なお、各ストーリーで共通の設定は `meta` に一括で定義できるため、そちらに定義しています。

```ts:src/stories/MyButton.stories.ts
import MyButton from "../components/MyButton.vue";
import type { Meta, StoryObj } from "@storybook/vue3";

type Story = StoryObj<typeof MyButton>;

const meta: Meta<typeof MyButton> = {
  title: "MyButton",
  component: MyButton,
  render: (args) => ({
    components: { MyButton },
    setup() {
      return { args };
    },
    template: "<MyButton v-bind='args' />",
  }),
  // ここにまとめて args を定義
  args: {
    label: "ボタン",
    variant: "primary",
    size: "medium",
  },
};

export const Default: Story = {};

export default meta;
```

`variant` `size` についても、 `Controls` タブから値を書き換えることで、スタイルの変更を反映させることができました。

![](https://storage.googleapis.com/zenn-user-upload/00d32263d9c9-20221226.gif)

しかし、 `variant` `size` の型は `String` であるものの、実際はそれぞれ特定の文字列しか受け付けません。 (`variant` の場合は `primary` or `secondary`)

上記 gif でも、入力途中の状態では壊れたスタイルになってしまう上、そもそも何を受け付けているかが不明です。

こういった問題を解決するため、 `Storybook` では入力を制限するための以下のコントロール UI を使用できます。

- 真偽値
- 数値
- JSON
- ラジオボタン
- チェックボックス
- セレクトボックス
- テキストボックス
- マルチラインテキストボックス
- カラーピッカー
- カレンダー

例として、`variant` にはラジオボタンを、 `size` にはセレクトボックスを使用してみます。

`args` ごとにどのコントロール UI を使用するかを設定するために、メタデータオブジェクトに `argTypes` を追加します。

```ts:src/stories/MyButton.ts
// 前略

const meta: Meta<typeof MyButton> = {
  title: "MyButton",
  component: MyButton,
  render: (args) => ({
    components: { MyButton },
    setup() {
      return { args };
    },
    template: "<MyButton v-bind='args' />",
  }),
  args: {
    label: "ボタン",
    variant: "primary",
    size: "medium",
  },
  // ここに args の型情報を定義
  argTypes: {
    variant: {
      control: {
        type: "inline-radio",
      },
      options: ["primary", "secondary"],
    },
    size: {
      control: {
        type: "select",
      },
      options: ["small", "medium", "large"],
    },
  },
};

// 以下略
```

ラジオボタンやセレクトボックスで切り替えができるようになりました。

![](https://storage.googleapis.com/zenn-user-upload/6e73ba32edc2-20221226.gif)

:::message
Vue 3 には、型レベルで `props` を定義する手段があります。ただし `vue-docgen-api` がインタフェースを取得する際にユニオン型の内容まで取り出せていないため、このような二度手間が必要になるようです。(`vue-docgen-api` のアップデートで解決している可能性があります)
:::

ここではラジオボタンをセレクトボックスを使用しましたが、その他のコントロール UI については関連リンクを参照してください。

# 関連リンク

https://storybook.js.org/docs/7.0/vue/essentials/controls
