+++  
author = "miyashi"  
title = "汎用的に使えるUIPickerViewを作る"  
date = "2020-05-22"  
description = ""  
tags = [  
    "Xcode",  
    "iOS",  
    "Swift",  
    "UIKit",  
    "UIPickerView",
]  
+++  
  
----  

## UIPickerViewとは
UIPickerViewはWebでいうフォームのOption要素のようなものです。
月日や都道府県などをユーザーに入力させる際に、キーボードではなく、選択肢から選んで入力することによって
誤った入力を防ぐことができます。


## UIPickerView実装の課題
iOSでのUIPickerViewを使うためには専用のプロトコルに準拠した実装をする必要があります。
普通に実装していると、特定のユースケースをプロトコルの実装に含めてしまい、  
UIPickerViewを使ったコンポーネントを実装するために似たようなコードを量産することになってしまいます。  


## 課題の解決案  
UIPickerViewをうまくラップすることで、似たようなコードを量産することなく、ユースケースごとに変化する部分だけを実装して、共通部分をまとめることができます。  
以下に通常の実装方法と改善案を比較します。  
今回は52パターンのトランプから一つを選ぶコードを実装します。
以下のコードはコピペでも動作するはずです。Playground等で試してみてください。

### 通常の実装方法
まずは普通にプロトコルに準拠したコードを書いてみる。  

```javascript
class TrumpPickerView: UIPickerView, UIPickerViewDelegate, UIPickerViewDataSource {

    // 問題点① データをViewに直接書くことで、他のユースケースで使えなくなる。
    var type: [String] {
        ["クローバー", "スペード", "ハート", "ダイヤ"]
    }
    
    var numbers: [String] {
        ["A", "2", "3", "4", "5", "6", "7", "8", "9", "10", "J", "Q", "K"]
    }

    override init(frame: CGRect) {
        super.init(frame: frame)
        delegate = self
        dataSource = self
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    // 以下PickerView専用のプロトコル

    func numberOfComponents(in pickerView: UIPickerView) -> Int {
        2　// 問題点②　データが増えたときにこの数字も修正する必要がでてくる。
    }
    
    func pickerView(_ pickerView: UIPickerView,
                     numberOfRowsInComponent component: Int) -> Int {

        // 問題点③　データが増えたときにcaseを増やす必要がある。
        switch component {
        case 0:
            return type.count
        case 1:
            return numbers.count
        default:

        // 問題点④　使わないのにdefaultが気持ち悪い。
            return 0
        }
    }
    
    func pickerView(_ pickerView: UIPickerView,
                     titleForRow row: Int,
                     forComponent component: Int) -> String? {
                         
        // 問題点⑤　3番同様データが増えたときにcaseを増やす必要がある。
        switch component {
        case 0:
            return type[row]
        case 1:
            return numbers[row]
        default:

        // 問題点⑥　4番同様defaultが気持ち悪い。
            return ""
        }
    }

    func pickerView(_ pickerView: UIPickerView,
                     didSelectRow row: Int,
                      inComponent component: Int) {
        // 選んだ結果を利用するための実装も必要
        print(row)
        print(component)
    }
}


```

通常の実装の利用例
```javascript
let pickerViewFrame = CGRect.init(x: 0, y: 0, width: 300, height: 200)
let v = TrumpPickerView(frame: pickerViewFrame)
```

### 改善案の実装
```javascript
class GeneralPickerView: UIPickerView,
                         UIPickerViewDelegate,
                        　UIPickerViewDataSource {
    
    var dataArray: [[String]]
    typealias SelectedHandler = ([String]) -> ()
    var selectedHandler: SelectedHandler
    
    init(frame: CGRect,
         dataArray: [[String]], // 表示するデータはコンストラクタで渡す。
         selectedHandler: @escaping SelectedHandler) {
        self.dataArray = dataArray
        self.selectedHandler = selectedHandler
        super.init(frame: frame)
        delegate = self
        dataSource = self
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    
    // 以下PickerView専用のプロトコル

    func numberOfComponents(in pickerView: UIPickerView) -> Int {
        dataArray.count
    }
    
    func pickerView(_ pickerView: UIPickerView,
                     numberOfRowsInComponent component: Int) -> Int {
        dataArray[component].count
    }
    
    func pickerView(_ pickerView: UIPickerView,
                     titleForRow row: Int,
                     forComponent component: Int) -> String? {
        dataArray[component][row]
    }
    
    func pickerView(_ pickerView: UIPickerView,
                     didSelectRow _: Int, inComponent _: Int) {

        // 変更がある度にselectedHandlerにデータが送られる。
        let selectedData: [String] = dataArray.enumerated().map { component, data in
            let selectedRow = pickerView.selectedRow(inComponent: component)
            return data[selectedRow]
        }
        
        selectedHandler(selectedData)
    }
}
```
改善案の利用例
```javascript
let pickerViewFrame = CGRect.init(x: 0, y: 0, width: 300, height: 200)
        
let dataArray =  [
    ["クローバー", "スペード", "ハート", "ダイヤ"],
    ["A", "2", "3", "4", "5", "6", "7", "8", "9", "10", "J", "Q", "K"]
]
        
let selectedHandler: GeneralPickerView.SelectedHandler = { selected in

    // 選択されたデータは一次元配列として帰ってくる
    print(selected[0]) // ハート
    print(selected[1]) // 3
}
        
let v = GeneralPickerView(
            frame: pickerViewFrame,
            dataArray: dataArray,
            selectedHandler: selectedHandler)

```

都道府県を選ぶ等、別のユースケースでも利用できる。
```javascript
let pickerViewFrame = CGRect.init(x: 0, y: 0, width: 300, height: 200)
        
// データが一次元配列でもOK
let dataArray =  [
    ["北海道", "青森", "東京", "埼玉",　"長野", "鳥取", "etc"]
]
        
let selectedHandler: GeneralPickerView.SelectedHandler = { selected in
    print(selected[0])　// 鳥取
}
        
let v = GeneralPickerView(
            frame: pickerViewFrame,
            dataArray: dataArray,
            selectedHandler: selectedHandler)
```

## まとめ  
ここまで完結に書くことができる理由はUIPickerViewは基本的にdataSouceが`[[String]]`だからこそ共通化できています。
UITableViewも同じようなプロトコルに準拠しますが、dataの型が定まらないため、同じようにはいきません。

このコードはSwift4,5であれば動くはずなので、ご利用くださいませ。

## 備考
保持しておいた前回の状態を初期値に反映する機能はこの記事では実装していません。
もし実装するのであれば、コンストラクタで初期値を渡すのがいいでしょう。
----  