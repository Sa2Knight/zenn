---
title: アドオンについて
---

`アドオン` は、 `Storybook` 本体からは分離されているものの、統合することで `Storybook` をさらに拡張するパッケージのことです。

`Storybook` の主要機能と言えるものも、ほとんどが公式アドオンの形式で配布されています。

公式アドオンの他にもサードパティのアドオンが多数配布されており、自作も可能です。

アドオンの一覧は以下から確認できます。

https://storybook.js.org/integrations

本書でも必要に応じてアドオンの使用しながらハンズオンを進めますが、使用するアドオンはほんの一握りです。

多くのアドオンは特定のユースケースでの課題を解決するための仕組みになっているため、 `Storybook` でこういうことがしたいと思ったら、アドオンを調べてみるのが良いでしょう。

# Essential addons

`Storybook` には、公式が開発した必須アドオンである、 `Essential addons` が用意されています。これは公式が推奨する７種のアドオンのプリセットであり、以下で構成されています。

|アドオン名|機能|
|----|----
|Docs|ストーリーからドキュメントを自動生成
|Controls|パラメータ(≒`props`)を GUI 上で動的に切り替える
|Actions|イベント(≒`emit`)をロギングする
|Viewport|画面サイズを変更する
|Backgrounds|背景色を変更する
|Toolbars & globals|ストーリーを跨いだグローバルパラメータの管理
|Measure & outline|CSSレイアウトのビジュアルデバッグ
|Highlight|描画したDOMをハイライトする

`Essential addons` はプリセットであるため、 `@storybook/addon-essentials` をインストールするだけですべてのアドオンがインストール、セットアップされます。

しかし、必須アドオンと言いつつも実際は使わないアドオンが含まれていることも多いため、本書では必要に応じて個別にアドオンをインストール、設定する方式で進めていきます。

# 関連リンク

https://storybook.js.org/docs/7.0/vue/addons/introduction
https://storybook.js.org/docs/7.0/vue/essentials/introduction