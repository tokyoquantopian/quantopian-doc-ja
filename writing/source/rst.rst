.. rst:

reSturedTextのルール
====================

.. rst-title:

見出し
------

下記のようにします。必要であれば階層を増やしても問題ありません。

.. code-block:: rst

   ========
   章見出し
   ========

   節見出し
   ========

   項見出し
   --------

各見出しにはリファレンスできるよう、ラベルを記述します。名前が重複しないよう、ラベルは ``_章-節-項`` の形式にします。

<例>

.. code-block:: rst

   .. _sos:

   =====
   SOS団
   =====

   .. _sos-about:

   SOS団とは
   =========

   .. _sos-about-history:

   SOS団の歴史
   -----------

番号は付与しません。自動的に割り振られます。

| 正）SOS団
| 誤）第1章 SOS団

.. rst-note_warning:

Note、Warning
-------------

noteとwarningのみを用います。それ以外のディレクティブを使いたい場合はissueを上げて協議します。

| "attention", "caution", "danger", "error", "hint", "important", "note", "tip", "warning", "admonition"

<例>

.. code-block:: rst

   .. note:: これはnoteです
      
      tipsやhintも含みます

.. note:: これはnoteです
  
   tipsやhintも含みます

.. code-block:: rst

   .. warning:: これはwarningです
      
      attentionやcautionも含みます

.. warning:: これはwarningです
  
  attentionやcautionも含みます

.. rst-url:

URL
---

脚注に記載します。 ``rubric::`` ディレクティブは **節（ファイル）の最後** に記載します。

<例>

.. code-block:: rst

   詳細については公式ドキュメント [#python_org]_ を参照してください。

   .. rubric:: 脚注

   .. [#python_org] https://docs.python.org/ja/3/index.html


詳細については公式ドキュメント [#python_org]_ を参照してください。

.. rst-emphasis:

強調
----

インラインリテラル \`\`word\`\`
  関数、クラス、引数、値

強い強調 \*\*word\*\*
  初出のモジュール、クラス

強調 \*word\*
  上記以外

``literalinclude::`` ディレクティブで強調する場合は ``:emphasize-lines:`` オプションを用います。

<例>

.. code-block:: rst

   .. literalinclude:: src/example.py
      :caption: example.py
      :language: python
      :emphasize-lines: 1,6-9
      :linenos:

.. literalinclude:: src/example.py
  :caption: example.py
  :language: python
  :emphasize-lines: 1,6-9
  :linenos:

.. rst-image:

画像
----

``figure::`` ディレクティブを用います。 ``:name:`` オプションとキャプションをつけます。

<例>

.. code-block:: rst

   .. figure:: https://www.python.org/static/img/python-logo.png
      :name: python_logo
   
      ここがキャプションになります

.. figure:: https://www.python.org/static/img/python-logo.png
   :name: python_logo

   ここがキャプションになります

参照する場合は ``:numref:`` ロールを用います。

.. code-block:: rst

   これはPythonの画像です（ :numref:`python_logo` ）

これはPythonの画像です（ :numref:`python_logo` ）

.. rst-table:

表（テーブル）
--------------

次のディレクティブを用います。 ``:name:`` オプションとキャプションをつけます。

<例>

- list-table
- cst-table
- table（grid tableはできるだけ使わない）

.. code-block:: rst

   .. list-table:: 好きな食べ物
      :name: foods
      :header-rows: 1
   
      * - なまえ
        - 食べ物
      * - 長門
        - カレー
      * - 小泉
        - りんご

.. list-table:: 好きな食べ物
   :name: foods
   :header-rows: 1

   * - なまえ
     - 食べ物
   * - 長門
     - カレー
   * - 小泉
     - りんご

参照する場合は ``:numref:`` ロールを用います。

<例>

.. code-block:: rst

   団員の好きな食べ物を :numref:`foods` に示します。

団員の好きな食べ物を :numref:`foods` に示します。


.. rst-crossref:

クロスリファレンス
------------------

脚注 + ``:ref:`` ロールを用います。

.. code-block:: rst

   nbsphinx [#ref1]_ を用います。

   .. rubric:: 脚注

   .. [#ref1] :ref:`nbsphinx`

nbsphinx [#ref1]_ を用います。


.. rst-index:

索引
----

<例>

``:index:`` ロールを用います。

.. code-block:: rst

   :index:`SOS団` は世界を大いに盛り上げる為の涼宮ハルヒの団です。

:index:`SOS団` は世界を大いに盛り上げる為の涼宮ハルヒの団です。

.. rst-code:

コード
------

コマンドは ``code-block`` ディレクティブを用います。言語名を明記し、 ``:name:`` および ``:caption:`` オプションをつけます。

<例>

.. code-block:: rst

   .. code-block:: python
      :name: zen_of_python
      :caption: Zen of Python

      import this

.. code-block:: python
   :name: zen_of_python
   :caption: Zen of Python

   import this

参照する場合は ``:numref:`` ロールを用います。

<例>

.. code-block:: rst

   :numref:`zen_of_python` を実行します。

:numref:`zen_of_python` を実行します。

Pythonスクリプトを参照する場合は ``literalinclude::`` ディレクティブを用います。 ``:linenos:`` オプションで行番号を表示します。コードの途中から切り出す場合は ``:lines:`` オプションでとりだす行番号を指定し、 ``:lineno-start:`` オプションで行番号を振り直します。

<例>

.. code-block:: rst

    .. literalinclude:: src/example.py
       :caption: example.py
       :language: python
       :lines: 11-
       :lineno-start: 11
       :linenos:

.. literalinclude:: src/example.py
   :caption: example.py
   :language: python
   :lines: 11-
   :lineno-start: 11
   :linenos:

.. rst-terminal:

ターミナル
----------

次のOSを対象とします。

- Windows
- macOS
- Linux(Ubuntu)

コマンドは ``code-block`` ディレクティブを用い、ハイライトする言語は次のとおりにします。macOSのシンタックスはzshとなっていますが、内容はbashで構いません。

.. list-table:: ターミナルブロックのルール
   :header-rows: 1

   * - OS
     - 言語
     - caption
     - プロンプト
   * - Windows
     - powershell
     - 冒頭に[Win] をつける
     - > 
   * - macOS
     - zsh
     - 冒頭に[macOS] をつける
     - $ 
   * - macOS/Linux
     - bash
     - 冒頭に[macOS/Linux] をつける
     - $ 
   * - Windows/macOS/Linux
     - bash
     - 冒頭に[Win/macOS/Linux] をつける
     - $ 

<例>

.. code-block:: rst

   .. code-block:: powershell
      :caption: [Win] 仮想環境を有効化

      > venv\Scripts\activate.ps1

.. code-block:: powershell
   :caption: [Win] 仮想環境を有効化

   > venv\Scripts\activate.ps1

TODO
----

TODOを残しておく場合は ``todo::`` ディレクティブを用います。

<例>

.. code-block:: rst

   .. todo:: 宇宙人の正体
   
      あとで詳しく書く


.. todo:: 宇宙人の正体

   あとで詳しく書く

.. rubric:: 脚注

.. [#python_org] https://docs.python.org/ja/3/index.html
.. [#ref1] :ref:`nbsphinx`

.. rst-todo: