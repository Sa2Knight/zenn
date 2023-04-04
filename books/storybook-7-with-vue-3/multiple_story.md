---
title: 複数のストーリーを定義する
free: true
---

前章にて、`CSF` では `export const` を使用して、一つのコンポーネントに対して複数のストーリーを定義できることがわかりました。

実際に `MyPage` コンポーネントのストーリーを複数作成してみましょう。

# ストーリーの追加

`MyPage` コンポーネントは、`label` に文字列を指定することで、ボタンのテキストを変更できます。現状の "ボタン" 以外にも、他のテキストを使ったストーリーを用意してみましょう。

```ts:src/stories/MyButton.stories.ts
import MyButton from "../components/MyButton.vue";
import type { Meta, StoryObj } from "@storybook/vue3";

type Story = StoryObj<typeof MyButton>;

const meta: Meta<typeof MyButton> = {
  title: "MyButton",
  component: MyButton,
};

// 「ボタン」
export const Default: Story = {
  render: (args) => ({
    components: { MyButton },
    setup() {
      return { args };
    },
    template: "<MyButton v-bind='args' />",
  }),
  args: {
    label: "ボタン",
  },
};

// 「ログイン」
export const Login: Story = {
  render: (args) => ({
    components: { MyButton },
    setup() {
      return { args };
    },
    template: "<MyButton v-bind='args' />",
  }),
  args: {
    label: "ログイン",
  },
};

// 「会員登録」
export const SignUp: Story = {
  render: (args) => ({
    components: { MyButton },
    setup() {
      return { args };
    },
    template: "<MyButton v-bind='args' />",
  }),
  args: {
    label: "会員登録",
  },
};

export default meta;
```

`Storybook` を確認すると、`Login` `SignUp` の２種類のストーリーが追加されて、サイドバーからストーリーを切り替えられるようになりました。

![](https://storage.googleapis.com/zenn-user-upload/eadc6f0b996e-20221225.gif)

# Args

ここで、ストーリーを複数定義するついでに、 `args` というフィールドを使用しました。

前回の例では、 `props` の注入は `<MyButton label="ボタン" />` のように、 `template` にハードコードしていました。

ハードコードでも動くには動きますが、通常は `args` という、 **`Vue` における `props` を `Storybook` から動的に注入できる仕組み** を利用します。

`args` は `render` 関数に引き渡されますが、これをテンプレートにバインドする必要があるため、 `Composition API` の仕組みに則り、`setup` 関数を使用してバインドしています。

# ストーリーの共通部分を抜き出す

ここまでで作成した各ストーリーは、「`MyButton` コンポーネントを描画し、 `args` をそのまま `props` に引き渡す」ことまで共通しており、具体的な `args` の値のみが異なっています。

各ストーリーで共通の設定については、`export default` で定義しているメタ情報に一元化できます。 `render` 関数をそこに移動することで、以下のように整理できます。

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
};

export const Default: Story = {
  args: {
    label: "ボタン",
  },
};

export const Login: Story = {
  args: {
    label: "ログイン",
  },
};

export const SignUp: Story = {
  args: {
    label: "会員登録",
  },
};

export default meta;
```

これは頻出パターンで、さらに `args` についても共通設定を抜き出すことが多いです。本書でも以降はこの形式を使用することが多いので覚えておいてください。

# 問題点

`Login` `SignUp` というストーリーを追加し、`label` が異なる場合の挙動を確認できました。

しかし、 `label` を空にした場合や、長文にした場合はどうなるでしょうか。また、`MyButton` コンポーネントには他にも `variant` `size` といった `props` が存在します。

現状だとそれらを確認するためにはソースコードを都度書き換えるしかなく、メンバー同士での共有、カタログ化には不向きです。

これを解決する、非常に強力な仕組みがアドオンで提供されています。まずは `Storybook` におけるアドオンについて確認しましょう。