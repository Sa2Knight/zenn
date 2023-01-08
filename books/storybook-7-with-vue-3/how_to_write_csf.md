---
title: CSF の書き方
free: true
---

`CSF` は慣れるまでとっつきにくい点もあるので、普段 `Storybook` を使用している方でも雛形をベースにコピー&ペーストして雰囲気で使いまわしていることもあるでしょう。

とはいえ、`CSF` の構造を理解しておくことで `Storybook` 周りのトラブルシューティングがグッと楽になるため、最低限の仕組みを抑えておくことをオススメします。

# CSF のルール

`CSF`の実態は 、`ESM` を用いて `export` するオブジェクトの集合です。

デフォルトエクスポートする一つのモジュールと、名前付きエクスポートする一つ以上のモジュールで役割が異なります。

- **`export default`**
  - 対象コンポーネントのメタデータ及び各ストーリーの共通設定を定義する
- **`export const` (named export)**
  - コンポーネントのストーリーを定義する

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

また、各ストーリーの共通設定をメタデータにまとめることもでき、本書でも随所で使用します。

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

また `export const` は複数宣言できることから、**一つのコンポーネントに対して複数のストーリーを定義することも出来ます。**

詳細は次章に譲りますが、特定の状態を持ったパターン別にストーリーを定義するやり方が主流です。

:::message
ストーリー名は変数名から機械的に決定します。あえて別名を付けたい場合、`name` フィールドで設定できます。
:::

:::message
`export const` したすべてのオブジェクトがデフォルトではストーリーとして扱われます。モックデータなど、ストーリー以外を export したい場合は、その旨を `export default` するメタデータの方で個別に設定出来ます。
:::

# 関連リンク

https://storybook.js.org/docs/7.0/vue/api/csf
