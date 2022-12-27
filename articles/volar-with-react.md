---
title: "Volar (Vue 3 + TypeScript) に @types/react が混ざると型エラーになる"
emoji: "🤖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["javascript", "typescript", "vue", "storybook"]
published: false
---

# 概要

本記事は、[Volar](https://github.com/johnsoncodehk/volar) を使って型安全に [Vue 3](https://vuejs.org/) + [TypeScript](https://www.typescriptlang.org/) を書いていたら急に型エラーが発生した問題の調査記録になります。

> Type '{ class: string; }' is not assignable to type 'DetailedHTMLProps<HTMLAttributes<HTMLDivElement>, HTMLDivElement>'.
> Object literal may only specify known properties, and 'class' does not exist in type 'DetailedHTMLProps<HTMLAttributes<HTMLDivElement>, HTMLDivElement>'.

# TL;DR

結論から言うと、以下ディスカッションの内容の通りです。

JSX issues in template
https://github.com/johnsoncodehk/volar/discussions/592

- `Volar` は `Vue` コンポーネントの型チェックに `JSX` を使用している
- 本来は `@vue/runtime-dom` に定義されている型を利用している
- `@types/react` が依存関係に入ると、 `JSX` が `React` 用ので上書きされてしまう
- `React` の `JSX` には `class` がなく `className` で置き換えられているため、 `Vue` と互換性がなく型エラーになる

`tsconfig.json` の `compilerOptions.type` フィールドを明示することで多くの場合は解決できる。

と、これで記事が終わっても良いんですが、意外とハマりやすそうなポイントなのに日本語の情報は無いなと思ったので、整理してまとめることにしました。

# `Volar` について

`Volar` は、`vscode` 拡張を中心とした `Vue.js` 向けのツールセットです。主に `.vue` ファイル向けの言語サーバーの提供や型チェック機能を含んでおり、 `Vue 3` + `TypeScript` で開発する上では実質必須のツールです。

「`Vue` は `TypeScript` との相性が悪い」「`Vue` は型安全性が低い」というイメージも今は昔。`Volar` の力もあり、現代の `Vue` は完全に型安全で使うことができます。

しかし、今回意図せず `<template>` 内の `<div class="card">` というシンプルなテンプレートで型エラーが発生するようになりました。

# 状況再現

`Vite` を用いて `Vue 3` + `TypeScript` のプロジェクトを作成します。

```bash
$ yarn create vite 20221227 --template vue-ts

$ cd 20221227
$ yarn install
```

スキャフォルドされたプロジェクトでは `vscode` の推奨拡張機能が定義されているので従ってインストールしておきます。 (普段からしてるけど)

```json:.vscode/extensions.json
{
  "recommendations": ["Vue.volar", "Vue.vscode-typescript-vue-plugin"]
}
```

`HelloWorld.vue` が用意されているので、確認すると、テンプレートまですべて型安全に扱える状態で、型チェックが通っていることが確認できます。

![](https://storage.googleapis.com/zenn-user-upload/671c0400b56a-20221227.png)

おもむろに `@types/react` を追加します。

```bash
$ yarn add -D @types/react
```

あらふしぎ、先程まで通っていた型チェックが通らなくなりました。

![](https://storage.googleapis.com/zenn-user-upload/214f4fb5f02c-20221227.png)

以下のエラーメッセージが表示されています。

> Type '{ class: string; }' is not assignable to type 'DetailedHTMLProps<HTMLAttributes<HTMLDivElement>, HTMLDivElement>'.
> Object literal may only specify known properties, and 'class' does not exist in type 'DetailedHTMLProps<HTMLAttributes<HTMLDivElement>, HTMLDivElement>'.

:::message
簡略化のために、`Vue` プロジェクトに直接 `@types/react` をインストールしています。
実際は `Storybook` など、`React` をベースに開発されたツールによって間接的にインストールされます。
:::

# 何が起こっているのか

そもそも `Volar` は、 `<template>` の型チェックに `JSX` を使用しています。

これは `Vue` が `Single File Component (.vue)` 形式と、関数形式それぞれでコンポーネントを定義できるため、どちらにも共通の仕組みで型チェックできるようにするためです。

通常は `@vue/runtime-dom` で定義されている `JSX.IntrinsicElements` を使用します。

https://github.com/vuejs/core/blob/c6e5bda27d13554675d68dbe33b07f3474467aa6/packages/runtime-dom/types/jsx.d.ts#L1326-L1342

`JSX.IntrinsicElements` は `NativeElements` を継承しており、`div` など通常の HTML 仕様に近いインタフェースを提供します。

https://github.com/vuejs/core/blob/c6e5bda27d13554675d68dbe33b07f3474467aa6/packages/runtime-dom/types/jsx.d.ts#L1320-L1324

`div` 要素は `HTMLAttributes` インタフェースを満たしているため、`<div class='card'>` は正しい型であると判断されます。

https://github.com/vuejs/core/blob/c6e5bda27d13554675d68dbe33b07f3474467aa6/packages/runtime-dom/types/jsx.d.ts#L1038
https://github.com/vuejs/core/blob/c6e5bda27d13554675d68dbe33b07f3474467aa6/packages/runtime-dom/types/jsx.d.ts#L240-L244

ここに `@react/types` がインストールされるとどうでしょうか。 `@react/types` も同様にトップレベルで `JSX.IntrinsicElements` が定義されています。

https://github.com/DefinitelyTyped/DefinitelyTyped/blob/8af1819bc428719c3fa562c9d43ad61c4c814416/types/react/index.d.ts#L3144-L3325

`React` を一度でも書いたことがあればご存じでしょうが、`React` における `JSX` には `class` がなく、 `className` になっています。

https://github.com/DefinitelyTyped/DefinitelyTyped/blob/8af1819bc428719c3fa562c9d43ad61c4c814416/types/react/index.d.ts#L1848-L1858

今回の場合は `React` のほうの `JSX.IntrinsicElements` が優先されてしまったのが原因のようです。

# 解決策

以下ディスカッションにて、いくつかの解決策があげられているので、それぞれ思考停止で取り入れる前に深堀りしてみます。

https://github.com/johnsoncodehk/volar/discussions/592

## `@types/react` を自動で読み込まないようにする

`tsconfig.json` で以下のように、 `compilerOptions.types` フィールドを明示的に設定します。

```json:tsconfig.json
{
  "compilerOptions": {
    // ...
    "types": [
      "vite/client", // if using vite
      // ...
    ]
  }
}
```

`tsconfig` における `compilerOptions.types` フィールドは、どの型パッケージをグローバルスコープに読み込むかを決定します。

https://www.typescriptlang.org/tsconfig#types

デフォルトでは `node_modules` 以下にあるすべての `@types` パッケージを対象とするため、 `@types/react` も読み込まれてしまいます。

それを上記のように、明示的に指定することで、それだけを読み込むように変更でき、`@types/react` が勝手に読み込まれなくなります。

この方法は、 `@types/react` を読み込んでいるコードがプロジェクトに存在しない場合のみ有効です。 (依存パッケージの中で、使われてないけど依存している場合など)

## `@types/react` を読み込んでいるモジュールを型チェックしない

特に `Storybook` で発生することが多いようです。

最近だと `Storybook 7.0.0-beta.14` では、アドオン経由で `@types/react` が入ってきてしまいます。

```bash
$ yarn why @types/react

=> Found "@types/react@18.0.26"
info Has been hoisted to "@types/react"
info Reasons this module exists
   - Specified in "devDependencies"
   - Hoisted from "@storybook#addon-essentials#@storybook#addon-docs#@mdx-js#react#@types#react"
```

この場合、前述の `tsconfig.json` の対応をしていても、以下のようなコードが出現したら終わりです。読み込んだモジュール経由で `@types/react` が読み込まれてエラーが再発します。

```ts
import { ArgsTable } from "@storybook/addon-docs";
```

ディスカッションでは、 `tsconfig.json` の　`exclude` オプションを使用することが提案されています。

今回の場合は `Storybook` 用のコードなので、 `*.stories.ts` を型チェックの対象外にします。

```json:tsconfig.json
{
  // ...
  "exclude": ["**/*.stories.ts"]
}
```

これで型チェック時に `@storybook/addon-docs` から対象外になるため、`@types/react` が読み込まれなくなり、エラーが解消します。

とはいえ、これをしてしまうと `Storybook` のコードに型チェックがかけられなくなるので、本末転倒でもあります。

## `@types/react` を改変する

`@types/react` をどうしても読み込んでしまうなら、それを空パッケージで差し替えたろうという力技です。

まず、`package.json` に以下を追加します。

```json:package.json
  "resolutions": {
    "@types/react": "file:stub/types__react"
  },
```

`package.json` の `resolutions` では、依存ツリー内に含まれる任意のパッケージの解決方法を強引に指定できます。

そしてパッケージの解決は `npm` に公開されているものでなく、ファイルシステムから直接指定することも出来ます。

よって、 `stub/types__react` に、何も定義しない空パッケージを作成することで、`JSX` ネームスペースが上書きされないようになります。

空パッケージの内容は以下のとおりです。

```json:stub/types_react/package.json
{
  "name": "@types/react",
  "version": "0.0.0"
}
```

```js:stub/types_react/index.d.ts
export {}
```

**ちなみに自社でハマったときはこの方法で対応してしまいました。**

# まとめ

本記事は、[Volar](https://github.com/johnsoncodehk/volar) を使ってるのにテンプレートで型エラーが発生する問題の深堀りをしました。

意外な落とし穴で、そこまでハマる機会はありませんが、あえて深堀りすることで OSS コードリーディングや TypeScript の理解を深める良い機会になりました。