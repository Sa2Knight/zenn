---
title: "プロジェクト作成"
free: true
---

本書では `Vue 3` + `TypeScript` のプロジェクトを、 `Vite` を用いて動かします。

`Vite` プロジェクトを手っ取り早く作成するため、 [`create-vue`](https://www.npmjs.com/package/create-vite) を使用します。

```bash
$ yarn create vite storybook-7-vue-3-sample --template vue-ts
```

これでプロジェクトの雛形が作成されました。

ディレクトリを移動し、パッケージをインストールして開発環境を動かしてみましょう。

```bash
$ cd storybook-7-vue-3-sample/
$ yarn install
$ yarn dev

  VITE v4.2.1  ready in 437 ms

  ➜  Local:   http://127.0.0.1:5173/
  ➜  Network: use --host to expose
  ➜  press h to show help
```

http://127.0.0.1:5173/ にアクセスすると、以下のような雛形アプリケーションが起動します。

![](https://storage.googleapis.com/zenn-user-upload/6ffeab321c8e-20221224.png)
