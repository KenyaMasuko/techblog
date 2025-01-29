---
title: Hi👋.length は３なのか
emoji: "👋"
type: "tech"
topics: [javascript, typescript, nodejs]
published: true
---

以下のコードを実行すると何が出力されるでしょうか？

```js
console.log("Hi👋".length);
```

実はこれを実行すると、4 になります。この記事では、なぜこのような結果になるのか、そして「正しく」文字数をカウントする方法をざっくりと解説します。

:::message
記事中「文字」に関する情報がでてきますが、詳細な説明等はありません。そのため誤りが含まれる場合があるかもしれませんが、その場合はご指摘いただけると幸いです。
:::

## JavaScript の.length メソッド

私は JavaScript において`.length`は、その文字列の長さを返すメソッドだと認識していました。例えば以下のようなコードを実行すると、2 が出力されます。

```js
"Hi".length;
```

はたして`.length`は本当に文字列の長さをただ返すだけなのでしょうか？
ここで MDN を確認してみます。

> このプロパティは、文字列内のコード単位の数を返します。JavaScript では UTF-16 エンコーディングを使用しており、すべての Unicode 文字が 1 つまたは 2 つのコード単位にエンコードされるため、length で返される値は文字列の実際の Unicode 文字数に一致しない可能性があります。よく使われるラテン、キリル、有名な漢字などはこのような問題にはなりませんが、絵文字、数学記号、難しい漢字などのような特定の文字体系では、length で返される値が文字列の実際の文字数と一致しなくなる可能性があるので、コード単位数と文字数の違いを考慮する必要があるかもしれません。

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/String/length#%E8%A7%A3%E8%AA%AC

この解説を読むと、単に文字数を返すだけと思っていた`.length`メソッドですが実は UTF-16 コード単位の数を返すということがわかりました。

そのため実際の文字列として表示される文字数と異なる場合があるということですね。例えば、以下のコードを実行すると 2 が出力されます。

```js
"👋".length; // 2
```

## なぜ「正しく」文字列がカウントされないのか

ここで改めて最初に提示したコードを見てみましょう。

```js
console.log("Hi👋".length);
```

このコードは、`Hi👋`というコード単位の数を返すコードです。このコードは、`Hi`と`👋`の 2 つの要素から構成されています。`Hi`は 2 つのコード単位、`👋`も 2 つのコード単位で構成されているため、合計で 4 という結果になります。

しかし、これで先ほどのコードが 4 文字としてカウントされるのは分かりましたが、一般的に`👋`を 2 文字としてカウントしたい人は少数派でしょう。では、`👋` を 1 文字として扱いたい場合はどうすればいいでしょうか？

## 文字列の長さを「正しく」カウントする方法

文字列の長さを「正しく」カウントするためには、以下のようなコードを実行することで期待する文字列の長さを取得することができます。

```js
const segmenter = new Intl.Segmenter("ja-JP", { granularity: "grapheme" });
[...segmenter.segment("Hi👋")].length; // 3
```

`Intl.Segmenter`を使用し、`granularity`オプションに`grapheme`を指定することで期待する文字数を得ることができます。
MDN では`grapheme`は以下のように説明されています。

> ロケールに応じた書記素クラスター（ユーザーが認識する文字）の境界で、入力を分割します。

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Intl/Segmenter/Segmenter#grapheme

「ユーザーが認識する文字」を境界として文字列を分割することで、私たち人間が期待する文字列の長さを取得することができるようですね。

## まとめ

今回の記事では`.length`が期待する文字数を返却しないケースがあるというのを取り上げました。`Intl.Segmenter`の詳細については他にとても勉強になる記事・ドキュメントがありますのでぜひそちらを参照してください。

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Intl/Segmenter/Segmenter
https://susisu.hatenablog.com/entry/2025/01/03/180241
https://qiita.com/suin/items/3da4fb016728c024eaca

ご覧いただきありがとうございました！
