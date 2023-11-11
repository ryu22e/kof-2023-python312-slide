#######################
Python 3.12の新機能紹介
#######################

Ryuji Tsutsui

関西オープンフォーラム2023セミナー資料

.. raw:: html

   <a rel="license" href="http://creativecommons.org/licenses/by/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by/4.0/88x31.png" /></a><br /><small>This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/4.0/">Creative Commons Attribution 4.0 International License</a>.</small>

はじめに
========

自己紹介
--------

* Ryuji Tsutsui @ryu22e
* `一般社団法人PyCon JP Association運営メンバー <https://www.pycon.jp/committee/members.html#ryuji-tsutsui>`_
* `株式会社hokan <https://www.corp.hkn.jp/>`_ 所属
* Python歴は12年くらい（主にDjango）
* Python Boot Camp、Shonan.py、GCPUG Shonanなどコミュニティ活動もしています
* 著書（共著）：『 `Python実践レシピ <https://gihyo.jp/book/2022/978-4-297-12576-9>`_ 』

PR
--

.. figure:: pyconjp-booth.*
   :alt: PyCon JP Associationブース

   今日ブース（エリアB No.11）で「Python Boot Camp」、「PyLadies Caravan」の宣伝をやっています。

今日話したいこと
----------------

* Python 3.12の新機能を紹介します

セミナーの構成
--------------

* 新しい構文
* パフォーマンスの改善
* デバッグ・モニタリング方法の改善
* その他新機能

このセミナーから得られるもの
----------------------------

* Pythonがどんな進化を遂げているかが分かる
* 実際にPython 3.12を使う際に役立つ情報が得られる

新しい構文
==========

* PEP 695 型ヒントの新構文
* PEP 701 f-stringがネスト可能に

PEP 695 型ヒントの新構文
------------------------

* ジェネリッククラス、ジェネリック関数に関する構文
* 型エイリアスに関する構文

型ヒントとは
------------

* Python 3.5で追加された機能
* コードに型ヒントを付けることで型チェッカー（Mypy、Pyrightなど）で型ヒントと矛盾するコードを書いていないか検証できる
* 実行時に型チェックはしてくれない

型ヒントの例
------------

.. revealjs-code-block:: python

    def example(message: str, count: int) -> str:
        return message * count

    print(example("hello", 3))  # OK
    print(example("hello", "3"))  # NG

型ヒントの例（続き）
--------------------

.. figure:: mypy-example.*
   :alt: Mypyの実行結果

   Mypyの実行結果

ジェネリッククラス、ジェネリック関数とは
----------------------------------------

特定の型に依存しないクラス、関数を定義できる。

具体的な型名を書かず抽象的な書き方にして、特定の型に依存しないクラスや関数を定義する。

ジェネリッククラスの例
----------------------

.. revealjs-code-block:: python

    from typing import Generic, TypeVar

    T = TypeVar('T')  # 抽象的な型名を定義

    # このクラスがジェネリッククラスであることを宣言する
    class Example(Generic[T]):
        def __init__(self, value: T) -> None:
            self.value = value

        def get_value(self) -> T:
            return self.value

        def get_type(self) -> type:
            return type(self.value)

    # クラス名の右に角括弧で具体的な型名を囲む
    example1 = Example[int](1)
    # 1 <class 'int'> が出力される
    print(example1.get_value(), example1.get_type())
    example2 = Example[str]('hello')
    # hello <class 'str'> が出力される
    print(example2.get_value(), example2.get_type())

ジェネリック関数の例
--------------------

.. revealjs-code-block:: python

    from typing import Sequence, TypeVar

    T = TypeVar('T')  # 抽象的な型名を定義

    def first(l: Sequence[T]) -> T:
        return l[0]

    print(first([1, 2, 3]))  # 1 が出力される
    print(first("python"))  # p が出力される

PEP 695登場以前のジェネリッククラス、ジェネリック関数の面倒な点
---------------------------------------------------------------

