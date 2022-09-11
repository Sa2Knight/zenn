---
title: "[アドオン編] Docs addon"
free: false
---

# Docs addon について

`Docs addon` は、`essentials addons` の一つで、 **Storybook上で、コンポーネントのドキュメンテーションを提供する** アドオンです。

これまでの章をハンズオンしていた方であれば、 `Docs` というタブに目を一度は開いていたかもしれませんが、 `Docs addon` では、**ドキュメンテーションを個別に記述しなくても、ストーリーの定義を元に自動で生成してくれる**仕組みがあります。

以下は `Button` コンポーネントのストーリーの `Docs` タブを開いた際の画面ですが、一つのドキュメント内で、コンポーネントのインタフェースと、使用例(個別のストーリー) が表示されていることがわかります。

![](https://storage.googleapis.com/zenn-user-upload/lrbajocev81c751ygvbiksi2q1vd)

# props と event の情報を追記する

現状では、 ドキュメント内に `props` と `event` が記述されていますが、 `description` が空になっていたり、 event の型が `unknown` になっていたりと不十分です。

ここの情報は `vue-docgen-api` によってソースコードを解析したものが表示されるので、 `vue-docgen-api` が理解できるようなコードを記述することで埋めることができます。

`vue-docgen-api` は、 `JSDoc` を使ったコードのタグ付が行えるので、コンポーネント側のコードを改修し、 `description` を記述してみましょう。

```html:src/components/Button.vue
<template>
  <button
    type="button"
    :class="['button', { primary, secondary }, size]"
    @click="onClick"
    :style="style"
    v-text="label"
  />
</template>

<script>
export default {
  props: {
    /**
     * ボタン内に表示するテキスト
     */
    label: {
      type: String,
      required: true
    },
    /**
     * プライマリーカラーを適用するか
     */
    primary: {
      type: Boolean,
      default: false
    },
    /**
     * セカンダリーカラーを適用するか
     */
    secondary: {
      type: Boolean,
      default: false
    },
    /**
     * ボタンのサイズ
     * @values small, medium, large
     */
    size: {
      type: String,
      default: 'medium',
      validator: function(value) {
        return ['small', 'medium', 'large'].indexOf(value) !== -1
      }
    },
    backgroundColor: {
      type: String,
      required: false
    }
  },
  computed: {
    style() {
      return {
        backgroundColor: this.backgroundColor
      }
    }
  },
  methods: {
    onClick() {
      /**
       * ボタンクリック時のイベント
       * @event onClick
       * @type {object}
       */
      this.$emit('onClick')
    }
  }
}
</script>

<style lang="scss" scoped>
  // 変更なし
</style>
```

JSDoc 形式で、各 props/event に説明を追記することで、Storybook 上にもそれが反映されていることが確認できました。

![](https://storage.googleapis.com/zenn-user-upload/0vkziduk6fkv3ujhpws9tjkwg24j)