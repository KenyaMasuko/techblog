---
title: "『RSC From Scratch. Part 1: Server Components』を読んだぜ"
emoji: "🧘‍♂️"
type: "tech"
topics: ["react", "nextjs", "web", "frontend", "note"]
published: true
---

みなさん初めまして、[けんや](https://twitter.com/kenchan_dayoooo)です。
先日、Dan先生が書いている『[RSC From Scratch. Part 1: Server Components](https://github.com/reactwg/server-components/discussions/5)』というReact Server Componentsについての解説記事（？）を読んでみました。
どうせなら初めての技術記事っぽいものを書いてみようと思い、学習ノート的な感じで学んだことをまとめてみました。

::: message
この記事は『[RSC From Scratch. Part 1: Server Components](https://github.com/reactwg/server-components/discussions/5)』の考察記事ではなく、あくまで私自身の学びのためにまとめた記事です。そのため内容についての発展的な考察等はほぼありませんのでご了承ください。
:::

# なぜ読もうと思ったか
今回私が『[RSC From Scratch. Part 1: Server Components](https://github.com/reactwg/server-components/discussions/5)』を読もうと思った動機は何かというと、React Server Components（RSC）についてのキャッチアップのためです。実務ではNext.jsのApp Routerを使用した開発を進めており、RSCについての理解をする必要がありました。そして以前参加したある技術系のイベントでこの解説記事について軽く取り上げられており[^1]、記憶の片隅に「いつか読むか〜」と思っていたのでちょうど良いと思い読んでみることにしました。

# 『RSC From Scratch. Part 1: Server Components』について
『[RSC From Scratch. Part 1: Server Components](https://github.com/reactwg/server-components/discussions/5)』はReduxの作者として有名な[Dan Abramov](https://twitter.com/dan_abramov)氏が執筆したRSCについてのDeep Diveです。この記事ついてDan氏は下記のように説明しています。
- **これは、ゼロから実装して新しい技術を学ぶのが好きな人のためのディープダイブです。**
  Webプログラミングの素養があり、Reactにある程度慣れていることが前提です。
  > 🔬 **This is a deep dive for people who like to learn new technologies by implementing them from scratch.**
  It assumes some background in web programming and some familiarity with React.

- **このディープダイブは、Server Componentsの使い方を紹介するものではありません。** 現在、ReactのウェブサイトでServer Componentsのドキュメントを作成中です。もし、お使いのフレームワークがServer Componentsをサポートしている場合は、そのドキュメントを参照してください。
  > 🚧 **This deep dive is not intended as an introduction to how to use Server Components.**
  We are working to document Server Components on the React website. In the meantime, if your framework supports Server Components, please refer to its docs.

- **教育的な理由から、私たちの実装はReactで実際に使われているものよりも大幅に効率が悪くなります。**
  将来的な最適化の機会については本文中に記すが、効率よりも概念の明確さを強く優先する。
  > 😳 For pedagogical reasons, our implementation will be significantly less efficient than the real one used by React.
  We will note future optimization opportunities in the text, but we will strongly prioritize conceptual clarity over efficiency.


また記事は大きく6つのセクションに分かれており、下記のようになっています（かっこ内は記事原文タイトル）。

1. JSXを発明しよう（**Let's invent JSX**）
2. コンポーネントを発明しよう（**Let's invent components**）
3. ルーティングを加えよう（**Let's add some routing**）
4. 非同期コンポーネントを発明しよう（**Let's invent async components**）
5. ナビゲーションの状態を維持しよう（**Let's preserve state on navigation**）
6. コードを整理しよう（**Let's clean things up**）

以降では上記のセクションを1つずつ、中身をある程度かいつまんでみていきます。

## 0. 時を遡ろう
このセクションはこれからRSCを実装していく「前提」の話をするようなセクションなので、勝手に「0.」としてあります（記事内では「**Let’s jump back in time...**」）。

この章では時をWeb開発が発展途上な2003年まで巻き戻し、RSCの実装をPHPのコードから開発を始めます（関係ないですが私はまだ5歳くらい...🐥）。
```php
<?php
  $author = "Jae Doe";
  $post_content = @file_get_contents("./posts/hello-world.txt");
?>
<html>
  <head>
    <title>My blog</title>
  </head>
  <body>
    <nav>
      <a href="/">Home</a>
      <hr>
    </nav>
    <article>
      <?php echo htmlspecialchars($post_content); ?>
    </article>
    <footer>
      <hr>
      <p><i>(c) <?php echo htmlspecialchars($author); ?>, <?php echo date("Y"); ?></i></p>
    </footer>
  </body>
</html>
```
これは個人ブログをイメージしたコードです。`http:localhost:3000/hello-world`にアクセスすると`./posts/hello-world.txt`から生成したHTMLが返却されます。

そしてこれと同等の内容をNode.jsに置き換えると下記のようになります。
```js
import { createServer } from 'http';
import { readFile } from 'fs/promises';
import escapeHtml from  'escape-html';

createServer(async (req, res) => {
  const author = "Jae Doe";
  const postContent = await readFile("./posts/hello-world.txt", "utf8");
  sendHTML(
    res,
    `<html>
      <head>
        <title>My blog</title>
      </head>
      <body>
        <nav>
          <a href="/">Home</a>
          <hr />
        </nav>
        <article>
          ${escapeHtml(postContent)}
        </article>
        <footer>
          <hr>
          <p><i>(c) ${escapeHtml(author)}, ${new Date().getFullYear()}</i></p>
        </footer>
      </body>
    </html>`
  );
}).listen(8080);

function sendHTML(res, html) {
  res.setHeader("Content-Type", "text/html");
  res.end(html);
}
```
テンプレートリテラルを使用してますが、文字列を直接操作しながらUIを実装してくのは理想的ではないです。本セクションはここにReact風のパラダイムを導入して、JSXで書き直すところからRSCの実装は始まります。

## 1. JSXを発明しよう
ここセクションではJSXを導入し、JSXが生成するツリーからHTML文字列への変換を実装します。

まずは元々あったテンプレートリテラルをJSXに置き換えてみます。
```js
createServer(async (req, res) => {
  const author = "Jae Doe";
  const postContent = await readFile("./posts/hello-world.txt", "utf8");
  sendHTML(
    res,
    <html>
      <head>
        <title>My blog</title>
      </head>
      <body>
        <nav>
          <a href="/">Home</a>
          <hr />
        </nav>
        <article>
          {postContent}
        </article>
        <footer>
          <hr />
          <p><i>(c) {author}, {new Date().getFullYear()}</i></p>
        </footer>
      </body>
    </html>
  );
}).listen(8080);
```
なんかスッキリしました。このJSXは下記のようなオブジェクトツリーへと変換されます。このオブジェクトツリーへの変換に関しては、記事内では深く取り扱われておりませんでした。
```json
{
  $$typeof: Symbol.for("react.element"),
  type: 'html',
  props: {
    children: [
      {
        $$typeof: Symbol.for("react.element"),
        type: 'head',
        props: {
          children: {
            $$typeof: Symbol.for("react.element"),
            type: 'title',
            props: { children: 'My blog' }
          }
        }
      },
      {
        $$typeof: Symbol.for("react.element"),
        type: 'body',
        props: {
          children: [
            {
              $$typeof: Symbol.for("react.element"),
              type: 'nav',
              props: {
                children: [{
                  $$typeof: Symbol.for("react.element"),
                  type: 'a',
                  props: { href: '/', children: 'Home' }
                }, {
                  $$typeof: Symbol.for("react.element"),
                  type: 'hr',
                  props: null
                }]
              }
            },
            {
              $$typeof: Symbol.for("react.element"),
              type: 'article',
              props: {
                children: postContent
              }
            },
            {
              $$typeof: Symbol.for("react.element"),
              type: 'footer',
              props: {
                /* さらに続いていきます... */
              }
            }
          ]
        }
      }
    ]
  }
}
```
そして最終的にはオブジェクトツリーをHTML文字列へと変換する必要があります。そのために`renderJSXToHTML`関数を実装します。この関数はオブジェクトツリーをHTML文字列に変換しきるまで再帰的に`renderJSXToHTML`を呼び出します。
```js
function renderJSXToHTML(jsx) {
  if (typeof jsx === "string" || typeof jsx === "number") {
    // 文字列なのでエスケープして直接HTMLに入れる
    return escapeHtml(jsx);
  } else if (jsx == null || typeof jsx === "boolean") {
    // 空のノードなのでHTMLには何も書き込まない
    return "";
  } else if (Array.isArray(jsx)) {
    // ノードの配列はそれぞれをHTMLにレンダリングして連結する
    return jsx.map((child) => renderJSXToHTML(child)).join("");
  } else if (typeof jsx === "object") {
    // オブジェクトがReact JSX要素（<div />など）であるかどうかをチェックする
    if (jsx.$$typeof === Symbol.for("react.element")) {
      // HTMLのタグへ変換する
      let html = "<" + jsx.type;
      for (const propName in jsx.props) {
        if (jsx.props.hasOwnProperty(propName) && propName !== "children") {
          html += " ";
          html += propName;
          html += "=";
          html += escapeHtml(jsx.props[propName]);
        }
      }
      html += ">";
      html += renderJSXToHTML(jsx.props.children);
      html += "</" + jsx.type + ">";
      return html;
    } else throw new Error("Cannot render an object.");
  } else throw new Error("Not implemented.");
}
```
そしてこのHTML文字列への変換がいわゆる"Server-Side Rendering（SSR）"というわけです。RSCとSSRを区別しなければならないといけないというのはうっすらと知ってはいたので、ここの意識はちゃんとしていきたいです。
:::details ここまでのコード
```js
import { createServer } from "http";
import { readFile } from "fs/promises";
import escapeHtml from "escape-html";

createServer(async (req, res) => {
  const author = "Jae Doe";
  const postContent = await readFile("./posts/hello-world.txt", "utf8");
  sendHTML(
    res,
    <html>
      <head>
        <title>My blog</title>
      </head>
      <body>
        <nav>
          <a href="/">Home</a>
          <hr />
        </nav>
        <article>{postContent}</article>
        <footer>
          <hr />
          <p>
            <i>
              (c) {author} {new Date().getFullYear()}
            </i>
          </p>
        </footer>
      </body>
    </html>
  );
}).listen(8080);

function sendHTML(res, jsx) {
  const html = renderJSXToHTML(jsx);
  res.setHeader("Content-Type", "text/html");
  res.end(html);
}

function renderJSXToHTML(jsx) {
  if (typeof jsx === "string" || typeof jsx === "number") {
    return escapeHtml(jsx);
  } else if (jsx == null || typeof jsx === "boolean") {
    return "";
  } else if (Array.isArray(jsx)) {
    return jsx.map((child) => renderJSXToHTML(child)).join("");
  } else if (typeof jsx === "object") {
    if (jsx.$$typeof === Symbol.for("react.element")) {
      let html = "<" + jsx.type;
      for (const propName in jsx.props) {
        if (jsx.props.hasOwnProperty(propName) && propName !== "children") {
          html += " ";
          html += propName;
          html += "=";
          html += escapeHtml(jsx.props[propName]);
        }
      }
      html += ">";
      html += renderJSXToHTML(jsx.props.children);
      html += "</" + jsx.type + ">";
      return html;
    } else throw new Error("Cannot render an object.");
  } else throw new Error("Not implemented.");
}
```
:::

## 2. コンポーネントを発明しよう
このセクションではJSXを使用してフロントエンド開発で馴染み深いコンポーネントの仕組みを発明します。

1 で実装した`renderJSXToHTML`をコンポーネントの発明にも利用しますが、現状のままではうまくいきません。たとえば`BlogPostPage`と`Footer`というコンポーネントを例に考えてみます。
```js
function BlogPostPage({ postContent, author }) {
  return (
    <html>
      <head>
        <title>My blog</title>
      </head>
      <body>
        <nav>
          <a href="/">Home</a>
          <hr />
        </nav>
        <article>
          {postContent}
        </article>
        <Footer author={author} />
      </body>
    </html>
  );
}

function Footer({ author }) {
  return (
    <footer>
      <hr />
      <p>
        <i>
          (c) {author} {new Date().getFullYear()}
        </i>
      </p>
    </footer>
  );
}
```
これらのコンポーネントが先ほど実装した`renderJSXToHTML`に渡ってきたとしても正しくHTML文字列へと変換はされません。なぜなら`renderJSXToHTML`が期待する`jsx.type`としてはHTMLのタグ（`"html"`,`"footer"`など）だからです。

なので`renderJSXToHTML`にコンポーネントが渡ってきた時の条件分岐を追加する必要があります。コンポーネントは関数で実装されているので、`function`が`jsx.type`として渡ってきた時の条件分岐をします。
```js
if (jsx.$$typeof === Symbol.for("react.element")) {
  if (typeof jsx.type === "string") { // これは<div>みたいなタグ？
    // HTMLタグ（<p>など）を処理する既存のコード
    let html = "<" + jsx.type;
    // ...
    html += "</" + jsx.type + ">";
    return html;
  } else if (typeof jsx.type === "function") { // これは＜BlogPostPage>のようなタグ？
    // propsでコンポーネントを呼び出し、返却されたJSXをHTMLに変換する
    const Component = jsx.type;
    const props = jsx.props;
    const returnedJsx = Component(props);
    return renderJSXToHTML(returnedJsx);
  } else throw new Error("Not implemented.");
}
```
これで無事にコンポーネントがHTML文字列に変換されるようになりました。
:::details ここまでのコード
```js
import { createServer } from "http";
import { readFile } from "fs/promises";
import escapeHtml from "escape-html";

createServer(async (req, res) => {
  const author = "Jae Doe";
  const postContent = await readFile("./posts/hello-world.txt", "utf8");
  sendHTML(res, <BlogPostPage author={author} postContent={postContent} />);
}).listen(8080);

function BlogPostPage({ postContent, author }) {
  return (
    <html>
      <head>
        <title>My blog</title>
      </head>
      <body>
        <nav>
          <a href="/">Home</a>
          <hr />
        </nav>
        <article>{postContent}</article>
        <Footer author={author} />
      </body>
    </html>
  );
}

function Footer({ author }) {
  return (
    <footer>
      <hr />
      <p>
        <i>
          (c) {author} {new Date().getFullYear()}
        </i>
      </p>
    </footer>
  );
}

function sendHTML(res, jsx) {
  const html = renderJSXToHTML(jsx);
  res.setHeader("Content-Type", "text/html");
  res.end(html);
}

function renderJSXToHTML(jsx) {
  if (typeof jsx === "string" || typeof jsx === "number") {
    return escapeHtml(jsx);
  } else if (jsx == null || typeof jsx === "boolean") {
    return "";
  } else if (Array.isArray(jsx)) {
    return jsx.map((child) => renderJSXToHTML(child)).join("");
  } else if (typeof jsx === "object") {
    if (jsx.$$typeof === Symbol.for("react.element")) {
      if (typeof jsx.type === "string") {
        let html = "<" + jsx.type;
        for (const propName in jsx.props) {
          if (jsx.props.hasOwnProperty(propName) && propName !== "children") {
            html += " ";
            html += propName;
            html += "=";
            html += escapeHtml(jsx.props[propName]);
          }
        }
        html += ">";
        html += renderJSXToHTML(jsx.props.children);
        html += "</" + jsx.type + ">";
        return html;
      } else if (typeof jsx.type === "function") {
        const Component = jsx.type;
        const props = jsx.props;
        const returnedJsx = Component(props);
        return renderJSXToHTML(returnedJsx);
      } else throw new Error("Not implemented.");
    } else throw new Error("Cannot render an object.");
  } else throw new Error("Not implemented.");
}
```
:::

## 3. ルーティングを加えよう
次にパスごとに表示するHTMLを分岐するためにルーティングを実装します。
ブログのレイアウトコンポーネント`BlogLayout`と一覧ページ`BlogIndexPage`を実装し、そしてURLのパスごとにページを選択する`matchRoute`関数も実装します。
```js
createServer(async (req, res) => {
  try {
    const url = new URL(req.url, `http://${req.headers.host}`);
    // URLにマッチした必要なデータを取得する
    const page = await matchRoute(url);
    // pageをBlogLayoutでラップする
    sendHTML(res, <BlogLayout>{page}</BlogLayout>);
  } catch (err) {
    console.error(err);
    res.statusCode = err.statusCode ?? 500;
    res.end();
  }
}).listen(8080);

async function matchRoute(url) {
  if (url.pathname === "/") {
    // ブログ記事の一覧を表示する
    // postsフォルダ内にある全てのファイルを読み込み、mapで投稿slugの配列を作成する
    const postFiles = await readdir("./posts");
    const postSlugs = postFiles.map((file) => file.slice(0, file.lastIndexOf(".")));
    const postContents = await Promise.all(
      postSlugs.map((postSlug) =>
        readFile("./posts/" + postSlug + ".txt", "utf8")
      )
    );
    return <BlogIndexPage postSlugs={postSlugs} postContents={postContents} />;
  } else {
    // 一覧ではない詳細ページのブログを表示する
    // postsフォルダから対応するファイルを読み込む
    const postSlug = sanitizeFilename(url.pathname.slice(1));
    try {
      const postContent = await readFile("./posts/" + postSlug + ".txt", "utf8");
      return <BlogPostPage postSlug={postSlug} postContent={postContent} />;
    } catch (err) {
      throwNotFound(err);
    }
  }
}

function throwNotFound(cause) {
  const notFound = new Error("Not found.", { cause });
  notFound.statusCode = 404;
  throw notFound;
}
```
重複しているコードもあったりしますが、意外と簡単にルーティングを実装することができました。
:::details ここまでのコード
```js
import { createServer } from "http";
import { readFile, readdir } from "fs/promises";
import escapeHtml from "escape-html";
import sanitizeFilename from "sanitize-filename";

createServer(async (req, res) => {
  try {
    const url = new URL(req.url, `http://${req.headers.host}`);
    const page = await matchRoute(url);
    sendHTML(res, <BlogLayout>{page}</BlogLayout>);
  } catch (err) {
    console.error(err);
    res.statusCode = err.statusCode ?? 500;
    res.end();
  }
}).listen(8080);

async function matchRoute(url) {
  if (url.pathname === "/") {
    const postFiles = await readdir("./posts");
    const postSlugs = postFiles.map((file) =>
      file.slice(0, file.lastIndexOf("."))
    );
    const postContents = await Promise.all(
      postSlugs.map((postSlug) =>
        readFile("./posts/" + postSlug + ".txt", "utf8")
      )
    );
    return <BlogIndexPage postSlugs={postSlugs} postContents={postContents} />;
  } else {
    const postSlug = sanitizeFilename(url.pathname.slice(1));
    try {
      const postContent = await readFile(
        "./posts/" + postSlug + ".txt",
        "utf8"
      );
      return <BlogPostPage postSlug={postSlug} postContent={postContent} />;
    } catch (err) {
      throwNotFound(err);
    }
  }
}

function BlogIndexPage({ postSlugs, postContents }) {
  return (
    <section>
      <h1>Welcome to my blog</h1>
      <div>
        {postSlugs.map((postSlug, index) => (
          <section key={postSlug}>
            <h2>
              <a href={"/" + postSlug}>{postSlug}</a>
            </h2>
            <article>{postContents[index]}</article>
          </section>
        ))}
      </div>
    </section>
  );
}

function BlogPostPage({ postSlug, postContent }) {
  return (
    <section>
      <h2>
        <a href={"/" + postSlug}>{postSlug}</a>
      </h2>
      <article>{postContent}</article>
    </section>
  );
}

function BlogLayout({ children }) {
  const author = "Jae Doe";
  return (
    <html>
      <head>
        <title>My blog</title>
      </head>
      <body>
        <nav>
          <a href="/">Home</a>
          <hr />
        </nav>
        <main>{children}</main>
        <Footer author={author} />
      </body>
    </html>
  );
}

function Footer({ author }) {
  return (
    <footer>
      <hr />
      <p>
        <i>
          (c) {author} {new Date().getFullYear()}
        </i>
      </p>
    </footer>
  );
}

function sendHTML(res, jsx) {
  const html = renderJSXToHTML(jsx);
  res.setHeader("Content-Type", "text/html");
  res.end(html);
}

function throwNotFound(cause) {
  const notFound = new Error("Not found.", { cause });
  notFound.statusCode = 404;
  throw notFound;
}

function renderJSXToHTML(jsx) {
  if (typeof jsx === "string" || typeof jsx === "number") {
    return escapeHtml(jsx);
  } else if (jsx == null || typeof jsx === "boolean") {
    return "";
  } else if (Array.isArray(jsx)) {
    return jsx.map((child) => renderJSXToHTML(child)).join("");
  } else if (typeof jsx === "object") {
    if (jsx.$$typeof === Symbol.for("react.element")) {
      if (typeof jsx.type === "string") {
        let html = "<" + jsx.type;
        for (const propName in jsx.props) {
          if (jsx.props.hasOwnProperty(propName) && propName !== "children") {
            html += " ";
            html += propName;
            html += "=";
            html += escapeHtml(jsx.props[propName]);
          }
        }
        html += ">";
        html += renderJSXToHTML(jsx.props.children);
        html += "</" + jsx.type + ">";
        return html;
      } else if (typeof jsx.type === "function") {
        const Component = jsx.type;
        const props = jsx.props;
        const returnedJsx = Component(props);
        return renderJSXToHTML(returnedJsx);
      } else throw new Error("Not implemented.");
    } else throw new Error("Cannot render an object.");
  } else throw new Error("Not implemented.");
}
```
:::


## 4. 非同期コンポーネントを発明しよう
どうやら`BlogIndexPage`と`BlogPostPage`は見た目が重複している箇所があるようです。`<Post />`コンポーネントを実装して共通化しましょう。また各`<Post />`コンポーネント単位で投稿データを取得できれば`matchRoute`関数の中で重複している投稿データの取得処理もなくせそうです。なんかコンポーネント単位でデータ取得するってRSCみがありますね。

そんなPostコンポーネントの実装です。
```js
async function Post({ slug }) {
  let content;
  try {
    content = await readFile("./posts/" + slug + ".txt", "utf8");
  } catch (err) {
    throwNotFound(err);
  }
  return (
    <section>
      <h2>
        <a href={"/" + slug}>{slug}</a>
      </h2>
      <article>{content}</article>
    </section>
  )
}
```
データ取得をする必要があるのでコンポーネント自体が非同期になります。
`<Post />`の中でデータ取得できるようになったので`matchRoute`関数の中で行っていたデータ取得処理もなくせます。

ルーティングの手段がなくなってしまったので、代わりにページに合わせたコンポーネントを表示する`<Route />`コンポーネントを実装します。
```js
function Router({ url }) {
  let page;
  if (url.pathname === "/") {
    page = <BlogIndexPage />;
  } else {
    const postSlug = sanitizeFilename(url.pathname.slice(1));
    page = <BlogPostPage postSlug={postSlug} />;
  }
  return <BlogLayout>{page}</BlogLayout>;
}
```
こうしてコンポーネント単位でデータを取得する非同期のPostコンポーネントを実装することができました。
:::details ここまでのコード
```js
import { createServer } from "http";
import { readFile, readdir } from "fs/promises";
import escapeHtml from "escape-html";
import sanitizeFilename from "sanitize-filename";

createServer(async (req, res) => {
  try {
    const url = new URL(req.url, `http://${req.headers.host}`);
    await sendHTML(res, <Router url={url} />);
  } catch (err) {
    console.error(err);
    res.statusCode = err.statusCode ?? 500;
    res.end();
  }
}).listen(8080);

function Router({ url }) {
  let page;
  if (url.pathname === "/") {
    page = <BlogIndexPage />;
  } else {
    const postSlug = sanitizeFilename(url.pathname.slice(1));
    page = <BlogPostPage postSlug={postSlug} />;
  }
  return <BlogLayout>{page}</BlogLayout>;
}

async function BlogIndexPage() {
  const postFiles = await readdir("./posts");
  const postSlugs = postFiles.map((file) =>
    file.slice(0, file.lastIndexOf("."))
  );
  return (
    <section>
      <h1>Welcome to my blog</h1>
      <div>
        {postSlugs.map((slug) => (
          <Post key={slug} slug={slug} />
        ))}
      </div>
    </section>
  );
}

function BlogPostPage({ postSlug }) {
  return <Post slug={postSlug} />;
}

async function Post({ slug }) {
  let content;
  try {
    content = await readFile("./posts/" + slug + ".txt", "utf8");
  } catch (err) {
    throwNotFound(err);
  }
  return (
    <section>
      <h2>
        <a href={"/" + slug}>{slug}</a>
      </h2>
      <article>{content}</article>
    </section>
  );
}

function BlogLayout({ children }) {
  const author = "Jae Doe";
  return (
    <html>
      <head>
        <title>My blog</title>
      </head>
      <body>
        <nav>
          <a href="/">Home</a>
          <hr />
        </nav>
        <main>{children}</main>
        <Footer author={author} />
      </body>
    </html>
  );
}

function Footer({ author }) {
  return (
    <footer>
      <hr />
      <p>
        <i>
          (c) {author} {new Date().getFullYear()}
        </i>
      </p>
    </footer>
  );
}

async function sendHTML(res, jsx) {
  const html = await renderJSXToHTML(jsx);
  res.setHeader("Content-Type", "text/html");
  res.end(html);
}

function throwNotFound(cause) {
  const notFound = new Error("Not found.", { cause });
  notFound.statusCode = 404;
  throw notFound;
}

async function renderJSXToHTML(jsx) {
  if (typeof jsx === "string" || typeof jsx === "number") {
    return escapeHtml(jsx);
  } else if (jsx == null || typeof jsx === "boolean") {
    return "";
  } else if (Array.isArray(jsx)) {
    const childHtmls = await Promise.all(
      jsx.map((child) => renderJSXToHTML(child))
    );
    return childHtmls.join("");
  } else if (typeof jsx === "object") {
    if (jsx.$$typeof === Symbol.for("react.element")) {
      if (typeof jsx.type === "string") {
        let html = "<" + jsx.type;
        for (const propName in jsx.props) {
          if (jsx.props.hasOwnProperty(propName) && propName !== "children") {
            html += " ";
            html += propName;
            html += "=";
            html += escapeHtml(jsx.props[propName]);
          }
        }
        html += ">";
        html += await renderJSXToHTML(jsx.props.children);
        html += "</" + jsx.type + ">";
        return html;
      } else if (typeof jsx.type === "function") {
        const Component = jsx.type;
        const props = jsx.props;
        const returnedJsx = await Component(props);
        return renderJSXToHTML(returnedJsx);
      } else throw new Error("Not implemented.");
    } else throw new Error("Cannot render an object.");
  } else throw new Error("Not implemented.");
}
```
:::

## 5. ナビゲーションの状態を維持しよう
ここまででサーバーでHTML文字列を生成してクライアントへ返却することはできました。ではさらにクライアントサイドでローカルの状態を保持できるよう、ナビゲーションが実行された時に変更された部分だけ更新したいですね。
そこでここでは3つのステップでナビゲーションを実装します。
1. ナビゲーションをインターセプトする
1. ネットワークを介してJSXを送る
1. クライアントでJSXを更新する

### 5.1 ナビゲーションをインターセプトする
デフォルトのナビゲーションをオーバーライドし、パスにあったHTMLをリクエストする`navigate`関数をページ遷移した時に呼び出します。これはクライアントで実行されます。
```js
// client.js

let currentPathname = window.location.pathname;

async function navigate(pathname) {
  currentPathname = pathname;
  // ナビゲートするHTMLを取得する
  const response = await fetch(pathname);
  const html = await response.text();

  if (pathname === currentPathname) {
    // HTMLの<body>タグの部分を取得する
    const bodyStartIndex = html.indexOf("<body>") + "<body>".length;
    const bodyEndIndex = html.lastIndexOf("</body>");
    const bodyHTML = html.slice(bodyStartIndex, bodyEndIndex);

    // ページ上のコンテンツを置き換える
    document.body.innerHTML = bodyHTML;
  }
}
```
ですがこれではまだHTMLを全て置き換えているだけです。
### 5.2 ネットワークを介してJSXを送る
次に更新差分を知るためにサーバーからJSXを送ってもらいます。そのためにリクエストのクエリパラメータを利用して、`?jsx`というパラメータでリクエストが来たらHTMLではなくJSXツリーを返却するようにします。
```js
// server.js

createServer(async (req, res) => {
  try {
    const url = new URL(req.url, `http://${req.headers.host}`);
    if (url.pathname === "/client.js") {
      // ...
    } else if (url.searchParams.has("jsx")) {
      url.searchParams.delete("jsx"); // Routerにはクエリパラメータは渡さない
      // sendJSXでJSXをJSONに変換して送る
      await sendJSX(res, <Router url={url} />);
    } else {
      await sendHTML(res, <Router url={url} />);
    }
    // ...
```
しかし一度だけ`sendJSX`関数を通すだけではコンポーネントがもし階層的になっていたりすると全てを綺麗にJSXツリーに変換することはできません。

それを解決するため、すべて変換しきるまで再帰的に自身を呼び出す`renderJSXtoClientJSX`を実装します。
```js
// server.js

async function renderJSXToClientJSX(jsx) {
  if (
    typeof jsx === "string" ||
    typeof jsx === "number" ||
    typeof jsx === "boolean" ||
    jsx == null
  ) {
    // そのまま返却する
    return jsx;
  } else if (Array.isArray(jsx)) {
    // 配列の各項目を再帰的に処理する
    return Promise.all(jsx.map((child) => renderJSXToClientJSX(child)));
  } else if (jsx != null && typeof jsx === "object") {
    if (jsx.$$typeof === Symbol.for("react.element")) {
      if (typeof jsx.type === "string") {
        // これは <div /> のようなコンポーネント
        // propsを調べてJSONに変換できることを確認する
        return {
          ...jsx,
          props: await renderJSXToClientJSX(jsx.props),
        };
      } else if (typeof jsx.type === "function") {
        // <Footer /> のようなカスタムコンポーネント
        // コンポーネントを呼び出しそれが返却するJSXに対してこの操作を繰り返す
        const Component = jsx.type;
        const props = jsx.props;
        const returnedJsx = await Component(props);
        return renderJSXToClientJSX(returnedJsx);
      } else throw new Error("Not implemented.");
    } else {
      // props などの任意のオブジェクト
      // 内部の全ての値を調べ、JSXがある場合はそれも処理する
      return Object.fromEntries(
        await Promise.all(
          Object.entries(jsx).map(async ([propName, value]) => [
            propName,
            await renderJSXToClientJSX(value),
          ])
        )
      );
    }
  } else throw new Error("Not implemented");
}
```
`sendJSX`内で`renderJSXToClientJSX`実行し、クライアントにJSX送ります。

### 5.3 クライアントでJSXを更新する
クライアントでJSXを更新するにはJSXの更新差分をReactで管理する必要があり、ページの初期ロードの段階でReactにDOMノードと対応するJSXを提供してあげなければいけません。
JSXをサーバーから取得して、取得したJSXをクライアントに埋め込む流れで実装してみます。

#### 5.3.1 JSXをサーバーから取得する
JSXをクライアントからサーバーにリクエストするため、URLのクエリパラメータに`?jsx`を付与してデータを取得する`fetchClientJSX`を実装します。
```js
// client.js

async function fetchClientJSX(pathname) {
  const response = await fetch(pathname + "?jsx");
  const clientJSXString = await response.text();
  const clientJSX = JSON.parse(clientJSXString);
  return clientJSX;
}
```
`fetchClientJSX`はサーバーから送られてくるJSONをパースして返却しますが、実はこの時点で送られてくるJSONをパースしてもセキュリティの面からReactが正しくJSXタグとして扱ってくれません。

Reactは`$$typeof: Symbol.for("react.element")`がないと正しくJSXタグとして扱わなくなり、オブジェクトツリーからJSONにシリアライズする際にこのようなSymbol値は抜け落ちてしまいます。そのためサーバー側で処理を加えてからJSONに送信することにしますが、実装自体は文字列を置き換えたりするだけなので割愛します。

ただ、JSONをパースする時に使用する[JSON.parse](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse#the_reviver_parameter)の第二引数に`reviser`関数を渡すことで、パースした値を変換できることを私自身初めて知りました。まだあまりJSONをパースする機会に出会ったことがありません🤫

#### 5.3.2 最初のJSXをHTMLに埋め込もう
そしたら先ほど手に入れた初期ロード時のJSXがどうなっているかをReactに教えてあげましょう。

初期のJSXをクライアントのグローバル変数として埋め込み利用可能だと仮定して、`sendHTML`で一番最初のロード時にHTMLを返却するのと同時にグローバル変数に初期のJSXを格納する方法をとります。
```js
async function sendHTML(res, jsx) {
  let html = await renderJSXToHTML(jsx);

  // HTMLの後にpayloadをシリアライズすることで、ペイントのブロックを回避する
  const clientJSX = await renderJSXToClientJSX(jsx);
  const clientJSXString = JSON.stringify(clientJSX, stringifyJSX);
  html += `<script>window.__INITIAL_CLIENT_JSX_STRING__ = `;
  html += JSON.stringify(clientJSXString).replace(/</g, "\\u003c");
  html += `</script>`;
  // ...
```
グローバル変数として利用可能になりました😃
![グローバル変数に格納されたJSX](https://storage.googleapis.com/zenn-user-upload/c837bf1e1933-20230729.png)

ということはこれでナビゲーションに状態を持たせることができるはずです。
![ページ遷移しても状態が変わらない](https://storage.googleapis.com/zenn-user-upload/38721dae5d90-20230730.gif)
ページ遷移してもインプットの状態は変わっていなそうですね！
:::details server.js  全体コード
```js
import { createServer } from "http";
import { readFile, readdir } from "fs/promises";
import escapeHtml from "escape-html";
import sanitizeFilename from "sanitize-filename";

createServer(async (req, res) => {
  try {
    const url = new URL(req.url, `http://${req.headers.host}`);
    if (url.pathname === "/client.js") {
      await sendScript(res, "./client.js");
    } else if (url.searchParams.has("jsx")) {
      url.searchParams.delete("jsx");
      await sendJSX(res, <Router url={url} />);
    } else {
      await sendHTML(res, <Router url={url} />);
    }
  } catch (err) {
    console.error(err);
    res.statusCode = err.statusCode ?? 500;
    res.end();
  }
}).listen(8080);

function Router({ url }) {
  let page;
  if (url.pathname === "/") {
    page = <BlogIndexPage />;
  } else {
    const postSlug = sanitizeFilename(url.pathname.slice(1));
    page = <BlogPostPage postSlug={postSlug} />;
  }
  return <BlogLayout>{page}</BlogLayout>;
}

async function BlogIndexPage() {
  const postFiles = await readdir("./posts");
  const postSlugs = postFiles.map((file) =>
    file.slice(0, file.lastIndexOf("."))
  );
  return (
    <section>
      <h1>Welcome to my blog</h1>
      <div>
        {postSlugs.map((slug) => (
          <Post key={slug} slug={slug} />
        ))}
      </div>
    </section>
  );
}

function BlogPostPage({ postSlug }) {
  return <Post slug={postSlug} />;
}

async function Post({ slug }) {
  let content;
  try {
    content = await readFile("./posts/" + slug + ".txt", "utf8");
  } catch (err) {
    throwNotFound(err);
  }
  return (
    <section>
      <h2>
        <a href={"/" + slug}>{slug}</a>
      </h2>
      <article>{content}</article>
    </section>
  );
}

function BlogLayout({ children }) {
  const author = "Jae Doe";
  return (
    <html>
      <body>
        <nav>
          <a href="/">Home</a>
          <hr />
          <input />
          <hr />
        </nav>
        <main>{children}</main>
        <Footer author={author} />
      </body>
    </html>
  );
}

function Footer({ author }) {
  return (
    <footer>
      <hr />
      <p>
        <i>
          (c) {author} {new Date().getFullYear()}
        </i>
      </p>
    </footer>
  );
}

async function sendScript(res, filename) {
  const content = await readFile(filename, "utf8");
  res.setHeader("Content-Type", "text/javascript");
  res.end(content);
}

async function sendJSX(res, jsx) {
  const clientJSX = await renderJSXToClientJSX(jsx);
  const clientJSXString = JSON.stringify(clientJSX, stringifyJSX);
  res.setHeader("Content-Type", "application/json");
  res.end(clientJSXString);
}

async function sendHTML(res, jsx) {
  let html = await renderJSXToHTML(jsx);
  const clientJSX = await renderJSXToClientJSX(jsx);
  const clientJSXString = JSON.stringify(clientJSX, stringifyJSX);
  html += `<script>window.__INITIAL_CLIENT_JSX_STRING__ = `;
  html += JSON.stringify(clientJSXString).replace(/</g, "\\u003c");
  html += `</script>`;
  html += `
    <script type="importmap">
      {
        "imports": {
          "react": "https://esm.sh/react@canary",
          "react-dom/client": "https://esm.sh/react-dom@canary/client"
        }
      }
    </script>
    <script type="module" src="/client.js"></script>
  `;
  res.setHeader("Content-Type", "text/html");
  res.end(html);
}

function throwNotFound(cause) {
  const notFound = new Error("Not found.", { cause });
  notFound.statusCode = 404;
  throw notFound;
}

function stringifyJSX(key, value) {
  if (value === Symbol.for("react.element")) {
    return "$RE";
  } else if (typeof value === "string" && value.startsWith("$")) {
    return "$" + value;
  } else {
    return value;
  }
}

async function renderJSXToClientJSX(jsx) {
  if (
    typeof jsx === "string" ||
    typeof jsx === "number" ||
    typeof jsx === "boolean" ||
    jsx == null
  ) {
    return jsx;
  } else if (Array.isArray(jsx)) {
    return Promise.all(jsx.map((child) => renderJSXToClientJSX(child)));
  } else if (jsx != null && typeof jsx === "object") {
    if (jsx.$$typeof === Symbol.for("react.element")) {
      if (typeof jsx.type === "string") {
        return {
          ...jsx,
          props: await renderJSXToClientJSX(jsx.props),
        };
      } else if (typeof jsx.type === "function") {
        const Component = jsx.type;
        const props = jsx.props;
        const returnedJsx = await Component(props);
        return renderJSXToClientJSX(returnedJsx);
      } else throw new Error("Not implemented.");
    } else {
      return Object.fromEntries(
        await Promise.all(
          Object.entries(jsx).map(async ([propName, value]) => [
            propName,
            await renderJSXToClientJSX(value),
          ])
        )
      );
    }
  } else throw new Error("Not implemented");
}

async function renderJSXToHTML(jsx) {
  if (typeof jsx === "string" || typeof jsx === "number") {
    return escapeHtml(jsx);
  } else if (jsx == null || typeof jsx === "boolean") {
    return "";
  } else if (Array.isArray(jsx)) {
    const childHtmls = await Promise.all(
      jsx.map((child) => renderJSXToHTML(child))
    );
    let html = "";
    let wasTextNode = false;
    let isTextNode = false;
    for (let i = 0; i < jsx.length; i++) {
      isTextNode = typeof jsx[i] === "string" || typeof jsx[i] === "number";
      if (wasTextNode && isTextNode) {
        html += "<!-- -->";
      }
      html += childHtmls[i];
      wasTextNode = isTextNode;
    }
    return html;
  } else if (typeof jsx === "object") {
    if (jsx.$$typeof === Symbol.for("react.element")) {
      if (typeof jsx.type === "string") {
        let html = "<" + jsx.type;
        for (const propName in jsx.props) {
          if (jsx.props.hasOwnProperty(propName) && propName !== "children") {
            html += " ";
            html += propName;
            html += "=";
            html += escapeHtml(jsx.props[propName]);
          }
        }
        html += ">";
        html += await renderJSXToHTML(jsx.props.children);
        html += "</" + jsx.type + ">";
        return html;
      } else if (typeof jsx.type === "function") {
        const Component = jsx.type;
        const props = jsx.props;
        const returnedJsx = await Component(props);
        return renderJSXToHTML(returnedJsx);
      } else throw new Error("Not implemented.");
    } else throw new Error("Cannot render an object.");
  } else throw new Error("Not implemented.");
}
```
:::
:::details client.js 全体コード
```js
import { hydrateRoot } from "react-dom/client";

const root = hydrateRoot(document, getInitialClientJSX());
let currentPathname = window.location.pathname;

async function navigate(pathname) {
  currentPathname = pathname;
  const clientJSX = await fetchClientJSX(pathname);
  if (pathname === currentPathname) {
    root.render(clientJSX);
  }
}

function getInitialClientJSX() {
  const clientJSX = JSON.parse(window.__INITIAL_CLIENT_JSX_STRING__, parseJSX);
  return clientJSX;
}

async function fetchClientJSX(pathname) {
  const response = await fetch(pathname + "?jsx");
  const clientJSXString = await response.text();
  const clientJSX = JSON.parse(clientJSXString, parseJSX);
  return clientJSX;
}

function parseJSX(key, value) {
  if (value === "$RE") {
    return Symbol.for("react.element");
  } else if (typeof value === "string" && value.startsWith("$$")) {
    return value.slice(1);
  } else {
    return value;
  }
}

window.addEventListener(
  "click",
  (e) => {
    if (e.target.tagName !== "A") {
      return;
    }
    if (e.metaKey || e.ctrlKey || e.shiftKey || e.altKey) {
      return;
    }
    const href = e.target.getAttribute("href");
    if (!href.startsWith("/")) {
      return;
    }
    e.preventDefault();
    window.history.pushState(null, null, href);
    navigate(href);
  },
  true
);

window.addEventListener("popstate", () => {
  navigate(window.location.pathname);
});
```
:::
## 6. コードを整理しよう
最後に実装してきたコードを整理しましょう。個人的にはこのセクションで「おお！」となる点がありました。

### 6.1 重複した実装を省く
初回ロード時にHTMLを送信する処理で、JSXのコンポーネントが2回実行されてしまっています。
```js
async function sendHTML(res, jsx) {
  // jsxには<Router />が渡ってくる
  let html = await renderJSXToHTML(jsx);

  // 同様に、jsxには<Router />が渡ってくる
  const clientJSX = await renderJSXToClientJSX(jsx);
```
これはそれぞれJSXをHTMLにするかクライアント向けのJSXツリーにするかが違うだけで、どちらも一度JSXツリーに変換されます。なので最初に`renderJSXToClientJSX`でJSXツリーを生成し、それを`renderJSXToHTML`に渡す形にします。
```js
async function sendHTML(res, jsx) {
  // 渡ってきた<Router />を<html>...</html>オブジェクトにする
  const clientJSX = await renderJSXToClientJSX(jsx);

  // すでに変換済みの<html>...</html>オブジェクトをHTML文字列へと変換するだけ
  let html = await renderJSXToHTML(clientJSX);
  // ...
```
これでコンポーネントは一度だけ実行されるようになりました。

### 6.2 HTMLを描画するのにReactを使う
最初はJSXで記述されたコンポーネントを実行するために`renderJSXToHTML`を使用していましたが、先ほどの実装で引数として渡されるのがすでに計算済みのJSXツリーになったので、内部で行っているJSXツリーへの変換は不要です。そのためこの部分を、React組み込みの`renderToString`[^2]で代替します。
```js
import { renderToString } from 'react-dom/server';

// ...

async function sendHTML(res, jsx) {
  const clientJSX = await renderJSXToClientJSX(jsx);
  let html = renderToString(clientJSX);
  // ...
```
そしてここでサーバー側で実行される処理が明確に2分化できました。

1. JSXをクライアントJSXツリーへと変換する処理
2. クライアントJSXツリーからHTMLを生成する処理

従来のSSRのReactアプリケーションではコンポーネントは1度サーバーで実行されて、それを再利用（ハイドレーション）する必要がありました。しかし今回実装したコードでは`<Router />`や`<BlogIndexPage />`などはサーバーでのみ実行されるコンポーネントとなり、`renderToString`と`hydrateRoot`[^3]に限って言えば`<Router />`や`<BlogIndexPage />`はツリー上から「**溶け去り**」（原文では`"melted away"`と表現）、すでに計算済みのJSXツリー上からはそれらコンポーネントの存在は消え去っています。

### 6.3 サーバーの処理を分割する
最後に先ほど2分化した処理は、それぞれステップ自体独立しているのでファイル分割することができます。

1. `renderJSTToClientJSX`→`rsc.js`
1. `renderToString`→`ssr.js`

このようにファイルを分割することでそれぞれのファイルを異なるプロセスで実行することができます。記事内では`rsc.js`はコンポーネントがDBにアクセスするならレイテンシの低いデータセンターの近くで動かしたり、`ssr.js`はHTMLを出力し静的アセットを提供する"エッジ"で動かすことができる、などの例が挙げられています。

またこれらの処理の2分化、ファイルの分割を経て私は1つの記事を思い出し「おお！」となりました。uhyoさんが以前書かれた記事の『[一言で理解するReact Server Components](https://zenn.dev/uhyo/articles/react-server-components-multi-stage)』という記事です。この記事内の「RSCによる多段階計算の例」ではRSCがstage0とstage1の2つの段階で計算されるいう例が挙げられています。私たちが見てきた実装がまさに、多段階計算の例に当てはまるのではないかと腹落ちした次第です。もしかしたら現状の実装では、多段階計算という例を正確に表現することは難しいかもしれませんが、RSCを理解する時のヒントにはなるのではないかと思っています。

最後に全体の流れをまとめると下記のようになります。
![初期ロード時の流れ](https://user-images.githubusercontent.com/810438/242937001-f3e95105-4acb-4ae7-9ce5-39bbe2afd515.png)
![ページ遷移時の流れ](https://user-images.githubusercontent.com/810438/242956087-c435e5bd-5421-4a6e-9d35-538a81a485bb.png)

# まとめ
いかがだったでしょうか？
学習ノートとして他人に説明するのが難しいかつ、文字に起こすともっと難しいと自身の知識のなさ・レベルの低さを痛感しました。もちろんわからない・曖昧な箇所は多くありましたし、それらの箇所についてもちゃんと向き合わないとなと思いました。笑
また今回はRSCについてでしたがクライアントコンポーネントについても1から実装するDeep Diveが執筆されればぜひやってみたいです。

私自身初めて記事を書くということもあり稚拙な文章・誤字脱字も多々あったかと思いますが最後までお読みいただいた方、本当にありがとうございました🥲

[^1]: https://speakerdeck.com/koba04/next-niyoruapurikesiyonkai-fa-nokorekara?slide=21
[^2]: https://react.dev/reference/react-dom/server/renderToString
[^3]: https://react.dev/reference/react-dom/client/hydrateRoot
