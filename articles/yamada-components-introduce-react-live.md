---
title: "ブラウザ上でReactのコードが書けるReact Liveの紹介"
emoji: "😎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["yamadaui", "react", "nextjs", "frontend", "web"]
published: false
---

# はじめに
こんにちは、[けんや](https://twitter.com/kenchan_dayoooo)です😎
この記事ではYamada UIのプレイグラウンドにも使用されている、[React Live](https://commerce.nearform.com/open-source/react-live/)というライブラリの紹介をします！

# React Liveについて
React Liveは、React コンポーネントをリアルタイムで編集・プレビューするためのライブラリです。
特にドキュメンテーションやインタラクティブなコーディングチュートリアルで使用されることが多く、[Yamada UIのドキュメントサイト](https://yamada-ui.com/ja)でも実際に使われていて、コードをいじりながらコンポーネントの具合を確かめることができます。

## React Liveを使ってみる
では一緒にReact Liveを使ってみましょう！
::: message
以降はpnpmとviteを使用して進めていきます。
:::

1. Reactの雛形を作成する
```terminal
$ pnpm create vite
```
で作成していきます。以降の環境構築の詳細については割愛させていただきます。詳細は[Viteのドキュメント](https://ja.vitejs.dev/guide/)をご覧ください！

2. React Liveをインストールする
```terminal
$ pnpm add react-live
```

3. `App.tsx`の中身を書き換える
`App.tsx`の中身はデフォルトを下記のように書き換えます。
```diff tsx:App.tsx
- import { useState } from 'react'
- import reactLogo from './assets/react.svg'
- import viteLogo from '/vite.svg'
- import './App.css'
+ import { LiveProvider, LiveEditor, LiveError, LivePreview } from "react-live";

function App() {
- const [count, setCount] = useState(0)

  return (
-    <>
-      <div>
-        <a href="https://vitejs.dev" target="_blank">
-          <img src={viteLogo} className="logo" alt="Vite logo" />
-        </a>
-        <a href="https://react.dev" target="_blank">
-          <img src={reactLogo} className="logo react" alt="React logo" />
-        </a>
-      </div>
-      <h1>Vite + React</h1>
-      <div className="card">
-        <button onClick={() => setCount((count) => count + 1)}>
-          count is {count}
-        </button>
-        <p>
-          Edit <code>src/App.tsx</code> and save to test HMR
-        </p>
-      </div>
-      <p className="read-the-docs">
-        Click on the Vite and React logos to learn more
-      </p>
-    </>
+  <LiveProvider code="<strong>Hello World!</strong>">
+    <LiveEditor />
+    <LiveError />
+    <LivePreview />
+  </LiveProvider>;
  )
}

export default App
```

コードをこのように書き換えた後`pnpm dev`でローカル環境を立ち上げると以下の画像のような見た目の画面が立ち上がります。
![](/images/yamada-components-introduce-react-live/react-live-preview.png)

3. コードをリアルタイムで編集する
あとは先ほど書いたコードをエディタ上で編集すればOKです！色々いじってみましょう！
![](/images/yamada-components-introduce-react-live/react-live-demo.gif)


# おわり
OSSの開発をしていると普段扱わないような技術やライブラリと出会ったりするため非常に楽しいし、普段の技術の引き出しも広がります。私自身もメンテナになって1、2ヶ月程度ですがこのようなたくさんの知らない世界に触れることができているので、ぜひ普段の開発生活にちょっとしたアクセントが欲しい方はYamada UIにコミットしてみてください🤪（[good first issue](https://github.com/yamada-ui/yamada-ui/labels/good%20first%20issue)めちゃくちゃあります！）

あなたからのプルリクエストを待っています👋
