---
title: サンプルアプリケーションの作成
free: true
---

`Storybook` を導入する前に、`Storybook` で描画するためのコンポーネントを３種作成します。

# 作成するコンポーネント

本章では、以下の３種類のコンポーネントを作成します。

|コンポーネント名|役割|
|---|---|
|MyButton|スタイリングされたボタン要素|
|MyHeader|タイトルといくつかの MyButton を表示するヘッダー|
|MyPage|MyHeader とコンテンツを表示する|

`MyPage` をアプリケーションのルートとすると、以下のような簡易的なアプリケーションが出来上がります。

![](https://storage.googleapis.com/zenn-user-upload/b45db66583ba-20221225.png)

それでは下位コンポーネントから順に実装していきましょう。

:::message
以降、 `<script setup>` 及び `TypeScript` を使用した、比較的モダンな `Vue` コンポーネントを作成します。とはいえ`Storybook` を使うための下準備に過ぎないので、不慣れであっても構わず進めてください。
:::

# `MyButton` コンポーネント

`MyButton` コンポーネントは、以下の `props` を受け取ります。

|props名|型|役割|
|----|---|---|
|`label`|String|ボタンに表示するテキスト|
|`variant`|String|ボタンのスタイル種別で、`"primary"` `"secondary"` から指定する|
|`size`|String|ボタンのサイズで、`"large"` `"medium"` `"small"` から指定する|

また、ボタンをクリックすることで `click` イベントが `emit` されます。

```vue:src/components/MyButton.vue
<script setup lang="ts">
const props = defineProps({
  label: {
    type: String,
    required: true,
  },
  variant: {
    type: String,
    default: "primary",
    validator: (value: string) => {
      return ["primary", "secondary"].includes(value);
    },
  },
  size: {
    type: String,
    default: "medium",
    validator: (value: string) => {
      return ["small", "medium", "large"].includes(value);
    },
  },
});

const emits = defineEmits(["click"]);
</script>

<template>
  <button @click="emits('click')" :class="[props.variant, props.size]">
    {{ props.label }}
  </button>
</template>

<style scoped>
button {
  font-family: "Nunito Sans", "Helvetica Neue", Helvetica, Arial, sans-serif;
  font-weight: 700;
  border: 0;
  border-radius: 3em;
  cursor: pointer;
  display: inline-block;
  line-height: 1;
  color: white;
  padding: 1rem;
}
button.primary {
  background-color: #1ea7fd;
}
button.secondary {
  background-color: #ff7e00;
}

button.large {
  font-size: 1.25rem;
  padding: 1.25rem 1.5rem;
}

button.medium {
  font-size: 1rem;
  padding: 1rem 1.25rem;
}

button.small {
  font-size: 0.875rem;
  padding: 0.75rem 1rem;
}
</style>
```

# `MyHeader` コンポーネント

`MyHeader` コンポーネントは、以下の `props` を受け取ります。

|props名|型|役割|
|----|---|---|
|`isLoggedIn`|Boolean|ユーザーがログイン中であるかのフラグ|

`MyHeader` は、ユーザーのログイン状態に応じてボタンを出し分けます。

- 未ログインの場合:「ログイン」「会員登録」ボタンを表示
- ログイン済みの場合:「ログアウト」ボタンを表示

それぞれのボタンが押下されると、 `login` `signUp` `logout` のイベントが `emit` されます。

また、ウィンドウサイズが一定より小さい場合は各種ボタンが非表示になります。(モバイルならハンバーガーメニューが出るイメージ)

```vue:src/components/MyHeader.vue
<script setup lang="ts">
import MyButton from "./MyButton.vue";

const props = defineProps({
  isLoggedIn: {
    type: Boolean,
    required: true,
  },
});

const emits = defineEmits(["login", "signUp", "logout"]);
</script>

<template>
  <header>
    <h1>Storybook 7 + Vue 3 + TypeScript サンプル</h1>
    <div class="buttons">
      <template v-if="props.isLoggedIn">
        <MyButton size="small" label="ログアウト" @click="$emit('logout')" />
      </template>
      <template v-else>
        <MyButton size="small" label="ログイン" @click="$emit('login')" />
        <MyButton size="small" label="会員登録" @click="$emit('signUp')" />
      </template>
    </div>
  </header>
</template>

<style scoped>
header {
  font-family: "Nunito Sans", "Helvetica Neue", Helvetica, Arial, sans-serif;
  border-bottom: 1px solid rgba(0, 0, 0, 0.1);
  padding: 15px 20px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding-top: 45px;
  background-color: #fff;
}
h1 {
  font-weight: 900;
  font-size: 20px;
  line-height: 1;
  margin: 6px 0 6px 10px;
  display: inline-block;
  vertical-align: top;
}
button + button {
  margin-left: 10px;
}
@media screen and (max-width: 720px) {
  .buttons {
    display: none;
  }
}
</style>
```

# `MyPage` コンポーネント

`MyPage` コンポーネントは、 `props` は受け取らずに自身でユーザー情報を管理し、各種認証操作をハンドリングします。(ここでは非常に簡略化していますが、グローバルストアや API を統合するイメージ)

また、コンテンツは通常３段組ですが、ウィンドウサイズに応じて２段組、１段組と切り替わるレスポンシブデザインになっています。

```vue:src/components/MyPage.vue
<script setup lang="ts">
import { ref } from "vue";
import MyHeader from "./MyHeader.vue";

type User = {
  id: number;
};

const user = ref<User | null>(null);

function login() {
  user.value = { id: 1 };
}

function logout() {
  user.value = null;
}

function signUp() {
  // TODO: 会員登録フォームに移動する
}
</script>

<template>
  <div>
    <MyHeader
      :isLoggedIn="!!user"
      @login="login"
      @logout="logout"
      @signUp="signUp"
    />
    <main>
      <div class="content-wrapper">
        <div class="content" :key="i" v-for="i in 10">
          {{ "コンテンツがここに表示されます".repeat(20) }}
        </div>
      </div>
    </main>
  </div>
</template>

<style>
.content-wrapper {
  background-color: #fff;
  padding: 25px;
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  column-gap: 25px;
  row-gap: 1em;
}
@media screen and (max-width: 720px) {
  .content-wrapper {
    grid-template-columns: repeat(1, 1fr);
  }
}
@media screen and (min-width: 721px) and (max-width: 1024px) {
  .content-wrapper {
    grid-template-columns: repeat(2, 1fr);
  }
}
</style>
```

# 動作確認

アプリケーションルートで `MyPage` コンポーネントを表示するように `App.vue` を書き直します。

```vue:src/App.vue
<script setup lang="ts">
import MyPage from "./components/MyPage.vue";
</script>

<template>
  <MyPage />
</template>
```

また、 `src/main.js` にある、スキャフォルドされたスタイルシートの `import` は不要なので削除します。

```ts:src/main.ts
import { createApp } from "vue";
// import "./style.css"; ここを削除
import App from "./App.vue";

createApp(App).mount("#app");
```


この状態で開発環境を起動すれば、３コンポーネントを組み合わせたサンプルアプリケーションを動かすことができます。

![](https://storage.googleapis.com/zenn-user-upload/27bd7df37e56-20230107.gif)

これでコンポーネントの作成は完了です。しばらくは `Storybook` を使用するため、開発サーバーは終了して構いません。