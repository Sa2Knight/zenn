---
title: "create-vue で学ぶ、Vue 3 + TypeScript のための tsconfig.json"
emoji: "😎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["javascript", "typescript", "vue"]
published: false
---

# 概要

本記事は、 [Vue.js](https://v3.ja.vuejs.org/) の公式ツールである [create-vue](https://github.com/vuejs/create-vue) によって生成された [tsconfig.json](https://www.typescriptlang.org/ja/tsconfig) を元に、 Vue 3 + TypeScript における、 `tsconfig` のベストプラクティスを理解しようという内容です。

# create-vue について

[create-vue](https://github.com/vuejs/create-vue) は、[Vite](https://ja.vitejs.dev/) ベースの Vue プロジェクトを CLI から簡易的に構築するためのツールです。

```bash
npm init vue@3
```

のようにコマンドを実行し、対話形式の質問に解答することで、ビルド設定から周辺ライブラリの追加、テストやリンター、フォーマッターのセットアップまで行ってくれる優れものです。

本記事では `create-vue` によって生成されたプロジェクト構成が、現在の Vue におけるベストプラクティスであるという前提の元、生成された `tsconfig` を深堀ります。

なお、本記事では `create-vue` のバージョンは [v3.3.4](https://github.com/vuejs/create-vue/releases/tag/v3.3.4) を使用します。

# プロジェクト作成

本記事では以下のようにプロジェクトをセットアップします。

```bash
$ npm init vue@3

Vue.js - The Progressive JavaScript Framework

✔ Project name: … vue-project
✔ Add TypeScript? …  Yes
✔ Add JSX Support? … No
✔ Add Vue Router for Single Page Application development? … No
✔ Add Pinia for state management? … No
✔ Add Vitest for Unit Testing? … Yes
✔ Add Cypress for End-to-End testing? … No
✔ Add ESLint for code quality? … No
```

TypeScript の有効化と、 [Vitest](https://vitest.dev/) のセットアップのみ行っています。

[VueRouter](https://router.vuejs.org/), [Pinia](https://pinia.vuejs.org/), [Cypress](https://www.cypress.io/), [ESLint](https://eslint.org/), [Prettier](https://prettier.io/) のセットアップも可能ですが、これらは TypeScript の設定には影響しないため、省略しています。

# package.json

スキャフォールドされたプロジェクトの `package.json` は以下になります。

```json:package.json
{
  "name": "vue-project",
  "version": "0.0.0",
  "scripts": {
    "dev": "vite",
    "build": "run-p type-check build-only",
    "preview": "vite preview --port 4173",
    "test:unit": "vitest --environment jsdom",
    "build-only": "vite build",
    "type-check": "vue-tsc --noEmit -p tsconfig.vitest.json --composite false"
  },
  "dependencies": {
    "vue": "^3.2.38"
  },
  "devDependencies": {
    "@types/jsdom": "^20.0.0",
    "@types/node": "^16.11.56",
    "@vitejs/plugin-vue": "^3.0.3",
    "@vue/test-utils": "^2.0.2",
    "@vue/tsconfig": "^0.1.3",
    "jsdom": "^20.0.0",
    "npm-run-all": "^4.1.5",
    "typescript": "~4.7.4",
    "vite": "^3.0.9",
    "vitest": "^0.23.0",
    "vue-tsc": "^0.40.7"
  }
}
```

TypeScript 関連として、以下パッケージが追加されています。

- [`@types/jsdom`](https://www.npmjs.com/package/@types/jsdom): [jsdom](https://github.com/jsdom/jsdom) の型パッケージ
- [`@types/node`](https://www.npmjs.com/package/@types/node): [Node.js](https://nodejs.org/en/) のビルトインモジュールの型パッケージ
- [`@vue/tsconfig`](https://github.com/vuejs/tsconfig): Vue プロジェクト向けの TSConfig のベース

本記事においては、`@vue/tsconfig` が特に重要で、このパッケージで公開されている設定ファイルをベースに `tsconfig` を構築します。

https://github.com/vuejs/tsconfig

# tsconfig.json

まずプロジェクトの TypeScript 設定のベースとなる、`tsconfig.json` ですが、以下のように定義されています。

```json:tsconfig.json
{
  "files": [],
  "references": [
    {
      "path": "./tsconfig.config.json"
    },
    {
      "path": "./tsconfig.app.json"
    },
    {
      "path": "./tsconfig.vitest.json"
    }
  ]
}
```

`references` という見慣れないフィールドが定義されていますが、これは `Project References` という TypeScript の機能です。

https://www.typescriptlang.org/docs/handbook/project-references.html

https://zenn.dev/katsumanarisawa/articles/58103deb4f12b4

`Project References` は複雑な仕組みではありますが、ここでは TypeScript プロジェクトを `tsconfig.config.json` `tsconfig.app.json` `tsconfig.vitest.json` の3種類の設定ファイルに基づいて論理分割していると考えて頂ければ大丈夫です。

ルートプロジェクト (`tsconfig.json`) 自体は、分割された3プロジェクトに依存しているだけで自身は何もしないため、 `files` フィールドが空になっています。

依存するそれぞれのプロジェクトは以下の役割を持っています。

|設定ファイル名|役割|
|---|---|
|`tsconfig.config.json`|`vite.config.ts` などの設定ファイルを TypeScript で書けるようにする|
|`tsconfig.app.json`|アプリケーションコードを TypeScript で書けるようにする|
|`tsconfig.vitest.json`|`vitest` によるテストコードを TypeScript で書けるようにする|

上記のようにプロジェクトを論理分割することで、単一のプロジェクトでまとめる場合に比べ、以下のようなメリットが得られます

- アプリコードからテストコードを `import` するような歪な依存を防げる
- アプリコードはブラウザで動作するが、テストコードは Node 上で動作するため、それぞれで使用できるモジュールを制限できる
- ビルドの対象を制限することでパフォーマンスが向上する

それらをふまえて、各設定ファイルについて覗いてみましょう。

# tsconfig.config.json

# tsconfig.app.json

# tsconfig.vitest.json