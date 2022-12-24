---
title: CSF の書き方
---

`CSF` は慣れるまでとっつきにくい点もあるので、普段 `Storybook` を使用している方でも雛形をベースにコピー&ペーストして雰囲気で使いまわしていることもあるでしょう。

とはいえ、`CSF` の構造を理解しておくことで `Storybook` 周りのトラブルシューティングがグッと楽になるため、最低限押さえていきます。

# CSF のルール

`CSF` は `ESM` を用いたモジュールで表現しますが、重要なのは以下二点のみです。

- **`export default` で、ストーリーで描画するコンポーネントのメタデータを定義する**
- **`export const` (named export) で、ストーリーを定義する**

`Counter.stories.ts` を例に順に見ていきます。

## export default

```ts
const meta: Meta<typeof Counter> = {
  title: "Counter",
  component: Counter,
};

export default meta;
```

変数 `meta` は、 `Counter` コンポーネントのストーリーに関するメタデータを定義しています。

`title` はコンポーネント名を、 `component` は対象コンポーネントそのものを指定し、それを `default export` します。

:::message
この他にも多くのフィールドを定義できますが、多くはアドオンとの連携用だったり、`Storybook` のデフォルトの挙動を書き換える用だったりで、使う機会は少なめです。
:::

## export const (named export)

変数 `Default` は、`Counter` コンポーネントを描画するストーリーです。

```ts
export const Default: Story = {
  render: () => ({
    components: { Counter },
    template: '<Counter />',
  }),
};
```

ストーリーはオブジェクト形式で定義し、 `render` 関数にて実際のレンダリング方法を定義します。

また `export const` は複数宣言できることから、一つのコンポーネントに対して複数のストーリーを定義することも出来ます。詳細は次章に譲りますが、特定の状態を持ったパターン別にストーリーを定義するやり方が主流です。

:::message
ストーリー名は変数名から機械的に決定します。あえて別名を付けたい場合、`name` フィールドを設定することができます。
:::

:::message
`export const` したすべてのオブジェクトがデフォルトではストーリーとして扱われます。モックデータなど、ストーリー以外を export したい場合は、その旨を `export default` するメタデータの方で定義することが出来ます。
:::

# 省略記法

今回の `counter.stories.ts` の場合、以下の設定が冗長です。

- コンポーネントのタイトルがファイル名と同一である
- 対象コンポーネントをそのままレンダリングするだけ

この場合、 `CSF 3` では以下のように省略可能で、 `Storybook` をシンプルに運用する場合は、多くのストーリーがこれだけで充分です。

```ts
import Counter from "../components/Counter.vue";
import type { Meta, StoryObj } from "@storybook/vue3";

type Story = StoryObj<typeof Counter>;

const meta: Meta<typeof Counter> = {
  component: Counter, // title はファイル名から決定する
};

export default meta;

export const Default: Story = {}; // 対象コンポーネントを描画する
```