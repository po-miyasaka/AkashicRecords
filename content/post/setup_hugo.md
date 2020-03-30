+++
author = "miyashi"
title = "はじめてのHUGO"
date = "2020-03-30"
description = ""
tags = [
    "HUGO",
]
+++

---
[`hugo`](https://gohugo.io/)+[`netlify`](https://www.netlify.com/)を使用してブログを開設してみた。  

themeの[`harbor`](https://themes.gohugo.io/harbor/)は
シンプルでかっこいい&検索窓ついていたので選んだ。  
(もしかしたら検索窓はthemeに依らず後付できるかもしれない)

# 躓いたところ

以下の①〜③のステップを踏むことでテーマを適用することができると思っていたが、
なぜか表示されなかったので、④〜⑥のようなワークアラウンドを使った。

①**themes**内で以下を実行
```
$ git submodule add https://github.com/matsuyoshi30/harbor.git harbor
```
②projectルートディレクトリ内で以下の２つを実行する。
```
$ git submodule init
$ git submodule　update
```
③config.toml内に以下を記載
```
themes = "harbor"
```

  
  
④ themes/harbor内の以下のファイルをそのままprojectルートに元からあるフォルダに差し替える
```
* archetypes/
* layouts/
* static/
* content
```

⑤　themeの作成者の方が提供している、[config.tomlのテンプレート](https://themes.gohugo.io/harbor/)をそのままコピペした。

あとは、`$ hugo server -D`を実行して`http://localhost:1313/`にアクセスすれば、うまく表示されるはず。

## その他
「hugo」と「hoge」が似ているので毎回Typoする。


---