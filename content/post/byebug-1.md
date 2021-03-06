+++  
author = "miyashi"  
title = "Railsのbyebugを使ったDebug方法"  
date = "2020-05-02"  
description = ""  
tags = [  
    "ruby",  
    "rails",  
    "byebug",  
]  
+++  

------------  
# 概要
Railsでブレークポイントを貼ってデバッグするには[`byebug`](https://github.com/deivid-rodriguez/byebug)というライブラリを使えるようだ。

# byebugのインストール
`byebug`は`Rails`にデフォルトで組み込まれていて、`Gemfile`をみると`test`と`development`環境でのみ使えるようにインストールされていた。

# ブレークポイントを設定する。
ブレークさせるためには任意のコードに`byebug`の一行を追加するだけ。  
該当のコードが実行されたタイミングでプロセスが一時停止して、コンソールでデバッグコマンドを実行することができるようになる。

# 良く使いそうなコマンド
#### ■ コンソールで変数の中身を確認する
値を確認するには特に特別なコマンドは必要ない。  
コンソールに確認したい値を記述するだけ。  

確認できる変数は、ブレークしたフレームからアクセスできる変数に限る。  
アクセスできる変数の一覧を調べるコマンドは次に記載   

#### ■ アクセスできるすべての変数を確認する。
`var all`コマンドを使えば、アクセスできる変数を一覧で見ることができる。  
厳密にはブレークした時点のローカル、グローバル、インスタンス変数が一覧化される。  
このコマンドの出力は膨大なので、以下のように出力を絞ることができる。  

* ローカル変数のみを表示  
`var local`
* グローバル変数のみを表示  
`var global`
* インスタンス変数のみを表示  
`var instance`

#### ■ 使えるメソッド一覧を表示
`puts methods`で使えるメソッドを一覧化できる。  
これはbyebugの機能ではなく`puts self.methods`と同じ意味。


#### ■ スタックトレースを表示
`where`でスタックトレースを表示することができる。

#### ■ 特定のフレームに移動
`frame 3`のように`where`コマンドで出力されたフレーム番号を入力すると特定のフレームにジャンプすることができる。

#### ■ 特定のThreadに移動
RailsはリクエストごとにThreadが作られる仕様のようだ。
`thread list`でThreadの一覧を表示することができる。
`thread current`で現在ブレークしているフレームのあるThreadが表示される。
`thread switch 2`のようにThread番号を指定すればThreadを切り替えることができる。



------------

