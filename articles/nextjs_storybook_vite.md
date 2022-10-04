---
title: "Next.js アプリの Storybook でも Vite を使いたい"
emoji: "📘"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["javascript", "nextjs", "storybook"]
published: true
---

# 概要

現状の [Next.js](https://nextjs.org/) は [webpack](https://webpack.js.org/) を利用しているため、通常なら [Storybook](https://storybook.js.org/) を導入する際にあわせて `webpack` を使用していますが、 `Storybook` ぐらいは [Vite](https://ja.vitejs.dev/) の爆速HMR を使用したいので差し替えてみた差異の作業メモになります。

:::message
本記事では `Storybook` 6.x を使用します。後継の 7.x ではビルダー周りの仕組みが大きく変わるためご注意ください。
:::

# webpack 5 ビルダーのアンインストール

`webpack` を使って Storybook をビルドしてると思うので、これを削除しちゃいます。
(`yarn` を使用しますが、 `npm` の場合は適宜読み替えてください)

```bash
$ yarn remove @storybook/builder-webpack5 @storybook/manager-webpack5
```

# vite と vite ビルダーのインストール

`Next.js` では `Vite` を使用していないので、 `Vite` 本体ごと追加でインストールします。

```bash
yarn add -D vite @storybook/builder-vite
```

ちなみに `@vitejs/plugin-react` については、 `@storybook/builder-vite` が依存しているため、直接インストールする必要はありません。
https://github.com/storybookjs/builder-vite/blob/858b856dcf99a920c831c2c6c428d693d7ea1892/packages/builder-vite/package.json#L33

# .storybook/main.js の修正

`Vite` を `Storybook` でのみ使用する場合は、 `vite.config.js` にあたる設定ファイルは不要です。

代わりに `Storybook` の設定ファイルである `.storybook/main.js` にて、 `core.builder` を `@storybook/builder-vite` にします。

また、必要に応じて `viteFinal` フィールドで `Vite` の設定を拡張できます。

```js:.storybook/main.js
const { mergeConfig } = require('vite')
const path = require('path')

module.exports = {
  stories: ['../stories/**/*.stories.mdx', '../stories/**/*.stories.@(js|jsx|ts|tsx)'],
  framework: '@storybook/react',
  core: {
    builder: '@storybook/builder-vite' // これを追加
  },
  // import に alias を設定するなど、Viteの設定を拡張する場合は viteFinal を定義する
  viteFinal: async config => {
    return mergeConfig(config, {
      resolve: {
        alias: {
          '@': path.resolve(__dirname, '../')
        }
      }
    })
  }
}
```