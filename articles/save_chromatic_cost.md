---
title: "Chromatic の実行を必要最小限にしてコストを削減する"
emoji: "⏰️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Chromatic", "GitHub Actions"]
published: true
---

# 概要

GitHub Actions で Chromatic Visual testing (VRT) の実行タイミングを最適化し、課金額を半分以下に削減した事例を紹介します。

# 背景

Chromatic は VRT の導入を簡単にでき、メンテナンスコストも低い便利なツールですが、その利便性と引き換えに、撮影したスクリーンショットの枚数に応じて従量課金が発生します。

この従量課金は比較的高額で、特に頻繁にテストを実行するプロジェクトでは費用が急増する可能性があります。

私が担当しているプロダクトでも、Chromatic を導入した初月から予想以上の請求額となり、急遽コスト削減の対策が必要となりました。

# 前提

- GitHub Actions を使用して Chromatic へのデプロイを行っている
- Chromatic をパスしていることがブランチ保護ルールに設定されている
- バックエンドとフロントエンドが混在したリポジトリだが、フロントエンドのコードは `client/` ディレクトリ以下に切り出されている

# 改善前のワークフロー

最初に Chromatic を導入した際は、すべてのコードプッシュに対してテストを実行する設定にしていました。

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

その結果、初月から非常に高額な請求が発生し、早急の改善が求められました。

# PR の状態で制限する

Chromatic の主な目的は、デフォルトブランチ (master) へのマージ前にデグレを防ぐことです。そのため、PR がある場合のみ実行するように設定しました。

```yml
on:
  pull_request:
    types: [opened, reopened, synchronize, ready_for_review]
```

また、PR がドラフト状態や、クローズ済みの場合はマージされることがないためスキップします。

```yml
jobs:
  deploy:
    runs-on: ubuntu-latest
    if: |
      github.event.pull_request.draft == false && github.event.pull_request.state == 'open'
```

PR のドラフトが解除されたり、再オープンされた場合には `ready_for_review` や `reopened` がトリガーとなって実行されるため、もれなく対応できます。

# PR にフロントエンドの差分がない場合はスキップ

VRT の差分はフロントエンド関連のコードに変更があった場合にのみ発生します。したがって、バックエンドのみ変更した PR での実行は不要です。

そこで、PR のコード差分から、クライアントコードが含まれていないかを確認します。

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
