---
title: "Next.jsのinstrumentationHookとは"
emoji: "🎷"
type: "tech"
topics: ["nextjs", "typescript", "frontend"]
published: true
published_at: 2024-12-08 19:00
publication_name: "dev_commune"
---

どうもこんにちは[けんや](https://x.com/kenchan_dayoooo)です。
みなさんは Next.js ディレクトリの root に置くファイルと言ったら何を思い浮かべますか？ 多くの方が`middleware` を思い浮かべると思いますが、同様にディレクトリの root にファイルを置く`instrumentationHook` について紹介したいと思います。

# はじめに

この記事は「[Commune Advent Calendar 2024](https://qiita.com/advent-calendar/2024/commune)」シリーズ 2 の 8 日目の記事です。

## Next.js の instrumentationHook 機能とは

### instrumentationHook について

機能自体を`instrumentationHook` と呼んできましたがこれは機能が experimental だった時に `next.config` に書くオプションだった時の名称です。Next.js の`v15.0.0-RC`で stable となり、ドキュメントでは Instrumentation という名称で説明がされているので、以降は Instrumentation と呼びます。

## Instrumentation の機能

Instrumentation の機能を使用すると、 Next.js のサーバインスタンスを起動した時に一度だけ実行される処理を定義することができます。

たとえば [Next.js のドキュメント](https://nextjs.org/docs/app/building-your-application/optimizing/instrumentation)で紹介されているのは、OpenTelemetry を使用したセットアップ例です。このような処理を Instrumentation で行うことで、サーバの起動時に一度だけトレースの設定を行うことができます。

Instrumentation の使い方としてはディレクトリの`root`か、または`src`ディレクトリを採用してる場合はその中に`instrumentation.ts`というファイルを作成します。そこに Next.js のサーバが起動した際同時に一度だけ実行したい処理を記述します。下記は OpenTelemetry のセットアップを行う例です。

```ts:instrumentation.ts
import { registerOTel } from "@vercel/otel";

export function register() {
  registerOTel("next-app");
}
```

これにより`register`関数がサーバの起動時に一度だけ実行されます（Instrumentation を使用した OpenTelemetry セットアップのコード例は[GitHub の example](https://github.com/vercel/next.js/tree/canary/examples/with-opentelemetry)に載っています）。

コミューンではフィーチャーフラグのクライアントの初期化に Instrumentation の機能を使用しています。

### Instrumentation 使用の際の注意点

Instrumentation の実行はすべての環境で呼び出されます。つまり`register`関数に実装した処理が特定のランタイムに依存するようだとその処理が動かなくなってしまします。そのため、特定のランタイムでしか動作しないような処理は、`register`関数内でそのランタイムをチェックするような処理を書く必要があります。

```ts:instrumentation.ts
export async function register() {
  // Node.js の場合のみ実行
  if (process.env.NEXT_RUNTIME === "nodejs") {
    await import("./instrumentation-node");
  }

  // Edge の場合のみ実行
  if (process.env.NEXT_RUNTIME === "edge") {
    await import("./instrumentation-edge");
  }
}
```

実際に私もこの点を考慮せずに実装して意図せず動作が正常に行われないということがありました。そのため、この点には注意が必要です。

# まとめ

Next.js の Instrumentation の機能を紹介してきましたがいかがでしたでしょうか？

余談ですが Instrumentation の単語の意味は「楽器の使用」的な意味だと思っていたのですが、エンジニアリングの文脈で使用すると「計装」という訳語になるそうです（筆者調べ）。以上、豆知識でした。

最後までお読みいただきありがとうございました！
