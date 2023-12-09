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

普段私たちが利用しているTypeScriptやJSXはJavaScriptのエンジンでそのまま実行はできません。
なので実行できる形に（つまりJavaScriptにトランスパイル）してあげる必要があります。

:::message
トランスパイル(トランスコンパイル)とは

フリー百科事典『ウィキペディア（Wikipedia）』https://ja.wikipedia.org/wiki/%E3%83%88%E3%83%A9%E3%83%B3%E3%82%B9%E3%82%B3%E3%83%B3%E3%83%91%E3%82%A4%E3%83%A9)
> あるプログラミング言語で書かれたプログラムのソースコードを入力として受け取り、別のプログラミング言語の同等のコードを目的コードとして生成する、ある種のコンパイラである
:::

トランスパイルのライフサイクルについては以下のようになっています。

```mermaid
graph LR
    A[ソースコード] --> B
    B[parser] --> C
    C[AST] --> D
    D[traverser] --> E
    E[AST] --> F
    F[generator] --> G
    G[JavaScript]
```
トランスパイルの処理の中ではAST(Abstract Syntax Tree)というデータ構造を中間表現として利用し、ソースコードを解析していきます。
※ASTについては後述します。

### parser
図のごとく、ソースコードをASTに変換する処理です。
ここではソースコードをNodeと呼ばれる単位に分割し、それぞれ意味のある形として構造化していきます。
こちらのフェーズについても詳細に書くと、下図のようなフローがあるのですが詳しい内容は割愛します。

```mermaid
graph LR
    A[ソースコード] --> B
    B[Lexer] --> C
    C[Token] --> D
    D[Parser] --> E
    E[AST]
```

もしご興味があれば、以下の記事が参考になります。
https://oxc-project.github.io/javascript-parser-in-rust/docs/overview

### traverser
このフェーズでは生成されたASTに対し、再帰的に解析を行います。
その過程で、ASTのノードに対して処理を行うことができます。
swcやbabelでは、このフェーズでノードに対して処理をするカスタムプラグインを作成することができます。
前述したJsx等をJavaScriptに変換する処理もこのフェーズで行われています。

例）
React Jsxを変換するpluginとしては以下のような物が利用されています。
https://babeljs.io/docs/babel-plugin-transform-react-jsx

### generator
最後に、ASTをJavaScriptに変換する処理です。
このフェーズでは、ASTを再帰的に解析し、JavaScriptのソースコードを生成します。


## ASTについて

Abstract Syntax Treeの略で、抽象構文木とも呼ばれます。
ソースコードの文法を抽象的に表現し、階層的に構造化したデータ構造です。
ASTは、Nodeという単位で構成されてます。
例えば以下のようなソースコードがあった場合、ASTはこのようになります。

```JavaScript
function hoge () {
    const huga = 'huga';
    console.log(huga);
}
```

```AST
{
  "type": "Program",
  "start": 0,
  "end": 65,
  "body": [
    {
      "type": "FunctionDeclaration",
      "start": 0,
      "end": 65,
      "id": {
        "type": "Identifier",
        "start": 9,
        "end": 13,
        "name": "hoge"
      },
      "expression": false,
      "generator": false,
      "async": false,
      "params": [],
      "body": {
        "type": "BlockStatement",
        "start": 17,
        "end": 65,
        "body": [
          {
            "type": "VariableDeclaration",
            "start": 20,
            "end": 40,
            "declarations": [
              {
                "type": "VariableDeclarator",
                "start": 26,
                "end": 39,
                "id": {
                  "type": "Identifier",
                  "start": 26,
                  "end": 30,
                  "name": "huga"
                },
                "init": {
                  "type": "Literal",
                  "start": 33,
                  "end": 39,
                  "value": "huga",
                  "raw": "'huga'"
                }
              }
            ],
            "kind": "const"
          },
          {
            "type": "ExpressionStatement",
            "start": 45,
            "end": 63,
            "expression": {
              "type": "CallExpression",
              "start": 45,
              "end": 62,
              "callee": {
                "type": "MemberExpression",
                "start": 45,
                "end": 56,
                "object": {
                  "type": "Identifier",
                  "start": 45,
                  "end": 52,
                  "name": "console"
                },
                "property": {
                  "type": "Identifier",
                  "start": 53,
                  "end": 56,
                  "name": "log"
                },
                "computed": false,
                "optional": false
              },
              "arguments": [
                {
                  "type": "Identifier",
                  "start": 57,
                  "end": 61,
                  "name": "huga"
                }
              ],
              "optional": false
            }
          }
        ]
      }
    }
  ],
  "sourceType": "module"
}
```

諸々説明が必要かと思いますが、
まず初めにJavaScriptではESTreeというMozillaが策定したASTの仕様が基礎になっています。

https://github.com/estree/estree

特徴としては、トップレベルのNodeであるProgramがsourceTypeというプロパティを持っており、その値がmoduleの場合はES Module、scriptの場合はScriptということを表しています。
上記例では、moduleとなっているのでES Moduleであることがわかります。

:::message
JavaScript ASTにはBabel/Babylon(Acorn系)やEsprima系等複数の種類が存在しているのですが、こちらについては割愛します。
:::

今回作成するpluginはSWCのpluginであるわけですが、SWCで採用しているASTは独自で定義したものを利用しておりESTreeとは異なります。
もし興味があれば、以下のリンクのページで試すことができます。

https://play.swc.rs/

さて少し話がそれましたが、まだ説明できていないNodeがありますね。
上述したProgram他にもStatementやExpression、Declarations、Identifier、Literal等様々なNodeが存在します。

大体ここにまとまっていますので、こちらも興味があれば見てみてください。
https://github.com/estree/estree/blob/master/es5.md


### Statement
文を表すNodeです。
BlockStatementであれば、{}で囲まれた部分を表します。

### Expression
式を表すNodeです。
上記例では、CallExpressionやMemberExpressionが該当します。

### Declarations
変数宣言を表すNodeです。
上記例では、FunctionDeclarationやVariableDeclarationとVariableDeclaratorが該当します。
見てもらえるとわかる通り、
FunctionDeclarationでは、関数の宣言を表現していますね。
VariableDeclarationはkindというプロパティにconst等のkeywordを情報を保持しており、
VariableDeclaratorでは、変数名を表すIdentifierと初期値を表すLiteralをそれぞれidとinitというプロパティに保持しています。

### Identifier
変数名や関数名等を表すNodeです。
上述したように、VariableDeclaratorやFunctionDeclaration等で利用されています。

### Literal
文字列や数値等のリテラルを表すNodeです。
こちらも上述したように、VariableDeclaratorやFunctionDeclaration等で利用されています。



もしASTを他にも色々試してみたい場合は、以下のサイトが便利です。
JavaScriptのASTはAcorn系のものを利用しているようです。

https://astexplorer.net/


とここまで、ESTreeベースで話を進めてきましたが上述した通りSWCでは独自のASTを利用しています。
とはいっても、ESTreeと大きく変わるわけではなく似た構造をしているのでこれらの知識を利用してプラグインの作成を進めていきたいと思います。


## SWCのカスタムプラグインを作成してみる

## まとめ

## 参考文献

https://oxc-project.github.io/javascript-parser-in-rust/docs/intro
https://astexplorer.net/
https://swc.rs/docs/plugin/ecmascript/getting-started
https://rustdoc.swc.rs/swc_ecma_visit/trait.VisitMut.html
https://www.wantedly.com/companies/wantedly/post_articles/389049

