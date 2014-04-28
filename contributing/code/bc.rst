.. note::

    * 対象バージョン：2.3以降
    * 翻訳更新日：2014/04/25

後方互換性の保証について
========================

私達はプロジェクトの円滑なアップグレードの保証を最も重視しています。
そのためSymfonyではすべてのマイナーリリース間の後方互換性を保証します。
`セマンティックバージョニング`_と言うバージョニング戦略をご存知の方も多いでしょう。
セマンティックバージョニングとは、メジャーリリース（2.0や3.0など）だけが後方互換性を破壊できる事を意味します。
マイナーリリース（2.5や2.6のような）では新機能の追加が行われますが、そのリリースブランチ（2.xの例）の既存のAPIの後方互換性は保たれます。

.. caution::

    この保証はSymfony 2.3 から施行されたため、その前のバージョンには適用されません。

しかし、一口に後方互換性と言っても様々な種類があります。
実際には、フレームワークに行われるほぼすべての変更はアプリケーションを破壊する恐れがあります。
例えば、クラスに新しいメソッドが追加されると、それを継承しているクラスが同名の異なるシグネチャのメソッドを持っていた場合、アプリケーションが壊れてしまいます。

また、全てのBCブレークがアプリケーションに影響を与えるわけではありません。
作ったクラスやアーキテクチャに大幅な変更が必要な物もあれば、一方で、単にメソッド名を変更することだけで解決する物もあります。

そこで私達はこのページを作りました。"Using Symfony Code"セクションでは、どのようにすればアプリケーションを壊す事なく、確実に同じメジャーリリースブランチの新しいバージョンにアップグレードする事ができるかを説明します。

次の"Working on Symfony Code"はSymfonyコントリビュータ向けのセクションです。
このセクションではユーザの円滑なアップグレードを保証するために、すべてのコントリビュータが準拠すべき規約の詳細を列挙します。

Using Symfony Code
------------------

プロジェクトでSymfonyを使用してる場合、以下のガイドラインに従うことで今後のSymfonyのすべてのマイナーリリースに円滑にアップグレードする事ができます。

インターフェースの利用
‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾

Symfonyが用意しているすべてのインターフェースはタイプヒントで利用する事ができ、宣言されている全てのメソッドをコールすることができます。
私達は、これらの規則を守るコードを壊さないことを保証します。

.. caution::

    例外として、``@internal``がタグ付けされているインターフェースが挙げられます。このようなインターフェースは使用、または実装すべきでありません。

インターフェースを実装する場合、まずそれがAPIインターフェースであるかを確認します。次のような、``@api``タグの付いている物がAPIインターフェースとなります::

    /**
     * HttpKernelInterface handles a Request to convert it to a Response.
     *
     * @author Fabien Potencier <fabien@symfony.com>
     *
     * @api
     */
    interface HttpKernelInterface
    {
        // ...
    }

APIインターフェースを実装していれば、将来そのコードが壊される事はありません。対照的に、通常のインターフェースはマイナーリリース時に、新しいメソッドを追加するなどの拡張がされる可能性がある為、アップグレードの際は事前に確認を行い、必要であれば手動でコードを移行しておく必要があります。

.. note::

    移行が必要な変更は最小限である事が保証され、それらの変更は常にSymfonyルートディレクトリに配置されているUPGRADEファイルに正確な移行の指示を明記します。

次の表は、どのようなユースケースが後方互換性が保たれるかを説明します。

+-------------------------------------------------+--------------------+--------------------+
| ユースケース                                    | 通常               | API                |
+=================================================+====================+====================+
| **あなたが...**                                 | **する場合は後方互換性は保たれる...**   |
+-------------------------------------------------+--------------------+--------------------+
| インターフェースに対するタイプヒント            | Yes                | Yes                |
+-------------------------------------------------+--------------------+--------------------+
| メソッドをコール                                | Yes                | Yes                |
+-------------------------------------------------+--------------------+--------------------+
| **あなたがインターフェースを実装し、かつ、...** | **する場合は後方互換性は保たれる...**   |
+-------------------------------------------------+--------------------+--------------------+
| メソッドを実装                                  | No [1]_            | Yes                |
+-------------------------------------------------+--------------------+--------------------+
| 実装したメソッドに引数を追加                    | No [1]_            | Yes                |
+-------------------------------------------------+--------------------+--------------------+
| 引数にデフォルト値を設定                        | Yes                | Yes                |
+-------------------------------------------------+--------------------+--------------------+

.. include:: _api_tagging.rst.inc

Using Our Classes
‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾

All classes provided by Symfony may be instantiated and accessed through their
public methods and properties.

.. caution::

    Classes, properties and methods that bear the tag ``@internal`` as well as
    the classes located in the various ``*¥¥Tests¥¥`` namespaces are an
    exception to this rule. They are meant for internal use only and should
    not be accessed by your own code.

