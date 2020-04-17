# 環境構築手順

## 仮想環境

venvの例

```bash
python3 -m venv venv
source venv/bin/activate
```

ビルドに必要なライブラリをインストール

```bash
pip install sphinx nbsphinx jupyter
```

コードに必要なライブラリをインストール

```bash
pip install pandas matplotlib
```

## Sphinx

以降は不要（メモとして残しているだけ）

### プロジェクトの作成（作業不要、メモ代わり）

```console
$ sphinx-quickstart 
Sphinx 2.2.0 クイックスタートユーティリティへようこそ。

以下の設定値を入力してください（Enter キーのみ押した場合、
かっこで囲まれた値をデフォルト値として受け入れます）。

選択されたルートパス: .

Sphinx 出力用のビルドディレクトリを配置する方法は2つあります。
ルートパス内にある "_build" ディレクトリを使うか、
ルートパス内に "source" と "build" ディレクトリを分ける方法です。
> ソースディレクトリとビルドディレクトリを分ける（y / n） [n]: y

プロジェクト名は、ビルドされたドキュメントのいくつかの場所にあります。
> プロジェクト名: jupyterbook
> 著者名（複数可）: 池内 孝啓, 片柳 薫子, 岩尾 エマ はるか, @driller
> プロジェクトのリリース []: 

If the documents are to be written in a language other than English,
you can select a language here by its language code. Sphinx will then
translate text that it generates into that language.

For a list of supported codes, see
https://www.sphinx-doc.org/en/master/usage/configuration.html#confval-language.
> プロジェクトの言語 [en]: ja

ファイル ./source/conf.py を作成しています。
ファイル ./source/index.rst を作成しています。
ファイル ./Makefile を作成しています。
ファイル ./make.bat を作成しています。

終了：初期ディレクトリ構造が作成されました。

マスターファイル ./source/index.rst を作成して
他のドキュメントソースファイルを作成します。次のように Makefile を使ってドキュメントを作成します。
 make builder
"builder" はサポートされているビルダーの 1 つです。 例: html, latex, または linkcheck。
```

### source/conf.py

下記を追加

```python
extensions = [
    "sphinx.ext.mathjax",
    "nbsphinx",
    "sphinx.ext.todo",
]

exclude_patterns = ["_build", "**.ipynb_checkpoints"]
numfig = True
```

## HTMLへのビルド

```bash
make html
```
