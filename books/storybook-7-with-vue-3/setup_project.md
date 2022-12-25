---
title: "プロジェクト作成"
---

本書では `vue 3` + `TypeScript` のプロジェクトを、 `vite` を用いて動かします。

`vite` プロジェクトを手っ取り早く作成するため、 [`create-vue`](https://www.npmjs.com/package/create-vite) を使用します。

```bash
$ yarn create vite storybook-7-vue-3-sample --template vue-ts
```

これでプロジェクトの雛形が作成されたため、パッケージをインストールして開発環境を動かしてみましょう。

```bash
$ cd storybook-7-vue-3-sample/
$ yarn install
$ yarn dev

  VITE v4.0.3  ready in 437 ms

  ➜  Local:   http://127.0.0.1:5173/
  ➜  Network: use --host to expose
  ➜  press h to show help
```

http://127.0.0.1:5173/ にアクセスし、以下のようなサンプルアプリケーションが起動していれば完了です。

![](https://storage.googleapis.com/zenn-user-upload/6ffeab321c8e-20221224.png)
