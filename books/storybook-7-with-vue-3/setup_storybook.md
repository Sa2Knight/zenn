---
title: Storybook のセットアップ
free: true
---

いよいよ、`Storybook` のセットアップに入ります。

当然 `Storybook` のインストールを行うわけですが、`Storybook` の実態はモノレポ形式になっており、プロジェクトの構成に応じて必要なパッケージを選定して導入する必要があります。

`npx sb init` のようなディレクトリ構成に基づいた自動セットアップコマンドも用意されていますが、今回は以下の理由によりこれを使用せずに手動でセットアップしていきます。

- 執筆時点で v7 が β バージョンであるため、正しく動作しない
- 推奨設定がすべて適用されるため、意図せず不要なパッケージまでインストールされてしまう
- パッケージが自動で追加、初期設定されるため、それらを理解せずに雰囲気で使ってしまう

# CLI

まず、`Storybook` の起動やビルドをコマンドラインで行うための CLI をインストールします。

```bash
$ yarn add -D storybook@7.0.0-beta.20
```

これで `yarn storybook <command>` の形式で `Storybook` を動かす準備ができました。

# 本体

本書では `Vue 3` 及び `Vite` を使用するので、それにあわせて `@storybook/vue3-vite` をインストールします。

```bash
$ yarn add -D @storybook/vue3-vite@7.0.0-beta.20
```

`@storybook/vue3-vite` は以下のパッケージを内包したプリセットになっています。

|パッケージ名|用途|
|----|----|
|`@storybook/core-server`|`Storybook` 本体のサーバーサイド(`Node.js`)|
|`@storybook/vue3`|`Vue` 3 のストーリーをレンダリングする|
|`@storybook/builder-vite`|`Storybook` を `Vite` でビルドする|
|`vue-docgen-api`|`Vue` コンポーネントからインタフェース情報などを抽出する|

あえてカスタマイズした使い方をしたい場合は、上記のパッケージを個別にインストール、設定もできますが、基本的には `@storybook/vue3-vite` のようなプリセットを使用すれば良いでしょう。

# `React`

本書では `Vue` を使うため違和感がありますが、`Storybook` には `React` が使用されているため、 別途 `React` のインストールが必要になります。

```bash
$ yarn add -D react react-dom
```

これで `Storybook` を起動するために必要な最低限のパッケージが揃いました。