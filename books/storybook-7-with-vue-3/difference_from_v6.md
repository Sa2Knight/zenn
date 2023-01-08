---
title: Storybook 6 との違い
free: true
---

`Storybook 7` は、 `Storybook 6` の後継にして、過去最大級のアップデートと言われています。

https://storybook.js.org/blog/7-0-beta/

上記ブログを要約すると、以下のようなアップデートが入りました。

## デザイン刷新

描画パフォーマンスや開発体験向上を目的としたレイアウト変更が行われています。

## 本体のプリビルド

以前の `Storybook` では、 `Storybook` 本体側が [`Webpack`](https://webpack.js.org/) (それも v4) に依存していました。そのため、 `Storybook` をインストールするだけで、もれなく `Webpack` もインストールされることから、インストールサイズが膨大で時間もかかる問題がありました。

さらに、`Storybook` を起動するたびに `Webpack` を用いて本体のビルドも同時に行われるため、これが大きなオーバーヘッドとなっていました。

`Storybook 7` では、プリビルド済みの本体が配布されるようになったため、パッケージサイズの大幅削減、初回起動の高速化が実現されています。

## MDX 2 サポート

`Storybook` では [`MDX`](https://github.com/mdx-js/mdx/) という、 `markdown` に `JSX` を埋め込むフォーマットを用いて、動作するコンポーネント付きのドキュメントを作成できます。`Storybook 7` では、 `MDX` の最新バージョンである `MDX 2` がサポートされ、より高速で効果的なドキュメント作成を行えるようになりました。

## Jest と Testing Library を用いたインタラクションテスト

`Storybook` で描画したコンポーネントに対して、クリックやキー入力する自動テストを、[Jest](https://jestjs.io/ja/) と [Testing Library](https://testing-library.com/) を用いて実装できるようになります。これによって、`Storybook` をコンポーネントカタログだけでなくテスト基盤としても使えるようになります。

## Playwright によるテストランナー

前述のインタラクションテストに加えて、[Playwright](https://github.com/microsoft/playwright) を用いたブラウザテストを、`Storybook` に対して行いやすくなりました。

## Component Story Format 3.0

`Storybook` では、コンポーネントを描画するストーリーを、`CSF (Component Story Format)` の形式で作成します。 `CSF 3` はその最新バージョンで、これまで冗長気味だったボイラーテンプレートを大幅に削減し、より少ないコード量でストーリーを表現できるようになりました。

なお、 `CSF` は純粋な JavaScript (`.js`) や TypeScript (`.ts`) で定義するため、 `Storybook` 以外からも利用可能です。

## フレームワークの統合

これまで、 [`Next.js`](https://nextjs.org/) や [`NuxtJS`](https://nuxtjs.org/) などのアプリケーションに `Storybook` を導入する場合は、個別の追加設定やサードパーティアドオンが必要でした。 `Storybook 7` ではこれを公式にサポートし、セットアップが簡略化されました。

## Vite のファーストクラスサポート

これまでの `Storybook` では、 `Webpack` をビルダーとして使用してきましたが、今回から [`Vite`](https://ja.vitejs.dev/) に役割を譲り、高速で軽量な `Storybook` を実現しました。