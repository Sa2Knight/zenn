---
title: インタラクションを作成する
---

本章では `Play function` 機能を用いて、**ストーリーに対するユーザーインタラクションを自動化する**方法を紹介します。

ここでいうインタラクションとは以下のようなユーザー操作を指します。

- DOM 取得
- 要素のクリック
- フォームへのキー入力

それらを組み合わせて自動化することで、以下を用意に実現できます。

- コンポーネントに対して特定の操作した際の状態を再現できる
- コンポーネントに対して特定の操作する自動テストを書ける([`Jest`](https://jestjs.io/ja/) など)

これまではコンポーネントのユニットテストに別の仕組みが必要だったり、画面上での確認ができないといった問題がありました。`Play function` を使用してそれらを `Storybook` と統合することで、よりフロントエンド開発を `Storybook` に集中させることが出来ます。

# 下準備

これまでの章で `.storybook/preview.js` をイジりましたが、不要な設定が多いので一度リセットしましょう。

```ts:.storybook/preview.ts
import { Decorator, Parameters } from "@storybook/vue3";
export const parameters: Parameters = {};
export const decorators: Decorator[] = [];
```

## フォームコンポーネントの作成

ここまで使用してきた `MyButton` `MyHeader` `MyPage` コンポーネントだけだと、ユーザーインタラクションに限りがあるため、本章に特化したコンポーネントを新たに作成します。

`SignUpForm` は簡単な会員登録フォームのイメージで、仕様は以下のとおりです。

- ユーザー名、年齢を入力できる
- 確定ボタン押下で、バリデーション実行後に `submit` イベントを発火する
- バリデーションエラー発生時に、画面にエラーメッセージを表示する

![](https://storage.googleapis.com/zenn-user-upload/f9a5156f9e24-20221228.png)

コードは以下になります。

```ts:src/components/SignUpForm
<script setup lang="ts">
import { reactive } from "vue";

const state = reactive({
  name: "",
  age: 18,
});
const errors = reactive({
  name: "",
  age: "",
});

const emits = defineEmits<{
  (e: "submit", payload: { name: string; age: number }): void;
}>();

function onSubmit() {
  errors.name = state.name === "" ? "名前を入力してください" : "";
  errors.age = state.age < 18 ? "18歳以上でなければ登録できません" : "";

  if (errors.name === "" && errors.age === "") {
    emits("submit", { name: state.name, age: state.age });
  }
}

function onReset() {
  state.name = "";
  state.age = 18;
}
</script>

<template>
  <div class="sign-up-form">
    <h1>会員登録</h1>
    <form class="form" @submit.prevent="onSubmit">
      <div class="form-group">
        <label for="name">Name</label>
        <input id="name" v-model="state.name" placeholder="Enter your email" />
        <span class="error">{{ errors.name }}</span>
      </div>
      <div class="form-group">
        <label for="age">Age</label>
        <input type="number" id="age" v-model="state.age" />
        <span class="error">{{ errors.age }}</span>
      </div>
      <div class="form-group">
        <button type="reset" @click="onReset">リセット</button>
        <button type="submit">確定</button>
      </div>
    </form>
  </div>
</template>

<style scoped>
.sign-up-form {
  width: 100%;
  margin: 0 auto;
  padding: 20px;
  border: 1px solid gray;
  border-radius: 5px;
  background-color: white;
}
.sign-up-form .form {
  width: 100%;
  display: flex;
  flex-direction: column;
  gap: 1rem;
}
.sign-up-form .form .form-group {
  display: flex;
  flex-direction: row;
  gap: 0.5rem;
}
.sign-up-form .form .form-group .error {
  color: red;
}
</style>
```

## ストーリーファイルを作成

`SignUpForm` コンポーネントのストーリーも最小限作っておきます。

`submit` イベントの発火を確認できるように、 `Actions addon` 用の `argTypes` も定義します。

```ts:src/stories/SignUpForm.stories.ts
import SignUpForm from "../components/SignUpForm.vue";
import type { Meta, StoryObj } from "@storybook/vue3";

type Story = StoryObj<typeof SignUpForm>;

const meta: Meta<typeof SignUpForm> = {
  title: "SignUpForm",
  component: SignUpForm,
  render: (args) => ({
    components: { SignUpForm },
    setup() {
      return { args };
    },
    template: "<SignUpForm v-bind='args' />",
  }),
  argTypes: {
    onSubmit: { action: "onSubmit" },
  },
};

export default meta;

export const Default: Story = {};
```

# アドオン及びパッケージインストール

下準備も整ったので、アドオン及びパッケージをインストールします。

```bash
$ yarn add -D @storybook/addon-interactions@7.0.0-beta.14 @storybook/testing-library
```

`@storybook/addon-interactions` は `Storybook` のアドオンであるため、例によって `.storybook/main.ts` に追記します。

```ts:.storybook/main.ts
import { StorybookConfig } from "@storybook/vue3-vite";

const config: StorybookConfig = {
  framework: "@storybook/vue3-vite",
  stories: ["../src/**/*.stories.@(js|ts)", "../src/**/*.mdx"],
  addons: [
    "@storybook/addon-controls",
    "@storybook/addon-actions",
    "@storybook/addon-viewport",
    "@storybook/addon-backgrounds",
    "@storybook/addon-measure",
    "@storybook/addon-outline",
    "@storybook/addon-docs",
    "@storybook/addon-interactions", // 追加
  ],
};

export default config;
```

[`@storybook/testing-library`](https://github.com/storybookjs/testing-library) は、 [`testing-library`](https://testing-library.com/) を `Storybook` 内で使用するためのパッケージです。 `testing-library` は `DOM` に対するテストコードのためのユーティリティぐらいの認識でも大丈夫です。`Play function` 内で、コンポーネント内の　`DOM` を参照、操作するのを簡便化するために使用します。

# Play function の作成

`Play function` はストーリーごとに、コンポーネントに対するユーザーインタラクションを定義することで、ストーリーがマウントされたタイミングで自動的に実行されます。

`SignUpForm` コンポーネントを操作する `Play function` を定義した、２種類のストーリーを定義してみましょう。

- フォームに適切な入力し、 `submit` イベントを発火させる
- フォームに不適切な入力し、エラーメッセージを表示させる

それぞれ `Complete` `Error` というストーリー名で以下のように定義します。

```ts:.storybook/main.ts
import { userEvent, within } from "@storybook/testing-library";

// 中略

export const Default: Story = {};

export const Complete: Story = {
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);

    const nameInput = canvas.getByLabelText("Name", { selector: "input" });
    const ageInput = canvas.getByLabelText("Age", { selector: "input" });
    const submitButton = canvas.getByRole("button", { name: "確定" });

    await userEvent.type(nameInput, "sasaki");
    await userEvent.clear(ageInput);
    await userEvent.type(ageInput, "30");
    await userEvent.click(submitButton);
  },
};

export const Error: Story = {
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);

    const nameInput = canvas.getByLabelText("Name", { selector: "input" });
    const ageInput = canvas.getByLabelText("Age", { selector: "input" });
    const submitButton = canvas.getByRole("button", { name: "確定" });

    await userEvent.type(nameInput, "");
    await userEvent.clear(ageInput);
    await userEvent.type(ageInput, "17");
    await userEvent.click(submitButton);
  },
};
```

`Storybook` を開くと、 `Complete` `Error` のストーリーが追加され、それらを開いた瞬間にインタラクションが完了しています。

![](https://storage.googleapis.com/zenn-user-upload/1bdbd0f6d63f-20221228.gif)

`Interactions` タブには実際に行われたインタラクションのログが表示され、インタラクションに失敗した場合はエラーログが表示されます。

このように `Play function` を使用することで、**実際に操作しないと再現できないコンポーネントの状態**を `Storybook` 上で用意に作成できます。

この仕組みは自動テストとの相性が抜群です。次の章では、作成したストーリーを `Storybook` で確認するだけでなく、自動テストと統合して振る舞いが変わらないことを担保しましょう。