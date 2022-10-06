---
title: "axios は v1.0.0 でどう変わるのか"
emoji: "👏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["javascript", "typescript"]
published: false
---

# 概要

本記事は、HTTP クライアントライブラリである [axios](https://github.com/axios/axios)の [v1.0.0](https://github.com/axios/axios/releases/tag/v1.0.0) が満を持してリリースされたため、何がどう変わったのか、マイグレーションしても良いのかについて個人的に調べてまとめた結果になります。

# TL;DR

- `axios` のモダン化に向けた節目としての `v1.0.0`
- `v1.0.0` では多数のバグ修正と、いくつかの小規模の機能追加がまとめて取り込まれた
- 破壊的変更は少なからずあるが、基本的な使い方や挙動を大きく変える規模の変更はない
- 公式マイグレーションガイドは記事執筆時点では提供されていない
- 不具合報告(特に `TypeScript` への対応漏れ)が多く上がっているため、ご利用は慎重に

# `axios` について

[axios](https://github.com/axios/axios) は、`JavaScript` 向けの HTTP クライアントライブラリの一種で、この種のパッケージとしては比較的古くから普及している老舗ライブラリです。

大きな特徴として、扱いが難しい [XMLHttpRequest](https://developer.mozilla.org/ja/docs/Web/API/XMLHttpRequest) を `Promise` でラップして使いやすくした上、 `Node.js` で実行する場合は `fetch` API に切り替えることで、同じコードとブラウザでも `Node` でも動くようにしたことです。

```js
axios.post('/user', {
    firstName: 'Fred',
    lastName: 'Flintstone'
  })
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  });
```

またリクエスト・レスポンスを都度改変するための `Interceptors` や、リクエスト・レスポンスデータを加工するフックを挟む `Transform` などのヘルパー機能も多く搭載されています。

# なぜ今になって `v1.0.0` なのか

本パッケージは2016年という、`ES2015` で `Promise` が導入されてまもない頃にリリースされ、デファクトスタンダードに近い立ち位置を得たため、多くの方が一度は使ったことあるのではないでしょうか。

その割には今になっての `v1.0.0` なので、 **「axios って一生 v1.0.0 にならないのかと思ってた」** **「というかまだ 0.x 系だったの知らなかった」** という人もいると思います。

以下 issue 及び discussions にて、今後 `axios` をどうしていきたいかが議論されており、モダン JavaScript のプラクティスに則ったライブラリを目指すための一環としての `v1.0.0` とされています。

https://github.com/axios/axios/issues/4209
https://github.com/axios/axios/discussions/4477

なお、モダン化というなら `XMLHttpRequest` に代わる [Fetch API](https://developer.mozilla.org/ja/docs/Web/API/Fetch_API) に切り替えていくのかとも思いましたが、以下のようなスタンスを持っているようです。

https://github.com/axios/axios/discussions/4477#discussioncomment-3091893

> もし axios が Response と Request オブジェクトを使った fetch APIに移行したら、このプロジェクトの存在はかなり無意味になると思います。ユーザーは1つのリクエストに1つの Promise、そしてクロスプラットフォームが axios の最も重要な利点だと考えていますが、もしこれを実行すれば、文字通りインターセプターを持つ別のフェッチライブラリができてしまうことになります。そのようなプロジェクトはすでに存在しています。axiosは日々の仕事を楽にするために、ユーザーエクスペリエンスをシンプルにする独自の方法を見つけるべきでしょう。

これに対して `XMLHttpRequest` を使い続けることに対しるデメリットも議論されていますが、`axios` ではレガシーブラウザ、や旧バージョンの `Node` の互換性を重視する姿勢のようです。

以上より、`axios` の `v1.0.0` 化はモダン化に向けた大きな一歩目である表明にはなるものの、積極的に後方互換を壊すような意図はないようです。

ちなみに、本記事では扱いませんが、 `v0.25` や `v0.27` での破壊的変更はそれなりにあるので、それ未満のバージョンから上げる場合はご注意ください。

 # マイグレーションガイドについて

前項で互換性はなるべく維持する方針であることはわかりましたが、それでも `v1.0.0` にはいくつかの破壊的変更を含みます。

[リリースノート](https://github.com/axios/axios/releases/tag/v1.0.0)を見た限りは大きな影響はありませんが、以下 Discussions によると、これからマイグレーションガイドは用意されるそうです。

https://github.com/axios/axios/discussions/4996

# 主な変更点

ここでは、`v1.0.0` で追加・修正された内容の一部を紹介します。

これらの変更は、記事執筆時点では[ドキュメントサイト](https://axios-http.com/docs/intro)には反映されていないためご注意ください。

## AxiosError オブジェクトにスタックトレースが追加された

https://github.com/axios/axios/pull/4624

エラー発生時にスタックトレースを AxiosError オブジェクトに追加する。

```ts
axios.get("/hoge").catch((e: AxiosError) => {
  console.log(e.stack); // 今までは stack にスタックトレースが含まれていなかった
});
```

ただし、内部で利用されている `Error.captureStackTrace` は非標準機能であるため、環境によって挙動が変わることがある。
https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Error#%E9%9D%99%E7%9A%84%E3%83%A1%E3%82%BD%E3%83%83%E3%83%89

## ビルドツールを `webpack` から `rollup` に変更

https://github.com/axios/axios/pull/4596

ユーザーに直接影響はありませんが、ライブラリのビルドには [Webpack](https://webpack.js.org/) よりも [rollup](https://rollupjs.org/guide/en/) が適しているとのことです。
事実、ビルド成果物のサイズが(minify後)2KB軽量化したそうです。

とはいえ以下で調べてみると、 `0.27` から `1.0.0` でだいぶ増加してますが…。
https://bundlephobia.com/package/axios@1.0.0

## 追加されたインターセプタをすべて破棄するメソッドの追加

https://github.com/axios/axios/pull/4248

`use` した `interceptors` を一括で解除(`eject`)するためのメソッドが追加されました。

```ts
const instance = axios.create();
instance.interceptors.request.use(() => {});
instance.interceptors.request.clear(); // ↑を解除

instance.interceptors.response.use(() => {});
instance.interceptors.response.clear(); // ↑を解除
```

が、これも例によって `TypeScript` の対応が漏れていたため PR を出しましたが、次のバージョンまでは `TypeScript` では使えません…。
https://github.com/axios/axios/pull/5010

## `transformResponse` のコールバックに HTTPステータスを追加

https://github.com/axios/axios/pull/4580

`transformResponse` は、レスポンス内容を書き換えるコールバック関数です。
ここに第三引数としてステータスコードが追加されました。

```ts
axios.get("https://dummyjson.com/products/1", {
  transformResponse(data, headers, status) {
    console.log(status); // 200
  },
});
```

## Web(XMLHttpRequest)で使用できるプロトコルに `blob` を追加

https://github.com/axios/axios/pull/4678

以下のように `blob` プロトコルでのリクエストが出来るようになりました。

```ts
axios.get("blob://hogehoge");
```

いつ使うのこんなのと思う人もいるかも知れませんが、弊社は一度この不具合を踏んでハマったことがあります。

## Web(XMLHttpRequest)で使用できるプロトコルに `data` を追加

前項とだいたい同じ。

https://github.com/axios/axios/pull/4678

```ts
axios.get("data:image/gif;base64,hogehoge");
```

## Node で使用できるプロトコルに `data` を追加

前項とだいたい同じ。

https://github.com/axios/axios/pull/4725

```ts
axios.get("data:image/gif;base64,hogehoge");
```

## toFormData へのオプション追加

https://github.com/axios/axios/pull/4704

オブジェクトを `FormData` 形式に変換する `toFormData` 関数に `formSerializer` オプションを指定できるようになり、シリアライズ時のいくつかの挙動を変更できるようになりました。

例えば `dots: true` を指定した場合、配列はドット記法に変換されるようになります。

```js
const obj = {
  nested: {
    arr: ["hello", "world"],
  },
};

const form = axios.toFormData(obj, undefined, {
  dots: true,
});

form.get("nested.arr.0"); //  dots: true の場合はこっちが "hello" に
form.get("nested.arr[0]"); // dots: false の場合はこっちが "hello" に
```

## リクエストペイロードに `HTMLFormElement` を指定可能に

https://github.com/axios/axios/pull/4735

以下のように Form 要素がある場合に

```html
<form id="form">
  <input type="text" name="foo" value="1" />
  <input type="text" name="deep.prop" value="2" />
  <input type="text" name="deep prop spaced" value="3" />
  <input type="text" name="baz" value="4" />
  <input type="text" name="baz" value="5" />

  <select name="user.age">
    <option value="value1">Value 1</option>
    <option value="value2" selected>Value 2</option>
    <option value="value3">Value 3</option>
  </select>
</form>
```

対象の Form 要素を取得してペイロードして渡すことで

```js
await axios.post("http://localhost:3000", document.querySelector("#form"));
```

Form の内容が `multipart/form-data` 形式に変換された状態でリクエストされます。

```
await axios.post("http://localhost:3000", document.querySelector("#form"));
------WebKitFormBoundaryD24ZKi5sunQBZrji
Content-Disposition: form-data; name="foo"

1
------WebKitFormBoundaryD24ZKi5sunQBZrji
Content-Disposition: form-data; name="deep.prop"

2
------WebKitFormBoundaryD24ZKi5sunQBZrji
Content-Disposition: form-data; name="deep prop spaced"

3
------WebKitFormBoundaryD24ZKi5sunQBZrji
Content-Disposition: form-data; name="baz"

4
------WebKitFormBoundaryD24ZKi5sunQBZrji
Content-Disposition: form-data; name="baz"

5
------WebKitFormBoundaryD24ZKi5sunQBZrji
Content-Disposition: form-data; name="user.age"

value2
------WebKitFormBoundaryD24ZKi5sunQBZrji--
```

また、以下のように `Content-Type: application/json` を明示した場合は `JSON` 形式に自動で変換されます。

```js
await axios.post("http://localhost:3000", document.querySelector("#form"));
```

```json
{"foo":"1","deep":{"prop":{"spaced":"3"}},"baz":["4","5"],"user":{"age":"value2"}}
```

## application/x-www-form-urlencoded への自動シリアライゼーション

`(post|put|patch)Form` メソッドにて、 `"content-type": "application/x-www-form-urlencoded"` を指定した場合、ペイロードを `multipart/form-data` 形式に自動で変換されます。

https://github.com/axios/axios/pull/4714

```ts
const data = {
  x: 1,
  arr: [1],
  arr2: [[2]],
  users: [{ name: "Peter", surname: "Griffin" }],
};

await axios.postForm("https://postman-echo.com/post", data, {
  headers: { "content-type": "application/x-www-form-urlencoded" },
});
```

ペイロードは以下のように変換されてリクエストされます。

```json
"x"

1
------WebKitFormBoundaryhtSCeXdfQVdT55OA
Content-Disposition: form-data; name="arr[]"

1
------WebKitFormBoundaryhtSCeXdfQVdT55OA
Content-Disposition: form-data; name="arr2[0][0]"

2
------WebKitFormBoundaryhtSCeXdfQVdT55OA
Content-Disposition: form-data; name="users[0][name]"

Peter
------WebKitFormBoundaryhtSCeXdfQVdT55OA
Content-Disposition: form-data; name="users[0][surname]"

Griffin
------WebKitFormBoundaryhtSCeXdfQVdT55OA--
```

`(post|put|patch)Form` メソッドは `v0.27.0` で追加されたショートカットメソッドで、`'Content-Type': 'multipart/form-data'` を自動で付与してくれます。
本来は `'Content-Type': 'multipart/form-data'` の場合に自動でペイロードを変換する仕組みがありましたが、ヘッダーを明示的に `application/x-www-form-urlencoded` で上書きされた場合でも同様の変換処理が走るようになったようです。

# 所感

最後に、調査する中での私の所感が以下になります。

- `axios` のような HTTPクライアントライブラリを深堀りすることで、Web や HTTP の学びを多く得られて良い
- 全体的に `TypeScript` の対応が弱いなと感じた
  - `JavaScript` で実装され、型定義ファイル(`index.d.ts`)を別途作成する構成になっているので漏れが出やすいというのは仕方ない
  - とはいえ現代で `JavaScript` では使えるけど `TypeScript` では使えないというのは厳しそうです。
- `FormData` を中心に、ペイロードの自動変換が強化された印象
  - 自動シリアライズや、シリアライズ用のヘルパー関数の提供あたりは嬉しいけど、どこまで HTTP クライアントライブラリの責務として扱うかは怪しい
  - `FormData` 自体を使う機会が最近は減ってきてる気もする
- 現代ではだんだん必要性が薄れてくるパッケージなのかなと思ったけど、シンプルな `Promise` ベースのインタフェースで、ブラウザ/Node ともに動くという強みは残ってそう
  - 今後は `Deno` や `Ban` で動くことも期待されて大変になるのかな
- 歴史があってユーザー数が多い割にはメンテナ・コントリビュータが少ない
  - メンテナーは実質一人の状態
  - コントリビュータも限られているので、OSS 貢献のチャンスは眠ってる
    - 実際、私もちょっと調査してて見つけた不具合を即日でマージしてもらえました