---
title: サンプルコンポーネントの作成
---

`Storybook` を導入する前に、`Storybook` で描画するためのサンプルコンポーネントを作成します。

`src/components/Counter.vue` を作成し、以下のようなカウンターコンポーネントを実装します。

```js:src/components/Counter.vue
<script setup lang="ts">
import { ref } from "vue";

const props = defineProps({
  initialCount: {
    type: Number,
    default: 0,
  },
  min: {
    type: Number,
    default: 0,
  },
  max: {
    type: Number,
    default: 10,
  },
});

const count = ref(props.initialCount);

function updateCount(newCount: number) {
  if (newCount >= props.min && newCount <= props.max) {
    count.value = newCount;
  }
}
</script>

<template>
  <div class="counter">
    <button @click="updateCount(count - 1)">-</button>
    <span class="value">{{ count }}</span>
    <button @click="updateCount(count + 1)">+</button>
  </div>
</template>
```

上記は `<script setup>` や `TypeScript` を使用した比較的モダンな Vue コンポーネントですが、実装スタイルに深い意味はありませんので、不慣れであっても構わず進めてください。

以下のような、上限・下限を持ったカウンターコンポーネントが出来上がりました。次はこのコンポーネントを `Storybook` で描画するためのセットアップを行います。

![](https://storage.googleapis.com/zenn-user-upload/9e662d28bdae-20221224.png)