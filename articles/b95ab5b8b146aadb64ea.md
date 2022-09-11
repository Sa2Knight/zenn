---
title: "dayjsの使い方とよく使うTips"
emoji: "⏰"
type: "tech"
topics:
  - "javascript"
  - "dayjs"
published: true
published_at: "2020-10-20 13:46"
---

# 概要

[dayjs](https://github.com/iamkun/dayjs) は、JavaScriptで日付時刻周りの操作を簡易的に行うためのライブラリで、同カテゴリで覇権を取っていた[moment.js](https://momentjs.com/)が[開発終了](https://momentjs.com/docs/#/-project-status/)することから、その移行先として注目を集めている。

本記事では、`dayjs`の特徴や`moment.js`からの移行作業などと言ったことは割愛し、単純に`dayjs`によるよく使うTipsを紹介する。

# バージョン情報

|ツール|バージョン|
|---|---|
|node|12.8.2|
|dayjs|1.9.1|

# Tips

## dayjsを使う

CommonJSの場合
```javascript
const dayjs = require('dayjs')
```

ESModulesの場合
```javascript
import dayjs from 'dayjs'
```

## 現在時刻のdayjsオブジェクトを生成

dayjsオブジェクトはJavaScriptのDateオブジェクトをラッピングしたもの

```javascript
> dayjs()
d {
  '$L': 'en',
  '$d': 2020-10-20T04:21:44.245Z,
  '$x': {},
  '$y': 2020,
  '$M': 9,
  '$D': 20,
  '$W': 2,
  '$H': 13,
  '$m': 21,
  '$s': 44,
  '$ms': 245
}
```

## 時刻文字列からdayjsオブジェクトを生成

時刻を省略した場合は0時00分に

```javascript
> dayjs('1992-05-29')
d {
  '$L': 'en',
  '$d': 1992-05-28T15:00:00.000Z,
  '$x': {},
  '$y': 1992,
  '$M': 4,
  '$D': 29,
  '$W': 5,
  '$H': 0,
  '$m': 0,
  '$s': 0,
  '$ms': 0
}
```

## dayjsオブジェクトからdateオブジェクトに変換

```javascript
> dayjs().toDate()
2020-10-20T04:23:16.372Z
```

## 年月日や時刻を個別に取得

```javascript
> now = dayjs()
> now.year()
2020
> now.month()
9 // 1月が0なので注意
> now.date()
20 // dateが日でdayが曜日なので注意
> now.day()
2 // 日曜日が0
```

## 年月日や時刻を個別に設定

取得に使ったメソッドに引数を指定するとセッターとなり、更新後のオブジェクトが戻り値になるのでチェイン出来る

```javascript
> dayjs().year(1992).month(5 - 1).date(29).hour(13).minute(20).second(1)
d {
  '$L': 'en',
  '$d': 1992-05-29T04:20:01.833Z,
  '$x': {},
  '$y': 1992,
  '$M': 4,
  '$D': 29,
  '$W': 5,
  '$H': 13,
  '$m': 20,
  '$s': 1,
  '$ms': 833
}
```

## 相対日付を取得

指定日から見た1年後の3日前を取得する。 

```javascript
> dayjs('2020/10/20').add(1, 'year').subtract(3, 'days')
d {
  '$L': 'en',
  '$d': 2021-10-16T15:00:00.000Z,
  '$x': {},
  '$y': 2021,
  '$M': 9,
  '$D': 17,
  '$W': 0,
  '$H': 0,
  '$m': 0,
  '$s': 0,
  '$ms': 0
}
```

## 日付と日付の差分を取得する

dayjsオブジェクトのdiffメソッドで、対象となるdayjsオブジェクトを渡してあげると、その差を秒で取得することができる。

```javascript
> const dayjs = require('dayjs')
> const from = dayjs('1992-05-29')
> const to = dayjs()
> to.diff(from)
896103349875
```

diffメソッドに第二引数を指定すると、形式を指定できる。

```javascript
> to.diff(from, 'year')
28 // 差が28年
> to.diff(from, 'day')
10371 // 差が10371日
```

## フォーマットを指定して文字列を取得

```javascript
> dayjs().format('YYYY-MM-DD')
'2020-10-20'
> dayjs().format('hh:mm:ss')
'01:37:47'
```

## UNIX TIMEを取得

```javascript
> dayjs().unix()
1603168706
```

## その他の便利機能(プラグイン)を使う

- 日付間の比較
- 日付リスト内の最大値、最小値を取得
- 週を使った複雑な操作
- うるう年確認
- ロケールのカスタム

などの、やや込み入った操作は、dayjsではプラグインをして提供されている。

例えば、`isBetween` プラグインの使用を以下のように宣言することで、対象の日付がある期間に含まれているかを確認することができる。

```javascript
const isBetween = require('dayjs/plugin/isBetween')
dayjs.extend(isBetween)

dayjs().isBetween('2020-01-01', '2020-12-13') // 現在は2020年内のためtrue
```