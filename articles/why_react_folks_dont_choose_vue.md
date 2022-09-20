---
title: "[Twitter] React ユーザーが Vue を選ばない理由"
emoji: "👻"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["javascript", "react", "vue"]
published: true
---

# 概要

本記事は、 `Vue.js` コアチームメンバーである [`@antfu7`](https://twitter.com/antfu7) 氏の以下ツイートに対する回答を個人的にまとめたものです。

https://twitter.com/antfu7/status/1567511783832961026

> React ユーザーの皆さん、
> 好奇心で聞くのですが、Vue を使ったり試したりするのを妨げているブロッカーや欠点は何ですか？

# 注意事項

対立煽りっぽいタイトルにはなっていますが、個人的な好奇心がモチベーションとなっており、特定の技術を贔屓、批判する意図はありません。

私自身は長らく `Vue` を愛用しており、業務でも大規模 Vue アプリのメンテに携わっている一方で、`React` は小規模プロジェクトや個人開発でしか利用していません。そのため、理解度に差がある状態であるため、改めて `Vue` と `React` の対比やそれぞれの良さを実感したいと思い、一通りのツイートに目を通した次第です。

また、 `Vue.js` の生みの親である [`Evan You`](https://twitter.com/youyuxi)氏が以下のようにツイートしている通り、回答に含まれる `Vue` に対する不満点の多くは Vue 3 及びそのエコシステムで解決されているものを含んでいます。

https://twitter.com/youyuxi/status/1567700474912190464

> 素晴らしい文化的な議論です。とはいえ、Reactユーザーの中には、Vueを数年前にちょっと試したときの経験に基づく認識がまだ残っている人がたくさんいるような気がします。
> 私はまだ、「React開発者のためのVue 202x版」ブログ記事を完成させる必要があります。

さらに、いくつかの回答に対しては `Vue` ユーザーの方々からのフォローも入っているので、合わせてみてみると `Vue` もイケるじゃんともなってくるのでオススメです。

# React ユーザーはなぜ Vue を選ばないのか

本題です。すべてのリプライに目を通しつつ、いくつかの観点別に、代表的なツイートを抜き出しています。

## エコシステムの成熟具合

`Vue` が順当な進化を経て機能上の差異がなくなってきたことにより、これが大きな理由となるケースが多いようです。

`React` は `Vue` と比べるとフレームワーク (`Nest.js`) やデータフェッチライブラリ (`SWR`, `tRPC`) などの成熟スピードに差があり、常に最善の選択ができることが理由としてあげられています。

特に `Vue` は Vue 3 への移行がエコシステム全体で追従しきれていないため、依然としてモダンな構成の選択が限られることが問題視されています。 (とはいえ `Vue` はコアチームから `VueRouter` `VueI18n` などの主要なライブラリが出ているのが救いではある)

個人的にも `React` のエコシステムに対する安心感を羨ましいと思うことがあり、特に `Storybook` のような開発支援系ツールは `React` を先行してサポートする傾向がある印象があります。

https://twitter.com/donutspree/status/1567523725091786759
https://twitter.com/EddyVinckk/status/1567516447336710144
https://twitter.com/NikhilVerma/status/1567533179652952065
https://twitter.com/songkeys/status/1567537487278637058
https://twitter.com/JoshWComeau/status/1567559618062082048
https://twitter.com/seanghay_yath/status/1567561042703822849
https://twitter.com/matijao_/status/1567544004819951622

## TypeScript のサポート不足

`React` は `TypeScript` との相性が非常に良く、複雑なセットアップを行わずともすぐに型安全な開発を行えます。

一方で `Vue` は SFC (`.vue` の拡張子を持った独自のファイル) を扱う都合に `TypeScript` で扱うのが困難です。`Vue 3` によって型サポートは大きく向上したものの、 `Volar` を主とする IDE とのインテグレーションの手間はまだまだ残っています。

`Vue` でも `vite` でスキャフォルドするなど、ベストプラクティスに則ったプロジェクト構成を採用すれば基本的には何とかなる印象ではありますが、やはり `React` は型安全の面でも先行しているように見えます。

https://twitter.com/sanxiaozhizi/status/1567529738230702083
https://twitter.com/Danny_H_W/status/1567519916156133376
https://twitter.com/NoWizardry/status/1567551925138395136
https://twitter.com/ragragg_/status/1567537229043572737

## `React` は　Vanila JS により近いため

多くのユーザーが `React` はコンポーネントを第一級オブジェクト(`first-class citizens`) であることから、純粋な `JavaScript` に近いことを理由にあげています。

対する `Vue` が SFC という、独自のコンパイラ (`vue-template-compiler` / `@vue/compiler-sfc`) を必要とする構成であることから、前述のエコシステムの成熟のしづらさにもつながっています。

また、`JSX` に対する評価が高く、 `Vue` の `template` が DSL 寄りであるのに対し、 `JSX` はレンダー関数の糖衣構文に過ぎないという点が重要なようです。もちろん `Vue` でも SFC を使わずに、 `JSX` を使ってレンダー関数を定義するという選択肢はありますが、一般的には SFC が使われることが大半で、[推奨もされている](https://vuejs.org/guide/scaling-up/sfc.html#why-sfc) ことから区別して扱っています。

なお、回答に対する反応として、「`JSX` が苦手で使いたくない」というテンプレート派や、「`Vue` のテンプレートは HTML 構文にほんの少しのカスタムディレクティブを追加しただけで DSL ではない」という声もありました。

https://twitter.com/ryanflorence/status/1567701540366073857
https://twitter.com/viniciusflv1/status/1567514645950349313
https://twitter.com/hardfist_1/status/1567552547220779008
https://twitter.com/mzaien_/status/1567533304231993345
https://twitter.com/jon_dewitt_ts/status/1567539033315614721
https://twitter.com/TunkShif/status/1567566536679854081
https://twitter.com/mfbyrktr/status/1567696832146317312

## React Native の存在

意外と多かったのがこちらです。 Web 技術を用いてネイティブアプリを開発する上で `React Native` を採用することが多いからという理由です。

`Vue` でも `capacitor` などのクロスプラットフォーム開発ツールを採用することでネイティブアプリを開発を行えますが、 `React Native` のシェアはまだまだ無視できないようです。

個人的にはクロスプラットフォーム開発のトレンドは移り変わりが激しいので、 `React` そのものを推す理由としては弱いかなと感じました。

https://twitter.com/lazminutes/status/1567512919071674370
https://twitter.com/a7medev/status/1567512972901203971
https://twitter.com/JofArnold/status/1567585671098863618
https://twitter.com/mabdullahsari/status/1567587383549231106
https://twitter.com/alexdotjs/status/1567631961417449475

## 好みでなくシェア率の問題

技術に対する大きなこだわりがない場合の最大の理由がこちらでした。

日本国内では `Vue` が一定のシェアを持っているため気づきにくいですが、 `React` 一強となっている英語圏においてはそのシェア率の高さがシェア率をさらに高めているのでしょうか。

https://twitter.com/Moose_Said/status/1567521537242038272
https://twitter.com/Adib_Hanna/status/1567514505139159042
https://twitter.com/darkjeda/status/1567527051770593280
https://twitter.com/_teo_garcia/status/1567530331254685698
https://twitter.com/ebns1na/status/1567536884003540992

# 感想

ツイートを読みながら思ったこと

- `React` を使うニーズの多くは、Vue 3 でも満たせるようになったから、今後の宣伝次第ではシェアを奪える可能性もある…？
- と言いつつ、 Vue 2 から Vue 3 へのマイグレーションの難しさ、未完成のエコシステムが足を引っ張りはするかも
- 第一級オブジェクトであることの重要性はやっぱり高いみたい。コンポーネントを第一級オブジェクトだからこそTSサポートもしやすく、エコシステムが成熟しやすいのかな
- シェア率、既存プロジェクトの数、会社の方針からっていう単純な回答が思ったより多い
- Composition API を評価する声はそこそこあったので、やっぱり `React` ユーザーにも刺さってはいるのか
- 双方向バインディングに関する声が少なからずあるけど、あれは糖衣構文で同じことは React でも出来るから論点としてはズレてる気がした
- 「React Hooks がクロージャーであることを理解すればその実態はシンプルだ」って話もあったけど、自分はそこの理解が甘いからまだ難しく感じてるのかも

# まとめ

本記事では、React ユーザーが Vue を選ばない理由のツイートについて整理しました。

取り上げたツイートは全体のごく一部ですが、一通りのツイートに目を通した限りは概ね本記事のような傾向だったと思います。

私自身、 `React` にあって `Vue` に足りないものを改めて実感しながらも、それでも `Vue` のここが良いよねというポイントを再認識したり、今後の `Vue` の進化にさらなる期待を感じる点もあったので良い機会になりました。

そもそも `Vue` と `React` は根本の思想が異なる点が多くアプローチも全然違うため、優劣を付けるのではなく、それぞれの強みを伸ばして互いに刺激しあって成長していけば良いなと感じました。