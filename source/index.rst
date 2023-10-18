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
- コードに型ヒントを付けることで型チェッカー（Mypy、Pyrightなど）で型と矛盾するコードを書いていないか検証できる
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
   :alt: pyrightの実行結果

   pyrightの実行結果

ジェネリッククラス、ジェネリック関数とは
----------------------------------------

特定の型に依存しないクラス、関数を定義できる

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

PEP 695でどう変わったか
-----------------------

PEP 701 f-stringがネスト可能に
------------------------------

f-stringとは
------------

PEP 701でどう変わったか
-----------------------

パフォーマンスの改善
====================

* PEP 684 インタプリタごとに固有のGILが使われるように変更

PEP 684 インタプリタごとに固有のGILが使われるように変更
-------------------------------------------------------

GILとは
-------

PEP 684でどう変わったか
-----------------------

デバッグ・モニタリング方法の改善
================================

* PEP 669 ``sys.monitoring`` の追加
* エラーメッセージの改善（PEP番号はなし）

PEP 669 ``sys.monitoring`` の追加
---------------------------------

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

``*kwargs`` 引数とは
--------------------

PEP 692でどう変わったか
-----------------------

PEP 698 メソッドをオーバーライドする際のtypoを防ぐ ``override`` デコレータの登場
--------------------------------------------------------------------------------

Pythonでメソッドをオーバーライドするには
----------------------------------------

``override`` デコレータを使うとどうなるか
-----------------------------------------

まとめ
======