* 毎回 ``T = TypeVar('T')`` を書くのが面倒
* ジェネリッククラスの場合、 ``Generic`` を継承する必要があるのが面倒

PEP 695でジェネリッククラス、ジェネリック関数はどう変わったか
-------------------------------------------------------------

``T = TypeVar('T')`` や ``Generic`` を書かない新構文が追加された。

Python 3.12でのジェネリッククラスの例
-------------------------------------

.. revealjs-code-block:: python

    class Example[T]:  # 角括弧でTを囲む（新構文）
        def __init__(self, value: T) -> None:
            self.value = value

        def get_value(self) -> T:
            return self.value

        def get_type(self) -> type:
            return type(self.value)

    example1 = Example[int](1)
    print(example1.get_value(), example1.get_type())
    example2 = Example[str]('hello')
    print(example2.get_value(), example2.get_type())

Python 3.12でのジェネリック関数の例
-----------------------------------

.. revealjs-code-block:: python

    from typing import Sequence

    def first[T](l: Sequence[T]) -> T:  # 関数名の右に角括弧でTを囲む（新構文）
        return l[0]

    print(first([1, 2, 3]))
    print(first("python"))

PEP 695で型エイリアスはどう変わったか
-------------------------------------

type文が追加された。

.. revealjs-code-block:: python

   >>> # Python 3.11
   >>> from typing import TypeAlias
   >>> Point: TypeAlias = tuple[float, float]
   >>> # Python 3.12
   >>> type Point = tuple[float, float]
   >>> type Point[T] = tuple[T, T]  # ジェネリックの構文も使える

type文と既存の型エイリアスの違い
--------------------------------

type文は遅延評価なので、type文の中にまだ定義されていない型を指定できる。

.. revealjs-code-block:: python

   >>> type Foo = int | Bar  # Barはこの時点では定義されていないがエラーにならない
   >>> type Bar = str
   >>> # Python 3.11までの書き方だとエラーになる
   >>> from typing import TypeAlias
   >>> Foo: TypeAlias = int | Bar
   Traceback (most recent call last):
     File "<stdin>", line 1, in <module>
   NameError: name 'Bar' is not defined

PEP 701 f-stringがネスト可能に
------------------------------

f-stringとは
------------

以下公式ドキュメントの引用。

    フォーマット済み文字リテラル (短くして f-string とも呼びます) では、文字列の頭に f か F を付け、式を {expression} と書くことで、 Python の式の値を文字列の中に入れ込めます。

https://docs.python.org/ja/3/tutorial/inputoutput.html#formatted-string-literals

f-stringの例
------------

.. revealjs-code-block:: python

   >>> name = "Python"
   >>> f"Hello, {name}!"  # 変数を埋め込める
   'Hello, Python!'
   >>> from datetime import datetime
   >>> f"Today is {datetime.now():%Y-%m-%d}"  # 式を埋め込める
   'Today is 2023-11-11'

公式ドキュメントに「式を埋め込めます」とは書いているものの…
-----------------------------------------------------------

（Python 3.11までは）厳密に言うと書けない式もある。

.. revealjs-code-block:: python

    >>> d = {"foo": 1, "bar": 2}
    >>> # "{d[" までを文字列を認識してしまう（一応 f"{d['foo']}" で回避できる）
    >>> f"{d["foo"]}"
      File "<stdin>", line 1
        f"{d["foo"]}"
              ^^^
    SyntaxError: f-string: unmatched '['
    >>> # バックスラッシュも使えない
    >>> f"{'\n'.join(['foo', 'bar'])}"
      File "<stdin>", line 1
        f"{'\n'.join(['foo', 'bar'])}"
                                      ^
    SyntaxError: f-string expression part cannot include a backslash

f-stringの仕様はどうなっている？
--------------------------------

f-string導入に関連するドキュメント `PEP 498 <https://peps.python.org/pep-0498/>`_ では具体的な仕様が定義されていなかった。

    The exact code used to implement f-strings is not specified.

    https://peps.python.org/pep-0498/#code-equivalence

