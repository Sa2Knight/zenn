---
title: "memlab を使って Web サイトのメモリリークを検出しよう"
emoji: "👌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["javascript", "memlab"]
published: true
---

# 概要

本記事は、メタ社(旧 FaceBook) が開発した OSS であるメモリリーク検知ツールである [memlab] をさっそく試してみた記録になります。

公式ドキュメント以上の付加価値はあまりありませんが、ざっくりと雰囲気を掴んでもらって使用を検討して頂ければ幸いです。

# memlabについて

`memlab` は、 [Puppeteer](https://pptr.dev/) API を用いたシナリオを作成することで、そのシナリオ実行によって発生するメモリリークの検出及びヒープ領域の分析を補助してくれるツールです。

https://facebookincubator.github.io/memlab/

本記事は実際に動かすところに重きを置くので、ツールの背景などの詳細は以下記事を参照ください。

https://www.publickey1.jp/blog/22/javascriptmemlab.html

ざっくり言うと、 `memlab` では以下のことが行なえます。

- `Puppeteer` ベースでの宣言的なシナリオの作成
- ヒープ領域とメモリリークの可視化
  - シナリオ内でのヒープのスナップショットの自動取得
  - スナップショットの解析とメモリリークのフィルタリング
  - 同種のメモリリークのグルーピングと集約
  - メモリリーク発生に至るまでのトレースの生成
- メモリリークのフィルタリングルールのカスタマイズ
- API を用いたヒープメモリへの簡易的なアクセス
- CLI を用いたレポートの参照

SPA におけるメモリリークのテストはブラウザのデベロッパーツールを活用するような複雑で困難なものが多いですが、それを簡易的に自動化できるツールと考えて頂ければ大丈夫です。

# インストール

ここではドキュメントに従ってグローバルインストールしておきますが、ローカルインストールでも問題ないと思います。

```bash
npm install -g memlab
```

```bash
$ memlab version
 memlab@1.1.27
 @memlab/heap-analysis@1.0.9
 @memlab/e2e@1.0.11
 @memlab/core@1.1.10
 @memlab/cli@1.0.11
 @memlab/api@1.0.11
```

# サンプルコード

まず、サンプルコードとして [YouTube](https://www.youtube.com/) にアクセスし、トップページから任意の動画を再生する導線を検証するコードを紹介します。

*以下コードは `memlab` のサンプルコードであり、実際の `YouTube` におけるメモリリークの検出として適切なコードではありません*

```js:01_hello_memlab.js
// 最初に読み込む URL (1)
function url() {
  return "https://www.youtube.com";
}

// メモリリークの有無を検出するための任意のアクション (2)
async function action(page) {
  await page.click('[id="video-title-link"]');
}

// アクション完了後に、初期状態に戻るための操作 (3)
async function back(page) {
  await page.click('[id="logo-icon"]');
}

// 少なくとも上記3種はエクスポートする
module.exports = { url, action, back };
```

`memlab` のシナリオコードでは、3種類の関数をエクスポートします。

**1. 読み込むURL**

シナリオ実行時に最初に開くURLを返す関数です。

ここでは `YouTube` のトップページの URL を返します。

**2. メモリリークを検出するための任意のアクション**

メモリリークが発生する可能性がある操作を `Puppeteer` API を用いて記述します。

ここでは先頭の動画へのリンクである要素(`#video-title-link`) を押下することで、動画再生ページに遷移します。

**3. アクション実行後に初期状態に戻るための手続き**

(2) でアクション完了後に、初期状態(トップページ)に戻るための手続きを `Puppeteer` API を用いて記述します。

ここでは (2) で動画再生画面に遷移していたので、 `YouTube` のロゴである　`#logo-icon` を押下することでトップページに戻ります。


以上、 (1)(2)(3) をエクスポートするモジュールを作成し、それを `memlab` で実行します。

# サンプルコードを実行する

前述のサンプルコードを、 `memlab run` コマンドで実行すると、以下のようにメモリリークが検出されます。

![](https://storage.googleapis.com/zenn-user-upload/83590d13ebfd-20220915.png)

特筆すべきは先頭行の

`page-load[26MB](baseline)[s1] > action-on-page[33.5MB](target)[s2] > revert[33MB](final)[s3]`

の部分です。これはそれぞれサンプルコードにおける `url` `action` `back` に該当するもので

- s1: `url` の返り値を元にサイトを開いた時点でのヒープ領域の使用量
- s2: `action` を実行した後のヒープ領域の使用量
- s3: `back` を実行した後のヒープ領域の使用量

を表し、 s1, s2, s3 それぞれのヒープ領域のスナップショットを元に、メモリリークが発生するかを判定しています。

ここでは `YouTube` がメモリリークを起こしているか、その検出方法が正しいかは重要ではないので、具体的な結果には触れずに、サンプルアプリケーションで再度試してみます。

# DOM によるメモリリークを検出する

本項では、`memlab` のモノレポで公開されているサンプルアプリケーションを使ってメモリリークの検出を体験します。

## サンプルアプリケーションのセットアップ

以下モノレポから、サンプルアプリケーションのセットアップを行います。

https://github.com/facebookincubator/memlab/tree/main/packages/e2e/static/example

```bash
$ git clone git@github.com:facebookincubator/memlab.git
$ cd memlab
$ npm install && npm run build
$ cd packages/e2e/static/example
$ npm install && npm run dev
```

モノレポなのもあって、2回インストール作業が必要ですが、これで `localhost:3000` でサンプルアプリケーションが立ち上がりました。

そこに対して `memlab` でメモリリークの検出を行ってみましょう。

## サンプルアプリケーションを確認する

`localhost:3000` で稼働しているサンプルアプリケーションのうち、`/examples/detached-dom` のページをテストします。

![](https://storage.googleapis.com/zenn-user-upload/5137a5e42009-20220915.png)

このページには `Create detached DOMs` というボタンがありますが、このボタンは押下するたびに、どこからも参照されない `div` 要素を 1024個 `window` オブジェクトに対して突っ込むという凶悪なコードになっています。

```jsx
export default function DetachedDom() {
  const addNewItem = () => {
    if (!window.leakedObjects) {
      window.leakedObjects = [];
    }
    // 凶悪なメモリリークを引き起こすコード
    for (let i = 0; i < 1024; i++) {
      window.leakedObjects.push(document.createElement('div'));
    }
    console.log('Detached DOMs are created. Please check Memory tab in devtools')
  };
  return (
    <div className="container">
      <div className="row">
        <Link href="/">Go back</Link>
      </div>
      <br />
      <div className="row">
        <button type="button" className="btn" onClick={addNewItem}>
          Create detached DOMs
        </button>
      </div>
    </div>
  );
}
```

言わずもがな、このページはボタンを押下すればするほど不要な DOM が生成され、ヒープ領域を逼迫しつづけます。

この事実を `memlab` で検出してみましょう。

## シナリオを作成する

`memlab` におけるテストシナリオの書き方は `YouTube` を使用した例と同様なので詳細は割愛しますが、以下のようになります。

```js
function url() {
  return "http://localhost:3000/examples/detached-dom";
}

async function action(page) {
  const elements = await page.$x("//button[contains(., 'Create detached DOMs')]");
  const [button] = elements;
  if (button) {
    await button.click();
  }
  await Promise.all(elements.map((e) => e.dispose())); // Puppeteer で確保しているオブジェクト開放する
}

async function back(page) {
  await page.click('a[href="/"]');
}

module.exports = { action, back, url };
```

先程の s1, s2, s3 に対応する処理が以下になります。

- s1: `localhost:3000/examples/detached-dom` にアクセスし
- s2: `Create detached DOMs` ボタンをクリック
- s3: トップページへのリンクをクリック

## メモリリークを検出する

前述のシナリオを `memlab` で実行します。

```bash
memlab run --scenario 02_find_memory_leaks.js
```

![](https://storage.googleapis.com/zenn-user-upload/495797a71ff8-20220915.png)

実行結果から、以下のことがわかりました。

- 1024 オブジェクトがリークしている (実際に1204回ループして `div` を生成しているため)
- リークしたオブジェクトのサイズは 143.3KB である
- リークしたオブジェクトは `window.leakedObjects` 内の `HTMLDivElement` である

## メモリリークを修正する

`memlab` によって検知されたメモリリークからも原因が特定できました。 `window.leakedObjects` に DOM をプッシュする処理は完全に不要であるため、コメントアウトしてしまいましょう。

```jsx
  const addNewItem = () => {
    // if (!window.leakedObjects) {
    //   window.leakedObjects = [];
    // }
    // for (let i = 0; i < 10000; i++) {
    //   window.leakedObjects.push(document.createElement('div'));
    // }
    // console.log('Detached DOMs are created. Please check Memory tab in devtools')
  };
```

これで再実行すると、無事にメモリリークの検出がなくなり、修正に成功しました。

![](https://storage.googleapis.com/zenn-user-upload/5189ec95d340-20220915.png)

# デフォルトで検出されないメモリリークを検出する

前項においては、 `window` オブジェクトからの参照が残ってしまった `div` 要素がメモリリークとして検出されました。

`memlab` はどういった基準でメモリリークを判断しているのでしょうか。

デフォルトでは、あるオブジェクトが以下の基準を満たした場合のみ、メモリリークとして検出しています。

- オブジェクトは action 関数でトリガーされたインタラクションによって割り当てられている
- オブジェクトは back 関数実行後も開放されていない
- オブジェクトは切り離された DOM 要素またはアンマウントされた React Fiber ノードである

前項の例では、 `action` 関数での操作(ボタンクリック)によって DOM 要素が生成され、それが `back` 関数実行後も開放されていなかったことから、メモリリークと判定されました。

ではそれ以外のパターンとしてどういったものが考えられるか、それをどう対処するのかの例を紹介します。

## サンプルアプリケーションを確認する

サンプルアプリケーション自体は前回の例と同じものを使用するので、引き続き `localhost:3000` で動いているものを使用します。

今回は `/examples/oversized-object` に該当するページが対象となります。

![](https://storage.googleapis.com/zenn-user-upload/8e67d5f38d7e-20220915.png)

本ページのコードは以下のようになっており、巨大オブジェクト(`bigArray`) に依存したイベントハンドラを、元にイベントの購読を行っています。

*コードに React の文脈を含みますが、ご存知でない方は雰囲気で読んでください*

```jsx
export default function OversizedObject() {
  // 巨大オブジェクト
  const bigArray = Array(1024 * 1024 * 2).fill(0);

  // 巨大オブジェクトに依存したイベントハンドラ
  const eventHandler = () => {
    console.log('Using hugeObject', bigArray);
  };

  // イベントの購読 (解除を忘れている)
  useEffect(() => {
    window.addEventListener('custom-click', eventHandler);

  }, []);
  return (
    <div className="container">
      <div className="row">
        <Link href="/">Go back</Link>
      </div>
      <br/>
      <div className="row">
        Object<code>bigArray</code>is leaked. Please check <code>Memory</code>{' '}
        tab in devtools
      </div>
    </div>
  );
}
```

ここでは `useEffect` を用いて、ページ読み込み時にコールバックでイベントを購読していますが、その解除を適切に行なっていないため、ページを行き来するたびにイベントがどんどん登録され、その分だけ巨大オブジェクトがヒープ領域に展開され続けます。

これは言わずもがなメモリリークであるため、 `memlab` で検出してみましょう。

## シナリオを作成する

シナリオを書くのも3度目なので、掲載のみで説明は割愛します。

```js
function url() {
  return "http://localhost:3000/";
}
// action where you suspect the memory leak might be happening
async function action(page) {
  await page.click('a[href="/examples/oversized-object"]');
}
// how to go back to the state before action
async function back(page) {
  await page.click('a[href="/"]');
}
module.exports = { action, back, url };
```

## メモリリークを検出する(失敗例)

```bash
$ memlab run --scenario 03_find_oversiz3ed_object.j
```

![](https://storage.googleapis.com/zenn-user-upload/376fbb615f89-20220915.png)

グラフからメモリ使用量が戻っていないことがわかるのに、メモリリークとして検出されていません！！

これだけ明らかなのに検出されないのは、前述のデフォルトでのメモリリーク検出基準を満たしていないからです。

ルールをもう一度見てみましょう。

- オブジェクトは action 関数でトリガーされたインタラクションによって割り当てられている
- オブジェクトは back 関数実行後も開放されていない
- オブジェクトは切り離された DOM 要素またはアンマウントされた React Fiber ノードである

今回の場合、ページを開いた時点で `useEffect` によって発生する副作用だったため、 `action` 関数からトレースできていないし、そもそも `DOM` 要素でなくプリミティブな `Array` であるため対象外という結果になりました。

では `memlab` はこの程度のメモリリークも検出できないのでしょうか？

そんなことはなく、多くの方法でメモリリークを分析、検出することができます。

今回はその中から、 `leakFilter` を定義するパターンを試してみましょう。

## `leakFilter` を実装する

`leakFilter` 関数は、シナリオコード(`url` `action` `back` を定義したモジュール) に追加する形で実装する、「それがメモリリークであるかの判断を行う関数」です。

以下のように、怪しいノード及びスナップショット情報が関数に渡されるので、今回はサイズが 1MB 以上であればアウトにします。

```js
// 前略

function leakFilter(node, _snapshot, _leakedNodeIds) {
  return node.retainedSize > 1000 * 1000;
}

module.exports = { action, back, url, leakFilter };
```

## メモリリークを検出する(リベンジ)

`leakFilter` が実装されたことで、デフォルトの検出ルールに加えて、サイズが 1MB 以上のノードが残っていたらアウトという判定になりました。

これで再度 `memlab` を実行します。

![](https://storage.googleapis.com/zenn-user-upload/a477296cb237-20220915.png)

見事メモリリークを検出し、8.3MB の `bigArray` が `eventHandler` 内にあることを教えてくれました。

## メモリリークを修正する

検出されたメモリリークを修正します。

ページ離脱時にイベントの購読を解除しなかったことが原因であるため、 `useEffect` でクリーンアップ関数を書いてあげましょう。

```jsx
  useEffect(() => {
    // 購読する
    window.addEventListener('custom-click', eventHandler);

    return () => {
      // 解除する
      window.removeEventListener('custom-click', eventHandler);
    }
  }, []);
```

これで最後 `memlab` を実行すると、メモリリークが修正されたことを確認できました。

![](https://storage.googleapis.com/zenn-user-upload/536c38b06c65-20220915.png)

グラフからも、トップページに戻ることでメモリが開放されていることが可視化されています。

# まとめ

本記事では、メタ社(旧 FaceBook) が開発した OSS である memlab をさっそく試し、いくつかのメモリリークの検出と、その修正までを試しました。

個人的にはメモリリークの基準を自分で定義して分析するというのは、ブラウザの低レイヤーでのヒープの振る舞いを意識する良いキッカケになったので、これを活用してパフォーマンスの良いサービスを作れるようになれればと思います。

`memlab` は記事執筆時点でリリースされたばかりの新鋭ツールであるため、今後どんどん強化されたり、ノウハウの記事が増えていったりすると思うので、今後の発展も非常に楽しみにしています。