Just like with interfaces, we also distinguish between regular and API classes.
Like API interfaces, API classes are marked with an ``@api`` tag::

    /**
     * Request represents an HTTP request.
     *
     * @author Fabien Potencier <fabien@symfony.com>
     *
     * @api
     */
    class Request
    {
        // ...
    }

The difference between regular and API classes is that we guarantee full
backwards compatibility if you extend an API class and override its methods. We
can't give the same promise for regular classes, because there we may, for
example, add an optional argument to a method. Consequently, the signature of
your overridden method wouldn't match anymore and generate a fatal error.

.. note::

    As with interfaces, we limit ourselves to changes that can be upgraded
    easily. We will document the precise ugprade instructions in the UPGRADE
    file in Symfony's root directory.

In some cases, only specific properties and methods are tagged with the ``@api``
tag, even though their class is not. In these cases, we guarantee full backwards
compatibility for the tagged properties and methods (as indicated in the column
"API" below), but not for the rest of the class.

To be on the safe side, check the following table to know which use cases are
covered by our backwards compatibility promise:

+-----------------------------------------------+---------------+---------------+
| Use Case                                      | Regular       | API           |
+===============================================+===============+===============+
| **If you...**                                 | **Then we guarantee BC...**   |
+-----------------------------------------------+---------------+---------------+
| Type hint against the class                   | Yes           | Yes           |
+-----------------------------------------------+---------------+---------------+
| Create a new instance                         | Yes           | Yes           |
+-----------------------------------------------+---------------+---------------+
| Extend the class                              | Yes           | Yes           |
+-----------------------------------------------+---------------+---------------+
| Access a public property                      | Yes           | Yes           |
+-----------------------------------------------+---------------+---------------+
| Call a public method                          | Yes           | Yes           |
+-----------------------------------------------+---------------+---------------+
| **If you extend the class and...**            | **Then we guarantee BC...**   |
+-----------------------------------------------+---------------+---------------+
| Access a protected property                   | No [1]_       | Yes           |
+-----------------------------------------------+---------------+---------------+
| Call a protected method                       | No [1]_       | Yes           |
+-----------------------------------------------+---------------+---------------+
| Override a public property                    | Yes           | Yes           |
+-----------------------------------------------+---------------+---------------+
| Override a protected property                 | No [1]_       | Yes           |
+-----------------------------------------------+---------------+---------------+
| Override a public method                      | No [1]_       | Yes           |
+-----------------------------------------------+---------------+---------------+
| Override a protected method                   | No [1]_       | Yes           |
+-----------------------------------------------+---------------+---------------+
| Add a new property                            | No            | No            |
+-----------------------------------------------+---------------+---------------+
| Add a new method                              | No            | No            |
+-----------------------------------------------+---------------+---------------+
| Add an argument to an overridden method       | No [1]_       | Yes           |
+-----------------------------------------------+---------------+---------------+
| Add a default value to an argument            | Yes           | Yes           |
+-----------------------------------------------+---------------+---------------+
| Call a private method (via Reflection)        | No            | No            |
+-----------------------------------------------+---------------+---------------+
| Access a private property (via Reflection)    | No            | No            |
+-----------------------------------------------+---------------+---------------+

.. include:: _api_tagging.rst.inc

Working on Symfony Code
-----------------------

Do you want to help us improve Symfony? That's great! However, please stick
to the rules listed below in order to ensure smooth upgrades for our users.

Changing Interfaces
‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾

This table tells you which changes you are allowed to do when working on
Symfony's interfaces:

==============================================  ==============  ==============
Type of Change                                  Regular         API
==============================================  ==============  ==============
Remove entirely                                 No              No
Change name or namespace                        No              No
Add parent interface                            Yes [2]_        Yes [3]_
Remove parent interface                         No              No
**Methods**
Add method                                      Yes [2]_        No
Remove method                                   No              No
Change name                                     No              No
Move to parent interface                        Yes             Yes
Add argument without a default value            No              No
Add argument with a default value               Yes [2]_        No
Remove argument                                 Yes [4]_        Yes [4]_
Add default value to an argument                Yes [2]_        No
Remove default value of an argument             No              No
Add type hint to an argument                    No              No
Remove type hint of an argument                 Yes [2]_        No
Change argument type                            Yes [2]_ [5]_   No
Change return type                              Yes [2]_ [6]_   No
==============================================  ==============  ==============

Changing Classes
‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾

This table tells you which changes you are allowed to do when working on
Symfony's classes:

