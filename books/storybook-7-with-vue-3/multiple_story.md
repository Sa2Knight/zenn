---
title: 複数のストーリーを定義する
---

前章にて、`CSF` では `export const` を使用して、一つのコンポーネントに対して複数のストーリーを定義できることがわかりました。

実際に `MyPage` コンポーネントのストーリーを複数作成してみましょう。

# ストーリーの追加

`MyPage` コンポーネントは、`label` に文字列を指定することで、ボタンのテキストを変更できます。現状の "ボタン" 以外にも、他のテキストを使ったストーリーを用意してみましょう。

```ts:src/stories/counter.stories.ts
// ...前略

// 「ボタン」
export const Default: Story = {
  render: () => ({
    components: { MyButton },
    template: "<MyButton label='ボタン' />",
  }),
};

// 「ログイン」
export const Login: Story = {
  render: () => ({
    components: { MyButton },
    template: "<MyButton label='ログイン' />",
  }),
};

// 「会員登録」
export const SignUp: Story = {
  render: () => ({
    components: { MyButton },
    template: "<MyButton label='会員登録' />",
  }),
};
```

`Storybook` を確認すると、以下のように `Login` `SignUp` の２種類のストーリーが追加されて、サイドバーからストーリーを切り替えられることがわかりました。

![](https://storage.googleapis.com/zenn-user-upload/eadc6f0b996e-20221225.gif)

# 問題点

`Login` `SignUp` というストーリーを追加することで、`label` が異なる場合の挙動を確認できました。

しかし、 `label` を空にした場合や、長文にした場合はどうなるでしょうか。また、`MyButton` コンポーネントには他にも `variant` `size` といった `props` が存在します。

現状だとそれらを確認するためにはソースコードを都度書き換えるしかなく、メンバー同士での共有、カタログ化には不向きです。

これを解決する、非常に強力な仕組みがアドオンで提供されています。まずは `Storybook` におけるアドオンについて確認しましょう。