+++  
author = "miyashi"  
title = "Rubyの備忘録1 Optional周り"  
date = "2020-04-07  
description = ""  
tags = [  
    "ruby",  
    "rails",  
]  
+++  
---

**※ 一番下の追記が一番重要です。**


# ある値がｎilかどうかを確かめるには、`nil?`を使う。  

(例)
```
user.girlfriend.nil?
# girlfriendがnilの場合　=> true
# girlfriendが存在する場合　=> false
```

# Optional Chainingを使う

> `Optional Chaining`とは  
> レシーバが`nil`の場合は評価を打ち切って`nil`を返し、  
> `nil`でなければ、後続の評価をする便利な記法

Rubyで`Optional Chaining`する場合は`Optional`な値に`&`をつけるだけでOK。  
(例)
```ruby
user.girlfriend&.name
# girlfriendがnilの場合　=> nil
# girlfriendが存在する場合　=> "Tailor Swift"などの値
```

`Optional Chaining`は名前のとおり、鎖のように繋げて使えるのが便利  
(例)
```ruby
user.girlfriend&.pet&.name
```
---

# 備考
ちなみに、`""`のような空文字はnilとしては扱われないので
`empty?`を使って判定する必要がある。

# 追記
我らがRailsには`blank?`という拡張された便利プロパティが実装されており、
これは`nil, "", " ",  {} ,[]`のときに`true`になる。
しかし、全角スペースの場合は`false`になる。