PEP 701でどう変わったか
-----------------------

パーサーが改善され、f-stringにどんな式でも埋め込めるようになった。

.. revealjs-code-block:: python

    >>> d = {"foo": 1, "bar": 2}
    >>> f"{d["foo"]}"
    '1'
    >>> f"{'\n'.join(['foo', 'bar'])}"
    'foo\nbar'
    >>> f"{f"{f"{f"{f"{f"{1+1}"}"}"}"}"}"
    '2'
    >>> def example(s):
    ...     return f"result: {s}"
    ...
    >>> import random
    f"{example(f"{random.randint(1, 10)}")}"
    'result: 4'

PEP 701でどう変わったか（続き）
-------------------------------

f-stringの途中で改行やコメントも入れられる。

VS Codeのシンタックスハイライトも効く。

.. figure:: pep701_example_py.*
   :alt: VS Codeのシンタックスハイライト

パフォーマンスの改善
====================

* PEP 684 インタプリタごとに固有のGILが使われるように変更
* PEP 709 内包表記のパフォーマンス改善

PEP 684 インタプリタごとに固有のGILが使われるように変更
-------------------------------------------------------

GIL（global interpreter lock）とは
----------------------------------

以下公式ドキュメントの引用。

    CPython インタプリタが利用している、一度にPythonのバイトコードを実行するスレッドは一つだけであることを保証する仕組みです。

https://docs.python.org/ja/3/glossary.html#term-global-interpreter-lock

GILの例
-------

以下のコードはマルチスレッドを使っているにも関わらず、 ``print_hello()`` 関数が同時に実行されない。

.. revealjs-code-block:: python

    import threading

    def print_hello():  # この関数はGILにより同時に実行されない
        print("Hello!")

    threads = []
    for _ in range(3):
        thread = threading.Thread(target=print_hello)
        threads.append(thread)
        thread.start()

    for thread in threads:
        thread.join()

PEP 684でどう変わったか
-----------------------

- インタプリタが固有のGILを持つサブインタプリタを作成できるようになった
- つまり、異なるサブインタプリタ間ではGILが起こらない

これで問題は解決した、と言いたいところだが…
-------------------------------------------

* 今回追加されたのはC言語から利用できる ``Py_NewInterpreterFromConfig()`` 関数。Pythonコードからは利用できない
*  `PEP 554 <https://peps.python.org/pep-0554/>`_ で追加される ``interpreters`` モジュールを使うことで、初めてPEP 684の恩恵を受けられる（Python 3.13で実装予定）

Python 3.13のリリース予定日は？
-------------------------------

PEP 719によると最終版は2024年10月1日リリース予定。気長に待とう！

https://peps.python.org/pep-0719/

``Python 3.13.0a1`` で ``interpreters`` モジュールを試してみると
----------------------------------------------------------------

未実装だったので試せず😢

.. revealjs-code-block:: shell

    >>> import interpreters
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    ModuleNotFoundError: No module named 'interpreters'

PEP 709 内包表記のパフォーマンス改善
------------------------------------

内包表記とは
------------

内包表記とは、リスト、辞書、セットを一行で作成できる構文。

.. revealjs-code-block:: python

   def example(numbers):
       """numbers: 整数のリスト"""
       # リストの各要素に"No."を付けたリストを作成する
       return [f"No.{i}" for i in numbers]

同じことをfor文でやるとこうなる
-------------------------------

.. revealjs-code-block:: python

   def example(numbers):
       results = []
       for i in numbers:
           results.append(f"No.{i}")
        return results

PEP 709登場前の内包表記の問題
-----------------------------

内包表記は実行ごとに新しい関数オブジェクトを作成するため、パフォーマンスに問題がある。

PEP 709でどう変わったか
-----------------------

関数オブジェクトを作成していた部分がインライン化された。

違いを確認してみる
------------------

Pythonの ``dis`` という標準モジュールを使って、CPythonのバイトコードを逆アセンブルしてみる。

``dis`` モジュールの使い方
--------------------------

