---
title: ストーリーをラップする
free: true
---

# Decorator について

`Decorator` は、ストーリーとは別のコンポーネントを用いてストーリーをラップする機能です。

平たく言えばストーリーの親コンポーネントを定義できるようになるため、ストーリーのレイアウトを整えたり、各種セットアップを一元化などができます。

# グローバル Decorator を定義する

`Decorator` は `Parameter` と同様、コンポーネントやストーリーごとに定義も出来ますが、 `preview.js` でグローバル Decorator を定義することが多いです。

ここではシンプルに、ストーリーの周囲にマージンを設けるデコレータを定義します。

```ts:.storybook/preview.ts
import { Decorator } from "@storybook/vue3";

export const decorators: Decorator[] = [
  () => {
    return {
      template:
        '<div style="margin: 5em; border: 1px solid; border-color: white"><story /></div>',
    };
  },
];
```

`decorators` には、複数の `Decorator` を配列で指定しネストできます。

ここでは単に `<story />` を `margin` と `border` を設定した `<div>` でラップするだけの `decorator` を用意しました。

大した意味はありませんが、すべてのストーリーに `margin` と `border` が付与されるようになりました。

![](https://storage.googleapis.com/zenn-user-upload/3c02a8632a4f-20221227.png)

この例だけだとイマイチ使いみちが見えないですが、 `Decorator` はストーリーが内部的に依存するコンテキストをセットアップする際に非常に役立ちます。

本書でも `Decorator` を活用する例が登場するため、ここではラッパーコンポーネントを作れるという理解をしていただければ大丈夫です。

# 関連リンク

https://storybook.js.org/docs/7.0/vue/writing-stories/decorators