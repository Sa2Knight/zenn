---
title: "[アドオン編] Viewport addon"
free: true
---

# Viewport addon について

`Viewport addon` は、`essentials addons` の一つで、 **コンポーネントを描画する際の画面サイズを任意に変更する** アドオンです。

モバイルファーストな時代である現代において、コンポーネントが画面サイズによって形を変えるレスポンシブな構成になることはよくあります。

そんな中で、一つのコンポーネントをある端末見た時は正常に描画されているが、別な画面サイズの端末で見ると崩れてしまっているということも多いでしょう。

`Viewport addon` を活用することによって、様々な画面サイズでコンポーネントを描画したり、画面サイズ別のストーリーを定義することができます。

![](https://storage.googleapis.com/zenn-user-upload/dnms93yab03xzi02ie97rc7yykys)

# Page コンポーネントのストーリーを作成する

本章では、 `Page` コンポーネントのストーリーを作成し、 `Viewport addon` を試していきます。

まずはざっと最低限だけ作っておきます。

```js:src/components/Page.stories.js
import Page from './Page'

export default {
  title: 'Page',
  component: Page
}

const Template = (args, { argTypes }) => ({
  props: Object.keys(argTypes),
  components: { Page },
  template: `<Page />`
})

export const Default = Template.bind({})
Default.args = {}
```

# 画面上から ViewPort を変更する

`Essentials addons` を有効化している場合、既に `Viewport addon` を使用できる状態になっています。

画面上部の Viewport アイコンをクリックすると、デフォルトの Viewport 一覧が表示されるので、いずれかをクリックすると、異なる画面サイズでコンポーネントが描画されるようになります。

![](https://storage.googleapis.com/zenn-user-upload/cwi2ivzckhca2rk4iggheysmr7rv)

# 使用する ViewPort のプリセットを変更する

デフォルトで使用できる ViewPort のプリセットには、以下の3種類が含まれています。

- Small mobile (320x568)
- Large mobile (414x896)
- Tablet (834x1112)

多くの場合はこの3種類があれば概ね充分そうですが、各種端末別の画面サイズで検証したいケースも考えられます。

そういった場合は、 `INITIAL_VIEWPORTS` というもう一つ用意されたプリセットを使用することができます。

使用するプリセットの設定は、 `.storybook/preview.js` に記述します。 `preview.js` は本書では初めて登場しますが、主にストーリーを描画方法に関する設定を宣言するファイルと思っていただければ大丈夫です。

以下は、 Storybook 全体で使用する Viewport のプリセットを、 `INITIAL_VIEWPORTS` に差し替えるコードになります。

```js:.storybook/preview.js
import { INITIAL_VIEWPORTS } from '@storybook/addon-viewport'

export const parameters = {
  viewport: {
    viewports: INITIAL_VIEWPORTS
  }
}
```

この状態で Storybook を再起動すると、以下のように端末別の ViewPort が選択できるようになります。

![](https://storage.googleapis.com/zenn-user-upload/fmuyoupeuclrmsza72x9e1t36x8a)

# カスタム Viewport を利用する

`INITIAL_VIEWPORTS` でもまだ足りないという場合は、任意の画面サイズに名前をつけたカスタム ViewPort を定義することができます。

カスタム ViewPort もプリセットと同様に、 `.storybook/preview.js` に以下のように記述します。

```js:.storybook/preview.js
export const parameters = {
  viewport: {
    viewports: {
      kindleFire2: {
        name: 'Kindle Fire 2',
        styles: {
          width: '600px',
          height: '963px'
        }
      },
      kindleFireHD: {
        name: 'Kindle Fire HD',
        styles: {
          width: '533px',
          height: '801px'
        }
      }
    }
  }
}
```

これで任意のサイズのViewPortを使用できるようになりました。

![](https://storage.googleapis.com/zenn-user-upload/bs2tmd5h4lcx1kctugpsqs939yzu)

# コンポーネントごとにデフォルトの ViewPort を設定する

PCでもスマートフォンでも利用できるサービスの場合、PCビューの場合のみ描画するコンポーネントや、逆にモバイルビュー専用のコンポーネントというものが出てくることがあります。そのような場合、モバイル専用のコンポーネントは、常にモバイルの画面サイズで検証するべきでしょう。

`Viewport addon` では、コンポーネントごとにデフォルトの ViewPort を設定することができます。

以下では、 `Header` コンポーネントのストーリーは全て、 `iphone6` の画面サイズで描画されるようになります。 (ツールバー上から変更することは可能です)

```js:src/components/Header.stories.js
import Header from './Header'
import { INITIAL_VIEWPORTS } from '@storybook/addon-viewport'

export default {
  title: 'Header',
  component: Header,
  parameters: {
    viewport: {
      viewports: INITIAL_VIEWPORTS,
      defaultViewport: 'iphone6'
    }
  }
}
```

# ストーリーごとにデフォルトの ViewPort を設定する

前項ではコンポーネントのストーリーを全て特定の ViewPort を使用するようにしましたが、画面サイズによって見た目や振る舞いが変わるコンポーネントの場合は、その数だけストーリーを定義し、ストーリーごとに ViewPort を設定したくなります。

以下では、 `Page` コンポーネントに、 `ForTablet` `ForSmartPhone` という2種類のストーリーを定義しており、それぞれに適した ViewPort を割り当てています。

```js:src/components/Page.stories.js
import Page from './Page'
export default {
  title: 'Page',
  component: Page
}

const Template = (args, { argTypes }) => ({
  props: Object.keys(argTypes),
  components: { Page },
  template: `<Page />`
})

export const ForTablet = Template.bind({})
ForTablet.parameters = {
  viewport: {
    defaultViewport: 'ipad'
  }
}

export const ForSmartPhone = Template.bind({})
ForSmartPhone.parameters = {
  viewport: {
    defaultViewport: 'iphone6'
  }
}
```

![](https://storage.googleapis.com/zenn-user-upload/bu91f03gzpse38thf3ycfr3pnvqv)

![](https://storage.googleapis.com/zenn-user-upload/xj2onry4jvpoj0404uw1o2dw3uq4)