.. revealjs-code-block:: shell

   $ python -m dis {Pythonコードのファイル名}

対象のPythonコード
------------------

以下の ``pep709_example.py`` を用意する。

.. revealjs-code-block:: python

   def example(numbers):
       return [f"No.{i}" for i in numbers]

Python 3.11の場合
-----------------

.. figure:: pep709-example-before.*
   :alt: ``python3.11 -m dis pep709_example.py`` の実行結果（一部抜粋）

   ``python3.11 -m dis pep709_example.py`` の実行結果（一部抜粋）

Python 3.12の場合
-----------------

.. figure:: pep709-example-after.*
   :alt: ``python3.12 -m dis pep709_example.py`` の実行結果（一部抜粋）

   ``python3.12 -m dis pep709_example.py`` の実行結果（一部抜粋）

デバッグ・モニタリング方法の改善
================================

* PEP 669 ``sys.monitoring`` の追加
* Linux perfのCPythonサポート（PEP番号はなし）
* エラーメッセージの改善（PEP番号はなし）

PEP 669 ``sys.monitoring`` の追加
---------------------------------

PEP 669登場前
-------------

* 従来のプロファイラー（ ``profile`` 、 ``cProfile`` など）はパフォーマンスに重大な問題を引き起こすことがあった
* 稼働中のアプリケーションにいつ、どの関数、メソッドが呼び出されているか調べるには高速なプロファイラーが必要

PEP 669 ``sys.monitoring`` の追加
---------------------------------

* ``sys.monitoring`` はCPython上で動作する高速なプロファイラー
* 関数やメソッド呼び出しなどのタイミングで呼び出すフック関数を登録できる

``sys.monitoring`` の主な使い方(1)
----------------------------------

以下で計測の準備。

* ``sys.monitoring.use_tool_id()`` : ツールIDを登録
* ``sys.monitoring.register_callback()`` : フック関数を登録
* ``sys.monitoring.set_events()`` : 監視するイベントを登録

``sys.monitoring`` の主な使い方(2)
----------------------------------

以下で計測対象コードの実行。

* 組み込み関数の ``compile()`` 、 ``exec()`` 関数を使ってコードを実行

``sys.monitoring`` の主な使い方(3)
----------------------------------

以下で計測の終了（これを書かないと計測対象以外のコード実行も計測してしまう）。

* ``sys.monitoring.set_events()`` : 監視するイベントの登録解除
* ``sys.monitoring.free_tool_id()`` : ツールIDを解放

``sys.monitoring`` のサンプルコード
-----------------------------------

前述の関数を使ったサンプルコード。

https://gist.github.com/ryu22e/87411710176fd1d0ba0f95b0e5f9d6e0

Linux perfのCPythonサポート（PEP番号はなし）
--------------------------------------------

Linux perfとは
--------------

* Linuxカーネル2.6.31以降で利用可能なパフォーマンス分析ツール
* プログラムのどこでどれだけCPUを使っているか計測できる

Linux perfの使用例
------------------

``python my_script.py`` を実行したときのCPU使用率を計測するには以下のコマンドを実行して ``perf.data`` を出力する。

.. revealjs-code-block:: shell

   $ perf record -F 9999 -g -o perf.data python my_script.py

.. revealjs-break::

出力された ``perf.data`` を ``perf report`` コマンドで確認する。

.. revealjs-code-block:: shell

   $ perf report --stdio -n -g

実際に計測してみる
------------------

計測対象のコードは以下の通り。

.. revealjs-code-block:: python

    def foo(n):
        result = 0
        for _ in range(n):
            result += 1
        return result

    def bar(n):
        foo(n)

    def baz(n):
        bar(n)

    if __name__ == "__main__":
        baz(1000000)

Python 3.11でLinux perfを使った場合
-----------------------------------

以下を参照:

https://gist.github.com/ryu22e/0f5f52712194e4e38c211958288e6267#file-python3-11-md

Python 3.12でLinux perfを使った場合
-----------------------------------

以下を参照:

