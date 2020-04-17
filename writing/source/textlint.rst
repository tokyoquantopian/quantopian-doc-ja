.. _textlint:

textlint
========

使用するルールプリセット
------------------------

- `textlint-rule-preset-ja-technical-writing <https://github.com/textlint-ja/textlint-rule-preset-ja-technical-writing>`_
- `textlint-rule-preset-JTF-style <https://github.com/textlint-ja/textlint-rule-preset-JTF-style>`_
- `textlint-plugin-rst <https://github.com/jimo1001/textlint-plugin-rst>`_
- `textlint-rule-prh <https://github.com/textlint-rule/textlint-rule-prh>`_

  - `WEB+DB_PRESS.yml <https://github.com/prh/rules>`_


ルールの追加
------------

``quantopian-doc.yml`` に追記します。

<例>

.. code:: yaml

   version: 1
   rules:
     - pattern: 取引アルゴリズム
       expected: トレーディングアルゴリズム

インストール
------------

Docker（後述）から実行する場合は不要です。

.. code:: bash

   npm install --save-dev \
   textlint \
   textlint-rule-preset-ja-technical-writing \
   textlint-rule-preset-jtf-style \
   textlint-rule-prh \
   textlint-plugin-rst

   pip install docutils-ast-writer

実行
----

.rstファイルの場合

.. code:: bash

   textlint <path_to_rst>

.ipynbファイルの場合

.. code:: bash

   jupyter nbconvert <path_to_ipynb> \
     --to rst --stdout \
     --TemplateExporter.exclude_code_cell=True | \
     textlint --stdin

Dockerから実行する場合
``textlint`` コマンドのpathの先頭に ``/work`` をつけます

.. code:: bash

   docker run --rm -v $PWD:/work driller/textlint-rst textlint /work/<path_to_rst>