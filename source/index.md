# 型ヒントって“型”をつけるだけだと思ってない？ 型ヒントをさらに使いこなすための実践ノウハウ集

```{raw} html

<a rel="license" href="http://creativecommons.org/licenses/by/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by/4.0/88x31.png" /></a>
<small>This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/4.0/">Creative Commons Attribution 4.0 International License</a>.</small>
```

## はじめに

### 自己紹介

* Ryuji Tsutsui@ryu22e
* さくらインターネット株式会社所属
* 2011年頃からPythonを使い続けている（主にDjango）
* Python Boot Camp、Shonan.pyなどコミュニティ活動もしています
* 著書（共著）：『[Python実践レシピ](https://gihyo.jp/book/2022/978-4-297-12576-9)』

### 【PR】『Python実戦レシピ』第2版が出ます！

<https://amzn.asia/d/0jaaWbIX>

```{figure} _static/img/qrcode_www.amazon.co.jp.png
:alt: Amazon URLのQRコード

3月16日発売！
```

### 今​日話すこと

* Pythonの型ヒントの便利な機能の紹介

### 今日話さないこと

* Pythonの型ヒントそのものの解説

### この​トークの​対象者

* Pythonの基礎的な文法を知っている人
* Pythonの型ヒントを（何となくでいいので）知っている人

### この​トークで​得られる​こと

* 以下に関する知見
  * 型ヒントでプログラマのどんなミスを防げるのか

### トークの構成

* 型ヒントについておさらい
* if文やパターンマッチでの条件指定漏れを検出してくれる`assert_never()`関数
* 「自分自身」を表す特殊な型「`Self`型」
* 型エイリアスの使い方を解説
* `@override`デコレーターでメソッドオーバーライドのミスを検出する

## 型ヒントについておさらい

### 型ヒントとは

* 2015年9月にリリースされたPython 3.5に追加された機能
* あくまで静的解析ツールに渡す情報
  * ツールの例：Mypy、Pyright
* 実行時に型チェックは行わない

### コード例

```{code-block} python
:caption: 型ヒントの例 ― example.py

def repeat_message(message: str, n: int) -> str:
    return message * n

print(repeat_message("Hello!", 3))  # OK
# nに整数以外を渡しているのでNG
print(repeat_message("Hello!", "3"))
# 整数型の変数に文字列を渡しているのでNG
result: int = repeat_message("Hello!", 3)
```

### 型チェック実行結果

```{code-block} shell
:caption: example.pyの型チェック実行結果

% mypy example.py
example.py:6: error: Argument 2 to "repeat_message" has incompatible type "str"; expected "int"  [arg-type]
example.py:8: error: Incompatible types in assignment (expression has type "str", variable has type "int")  [assignment]
Found 2 errors in 1 file (checked 1 source file)
```

### たぶん、ここまではみんな知っているはず

型チェックは、もっと便利な機能があります！

## if文やパターンマッチでの条件指定漏れを検出してくれる`assert_never()`関数

## 「自分自身」を表す特殊な型「`Self`型」

## 型エイリアスの使い方を解説

## `@override`デコレーターでメソッドオーバーライドのミスを検出する

## 最後に

### まとめ

### ご清聴ありがとうございました