https://gist.github.com/ryu22e/0f5f52712194e4e38c211958288e6267#file-python3-12-md

Python 3.12でperfプロファイリングを有効にするには
-------------------------------------------------

1. 環境変数 ``PYTHONPERFSUPPORT=1`` を設定して実行
2. ``-X perf`` オプションを付けて実行
3. ``sys`` モジュールが提供するAPIを使ったコードを入れる（次のスライド参照）

sysモジュールを使ってperfプロファイリングを有効にする例
-------------------------------------------------------

.. revealjs-code-block:: python

    import sys

    sys.activate_stack_trampoline("perf")  # 計測開始
    do_profiled_stuff()  # 計測中
    sys.deactivate_stack_trampoline()  # 計測終了

    non_profiled_stuff()

    activate_stack_trampoline

エラーメッセージの改善（PEP番号はなし）
---------------------------------------

Pythonエラーメッセージは改善を続けている
----------------------------------------

Python 3.10（2021年10月4日リリース）で「Better error messages」が追加された。

CPythonのパーサーの改善により、エラーを指摘しているけどどう直していいか分からないメッセージが減った。

https://docs.python.org/ja/3/whatsnew/3.10.html#better-error-messages

Python 3.10でのエラーメッセージの例
-----------------------------------

Python 3.9

.. revealjs-code-block:: python

    >>> if a
      File "<stdin>", line 1
        if a
            ^
    SyntaxError: invalid syntax

Python 3.10

.. revealjs-code-block:: python

    >>> # :が足りないと指摘してくれる
    >>> if a
      File "<stdin>", line 1
        if a
            ^
    SyntaxError: expected ':'

「Better error messages」についてもっと詳しく知りたい人は
---------------------------------------------------------

Gihyoさんの連載『Python Monthly Topics』で鈴木たかのりさんが執筆された『Python 3.10から導入されたBetter error messagesの深掘り』がおすすめ。

https://gihyo.jp/article/2022/12/monthly-python-2212

Python 3.12でのエラーメッセージの例(1)
--------------------------------------

``NameError`` 時に標準モジュールと同じ名前だとimportを促すメッセージが出てくる。

Python 3.11

.. revealjs-code-block:: python

    >>> sys
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    NameError: name 'sys' is not defined

Python 3.12

.. revealjs-code-block:: python

    >>> sys
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    NameError: name 'sys' is not defined. Did you forget to import 'sys'?

Python 3.12でのエラーメッセージの例(2-1)
----------------------------------------

``NameError`` 時にクラスの属性と同じ名前だと ``self.属性名`` を書くよう促すメッセージが出てくる。

Python 3.11

.. revealjs-code-block:: python

    >>> class Example:
    ...     def __init__(self):
    ...             self.foo = 1
    ...     def hello(self):
    ...             a = foo
    ...
    >>> Example().hello()
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
      File "<stdin>", line 5, in hello
    NameError: name 'foo' is not defined

Python 3.12でのエラーメッセージの例(2-2)
----------------------------------------

Python 3.12

.. revealjs-code-block:: python

    >>> class Example:
    ...     def __init__(self):
    ...             self.foo = 1
    ...     def hello(self):
    ...             a = foo
    ...
    >>> Example().hello()
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
      File "<stdin>", line 5, in hello
    NameError: name 'foo' is not defined. Did you mean: 'self.foo'?

Python 3.12でのエラーメッセージの例(3-1)
----------------------------------------

import文の書き順の間違いを指摘してくれる。

Python 3.11

.. revealjs-code-block:: python

    >>> # importとfromの順番が逆
    >>> import os from environ
      File "<stdin>", line 1
        import os from environ
                  ^^^^
    SyntaxError: invalid syntax

Python 3.12でのエラーメッセージの例(3-2)
----------------------------------------

Python 3.12

.. revealjs-code-block:: python

    >>> import os from environ
      File "<stdin>", line 1
        import os from environ
        ^^^^^^^^^^^^^^^^^^^^^^
    SyntaxError: Did you mean to use 'from ... import ...' instead?

