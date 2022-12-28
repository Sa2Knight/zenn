---
title: CSF の書き方
free: true
---

`CSF` は慣れるまでとっつきにくい点もあるので、普段 `Storybook` を使用している方でも雛形をベースにコピー&ペーストして雰囲気で使いまわしていることもあるでしょう。

とはいえ、`CSF` の構造を理解しておくことで `Storybook` 周りのトラブルシューティングがグッと楽になるため、最低限押さえていきます。

# CSF のルール

`CSF` は `ESM` を用いたモジュールで表現しますが、重要なのは以下二点のみです。

- **`export default` で、ストーリーで描画するコンポーネントのメタデータを定義する**
- **`export const` (named export) で、ストーリーを定義する**

`MyButton.stories.ts` を例に順に見ていきます。

## export default

```ts
const meta: Meta<typeof Counter> = {
  title: "MyButton",
  component: MyButton,
};

export default meta;
```

変数 `meta` は、 `MyButton` コンポーネントのストーリーに関するメタデータを定義しています。

`title` はコンポーネント名を、 `component` は対象コンポーネントそのものを指定し、それを `default export` します。

:::message
この他にも多くのフィールドを定義できますが、多くはアドオンとの連携用だったり、`Storybook` のデフォルトの挙動を書き換える用だったりで、使う機会は少なめです。
:::

## export const (named export)

変数 `Default` は、`MyButton` コンポーネントを描画するストーリーです。

```ts
export const Default: Story = {
  render: () => ({
    components: { MyButton },
    template: "<MyButton label='ボタン' />",
  }),
};
```

ストーリーはオブジェクト形式で定義し、 `render` 関数にて実際のレンダリング方法を定義します。

また `export const` は複数宣言できることから、一つのコンポーネントに対して複数のストーリーを定義することも出来ます。詳細は次章に譲りますが、特定の状態を持ったパターン別にストーリーを定義するやり方が主流です。

:::message
ストーリー名は変数名から機械的に決定します。あえて別名を付けたい場合、`name` フィールドを設定できます。
:::

:::message
`export const` したすべてのオブジェクトがデフォルトではストーリーとして扱われます。モックデータなど、ストーリー以外を export したい場合は、その旨を `export default` するメタデータの方で定義することが出来ます。
:::

# 関連リンク

https://storybook.js.org/docs/7.0/vue/api/csf
