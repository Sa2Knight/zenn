---
title: Storybook 6 との違い
---

`Storybook 7` は、 `Storybook 6` の後継にして、過去最大級のアップデートになります。

https://storybook.js.org/blog/7-0-beta/

上記ブログの内容を要約すると、主に以下のようなアップデートが入りました。

## デザイン刷新

描画パフォーマンスや開発体験向上を目的としたレイアウト変更が行われています。

## 本体のプリビルド

以前の `Storybook` では、 `Storybook` 本体側が [`webpack`](https://webpack.js.org/) (それも v4) に依存していました。そのため、 `Storybook` をインストールするだけで、もれなく `webpack` もインストールされることから、インストールサイズが膨大になっていました。

さらに、`Storybook` を起動するたびに `webpack` を用いて本体のビルドを行っており、これが大きなオーバーヘッドとなっていました。

`Storybook 7` では、プリビルド済みの本体が配布されるようになったため、パッケージサイズの大幅削減、初回起動の高速化が実現されています。

## MDX 2 サポート

`Storybook` では [`MDX`](https://github.com/mdx-js/mdx/) という、 Markdown に JSX を埋め込むフォーマットを用いて、コンポーネントのインタラクティブなドキュメントを作成できます。`Storybook 7` では、 `MDX` の最新バージョンである `MDX 2` がサポートされ、より高速で効果的なドキュメント作成を行えるようになりました。

## Jest と Testing Library を用いたインタラクションテスト

`Storybook` で描画したコンポーネントに対して、クリックやキー入力する自動テストを、[Jest](https://jestjs.io/ja/) と [Testing Library](https://testing-library.com/) を用いて実装できるようになります。これによって、`Storybook` をコンポーネントのカタログだけでなくテスト基盤としても使えるようになります。

## Playwright によるテストランナー

前述のインタラクションテストに加えて、[Playwright](https://github.com/microsoft/playwright) を用いたブラウザテストを、`Storybook` に対して行いやすくなりました。

## Component Story Format 3.0

`Storybook` では、コンポーネントを描画するためのストーリーを `CSF (Component Story Format)` の形式で作成します。 `CSF 3` はその最新バージョンで、これまで冗長気味だったボイラーテンプレートを大幅に削減し、より少ないコード量でストーリーを表現できるようになりました。

## フレームワークの統合

これまで、 [`Next.js`](https://nextjs.org/) や [`NuxtJS`](https://nuxtjs.org/) などのアプリケーションに `Storybook` を導入する場合は、個別の追加設定やサードパーティアドオンが必要でした。 `Storybook 7` ではこれを公式にサポートし、セットアップが簡略化されました。

## Vite のファーストクラスサポート

これまでの `Storybook` では、 `webpack` をビルダーとして使用してきましたが、今回から [`vite`](https://ja.vitejs.dev/) に役割を譲り、高速で軽量な `Storybook` を実現しました。