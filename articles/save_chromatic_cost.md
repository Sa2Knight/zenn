---
title: "Chromatic の実行を必要最小限にしてコストを削減する"
emoji: "⏰️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Chromatic", "GitHub Actions"]
published: false
---

# 概要

Chromatic による Visual testing (a.k.a VRT) を、GitHub Actions から都度実行しているプロジェクトにて、その実行タイミングを必要十分になるように制限することで課金額を半分未満にまで削減した事例です。

# 背景

Chromatic は VRT の導入をコンフィグレスで即座にでき、メンテコストも低い便利なツールです。

ただ撮影したスクリーンショット枚数に応じる従量課金額は(個人の感覚ですが)非常に高額です。

私が担当しているプロダクトでも、無邪気に Chromatic を導入した結果、これはまずいというレベル請求になったため、急遽コスト削減の取組みを行いました。

# 前提

- GitHub Actions を使用して Chromatic へのデプロイを行っている
- リポジトリでは Chromatic をパスしていることが、PR をマージに必須
- バックエンドとフロントエンドが混在したリポジトリだが、フロントエンドのコードは `client/` ディレクトリ以下に切り出されている

# 改善前のワークフロー

最初に Chromatic を導入した際のワークフローは以下のようなものでした。

```yml
name: Chromatic Deployment

on:
  - push
  - pull_request

jobs:
  deploy:
    runs-on: ubuntu-latest
    env:
      CHROMATIC_PROJECT_TOKEN: ${{ secrets.CHROMATIC_PROJECT_TOKEN }}
    steps:
      # (環境セットアップのステップは省略)

      - name: Run Chromatic
        uses: chromaui/action@latest
        with:
          projectToken: ${{ secrets.CHROMATIC_PROJECT_TOKEN }}
          onlyChanged: true
          exitZeroOnChanges: true
```

無邪気にすべてのコードプッシュで Chromatic を実行しているため、初月からとんでもない請求が来てしまいました。

流石にまずいので、明らかに実行する必要がない場合には実行しないように改善を重ねました。

# PR の状態で制限する

まず、Chromatic はリリース対象となる master ブランチへのコードマージ前にデグレを検知する役目を果たしてくれば良いので、プルリクエストがある場合のみをトリガーにします。

```yml
on:
  pull_request:
    types: [opened, reopened, synchronize, ready_for_review]
```

また、PR がドラフト状態の場合や、クローズ済みの PR はマージされる心配がないためスキップします。

```yml
jobs:
  deploy:
    runs-on: ubuntu-latest
    if: |
      github.event.pull_request.draft == false && github.event.pull_request.state == 'open'
```

PR がドラフトからオープンされた場合には `ready_for_review` がトリガーとなって実行されます。

# PR にフロントエンドの差分がない場合はスキップ

VRT の差分は、原理上フロントエンド関連のコードに差分があるときしか発生しません。バックエンドのみ変更した PR で Chromatic を実行するのは完全に無意味です。

そこで、PR のベースブランチとの差分にクライアントコードが含まれていないかを確認します。

```yml
- name: Check for frontend changes
  id: check_changes
  run: |
    # PRにフロント差分が含まれていなければスキップする
    CHANGED_FILES=$(gh pr diff ${{ github.event.pull_request.number }} --name-only | grep -E '^client/|package.json|pnpm-lock.yaml|^\.storybook/' || true)
    echo "$CHANGED_FILES"
    if [ -z "$CHANGED_FILES" ]; then
      echo "skip=true" >> $GITHUB_OUTPUT
    else
      echo "skip=false" >> $GITHUB_OUTPUT
    fi
```

クライアントコードは基本的に `client/` ディレクトリに集約されているので、それに加えて `package.json` など、VRT に影響を与えるパスをチェック対象としています。

クライアントコードの差分の有無を元に、Chromatic の Skip オプションを有効化します。

```yml
- name: Run Chromatic
  uses: chromaui/action@latest
  with:
    projectToken: ${{ secrets.CHROMATIC_PROJECT_TOKEN }}
    onlyChanged: true
    exitZeroOnChanges: true
    skip: ${{ steps.check_changes.outputs.skip }} # これを追加
```

これは Chromatic の実行自体はスキップするが、コミットステータス上では成功として通知してくれる便利なオプションです。

これによって、実際は Chromatic を実行していなくても、ブランチ保護ルールを突破して PR をマージできるようになります。

# renovate/dependabot でも動かせるようにする

renovate/dependabot が作成した、ライブラリバージョンアップの PR でも、master ブランチにマージさせるには Chromatic を通す必要があります。

しかし、bot ユーザーが作成した PR では、以下の GitHub Secrets を参照できないという問題がありました。

```yml
env:
  CHROMATIC_PROJECT_TOKEN: ${{ secrets.CHROMATIC_PROJECT_TOKEN }}
  GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

これはいくつかの方法で回避可能ですが、今回は最も簡単な、`pull_request_target` をトリガーとする方法を採用しました。

```diff
on:
-  pull_request:
+  pull_request_target:
    types: [opened, reopened, synchronize, ready_for_review]
```

これによって、ワークフローを実行するコンテキストがベースブランチ側となり、secrets にアクセスできるようになります。

ただし、ワークフローが実行されるブランチもベースブランチ(≒master)となってしまうため、そのままだと意図した VRT 結果を得ることができません。

そこで、コードをチェックアウトするステップで、プルリクエストを作成したブランチ側のコミットをチェクアウトするように明示しました。

```yml
steps:
  - name: Checkout code
    uses: "actions/checkout@4"
    with:
      fetch-depth: 0
      ref: ${{ github.event.pull_request.head.sha }}
```

これで renovate/dependabot が作成した PR でも、ライブラリバージョンアップ後のコードを使って VRT を行えるようになりました。

ちなみに dependabot のほうは npm のバージョンアップを行っていないため、常にフロントエンドの差分が無い判定となりスキップされます。

# その他

本記事では、GitHub Actions での実行条件の見直しについて紹介しましたが、Chromatic のコスト削減のために、他にも以下のような取り組みもしました。

- VRT 専用のストーリーを用意し、通常のストーリーを VRT 対象外にする
- renovate の設定で、複数パッケージをまとめてバージョンアップするようにし、PR 数自体を減らす
- [TurboSnap](https://www.chromatic.com/docs/turbosnap/)を有効化し、キャッシュを活用する
