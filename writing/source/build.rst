.. _build:

ドキュメントのビルド
====================

.. _build-env:

環境構築
--------

ビルドにライブラリをインストール。

.. code:: bash

   pip install sphinx

.. _build-html:

HTML
----

.. code:: bash

   make html

``build/html`` に ``index.html`` が生成される。

Dockerからビルドする場合。

.. code:: bash

   docker run --rm -v $PWD:/build driller/nbsphinx-build:requirements make html

PDF
---

事前に `TeX Live <https://sphinx-users.jp/cookbook/pdf/latex.html>`_ をインストール。

.. code:: bash

   make latexpdf

``build/latex`` に ``projectname.pdf`` が生成される。

Dockerからビルドする場合。

.. code:: bash

   docker run --rm -v $PWD:/build driller/nbsphinx-build:requirements make latexpdf