Python 3.12でのエラーメッセージの例(4-1)
----------------------------------------

import文のtypoを指摘してくれる。

Python 3.11

.. revealjs-code-block:: python

    >>> # chainmapはChainMapのtypo
    >>> from collections import chainmap
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    ImportError: cannot import name 'chainmap' from 'collections' (/****/__init__.py)

Python 3.12でのエラーメッセージの例(4-2)
----------------------------------------

Python 3.12

.. revealjs-code-block:: python

    >>> from collections import chainmap
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    ImportError: cannot import name 'chainmap' from 'collections' (/****/__init__.py).
    Did you mean: 'ChainMap'?

約1分休憩
=========

給水します🚰

大阪でお勧めの美味しい店募集
----------------------------

明日1日フリーなので、いい店があったら行きたいです。このあとブース（エリアB No.11）にいるのでぜひ教えてください🙇

その他新機能
============

* PEP 692 ``**kwargs`` 引数に付けられる型ヒントに関する改善
* PEP 698 メソッドをオーバーライドする際のtypoを防ぐ ``override`` デコレーターの登場
* PEP 688 Pythonコードからバッファプロトコルにアクセスできるように

※ PEP 688はかなりニッチな機能なので時間がなければ省略

PEP 692 ``**kwargs`` 引数に付けられる型ヒントに関する改善
---------------------------------------------------------

Pythonの関数の引数指定方法
--------------------------

Pythonの関数の引数指定方法は以下の2つ。

.. revealjs-code-block:: python

    >>> def example(a, b):
    ...     ...
    ...
    >>> example(1, 2)  # 関数定義に書かれた順番に値を指定（位置引数）
    >>> example(a=1, b=2)  # 引数名と値をセットで指定（キーワード引数）

``**kwargs`` 引数とは
---------------------

* 引数名の先頭に ``**`` を付けると、どんなキーワード引数でも受け付ける引数になる
* 関数内では ``kwargs`` を辞書型の値として扱う
* 読み方は「クワーグス」（参考URL: https://youtu.be/WcTXxX3vYgY?t=9）
* keyword argumentsの略
* 文法的には先頭に ``**`` があればどんな名前でもいいが、慣習的に ``kwargs`` と書く

``**kwargs`` 引数の例
---------------------

.. revealjs-code-block:: python

    >>> def example(**kwargs):
    ...     print(kwargs)
    ...
    >>> example(foo=1, bar=2)
    {'foo': 1, 'bar': 2}
    >>> example(last_name="Tsutsui", first_name="Ryuji")
    {'last_name': 'Tsutsui', 'first_name': 'Ryuji'}

Python 3.11までの ``**kwargs`` 引数への型ヒントの付け方
-------------------------------------------------------

すべてのキーワード引数で同じ型を指定することしかできなかった。

.. revealjs-code-block:: python

    def example(**kwargs: str) -> None:
        ...

    example(foo="test1", bar="test2")  # すべてのキーワード引数が文字列なのでOK
    example(foo="test1", bar=2)  # bar引数が整数値なのでNG

PEP 692でどう変わったか
-----------------------

``typing.TypedDict`` と ``typing.Unpack`` を組み合わせて ``**kwargs`` 引数に型ヒントを付けられるようになった。

.. revealjs-code-block:: python

    from typing import TypedDict, Unpack, assert_type

    class Book(TypedDict):
        title: str
        price: int

    def add_book(**kwargs: Unpack[Book]) -> None:
        assert_type(kwargs, Book)  # エラーにならない

    add_book(title="Python実践レシピ", price=2790)  # OK
    add_book(
        title="Python実践レシピ",
        price="2,970円（本体2,700円＋税10%）",  # NG
    )

``**kwargs`` 引数に ``TypedDict`` を指定することによるメリット(1)
-----------------------------------------------------------------

typoを防げる。

