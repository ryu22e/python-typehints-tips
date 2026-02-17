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

### 【PR】『Python実践レシピ』第2版が出ます！

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

### まず、こんな列挙型を定義する

```{code-block} python
:caption: assert_never()関数の例(1) ― assert_never_example.py

import enum

class Color(enum.Enum):
    RED = 0
    BLUE = 1
    YELLOW = 2
```

### Colorをパターンマッチで判定するコードを書く

```{code-block} python
:caption: assert_never()関数の例(2) ― assert_never_example.py

# （省略）
def get_color_name_jp(color: Color) -> str:
    match color:
        case Color.RED:
            return "赤"
        case Color.BLUE:
            return "青"
        # Color.YELLOWを書き忘れている
        case _:
            return "Unknown"

print(get_color_name_jp(Color.RED))  # 「赤」
print(get_color_name_jp(Color.BLUE))  # 「青」
print(get_color_name_jp(Color.YELLOW))  # 「黄」ではなく「Unknown」
```

### assert_never_example.pyの実行結果

```{code-block} shell
:caption: assert_never_example.pyの実行結果

% python assert_never_example.py
赤
青
Unknown
```

### このミスを防ぐにはどうすればいいか？

* テストコードを書けば防げるが……
* テストケースに漏れがあればミスに気づかない

### こんな時便利なのが`assert_never()`関数

```{code-block} python
:caption: assert_never()関数の例(3) ― assert_never_example.py

from typing import assert_never

# （省略）
def get_color_name_jp(color: Color) -> str:
    match color:
        case Color.RED:
            return "赤"
        case Color.BLUE:
            return "青"
        # Color.YELLOWを書き忘れている
        case _:
            assert_never(color)  # ここを変更

# （省略）
```

### 型チェッカーが条件指定漏れを検出してくれる

```{code-block} shell
:caption: assert_never_example.pyの型チェック結果

% mypy assert_never_example.py
assert_never_example.py:17: error: Argument 1 to "assert_never" has incompatible type "Literal[Color.YELLOW]"; expected "Never"  [arg-type]
Found 1 error in 1 file (checked 1 source file)
```

### このコードを実行するとどうなるか？

`assert_never()`関数が`AssertionError`を送出する。

```{code-block} shell
:caption: assert_never_example.pyの実行結果

% python assert_never_example.py
赤
青
Traceback (most recent call last):
  File "/***/assert_never_example.py", line 21, in <module>
    print(get_color_name_jp(Color.YELLOW))
          ~~~~~~~~~~~~~~~~~^^^^^^^^^^^^^^
  File "/***/assert_never_example.py", line 17, in get_color_name_jp
    assert_never(color)  # ここを変更
    ~~~~~~~~~~~~^^^^^^^
  File "/***/typing.py", line 2582, in assert_never
    raise AssertionError(f"Expected code to be unreachable, but got: {value}")
AssertionError: Expected code to be unreachable, but got: <Color.YELLOW: 2>
```

## 「自分自身」を表す特殊な型「`Self`型」

### `Self`型を使わなかった場合の例

```{code-block} python
:caption: Self型を使わなかった場合の例 ― self_example.py

class Foo:
    def __init__(self: "Foo") -> None:
        ...  # 省略

    def call_by_foo(self: "Foo") -> "Foo":
        """自分自身を戻り値にするメソッド"""
        return self

foo = Foo()
foo.call_by_foo()
```

### このコードは問題がないように見えるが……

```{code-block} shell
:caption: 型チェックでもエラーにならない

$ mypy self_example.py
Success: no issues found in 1 source file
```

### Fooクラスを継承してBarクラスを作ってみる

```{code-block} python
:caption: Fooクラスを継承してBarクラスを作ってみる ― self_example.py

# （省略）

class Bar(Foo):
    def __init__(self: "Bar") -> None:
        ...  # 省略

    def call_by_bar(self: "Bar") -> "Bar":
        """自分自身を戻り値にするメソッド"""
        return self

bar = Bar()
# Foo.call_by_foo()メソッドはselfを返しているのだから、
# 本来はBarクラスのメソッドを呼べるはず
bar.call_by_foo().call_by_bar()
```

### 再度型チェックしてみると……

```{code-block} bash
:caption: 問題ないはずのコードが型チェッカーではエラーになる

$ mypy self_example.py
self_example.py:23: error: "Foo" has no attribute "call_by_bar"  [attr-defined]
Found 1 error in 1 file (checked 1 source file)
```

### なぜエラーになるのか

* 型ヒント上では、`Foo.call_by_foo()`メソッドの戻り値の型が`"Foo"`になっているから
* `Foo`クラスには`call_by_bar()`メソッドは定義されていない
* 文法上は問題ないのに、型ヒントのせいでエラーになってしまっている

### どうすればいいのか

動的に型名を置き換えてくれる`Self`型を使う。

### self_example.pyに`Self`型を使ってみる

```{code-block} python
:caption: `Foo`クラス、`Bar`クラスに`Self`型を使う ― self_example.py

from typing import Self

class Foo:
    def __init__(self: Self) -> None:
        ...  # 省略

    # SelfはFooクラスとFooクラスを継承したクラスを意味する
    def call_by_foo(self: Self) -> Self:
        """自分自身を戻り値にするメソッド"""
        return self
# Barクラスも同様に書き換える
# （省略）
```

### 再度self_example.pyを型チェック

```{code-block} bash
:caption: 今度はエラーにならない

$ mypy self_example.py
Success: no issues found in 1 source file
```

## 型エイリアスの使い方を解説

## `@override`デコレーターでメソッドオーバーライドのミスを検出する

## 最後に

### まとめ

### ご清聴ありがとうございました
