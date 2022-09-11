---
title: "Cypress で Vue 3 コンポーネントを楽々テストしちゃおう"
emoji: "🎉"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["javascript", "typescript", "vue", "cypress"]
published: false
---

# 概要

本記事では、オープンソースのE2Eテストフレームワークである [Cypress](https://www.cypress.io/) の [Component Testing](https://docs.cypress.io/guides/component-testing/) を用いて、 [Vue 3](https://v3.ja.vuejs.org/) のコンポーネントのテストを書く手法といくつかのレシピを紹介します。

想定読者は主に以下の方々です。

- モダンなコンポーネントテストの手法例を知りたい
- Cypress を E2E テストで使用しているが、コンポーネントテストでの利用にも興味がある
- Vue コンポーネントの最適なテスト手法を探している

基本的には公式ドキュメントの内容をベースにしていますが、要所を抜き出したり、サンプルコードをよりイメージしやすいものにしたり、実際のプロダクトレベルで使用するための各種調整などを追加しています。

# 技術構成

- `Cypress`: 10.7.0
- `Vue.js`: 3.2.37
- `TypeScript`: 4.6.4
- `Vite`: 3.1.0
- `yarn`: 1.22.19

技術構成を `React` や `Webpack` に差し替えてもコンポーネントテストは可能ですが、本記事では筆者自身の好みである上記構成で進めます。

# 注意事項

- `Cypress` 自体の説明は最低限に留めます
- `Cypress` の `Component Testing` は記事公開時点でβ機能です
- Vue コンポーネントは Vue 2 ユーザーの方にもわかりやすいよう、 `<script setup>` 含む `Composition API` は使用せず、 `Option API` で記述します

# Cypress について

`Cypress` は一言でいうと**モダンWebアプリケーションのための次世代フロントエンドツール**です。

**テストのセットアップ、作成、実行、デバッグを1パッケージですぐに利用可能**で、ブラウザと一体化した `Cypress` アプリケーション内に `iframe` でテスト対象アプリケーションを埋め込む独自の仕組みを提供します。

これだけだとどんなツールなのかイメージが湧きづらいと思いますが、公式サイトに掲載されている以下動画に目を通すとイメージが湧くと思います。

https://vimeo.com/237527670

また、公式ドキュメントに目を通すのが一番確実ですが、筆者が `Cypress` に初めて触れた際にざっくりとメモをとったスクラップが以下になりますのでご参考までどうぞ。

https://zenn.dev/sa2knight/scraps/b5946489946802

# Component Testing について

Component Testing (以降、コンポーネントテスト) は、E2Eテストを主軸としてきた `Cypress` に新たに追加されたテストモードで、React や Vue といったコンポーネント指向フレームワークにおける、コンポーネントレベルの単体テストを、ブラウザベースで行うものです。

E2E テストでは `iframe` 内にテスト対象の Webサイトを実際に読み込んで操作するというプロセスでしたが、**コンポーネントテストでは任意のビルドツールを使って、コンポーネント単体をビルドし、サンドボックス上にレンダリングすることで単体でのテストを行う**ことが出来ます。

類似のツールとして [Storybook](https://storybook.js.org/) の [Interaction tests](https://storybook.js.org/docs/react/writing-tests/interaction-testing) がありますが、 `Storybook` がコンポーネントのカタログからドキュメンテーションまで包括的に扱うのに対して、 `Cypress` はテストに特化しているので使い分けの判断はプロジェクトの方針によるのかなと個人的には思います。

# E2Eテストとコンポーネントテストの比較

公式ドキュメント内での比較を整理すると、以下のようになります。

||メリット|デメリット|
|----|----|----|
|E2E|アプリケーション全体を包括的にテストでき、UXレベルでのテストシナリオを書けることから、QA観点でのテストができる|セットアップやメンテナンスのコストが高く、アプリケーションが依存する外部のシステム、APIの考慮も必要|
|コンポーネント|コンポーネント単位の独立したテストができることから、実行が高速で信頼性が高く、セットアップも用意で、外部システムに依存することもない|アプリケーション全体の担保にはならないので、結合部分のテストがどのみち必要なことと、たいていはQAでなく開発者向けのテストになる|

たいていは E2E テストは認証認可や課金機能などのミッションクリティカルな機能の検証に使ったり、画面を跨いでのデータの永続性といった単体では見れない範囲に限定して実施するのが望ましいようです。

# セットアップ

本記事で `Cypress` を使用するテスト対象のプロジェクトを作成します。

## Vue 3 + TypeScript + Vite のスキャフォルド

`Vite` でスキャフォルドするのが最も早いので今回はそれでセットアップします。

```bash
$ yarn create vite --template vue-ts
$ cd vite-project
$ yarn install
```

今回はコンポーネント単体のビルドに Vite を使用するだけなので、Vue アプリケーション自体は不要のため、開発サーバーを立ち上げる必要がありません。

## Cypress のセットアップ

`cypress` を追加します。

```bash
$ yarn add -D cypress
```

`cypress`　をコンポーネントテストモードで起動します。通常は E2E テストモードかを選択する画面を挟みますが、 `--component` オプションを付与するとそれをショートカットできます。

```bash
$ yarn cypress open --component
```

初回起動時(プロジェクトルートに `cypress` ディレクトリがない場合)はプロジェクトのセットアップナビゲーションが始まります。

![](https://storage.googleapis.com/zenn-user-upload/4fa690722e98-20220907.png)

初めにフレームワーク及びバンドラツールの選択画面になります。自動で検知されることもありますが、設定されていない場合、 `Vue.js 3` 及び `Vite` を選択しましょう。

続けて　`Install Dev Dependencies` 画面になりますが、本記事の手順通りなら既にインストール済みなので `Continue` を押下するだけです。

![](https://storage.googleapis.com/zenn-user-upload/01a804925cb9-20220907.png)

`cypress` ディレクトリが作成され、以下のような初期設定ファイルが自動生成された旨が表示されます。

|ファイル名|内容|
|---|---|
|cypress.config.ts|コンポーネントテストで Vue + Vite を使うことが宣言されてる|
|cypress/support/component.ts|コンポーネントテスト用の型定義と `mount` コマンドが追加されてる|
|cypress/support/command.ts|カスタムコマンドを定義するための場所|
|cypress/support/component-index.html|ビルドしたコンポーネントをレンダリングするための HTML|
|cypress/fixtures/example.json|テストデータのサンプル(不要)|

続けて `Continue` を押下すると、テストに使用するブラウザの選択画面になるため、任意のブラウザを選択してください。

![](https://storage.googleapis.com/zenn-user-upload/4aa649a92328-20220907.png)

これで `Cypress` のセットアップは完了しましたが、まだテストコードはもちろん、テスト対象のコンポーネントもない状態なので、 `Cypress` は一旦終了して先にコンポーネントの実装を行います。

## テスト用コンポーネントを作成

以下のようなシンプルな `Stepper` コンポーネントを作成します。

```vue:src/components/Stepper.vue
<template>
  <div>
    <button data-cy="decrement" @click="count--">-</button>
    <span data-cy="counter">{{ count }}</span>
    <button data-cy="increment" @click="count++">+</button>
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue";

export default defineComponent({
  props: {
    initial: {
      type: Number,
      default: 0,
    },
  },
  emits: ["change"],
  data() {
    return {
      count: this.initial,
    };
  },
});
</script>
```

仕様は単純で以下の通りです。

- カウント値 `count` を内部で持っている
- `count` は `props`(`initial`) 経由で初期値を指定することができ、省略時　0 になる
- `+` ボタンを押下すると、 `count` がインクリメントされる
- `-` ボタンを押下すると、 `count` がデクリメントされる
- 最新の `count` の値が表示される

## テストコードを作成

前項で `Stepper.vue` を作成した状態で、再度 `Cypress` を起動します。その際、 `--browser` オプションでブラウザの指定もすることでさらに操作を省略できます。

```bash
$ yarn cypress open --component --browser chrome
```

Chrome が起動し、 `Create your first spec` 画面が表示されるので、 `Create from component` を押下し、先程作成した `Stepper` コンポーネントを選択します。

![](https://storage.googleapis.com/zenn-user-upload/d4b0f6c5805b-20220907.png)

するとコンポーネントと同ディレクトリに、 `Stepper.cy.ts` というテストコードが作成され、最初から `Stepper` コンポーネントをマウントする最低限のコードが記述され、実行可能な状態となっています。

```ts:src/components/Stepper.cy.ts
import Stepper from './Stepper.vue'

describe('<Stepper />', () => {
  it('renders', () => {
    // see: https://test-utils.vuejs.org/guide/
    cy.mount(Stepper)
  })
})
```

![](https://storage.googleapis.com/zenn-user-upload/71ac65d415e9-20220907.png)

見事、`Stepper` コンポーネントが Cypress アプリケーション上の `iframe` 内にレンダリングされ、コンポーネント単体でのテストが可能な状態になりました。

## TypeScript 対応

前項で作成されたテストコードには、 `describe` `it` `cy` といった、 `Cypress` モジュール上のオブジェクトに暗黙に依存しているため、そのままだと TypeScript で解釈することができません。

これらの型宣言が行われている、 `cypress/support/*.ts` を、 `tsconfig` の `include` フィールドに追加することで、型サポートを受けることができます。

```json:tsconfig.json
  "include": [
    "src/**/*.ts",
    "src/**/*.d.ts",
    "src/**/*.tsx",
    "src/**/*.vue",
    "cypress/**/*.ts",
  ],
```

これでセットアップはすべて終了です。作成した `Stepper` コンポーネントを使ったテストコードを書いていきましょう。

# レンダリング内容の検証

改めて、生成されたテストコードを見てみましょう。

```ts:src/components/Stepper.cy.ts
import Stepper from './Stepper.vue'

describe('<Stepper />', () => {
  it('renders', () => {
    // see: https://test-utils.vuejs.org/guide/
    cy.mount(Stepper)
  })
})
```

最も重要なのは `cy.mount` です。

[vue-test-utils](https://test-utils.vuejs.org/) の　`mount` に似ていますが、ここでは戻り値 (`vue-test-utils` では慣習的に `wrapper`) を使用することはありません。

`Cypress` のテストコード内で `mount` されたコンポーネントは (ここでは `vite` によって) ビルドされ、`Cypress` アプリケーション上の `iframe` 内に描画されます。

そのため、あとは `cy` モジュールを経由することで、E2Eテストと同様の API を用いてテストすることができます。

`Stepper` コンポーネントの `count` は、 `props` を省略した場合は 0 が初期描画されるので、それをテストしてみましょう。

```ts:src/components/Stepper.cy.ts
const counterSelector = '[data-cy=counter]'

describe('<Stepper />', () => {
  it('ステッパーの初期値は 0 である', () => {
    cy.mount(Stepper)
    cy.get(counterSelector).should('have.text', '0')
  })
})
```

`Cypress` では要素セレクタに慣習的に `data-cy` 属性を使用するので、カウント値を表示している要素のセレクタを使用します。

あとはE2Eテストと同様に、 `cy.get` `should` を使用して、初期値である　`0` が描画されていることを確認して完了です。

![](https://storage.googleapis.com/zenn-user-upload/619489c37758-20220908.png)

# props の検証

`Stepper` コンポーネントの `props`　である `initial` を指定して、それが初期値として描画されていることをテストします。

```ts:src/components/Stepper.cy.ts
  it("ステッパーの初期値を props で指定することができる", () => {
    cy.mount(Stepper, { props: { initial: 100 } });
    cy.get(counterSelector).should("have.text", "100");
  });
```

`mount` メソッドの第二引数でマウントオプションが指定できるので、そこで `props` を挿入するだけになります。

# インタラクションの検証

`Stepper` コンポーネントは、 `+` `-` 2種類のボタンがあり、それらをクリックすることで `count` を増減させられます。

両ボタンのセレクタを追加し、あとはE2Eテストと同様のAPIを使用できます。

```ts:src/components/Stepper.cy.ts
const incrementSelector = "[data-cy=increment]";
const decrementSelector = "[data-cy=decrement]";
```

```ts:src/components/Stepper.cy.ts
  it("+ボタンが押下されると、カウンターがインクリメントされる", () => {
    cy.mount(Stepper);
    cy.get(incrementSelector).click();
    cy.get(counterSelector).should("have.text", "1");
  });

  it("-ボタンが押下されると、カウンターがデクリメントされる", () => {
    cy.mount(Stepper);
    cy.get(decrementSelector).click();
    cy.get(counterSelector).should("have.text", "-1");
  });
```

# Event の Emit の検証

ここまでの `Stepper` コンポーネントは、ボタン押下時にカウント値が増減するだけでしたが、親コンポーネントからその変更を監視したくなることがあると思います。

`Stepper` コンポーネントを改修し、両ボタンクリック時に親コンポーネントに対して `change` イベントを発火するようにしましょう。

```vue:src/components/Stepper.vue
<template>
  <div>
    <button data-cy="decrement" @click="decrement">-</button>
    <span data-cy="counter">{{ count }}</span>
    <button data-cy="increment" @click="increment">+</button>
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue";

export default defineComponent({
  props: {
    initial: {
      type: Number,
      default: 0,
    },
  },
  emits: ["change"],
  data() {
    return {
      count: this.initial,
    };
  },
  methods: {
    increment() {
      this.count++;
      this.$emit("change", this.count);
    },
    decrement() {
      this.count--;
      this.$emit("change", this.count);
    },
  },
});
</script>

```

上記コンポーネントに対して、 `change` イベント経由で最新のカウント値を受け取るれるかをテストします。

```ts:src/components/Stepper.cy.ts
  it("+ボタンが押下されると、change イベントでカウンターの値が取得できる", () => {
    const onChangeSpy = cy.spy().as("onChangeSpy");
    cy.mount(Stepper, { props: { onChange: onChangeSpy } });
    cy.get(incrementSelector).click();
    cy.get("@onChangeSpy").should("be.calledWith", 1);
  });

  it("-ボタンが押下されると、change イベントでカウンターの値が取得できる", () => {
    const onChangeSpy = cy.spy().as("onChangeSpy");
    cy.mount(Stepper, { props: { onChange: onChangeSpy } });
    cy.get(decrementSelector).click();
    cy.get("@onChangeSpy").should("be.calledWith", -1);
  });
```

`cy.spy` で、スパイ関数を作成し、それを後から Cypress API で参照できるように、 `as` で別名を付け、 `change` イベントにバインドします。

ここで `change` イベントなのに、 `onChange` というイベント名、かつイベントなのに `props` で渡すという、 Vue らしくないコードに見えますが、これは `vue-test-utils` の慣習に寄せているそうです。

> You may notice the syntax above in the non-JSX example relies on binding events to the props key in mount. While this isn't "idiomatic Vue", it's the current signature of Vue Test Utils.

# スロットの検証

スロットに対して任意の要素を注入できることをテストすることも可能です。

これまでの `Stepper` コンポーネントでは約不足なので、スロットをふんだんに使った、`Modal` コンポーネントを実装します。

```vue:src/components/Modal.vue
<template>
  <div class="modal">
    <div data-cy="modal-header">
      <slot name="header">
        <span>No Title</span>
      </slot>
      <hr />
    </div>
    <div data-cy="modal-content">
      <slot />
    </div>
    <template v-if="$slots.footer">
      <div data-cy="modal-footer">
        <hr />
        <slot name="footer" />
      </div>
    </template>
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue";
export default defineComponent({});
</script>

<style>
.modal {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  background-color: white;
  border-radius: 5px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.26);
  width: 30rem;
  height: 25rem;
}
</style>
```

非常に簡素ではありますが、上記コンポーネントは3種類のスロットを提供します。

- `header`: ヘッダーに差し込む要素で、省略時フォールバック用の初期値 `No Title` を持つ
- `default`: デフォルトスロットでモーダルコンテンツを差し込む要素
- `footer`: フッターに差し込む要素で、省略時はフッター自体が非表示になる

上記の仕様を元に、以下の観点のテストを行います。

- `header` を注入できる
- `header` を省略するとフォールバックする
- `default` を注入できる
- `footer` を注入できる
- `footer` を省略するとフッター要素自体が表示されない

作成したテストコードが以下になります。

```ts:src/components/Modal.cy.ts
import Modal from "./Modal.vue";

const headerSelector = "[data-cy=modal-header]";
const contentSelector = "[data-cy=modal-content]";
const footerSelector = "[data-cy=modal-footer]";

describe("<Modal />", () => {
  it("ヘッダーにスロットを注入しない場合、ヘッダーはデフォルトタイトルにフォールバックする", () => {
    cy.mount(Modal);
    cy.get(headerSelector).should("have.text", "No Title");
  });

  it("ヘッダーに名前付きスロットを注入した場合、ヘッダーにスロットの内容が表示される", () => {
    cy.mount(Modal, { slots: { header: "Custom Title" } });
    cy.get(headerSelector).should("have.text", "Custom Title");
  });

  it("デフォルトスロットを注入した場合、モーダルコンテンツにスロットの内容が表示される", () => {
    cy.mount(Modal, { slots: { default: "Modal Content" } });
    cy.get(contentSelector).should("have.text", "Modal Content");
  });

  it("フッターにスロットを注入しない場合、フッター要素自体が表示されない", () => {
    cy.mount(Modal);
    cy.get(footerSelector).should("not.exist");
  });

  it("フッターに名前付きスロットを注入した場合、フッターにスロットの内容が表示される", () => {
    cy.mount(Modal, { slots: { footer: "Custom Footer" } });
    cy.get(footerSelector).should("have.text", "Custom Footer");
  });
});
```

![](https://storage.googleapis.com/zenn-user-upload/6f858fdc16ce-20220908.png)

`mount` 関数の第二引数で `slots` を指定して注入することができます。

上記コードではテキストノードのみを注入していますが、以下のように　`h` 関数を使って仮想DOMを注入することも可能です。

```ts
  it("ヘッダーに名前付きスロットを注入した場合、ヘッダーにスロットの内容が表示される", () => {
    cy.mount(Modal, {
      slots: { header: h("h1", { "data-cy": "custom-element" }, "タイトル") },
    });
    cy.get("[data-cy=custom-element]").should("have.text", "タイトル");
  });
```

また、今回は登場しませんでしたが、スロットスコープから `props` を受け取る場合は、関数経由で参照することが出来ます。

```ts
cy.mount(Modal, { slots: { default: props => props.value } });
```

# Vue プラグイン対応

最後に、Vue アプリケーションを開発する場合に多くのケースで導入される [Vue I18n](https://kazupon.github.io/vue-i18n/), [Vue Router](https://router.vuejs.org/), [Pinia](https://pinia.vuejs.org/) に依存したコンポーネントのテストをできるようにします。

やや強引ですが、上記3ライブラリすべてに依存する以下のようなコンポーネントを用意し、それを単体テストできるようにします。

## インストール

```bash
$ yarn add pinia vue-i18n vue-router
```

## Pinia

`pinia` は `vuex` の後継であるグローバルステート管理ライブラリです。

https://zenn.dev/sa2knight/articles/74f066b7be24c3

今回はシンプルにカウンターを増減するだけのストアを用意します。

```ts:src/store.ts
import { createPinia, defineStore } from 'pinia'

export const useCounterStore = defineStore('counter', {
  state: () => {
    return {
      count: 0
    }
  },
  actions: {
    increment() {
      this.count++
    },
    decrement() {
      this.count--
    }
  }
})

export default createPinia()
```

## VueI18n

`VueI18n` は Vue における多言語対応をサポートするライブラリです。

今回は一種類のメッセージを、日本語・英語それぞれ定義しておきます。

```ts:src/i18n.ts
import { createI18n } from 'vue-i18n'

const messages = {
  en: {
    message: {
      hello: 'hello world'
    }
  },
  ja: {
    message: {
      hello: 'こんにちは、世界'
    }
  }
}

export default createI18n({ locale: 'ja', messages })
```

## VueRouter

`VueRouter` は Vue におけるルーティングライブラリです。

通常はコンポーネントの単体テストにおいてルーティングを意識する必要はありませんが、VueRouter に対して依存したコードがコンポーネント内で即時実行されることは少なくないと思うので、適当なルートを用意しておきます。

```ts:router.ts
import { createRouter, createWebHistory } from 'vue-router'

const Home = { template: '<div>Home</div>' }
const About = { template: '<div>About</div>' }

const routes = [
  { path: '/', component: Home },
  { path: '/about', component: About }
]

export default createRouter({ history: createWebHistory(), routes })
```

## コンポーネントからの利用

`pinia` `VueI18n` `VueRouter` すべてに依存したコンポーネント `SuperComponent.vue` を実装します。細かいコードまで追う必要はありませんが、各プラグインのセットアップが完了していないと一切動かないコンポーネントと理解していただければ大丈夫です。

```vue:src/components/Child.vue
<template>
  <!-- vuei18n 依存-->
  <h1 data-cy="message">{{ $t('message.hello') }}</h1>
  <button data-cy="toggle" @click="toggleLocale">toggleLocale</button>

  <!-- pinia 依存-->
  <div>
    <button data-cy="decrement" @click="store.decrement">-</button>
    <span data-cy="count">{{ store.count }}</span>
    <button data-cy="increment" @click="store.increment">+</button>
  </div>

  <!-- vue-router 依存-->
  <div>
    <router-link data-cy="link" to="/about">About</router-link>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import { useCounterStore } from '../store'

export default defineComponent({
  setup() {
    const store = useCounterStore()
    return { store }
  },
  methods: {
    toggleLocale() {
      this.$i18n.locale = this.$i18n.locale === 'ja' ? 'en' : 'ja'
    }
  }
})
</script>
```

## プラグイン注入

`pinia` `VueI18n` `VueRouter` いずれも [Vueプラグイン](https://v3.ja.vuejs.org/guide/plugins.html#%E3%83%95%E3%82%9A%E3%83%A9%E3%82%AF%E3%82%99%E3%82%A4%E3%83%B3) として提供されているため、アプリケーションルートである `main.ts` にて、 Vueインスタンスにプラグインを注入します。

```ts:src/main.ts
import { createApp } from 'vue'
import SuperComponent from './components/SuperComponent.vue'
import i18n from './i18n'
import router from './router'
import store from './store'

const app = createApp(SuperComponent) // SuperComponent をルートに
app.use(router) // 注入!
app.use(store) // 注入!!
app.use(i18n) // 注入!!!
app.mount('#app')
```

## アプリケーション上での動作確認

ここまでで、 `yarn dev` で開発サーバーを起動した場合に、 `SuperComponent` が意図通り描画され、各操作ができることが確認できました。

```bash
$ yarn dev
```

![](https://storage.googleapis.com/zenn-user-upload/ed6ec6462466-20220908.png)

## 単体テストを書く

アプリケーション上では動作しましたが、 `Cypress` を用いた単体テストではどうでしょうか。

これまでと同様の手順で、 `SuperComponent.cy.ts` を作成し、テストの実行を試みます。

![](https://storage.googleapis.com/zenn-user-upload/391e69de4a17-20220908.png)

`pinia` のセットアップが行われていないというエラーが発生しました。

これは `yarn dev` の場合は `src/main.ts` が最初に実行されるため、そこで各種プラグインのセットアップが行われますが、 `Cypress` の場合は `SuperComponent.vue` を単体でビルドしているだけで、 `src/main.ts` は参照されていないため、セットアップが出来ていないことに起因します。

ではどうするか。 `cy.mount` 関数を拡張して、プラグインのセットアップを行った Vue インスタンス上でコンポーネントを描画するようにします。

現状では `cypress/support/component.ts` にて、以下のようにデフォルトの `mount` 関数をそのままコマンド登録しています。

```ts:cypress/support/component.ts
import { mount } from 'cypress/vue'
Cypress.Commands.add('mount', mount)
```

これを拡張し、以下のように対象の Vue コンポーネント経由でグローバルフィールドにアクセスし、プラグインを追加します。

```ts:cypress/support/component.ts
import { mount } from 'cypress/vue'
import store from '../../src/store'
import i18n from '../../src/i18n'
import router from '../../src/router'

Cypress.Commands.add('mount', component => {
  return mount(component, {
    global: {
      plugins: [store, i18n, router]
    }
  })
})
```

*※もちろん必要に応じてモックを定義したり初期化したりテストコード側から注入するなどの調整が必要になることはあります*

これで描画されるコンポーネントでもプラグインを参照できるようになりました。せっかくなのでテストコードをこのまま書いていきましょう。

```ts:cypress/
import SuperComponent from './SuperComponent.vue'

describe('<Child />', () => {
  beforeEach(() => {
    cy.mount(SuperComponent)
  })

  describe('メッセージ', () => {
    it('デフォルトで日本語メッセージが表示されている', () => {
      cy.get('[data-cy=message]').should('have.text', 'こんにちは、世界')
    })

    it('トグルボタンを押下すると、英語メッセージが表示される', () => {
      cy.get('[data-cy=toggle]').click()
      cy.get('[data-cy=message]').should('have.text', 'hello world')
    })
  })

  describe('カウンター', () => {
    it('カウンターの初期値は0', () => {
      cy.get('[data-cy=count]').should('have.text', '0')
    })

    it('+ボタンと-ボタンで押下で、カウンターの値を増減できる', () => {
      cy.get('[data-cy=increment]').click()
      cy.get('[data-cy=increment]').click()
      cy.get('[data-cy=count]').should('have.text', '2')
      cy.get('[data-cy=decrement]').click()
      cy.get('[data-cy=count]').should('have.text', '1')
    })
  })

  describe('フッターリンク', () => {
    it('フッターに /about へのリンクが表示されている', () => {
      cy.get('[data-cy=link]').should('have.attr', 'href', '/about')
    })
  })
})
```

これですべてのテストがパスしました。

![](https://storage.googleapis.com/zenn-user-upload/c00021049e08-20220908.png)

# まとめと所感

本記事では、オープンソースのE2Eテストフレームワークである [Cypress](https://www.cypress.io/) の [Component Testing](https://docs.cypress.io/guides/component-testing/) を用いて、 [Vue 3](https://v3.ja.vuejs.org/) のコンポーネントのテストを書く手法といくつかのレシピを紹介しました。

個人的には Vue コンポーネントの単体テストを行う最も敷居の低くコスパの高い手法だったなと感動し、その勢いのまま記事化するぐらいには刺さりました。

[Storybook](https://storybook.js.org/) の [Interaction tests](https://storybook.js.org/docs/react/writing-tests/interaction-testing) についても同様の感動はしたものの、(やり方が悪かったのか) 微妙にテストが安定しなく、まだまだ使い勝手が良くないと感じていましたが、 `Cypress` は自動リトライ機構が含まれているのもあって、抜群の安定感があるなと感じました。

本機能は記事公開時点ではβ機能ではありますが、 `Cypress` を使った E2E テストを運用できているプロジェクトなら既に実用できるレベルだと感じているので、これからもウォッチしていきたいと思います。