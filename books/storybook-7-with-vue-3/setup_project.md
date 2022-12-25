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

# `vscode` / `Volar` を使用している場合

開発環境として [`vscode`](https://code.visualstudio.com/) 及び [`Volar`](https://github.com/johnsoncodehk/volar) を使用している場合は、 `tsconfig.json` を以下のように修正してください。

```json:tsconfig.json
{
  "compilerOptions": {
    "target": "ESNext",
    "useDefineForClassFields": true,
    "module": "ESNext",
    "moduleResolution": "Node",
    "strict": true,
    "jsx": "preserve",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "esModuleInterop": true,
    "lib": ["ESNext", "DOM"],
    "skipLibCheck": true,
    "noEmit": true,
    // ここを追加する
    "types": [
      "vite/client"
    ]
  },
  "include": ["src/**/*.ts", "src/**/*.d.ts", "src/**/*.tsx", "src/**/*.vue"],
  "references": [{ "path": "./tsconfig.node.json" }],
}
```

本書の本筋とは関係ないので詳細は割愛しますが、 `Storybook` を使用する場合はテンプレートの正しい型チェックのため必要になります。

https://github.com/johnsoncodehk/volar/discussions/592