.. _toctree:

ファイルの作成とtoctreeへの登録
===============================

.. _toctree-directory:

ディレクトリ構成
----------------

.. code-block::

   ├── build
   └── source
       ├── _static
       ├── _templates
       ├── conf.py
       └── index.rst

build
  ビルド処理で生成されたHTMLやPDFが格納
source
  原稿のファイル（.rstや.ipynb）
source/index.rst
  マスタドキュメント

.. _toctree-chapter:

章の立て方
----------

1. ``source`` 直下に章名のディレクトリを作成する
2. 作成したディレクトリに章のトップとなるファイル（例: title.rst）を作成する
3. ``source/index.rst`` のtoctreeに追加する

<例: 章の名前をSOS、トップとなるファイルをtitle.rstとした場合>  

.. code-block:: rst
   :caption: source/index.rst

   .. toctree::
      :maxdepth: 2
      :caption: Contents:

      SOS/title

.. _toctree-section:

節の立て方
----------

1. 章のディレクトリに節単位でファイルを作成する
2. トップとなるファイルのtoctreeに作成したファイルを追加する

<例: 節ファイルabout.rstを追加する場合>

.. code-block:: rst
   :caption: source/SOS/title.rst

   .. toctree::
      :maxdepth: 2
      :caption: Contents:

      SOS/about