.. revealjs-code-block:: python

    from typing import TypedDict, Unpack, assert_type

    class Book(TypedDict):
        title: str
        price: int

    def add_book(**kwargs: Unpack[Book]) -> None:
        assert_type(kwargs, Book)  # エラーにならない

    # pricaはBookクラスに存在しないのでエラーになる
    add_book(title="Python実践レシピ", prica=2790)

``**kwargs`` 引数に ``TypedDict`` を指定することによるメリット(2-1)
-------------------------------------------------------------------

「引数の指定を省略している場合」と「明示的に引数を指定している場合」を区別できる。

.. revealjs-code-block:: python

    class Auth:
        """認証情報"""
        ...

    def request(url: str, auth: Auth | None = None) -> None:
        """url引数で指定したURLにリクエストを送る。
        認証が必要な場合はauth引数に認証情報を指定"""
        ...

    # auth引数を省略している
    request("https://example.com")
    # auth引数に明示的にNoneを指定している
    request("https://example.com", auth=None)

``**kwargs`` 引数に ``TypedDict`` を指定することによるメリット(2-2)
-------------------------------------------------------------------

この問題は `HTTPX <https://www.python-httpx.org/>`_ というライブラリで実際に議論の対象になった。
`PEP 692のドキュメント <https://peps.python.org/pep-0692/>`_ のMotivationの項にもこの問題が書かれている。

https://github.com/encode/httpx/issues/1384

``**kwargs`` 引数に ``TypedDict`` を指定することによるメリット(2-3)
-------------------------------------------------------------------

明示的に空の認証情報を指定する際のクラスを用意する手もあるが、関数のインターフェースが変わってしまう。

.. revealjs-code-block:: python

    # Authのコードは省略
    class Empty(Auth):
       ...

    EMPTY = Empty()

    def request(url: str, auth: Auth | None | Empty = None) -> None:
        ...

    # auth=Noneをauth=EMPTYに書き換える
    request("https://example.com", auth=EMPTY)

``**kwargs`` 引数に ``TypedDict`` を指定することによるメリット(2-4)
-------------------------------------------------------------------

``**kwargs`` 引数を使うと解決する。

.. revealjs-code-block:: python

    from typing import TypedDict, Unpack, NotRequired

    class OtherParams(TypedDict):
        auth: NotRequired[Auth]  # 入力必須ではない場合はNotRequiredを指定

    # authの代わりに**kwargsを指定する
    def request(url: str, **kwargs: Unpach[OtherParams]) -> None:
        if "auth" not in kwargs:
            print("auth引数が省略された場合の処理が呼ばれた")
        elif "auth" in kwargs and kwargs["auth"] is None:
            print("auth引数に明示的にNoneを渡した場合の処理が呼ばれた")

PEP 698 メソッドをオーバーライドする際のtypoを防ぐ ``override`` デコレーターの登場
----------------------------------------------------------------------------------

Pythonでメソッドをオーバーライドするには
----------------------------------------

メソッド名、引数を一致させる。

.. revealjs-code-block:: python

    >>> class Base:
    ...     def say_hello(self, name):
    ...         print("Hello, " + name)
    ...
    >>> class Example(Base):
    ...     def say_hello(self, name):
    ...         print("こんにちは、" + name)
    ...
    >>> example = Example()
    >>> example.say_hello("Taro")
    こんにちは、Taro

typoがあるとオーバーライドできない
----------------------------------

.. revealjs-code-block:: python

    >>> class Example(Base):
    ...     def say_hallo(self, name):  # halloはtypo
    ...         print("こんにちは、" + name)
    ...
    >>> example = Example()
    >>> example.say_hello("Taro")  # 基底クラスBaseのsay_helloメソッドが呼ばれる
    Hello, Taro

``override`` デコレーターを使うとどうなるか
-------------------------------------------

``typing.override`` デコレーターを付けることでtypoしても型チェッカーが教えてくれる。

.. revealjs-code-block:: python

    from typing import Self, override

    class Base:
        def say_hello(self: Self, name: str) -> None:
            print("Hello, " + name)

    class Example(Base):
        @override
        def say_hallo(self: Self, name: str) -> None:  # halloはtypo
            print("こんにちは、" + name)

