+++  
author = "miyashi"  
title = "[GraphQL] Apollo-iOS を使ってみて感じたイケてるところ。イケてないところ。"  
date = "2020-12-18"  
description = ""  
tags = [  
    "GraphQL",  
    "Apollo-iOS",  
    "Swift",  
    "iOS",
    "UIKit",  
]  
+++  

---
この記事は [iOS #2 Advent Calendar 2020](https://qiita.com/advent-calendar/2020/ios-2) の18日目の記事です。 

  
# 概要  
今業務で[Apollo-iOS](https://github.com/apollographql/apollo-ios)(GraphQL)を導入してます。
ネット上のいろんなサイトを参考にして、実装しましたが、思い返すと、Apollo-iOSの使い方について言及されている記事は多いのですが、実際使ってみての感想はあまり見たことがないので、
今回はイケてると思った所とイケてないなと思った所について、主観で好き勝手書いてみました。  

これからGraphQL導入しようか迷っているという方に流し目に読んでいただけると幸いです.  
捉え方は要件や環境によると思うので、当てはまらない場合もあると思います。  
また、私の知識が及んでいないゆえの間違いなど気づいた点があれば[Twitter](https://twitter.com/po_miyasaka)等でコメントいただけると嬉しいです。  

# GraphQLとは  
簡単に言うとサーバーとクライアントとの通信をする際のプロトコルのようなもので、特定のライブラリやツールを指すものではありません。
詳しくはすでに様々なサイトで紹介されているので、参考になるであろうリンクを記載しておきます。
[GraphQL.org](https://graphql.org/)
[世のフロントエンドエンジニアにApollo Clientを布教したい](https://qiita.com/seya/items/26c8a0dc549a10efcdf8#graphql%E3%81%A8%E3%81%AF)
[Web API初心者と学ぶGraphQL](https://qiita.com/SiragumoHuin/items/cc58f456bc43a1be41b4)

# Apollo-iOSとは
iOSアプリからGraphQLでサーバーとやり取りするためのライブラリです。
こちらも参考となるリンクを貼っておきます。
[Apollo-iOS](https://github.com/apollographql/apollo-ios)
[Apollo-iOS](https://www.apollographql.com/docs/ios/)



#  イケてる所  
## サーバーエンジニアとの認識齟齬がなくなる。
これはApollo-iOSというよりもGraphQLについてですが、[GraphiQL](https://github.com/graphql/graphiql)や[PlayGround](https://github.com/graphql/graphql-playground)などのツールを導入することで、ブラウザ上でリクエストを簡単に試したり、付属するDoc機能でAPIの仕様を簡単に把握することができます。
[初めてのGraphQL](https://www.oreilly.co.jp/books/9784873118932/)という本でも紹介されていた [サンプル](http://snowtooth.moonhighway.com/)でPlayGroundを実際に触ることができます。

<a href='/images/post/graphql-impressions/playground.png'>{{< figure src="/images/post/graphql-impressions/playground.png" title="" class="center" width="600" >}}</a>


こうしたGraphQLから提供されたツールを利用することで、API仕様についてのレビューをより効率的に行えるようになり、
先にインターフェースの仕様をかっちりと決めて、その後にサーバーとクライアントがそれぞれ実装に入るという[スキーマ駆動](https://blog.spacemarket.com/code/schema-driven/)がしやすい点がイケてました。

また、[ページネーション](https://graphql.org/learn/pagination/)や[認証](https://graphql.org/learn/authorization/)などのよくある処理について、実現パターンが[GraphQLコミュニティ](https://graphql.org/learn/authorization/)で策定されており、これを参照することで実装方法や仕様の一貫性を保つことができる点もいいなと思いました。

##  Interceptorによって通信処理がカスタマイズしやすい
こちらはApollo-iOSについてです。
Apollo-iOSでは、通信、パース、キャッシュの操作などの処理を[Interceptor](https://github.com/apollographql/apollo-ios/blob/main/Sources/Apollo/ApolloInterceptor.swift#L2)として個別に定義し、それらを配列にして登録することで順番に処理されるようになります。

デフォルトでは[LegacyInterceptorProvider](https://github.com/apollographql/apollo-ios/blob/main/Sources/Apollo/InterceptorProvider.swift#L31)が定義されていて、この中に[Interceptorの配列](https://github.com/apollographql/apollo-ios/blob/main/Sources/Apollo/InterceptorProvider.swift#L57)が定義されています。

[Interceptorプロトコル](https://github.com/apollographql/apollo-ios/blob/main/Sources/Apollo/ApolloInterceptor.swift#L2)に準拠した構造体を独自の`InterceptorProvider`に組み込むことで、通信前後に任意の処理を行うことができます。

例えば、リクエストヘッダーにアクセストークンを付与することも以下のように簡単に行なえます。

#  イケてない所
## エラー周り
### エラーはスキーマで定義されない
一般的なGraphQLライブラリ（Apollo-iOSについても同様）の挙動では、エラーはHTTPステータス200として以下のようなJSON形式で返ってきます。

`message` や`locations`　の値はライブラリによって自動的に格納され、`extensions`にサーバーエンジニアが任意で設定した情報が格納されています。

```
{
    "errors": [
      {
        "message": "Cannot query field \"n\" on type \"Lift\".",
        "locations": [
          {
            "line": 3, 
            "column": 5
          }
        ],
        "extensions": {
          "code": “HogeError”
          “message”: “HogeMessage”
        }
      }
    ]
 }
```

エラーの情報は正常系とは違いスキーマ等に明示されず、独自の型を設定することもできません。また、仕様をクライアントエンジニアとサーバーエンジニアで共有する手段は特に用意されていません。

Apollo-iOSで`extensions`の`code` にアクセスするには以下のようにします。
```
response?.parsedResponse?.errors?[0].extensions?[“code”] == “HogeError”
```

上記を見る限りパターンマッチでタイポしたりすると、ハンドリングができなくなったりするリスクがありそうです。

できることとしては、タイポしないようにクライアント側にEnumで列挙したり、スキーマのDocにエラーの情報も追加で一覧化してもらうといった形で、見える化するくらいしかないのかなと思っています。

なお、この場合の200系のエラーはデフォルトでは、`Result.failure`ではなく`Result.success`に包まれてハンドラーに返ってくる点にも注意が必要です。

### 各Interceptorのエラー型についてApollo-iOSの内部実装を確認する必要がある。
これはしょうがない気もしますが、Interceptorにそれぞれ独自のエラーが定義されており( [例: MaxRetryInterceptor](https://github.com/apollographql/apollo-ios/blob/main/Sources/Apollo/MaxRetryInterceptor.swift#L9))、それぞれの内部実装を確認しながら、Errorをそれぞれの具体的な型にキャストして適したハンドリングをする必要があります。

## GraphQLの思想が今日のモバイルアーキテクチャと合わない

GraphQLの思想として、「画面ごとに必要な情報だけ取得して表示できる」というのがありますが、その思想のまま実装すると画面ごとに独立した構造体が定義されることになります。
クエリから最低限の構造体を自動生成しそのままViewで使用するというのは非常にシンプルに聞こえますが、このような実装をするアプリは少ないんじゃないかと思います。

上記の実装の場合の問題点としては、
アプリ全体がApollo-iOSに依存することや、自動生成された構造体が扱いづらいことです。
例えば、モックのデータが用意しづらかったり、クエリのプロパティをフラグメントで共通化すると、コードの構造が変わるところなどが挙げられます。

よって、結局はアプリで使いやすい[ドメインオブジェクト](https://qiita.com/takasek/items/70ab5a61756ee620aee6)にマッピングすることになります。その時は複数の画面でも使用できるようにオプショナルな値を持つことになると思います。
そしてマッピングのためには扱いづらい自動生成されたコードと向き合う必要があります。マッピングのテストのためのモックの用意も結構大変です。

ちなみに、モックは以下の方法で用意できます。(もっといい方法がありそうですが)

1. 自動生成されたコードに以下のプロパティがあるので、
`public private(set) var resultMap: [String: Any?]`
実行中にデバッグコンソールで`po print(resultMap)`で値を表示する。

2. 1.の出力はそのままSwiftのコードとしても機能するはずなので、`[String: Any]?`の変数としてテストターゲットに実装する。(コンパイラが型を認識しない場合は型を明示すればイケるはず)

3. 自動生成された構造体のコードに以下のようなイニシャライザがあるので、2.で生成した値から構造体を生成。
```
    public init(unsafeResultMap: [String: Any?]) {
      self.resultMap = unsafeResultMap
    }
```


## Interceptorの単体テストは難しい
[Interceptor](https://github.com/apollographql/apollo-ios/blob/main/Sources/Apollo/ApolloInterceptor.swift#L2)の処理は以下のメソッド内部に書くので、Apollo独自の型の引数のモックを用意する必要があります。
AssertionはRequestChainを継承したモックを作って、
`chain.proceedAsync()` や`chain.handleErrorAsync()`などの実行をトラップするのがいいのかなと思います。
```swift
  func interceptAsync<Operation: GraphQLOperation>(
    chain: RequestChain,
    request: HTTPRequest<Operation>,
    response: HTTPResponse<Operation>?,
    completion: @escaping (Result<GraphQLResult<Operation.Data>, Error>) -> Void)

```


## QueryとMutationで実行するメソッドが違う。
Queryはサーバーから値を取得するとき、Mutationはサーバーの値を変える時に使用されます。
Apollo-iOSではQueryのリクエストは`fetch`、Mutationのリクエストは`perform`を使用します。単純に実装すると同じような処理を2つ実装する必要がでてくるので、共通化するために、ちょっと複雑なコードを書く必要がありました。

## 自動生成したコード内で強制アンラップ[祭り](https://www.youtube.com/watch?v=yEaigqvyzqA)
これ原因でクラッシュしたことはありません。


# まとめ
GraphQLはチーム全体としては導入のメリットはかなりあると思います。

しかしApollo-iOSはまだまだ発展途上な部分が多いと思います。

とはいえApollo-iOSの開発は盛んなので、どんどん改善されていくと思います。
私もできる所から貢献して行こうと思っています。

---
