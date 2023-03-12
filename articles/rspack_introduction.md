---
title: "Rust + Webpack = Rspack の紹介"
emoji: "🦀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["javascript", "webpack", "rspack"]
published: false
---

# 概要

本記事では、JavaScript モジュールバンドラの一種である、 [`Rspack`](https://www.rspack.dev/) について、公式ドキュメントからわかることを要約し、自分なりの所感を付け加えて紹介します。

![](https://storage.googleapis.com/zenn-user-upload/045262a8ae3b-20230312.png)
*https://www.rspack.dev/misc/branding.html*

※ 本記事では、[`Webpack`](https://webpack.js.org/) に関する最低限の知識を前提としています。
※ 本記事は、2023/03/12 時点の情報であり、古い内容が残っている場合があります

# TL;DR;

- `Rspack` は `Webpack` との互換性とパフォーマンスを両立したモジュールバンドラ
- `Webpack` より5〜10倍高速でありながらも、多くの設定、`loader`, `plugin` に互換性を持つ
- `Webpack` の一般的な設定パターンがビルトインで用意されており、シンプルに使える
- 現在はアーリーステージで、[`Vue.js`](https://vuejs.org/) のサポートも不十分という状態なので今後に期待

# Rspack とは

Webアプリケーション開発において、モジュールバンドルは(今の所)欠かせないツールの1つです。

モジュールバンドルの代表的なツールとして、Webpackが挙げられます。しかし、近年では、プロダクトの大規模化や複雑化に伴い、`Webpack` では開発サーバーやプロダクションビルドの低速化する問題が浮き彫りとなっています。

この問題に対するアプローチとして、 [`Vite`](https://ja.vitejs.dev/) や [`Turbopack`](https://turbo.build/pack) が台頭してきています。

しかし、それらのツールは、`Webpack` とは全く異なるアプローチを採用しているため、`Webpack` とは互換性がなく、既存のプロダクトを移行する際に困難を伴うことがあります。

`Rspack` は、これらの問題に対応するために開発されたツールです。**[`Rust`](https://www.rust-lang.org/ja) ベースの高速なモジュールバンドラーでありながら、`Webpack` と互換性を保つ**ように設計されており、既存の設定ファイル、ローダー、プラグインをそのまま活用することができます。

以下は `Rspack` における設定ファイルで、`webpack.config.js` に近いことがわかります。

```js:rspack.config.js
module.exports = {
  context: __dirname,
  // エントリーは Webpack と同様、バンドルの開始地点
  entry: {
    main: "./src/main.jsx"
  },
  // Webpack の HtmlWebpackPlugin にあたる仕組みがビルトイン化されている(後述)
  builtins: {
    html: [
      {
        template: "./index.html"
      }
    ]
  },
  // ローダーの適用方法は Webpack と同様
  module: {
    rules: [
      {
        test: /\.svg$/,
        type: "asset"
      }
    ]
  }
};
```

# 発音について

`Rspack` の発音記号は `/'ɑrespæk/` でカタカナにするなら「アーレスペック」に近いようです。
(個人的には `Rust` + `Webpack` で「ラスパック」と呼びたくなってしまいます)

# Webpack との互換性について

`Rspack` は、`Webpack` との互換性を強みとしていますが、ここでいう互換性は以下を指します。

- 設定ファイルの[データ構造](https://www.rspack.dev/guide/config-diff.html)が `Webpack` のそれと近い
- [主要なローダー](https://www.rspack.dev/guide/loader-compat.html)を使用できる
- [主要なプラグイン](https://www.rspack.dev/guide/plugin-compat.html)が使用できる(または同等の機能がある)
- JavaScript を用いて、`Webpack` と同様にカスタムローダー、プラグインの実装が可能

`Rspack` は 100% の互換性を目指してません。上記リンクの通り、主要な仕組みを優先して対応し、他はコミュニティからフィードバックに応じて、必要であれば互換性を広げる方針のようです。

# モジュール解決のデフォルトサポート

いくつかのファイル形式については、ローダーの設定を書かずとも自動で適切に変換されるようになりました。以下はその例です。

## TypeScript (.ts)

`Webpack` で [`TypeScript`](https://www.typescriptlang.org/) を使用する場合、[`ts-loader`](https://github.com/TypeStrong/ts-loader) や [`babel-loader`](https://github.com/babel/babel-loader) を使ってトランスパイルする必要がありましたが、 `Rspack` ではビルトインで `TypeScript` がサポートされています。

`Rspack` は、.ts ファイルをインポートすると、内部では `SWC` を用いて高速なトランスパイルをします。

ただし、[`SWC`](https://github.com/swc-project/swc) はトランスパイルのみで型チェックは行わないため、型チェックについては別途 `tsc` などを使用する必要があります

## JSX (.jsx / .tsx)

`Rspack` は [`JSX`](https://beta.reactjs.org/learn/writing-markup-with-jsx) もビルトインでサポートしており、`babel-loader` や [`@babel/preset-react`](https://babeljs.io/docs/babel-preset-react) などの設定を行う必要がなく、`.jsx` `.tsx` ファイルに対してトランスパイルが自動的に適用されます。

## CSS (.css / .module.css)

CSS についても `.css` `.module.css` ともにサポートされており、[`css-loader`](https://github.com/webpack-contrib/css-loader) や [`style-loader`](https://github.com/webpack-contrib/style-loader) を設定する必要がなくなります。

# ローダーの対応状況

`Rspack` では、`Webpack` における主要なローダーの多くがサポートされています。

以下がそのまま利用可能なローダーになります。

- `sass-loader`
- `less-loader`
- `postcss-loader`
- `yaml-loader`
- `json-loader`
- `stylus-loader`
- `@mdx-js/loader`
- `@svgr/webpack`
- `raw-loader`
- `url-loader`
- `css-loader`

以下については注意が必要です。

## vue-loader

**[`vue-loader`](https://github.com/vuejs/vue-loader) は、`Vue.js` での開発においては必須と言えるローダーですが、現状はサポートされていません**

詳細は不明ですが、現状の `Rspack` が提供する API では、`vue-loader` を動かすことが不可能なようです。

ただし `Rspack` 側としても、`vue-loader` のサポートは優先度が高いと判断しているようで、[2023年の4月から6月を目処に対応する](https://www.rspack.dev/misc/FAQ.html#when-will-rspack-support-vue)ようです。

なお、 `vue-loader` を直接使うことは出来ずとも、同等の動作をするローダーを実装することは既に可能なようです。

https://github.com/web-infra-dev/rspack/blob/main/examples/vue/vue-loader.js

## babel-loader

[`babel-loader`](https://github.com/babel/babel-loader) 自体は対応していますが、パフォーマンスに懸念があるため、推奨されていません。

`Rspack` では内部的に `SWC` を使用した `TypeScript` のトランスパイルと、レガシーブラウザ向けのコード変換機能を持っているため、ほとんどのケースでは `babel-loader` は必要ありません。

ただし、特定の理由で `babel-loader` を使用する場合は、パフォーマンス低下に注意する必要があります。

# プラグインの対応状況

いくつかの主要なプラグインはそのまま利用可能です。

- `DefinePlugin`
- `copy-webpack-plugin`
- `mini-css-extract-plugin`
- `terser-webpack-plugin`
- `progressPlugin`
- `webpack-bundle-analyzer`
- `webpack-stats-plugin`
- `tsconfig-paths-webpack-plugin`
- `HotModuleReplacementPlugin`

また、いくつかのプラグインについては、同等の組み込み機能が用意されている場合があります。

組み込み機能は、設定ファイルの [`builtins`](https://www.rspack.dev/config/builtins.html) から設定することができます。

# 開発サーバー

`Webpack` を用いた開発サーバーを立ち上げる場合、 [`webpack-dev-server`](https://github.com/webpack/webpack-dev-server) を別途導入しますが、 `RSpack` では開発用サーバーが組み込みで用意されており、以下コマンドですぐに開発を始めることができます。

```bash
$ yarn rspack serve
```

当然、設定は `webpack-dev-server` との互換性が充分にあり、 `HMR` や `Proxy` といった機能も使用可能です。

# プロダクションビルド

ここでは詳細は割愛しますが、以下のような `Webpack` ではお馴染みのプロダクションビルド時の最適化についても対応しています。

- ミニフィケーション
- コード分割(チャンク化)
- ツリーシェイキング(デッドコード検出)
- ソースマップ生成

# パフォーマンス

さて、肝心のパフォーマンスですが、`Rspack` を `Webpack + babel` の組み合わせと比較した場合、以下のようなベンチマークが取れたようです。

- 開発環境の初回起動時: 11倍高速
- ルートモジュール変更時のHMR: 3倍高速
- リーフモジュール変更時のHMR: 3倍高速
- プロダクションビルド: 7倍高速

※ ベンチマーク手法の詳細は [Benchmark case](https://www.rspack.dev/misc/benchmark.html#benchmark-case) 参照

もちろん、 `Webpack` を使いつつも、 `babel` を `SWC` に置き換えたり、一部で [`esbuild`](https://github.com/evanw/esbuild) を使用するなどでパフォーマンスを改善することは可能です。

とはいえ、コア機能であるモジュール解決やバンドル化、チャンク切り出しなどの仕組みを `JavaScript` でなく `Rust` で書くことで、マルチコアCPU による並列化を最大限に生かして効率化できているようです。

# まとめと所感

本記事では、`Webpack` との互換性を持ちつつも `Rust` によって高速に動作するように書き換えられたモジュールバンドらである、`Rspack` について紹介しました。

近年では `Vite` や `Turbopack` など、新たなアプローチによってパフォーマンスを大幅に向上させるツールが台頭していますが、そこに新たな仲間が加わりました。

新規プロダクトについては既に `Vite` を使えば最高の開発体験を得られているとも言えますが、一方で、`Webpack` を用いている既存プロダクトを `Vite` に移行するというのは一筋縄では行かないことがあります。

そういった事情から、いつまでも `Webpack` を使い続けてしまうプロダクトにとって、 `Rspack` は一筋の光になりえます。もし `Vite` と比べて遥かに低コストで移行ができると言うならこれ以上のソリューションはないでしょう。

また、個人的には終わりを迎えてしまった `webpack` に関する知識・ノウハウが無駄にならずに今後も活かせる可能性に心躍っています。
(これは逆に早く無くなってほしいという声もありそうですが)

とは言え、自社では `Vue.js` を使用している都合、`vue-loader` の完全サポートを待ち望みつつウォッチしていくことになりそうです。