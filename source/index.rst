#######################
Python 3.12の新機能紹介
#######################
.. raw:: html

   <a rel="license" href="http://creativecommons.org/licenses/by/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by/4.0/88x31.png" /></a><br /><small>This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/4.0/">Creative Commons Attribution 4.0 International License</a>.</small>

はじめに
========

自己紹介
--------

* Ryuji Tsutsui @ryu22e
* 株式会社hokan所属
* Python歴は12年くらい（主にDjango）
* Python Boot Camp、Shonan.py GCPUG Shonanなどコミュニティ活動もしています
* 著書（共著）：『 `Python実践レシピ <https://gihyo.jp/book/2022/978-4-297-12576-9>`_ 』

今日話したいこと
----------------

* Python 3.12の新機能を紹介します

このセミナーから得られるもの
----------------------------

* Pythonがどんな進化を遂げているかが分かる
* 実際にPython 3.12を使う際に役立つ情報が得られる

新しい構文
==========

* PEP 695 ジェネリッククラス、ジェネリック関数を簡潔に書ける新構文の登場
* PEP 701 f-stringがネスト可能に

PEP 695 ジェネリッククラス、ジェネリック関数を簡潔に書ける新構文の登場
----------------------------------------------------------------------

型ヒントとは
------------

- Python 3.5で追加された機能
- コードに型ヒントを付けることで型チェッカー（Mypy、Pyrightなど）で型ヒントと矛盾するコードを書いていないか検証できる
- 実行時に型チェックはしてくれない

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

ジェネリッククラスの例
----------------------

.. revealjs-code-block:: python

    from typing import Generic, TypeVar

    T = TypeVar('T')

    class Example(Generic[T]):
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

ジェネリック関数の例
--------------------

.. revealjs-code-block:: python

    from typing import Sequence, TypeVar

    T = TypeVar('T')

    def first(l: Sequence[T]) -> T:
        return l[0]

    print(first([1, 2, 3]))
    print(first("python"))

PEP 695でどう変わったか
-----------------------

``T = TypeVar('T')`` という記述が不要になった

Python 3.12でのジェネリッククラスの例
-------------------------------------

.. revealjs-code-block:: python

    class Example[T]:  # 角括弧でTを囲む
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

    def first[T](l: Sequence[T]) -> T:  # 関数名の右に角括弧でTを囲む
        return l[0]

    print(first([1, 2, 3]))
    print(first("python"))

PEP 701 f-stringがネスト可能に
------------------------------

f-stringとは
------------

    フォーマット済み文字リテラル (短くして f-string とも呼びます) では、文字列の頭に f か F を付け、式を {expression} と書くことで、 Python の式の値を文字列の中に入れ込めます。

https://docs.python.org/ja/3.11/tutorial/inputoutput.html#formatted-string-literals

公式ドキュメントに「式を埋め込めます」とは書いているものの…
-----------------------------------------------------------

（Python 3.11までは）厳密に言うと書けない式もある

.. revealjs-code-block:: python

    >>> d = {"foo": 1, "bar": 2}
    >>> f"{d["foo"]}"  # "{d[" までを文字列を認識してしまう
      File "<stdin>", line 1
        f"{d["foo"]}"
              ^^^
    SyntaxError: f-string: unmatched '['
    >>> f"{d[\"foo\"]}"  # バッククォートでエスケープしてもダメ
      File "<stdin>", line 1
        f"{d[\"foo\"]}"
                       ^
    SyntaxError: f-string expression part cannot include a backslash

PEP 701でどう変わったか
-----------------------

パーサが改善され、f-stringにどんな式でも埋め込めるようになった。

.. revealjs-code-block:: python

    >>> d = {"foo": 1, "bar": 2}
    >>> f"{d['foo']}"
    '1'
    >>> f"{f"{f"{f"{f"{f"{1+1}"}"}"}"}"}"
    '2'

PEP 701でどう変わったか（続き）
-------------------------------

f-stringの途中で改行やコメントも入れられる。

VS Codeのシンタックスハイライトも効く。

.. figure:: pep701_example_py.*
   :alt: VS Codeのシンタックスハイライト

パフォーマンスの改善
====================

* PEP 684 インタプリタごとに固有のGILが使われるように変更

PEP 684 インタプリタごとに固有のGILが使われるように変更
-------------------------------------------------------

GIL（global interpreter lock）とは
----------------------------------

以下公式ドキュメントの引用

    CPython インタプリタが利用している、一度にPythonのバイトコードを実行するスレッドは一つだけであることを保証する仕組みです。

https://docs.python.org/ja/3/glossary.html#term-global-interpreter-lock

GILの例
-------

以下のコードはマルチスレッドを使っているにも関わらず、 ``print_hello`` 関数が同時に実行されない。

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
*  `PEP 554 <https://peps.python.org/pep-0554/>`_ が実装されることで初めてPEP 684の恩恵を受けられる（Python 3.13で実装予定）

Python 3.13のリリース予定日は？
-------------------------------

最終版は2024年10月1日リリース予定。気長に待とう！

https://peps.python.org/pep-0719/

デバッグ・モニタリング方法の改善
================================

* PEP 669 ``sys.monitoring`` の追加
* エラーメッセージの改善（PEP番号はなし）

PEP 669 ``sys.monitoring`` の追加
---------------------------------

関数やメソッド呼び出しなどのタイミングで呼び出すフック関数を登録できるようになった。

``sys.monitoring`` の主な使い方
-------------------------------

以下の関数を使う。

* ``sys.monitoring.use_tool_id`` : ツールIDを登録
* ``sys.monitoring.register_callback`` : フック関数を登録
* ``sys.monitoring.set_events`` : 監視するイベントを登録・登録解除
* ``sys.monitoring.free_tool_id`` : ツールIDを解放

``sys.monitoring`` のサンプルコード
-----------------------------------

前述の関数を使ったサンプルコード

TODO Gist URLを書く

エラーメッセージの改善（PEP番号はなし）
---------------------------------------

その他新機能
============

* PEP 688 Pythonコードからバッファプロトコルにアクセスできるように
* PEP 692 ``*kwargs`` 引数に付けられる型ヒントに関する改善
* PEP 698 メソッドをオーバーライドする際のtypoを防ぐ ``override`` デコレータの登場

PEP 688 Pythonコードからバッファプロトコルにアクセスできるように
----------------------------------------------------------------

PEP 692 ``*kwargs`` 引数に付けられる型ヒントに関する改善
--------------------------------------------------------

Pythonの関数の引数指定方法
--------------------------

Pythonの関数の引数指定方法は以下の2つ。

.. revealjs-code-block:: python

    >>> example(1, 2)  # 位置引数
    >>> example(a=1, b=2)  # キーワード引数

``*kwargs`` 引数とは
--------------------

* 引数名の先頭に ``**`` を付けると、どんなキーワード引数でも受け付ける引数になる
* 関数内では ``kwargs`` を辞書型の値として扱う
* ``kwargs`` という名前は別の名前でも良いが、慣例として ``kwargs`` （読み: クワーグス。keyword argumentsの略）とすることが多い

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

    add_book(title="Python実践レシピ", price=2790)
    add_book(title="Python実践レシピ", price="2,970円（本体2,700円＋税10%）")

PEP 698 メソッドをオーバーライドする際のtypoを防ぐ ``override`` デコレータの登場
--------------------------------------------------------------------------------

Pythonでメソッドをオーバーライドするには
----------------------------------------

``override`` デコレータを使うとどうなるか
-----------------------------------------

まとめ
======
