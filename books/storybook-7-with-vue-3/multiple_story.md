---
title: 複数のストーリーを定義する
---

前章にて、`CSF` では `export const` を使用して、一つのコンポーネントに対して複数のストーリーを定義できることがわかりました。

実際に `Counter` コンポーネントのストーリーを複数作成してみましょう。

# With Initial Count ストーリー

`Counter` コンポーネントは、`initialCount` という `props` でカウンターの初期値が決定します。これを指定した場合と、省略した場合での挙動を確認できるようにしましょう。

```ts:src/stories/counter.stories.ts
// ...前略

// 初期値を指定しないパターンのストーリー
export const Default: Story = {
  render: () => ({
    components: { Counter },
    template: "<Counter />",
  }),
};

// 初期値を指定したパターンのストーリー
export const WithInitialCount: Story = {
  render: (args) => ({
    components: { Counter },
    setup() {
      return { args };
    },
    template: '<Counter v-bind="args" />',
  }),
  args: {
    initialCount: 5,
  },
};

```

`Storybook` を確認すると、以下のように `With Initial Count` というストーリーが追加され、カウンターの初期値が設定された状態でレンダリングされます。

![](https://storage.googleapis.com/zenn-user-upload/428ab15c74ff-20221225.png)

# Args

追加したストーリーでは、新たに `args` というフィールドを指定しました。これは `Vue` における `props` であると解釈してください。コンポーネントをレンダリングする際は通常、親コンポーネントから `props` が渡されますが、 `args` フィールドがその役割担います。

`args` は `render` 関数に渡されるので、あとは `composition API` の仕組みを使って `setup` 関数経由でテンプレートにバインドしています。

# 問題点

`With Initial Count` というストーリーを追加することで、`initialCount` が `5` である場合の挙動を確認できました。

しかし、 `initialCount` を `-1` にした場合、 `1000` にした場合はどのような挙動になるでしょうか。現状だとソースコードを都度書き換えるしかなく、メンバー同士での共有、カタログ化には不向きです。

これを解決する、非常に強力な仕組みがアドオンで提供されています。まずは `Storybook` におけるアドオンについて確認しましょう。