typoしているコードを型チェックすると
------------------------------------

該当箇所がエラーになる。

.. figure:: pep698_example.*
   :alt: Mypyの実行結果

   Mypyの実行結果

PEP 688 Pythonコードからバッファプロトコルにアクセスできるように
----------------------------------------------------------------

（残り4分切っていたらスキップしてまとめに入る）

バッファプロトコルとは何か、の前にプロトコルとは何か
----------------------------------------------------

特定の動作を実現するために必要なオブジェクトの実装に関するルール。

プロトコルの例(1)
-----------------

``len()`` 関数は引数の長さを返す組み込み関数。リスト、タプルなら要素数、文字列なら文字数を返す。

.. revealjs-code-block:: python

    >>> len([1, 2, 3])  # リストなら要素数
    3
    >>> len((1, 2, 3))  # タプルなら要素数
    3
    >>> len('Python')  # 文字列なら文字数
    6

プロトコルの例(2)
-----------------

``len()`` 関数には何でも渡せるわけではない。

.. revealjs-code-block:: python

    >>> len(1)
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    TypeError: object of type 'int' has no len()
    >>> len(None)
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    TypeError: object of type 'NoneType' has no len()

.. revealjs-break::

.. revealjs-code-block:: python

    >>> class Example:
    ...     ...
    ...
    >>> len(Example())
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    TypeError: object of type 'Example' has no len()

プロトコルの例(3)
-----------------

どんなオブジェクトなら渡していいかを決めるのがプロトコル。

``len()`` 関数の場合は ``__len__()`` メソッドを実装しているオブジェクトなら渡していい。

.. revealjs-code-block:: python

    >>> class Example:
    ...     def __len__(self):
    ...         return 123
    ...
    >>> len(Example())
    123

プロトコルについてもっと詳しく知りたい人は
------------------------------------------

Takayuki ShimizukawaさんのPyCon JP 2017の発表「len()関数がオブジェクトの長さを手にいれる仕組み」がおすすめ。

https://pycon.jp/2017/ja/schedule/presentation/22/

それではバッファプロトコルとは何か
----------------------------------

Pythonより低レイヤーなメモリ配列またはバッファへのアクセスを提供するプロトコル。

組み込みオブジェクトだと ``bytes`` 、 ``bytearray`` などがバッファプロトコルをサポートしている。

PEP 688登場以前にあったバッファプロトコルの問題点
-------------------------------------------------

バッファプロトコルをサポートしているかどうかはC言語側で実装するので、Pythonコードで表現する手段がなかった。

.. revealjs-break::

関数、メソッドの引数をバッファプロトコルをサポートするオブジェクトのみにしたい場合、型ヒントで表現する手段がなかった。

.. revealjs-break::

``typing.ByteString`` はあるが、組み込み型のみ対象で、独自クラスは対象外。

PEP 688でどう変わったか
-----------------------

``__buffer__()`` メソッドを実装したオブジェクトはバッファプロトコルをサポートしているとみなされるようになった。

.. revealjs-break::

バッファプロトコルをサポートするオブジェクトを意味する ``collections.abc.Buffer`` 型が追加され、バッファプロトコルをサポートするオブジェクトの型ヒントに指定できるようになった。

まとめ
======

* 型ヒントの新構文でジェネリックや型エイリアスが楽に書ける。f-stringはネスト可能に
* GILの改善や内包表記のインライン化などによりPythonのパフォーマンスが向上
* 新たなデバックやモニタリング方法が登場。エラーメッセージもより親切に
* 型ヒントの細かい部分の改善により、より型安全なコードが書けるように

ご清聴ありがとうございました
============================

.. figure:: thank-you-for-your-attention.*
   :alt: AIが考えた「Python 3.12の素晴らしい進化に興奮を隠しきれないプログラマーたち」

   AIが考えた「Python 3.12の素晴らしい進化に興奮を隠しきれないプログラマーたち」