==================================================  ==============  ==============
Type of Change                                      Regular         API
==================================================  ==============  ==============
Remove entirely                                     No              No
Make final                                          No              No
Make abstract                                       No              No
Change name or namespace                            No              No
Change parent class                                 Yes [7]_        Yes [7]_
Add interface                                       Yes             Yes
Remove interface                                    No              No
**Public Properties**
Add public property                                 Yes             Yes
Remove public property                              No              No
Reduce visibility                                   No              No
Move to parent class                                Yes             Yes
**Protected Properties**
Add protected property                              Yes             Yes
Remove protected property                           Yes [2]_        No
Reduce visibility                                   Yes [2]_        No
Move to parent class                                Yes             Yes
**Private Properties**
Add private property                                Yes             Yes
Remove private property                             Yes             Yes
**Constructors**
Add constructor without mandatory arguments         Yes [2]_        Yes [2]_
Remove constructor                                  Yes [2]_        No
Reduce visibility of a public constructor           No              No
Reduce visibility of a protected constructor        Yes [2]_        No
Move to parent class                                Yes             Yes
**Public Methods**
Add public method                                   Yes             Yes
Remove public method                                No              No
Change name                                         No              No
Reduce visibility                                   No              No
Move to parent class                                Yes             Yes
Add argument without a default value                No              No
Add argument with a default value                   Yes [2]_        No
Remove argument                                     Yes [4]_        Yes [4]_
Add default value to an argument                    Yes [2]_        No
Remove default value of an argument                 No              No
Add type hint to an argument                        Yes [8]_        No
Remove type hint of an argument                     Yes [2]_        No
Change argument type                                Yes [2]_ [5]_   No
Change return type                                  Yes [2]_ [6]_   No
**Protected Methods**
Add protected method                                Yes             Yes
Remove protected method                             Yes [2]_        No
Change name                                         No              No
Reduce visibility                                   Yes [2]_        No
Move to parent class                                Yes             Yes
Add argument without a default value                Yes [2]_        No
Add argument with a default value                   Yes [2]_        No
Remove argument                                     Yes [4]_        Yes [4]_
Add default value to an argument                    Yes [2]_        No
Remove default value of an argument                 Yes [2]_        No
Add type hint to an argument                        Yes [2]_        No
Remove type hint of an argument                     Yes [2]_        No
Change argument type                                Yes [2]_ [5]_   No
Change return type                                  Yes [2]_ [6]_   No
**Private Methods**
Add private method                                  Yes             Yes
Remove private method                               Yes             Yes
Change name                                         Yes             Yes
Reduce visibility                                   Yes             Yes
Add argument without a default value                Yes             Yes
Add argument with a default value                   Yes             Yes
Remove argument                                     Yes             Yes
Add default value to an argument                    Yes             Yes
Remove default value of an argument                 Yes             Yes
Add type hint to an argument                        Yes             Yes
Remove type hint of an argument                     Yes             Yes
Change argument type                                Yes             Yes
Change return type                                  Yes             Yes
==================================================  ==============  ==============

.. [1] Your code may be broken by changes in the Symfony code. Such changes will
       however be documented in the UPGRADE file.

.. [2] Should be avoided. When done, this change must be documented in the
       UPGRADE file.

.. [3] The added parent interface must not introduce any new methods that don't
       exist in the interface already.

.. [4] Only the last argument(s) of a method may be removed, as PHP does not
       care about additional arguments that you pass to a method.

.. [5] The argument type may only be changed to a compatible or less specific
       type. The following type changes are allowed:

       ===================  ==================================================================
       Original Type        New Type
       ===================  ==================================================================
       boolean              any `scalar type`_ with equivalent `boolean values`_
       string               any `scalar type`_ or object with equivalent `string values`_
       integer              any `scalar type`_ with equivalent `integer values`_
       float                any `scalar type`_ with equivalent `float values`_
       class ``<C>``        any superclass or interface of ``<C>``
       interface ``<I>``    any superinterface of ``<I>``
       ===================  ==================================================================

.. [6] The return type may only be changed to a compatible or more specific
       type. The following type changes are allowed:

       ===================  ==================================================================
       Original Type        New Type
       ===================  ==================================================================
       boolean              any `scalar type`_ with equivalent `boolean values`_
       string               any `scalar type`_ or object with equivalent `string values`_
       integer              any `scalar type`_ with equivalent `integer values`_
       float                any `scalar type`_ with equivalent `float values`_
       array                instance of ``ArrayAccess``, ``Traversable`` and ``Countable``
       ``ArrayAccess``      array
       ``Traversable``      array
       ``Countable``        array
       class ``<C>``        any subclass of ``<C>``
       interface ``<I>``    any subinterface or implementing class of ``<I>``
       ===================  ==================================================================

.. [7] When changing the parent class, the original parent class must remain an
       ancestor of the class.

.. [8] A type hint may only be added if passing a value with a different type
       previously generated a fatal error.

.. _セマンティックバージョニング: http://semver.org/
.. _スカラー型: http://php.net/manual/ja/function.is-scalar.php
.. _boolean値: http://php.net/manual/ja/function.boolval.php
.. _string値: http://www.php.net/manual/ja/function.strval.php
.. _integer値: http://www.php.net/manual/ja/function.intval.php
.. _float値: http://www.php.net/manual/ja/function.floatval.php

.. 2014/04/25 issei-m 6c1ded9af043f1711d6349db91711b2e5fc33bb4
