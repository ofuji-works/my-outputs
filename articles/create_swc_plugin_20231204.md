---
title: "SWCプラグインを作ってJavaScriptトランスパイルの仕組みに触れる"
emoji: "📚"
type: "tech"
topics: ["swc", "rust", "JavaScript"]
published: false
publication_name: "sun_asterisk"
---

## はじめに

### 自己紹介

Frontend Engineerのofujiです。
業務ではリードエンジニアとして、開発を行っています。
最近では主にReactとTypeScriptを使ったWebアプリケーションの開発を行うことが多いです。

個人では、Rustが好きで簡単なアプリケーションを作るなりして楽しんでいます。
実は今回の記事もその一貫になります。

### この記事の目的
今回の記事では、SWCというJavaScriptのトランスパイラを使ってカスタムプラグインを作成することで、JavaScriptのトランスパイルの仕組みに触れることを目的としています。

### この記事の構成

この記事は以下の内容で構成されます。

1. **はじめに**
2. SWCとは
3. JavaScriptのトランスパイルについて
4. ASTについて
5. SWCのカスタムプラグインを作成してみる
6. まとめ
7. 参考文献

## SWCとは

SWCとは、Rustで書かれたJavaScript/TypeScriptのトランスパイラです。
同等の機能を持つBabelと比較して、高速であることが特徴です。
(シングルスレッド上でBabelの20倍早いとかなんとか)

https://swc.rs/

身近なところでは、Next.jsでよく話題になったりしますよね。
https://nextjs.org/docs/architecture/nextjs-compiler

主に機能としては、CompilationやMinification、Bundling等があります。

## JavaScriptのトランスパイルについて

## ASTについて

## SWCのカスタムプラグインを作成してみる

## まとめ

## 参考文献

https://oxc-project.github.io/javascript-parser-in-rust/docs/intro
https://astexplorer.net/
https://swc.rs/docs/plugin/ecmascript/getting-started
https://rustdoc.swc.rs/swc_ecma_visit/trait.VisitMut.html
https://www.wantedly.com/companies/wantedly/post_articles/389049

