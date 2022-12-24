---
title: "Storybook とは"
---

`Storybook` は UI 開発のためのツールの一種で、**アプリケーションから分離したサンドボックス環境を用いて UI コンポーネントを開発する**ことが主な用途です。

これによって、本来必要となる開発環境や、サーバーサイド、データベースなどを用意することなく、素早くコンポーネント開発に集中できます。

また、作成した Storybook は**ビルドしてチーム間で共有したり、自動テストに使用するなど、フロントエンド開発全体の中心にもなり得る**ツールとなっています。

Storybook に初めて触れる方は、百聞は一見に如かずということで、[GitLab](https://about.gitlab.com/) が公開している [gitlab-ui](https://gitlab.com/gitlab-org/gitlab-ui) の Storybook を触ってみましょう。

https://gitlab-org.gitlab.io/gitlab-ui/

まずは、なんとなく Storybook がどのようなツールなのかのイメージが出来ていれば OK です。本書ではハンズオン形式で少しずつ Storybook (+ Vue 3) の世界に踏み込んでいきます。
