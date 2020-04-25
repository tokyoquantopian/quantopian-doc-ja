Quantopianへようこそ! この入門チュートリアルでは、Quantopianでのクオンツトレーディング戦略の研究と開発について説明します。このチュートリアルでは、Quantopian APIの基本的な機能を多く取り上げており、Quantopian を初めて利用する方を対象にしています。チュートリアルを始めるために必要なのは、基本的な `Python <https://docs.python.org/2.7/>`__ のプログラミングスキルだけです。


取引アルゴリズムとは？
-------------------------

取引アルゴリズムとは、資産を売買するための一連のルールを定義したコンピュータプログラムです。ほとんどの取引アルゴリズムは、過去のデータに基づいて行われた研究から得られた数学的または統計的なモデルに基づいて意思決定を行います。

何から始めればいいですか？
--------------------------

取引アルゴリズムを書くために最初にすることは、戦略のベースとなる経済的または統計的な関係を見つけることです。QuantopianのResearch環境を使って、利用できる過去データセットにアクセスし、分析を行うことができます。Research は `Jupyter
Notebook <http://jupyter-notebook-beginner-guide.readthedocs.io/en/latest/what_is_jupyter.html>`__ 環境で提供されており、Pythonのコードを 'セル' と呼ばれる場所で実行することができます。

例えば、以下のコードは、Apple Inc. (AAPL)の毎日の終値と20日と50日移動平均線をプロットしています。


.. code:: ipython2

    # Research 環境用関数
    from quantopian.research import prices, symbols
    
    # Pandas library: https://pandas.pydata.org/
    import pandas as pd
    
    # AAPL の過去の価格データを取得する
    aapl_close = prices(
        assets=symbols('AAPL'),
        start='2013-01-01',
        end='2016-01-01',
    )
    
    # AAPL の価格データより20日と50日の移動平均を算出する
    aapl_sma20 = aapl_close.rolling(20).mean()
    aapl_sma50 = aapl_close.rolling(50).mean()
    
    # 結果を結合して pandas の DataFrameに入れ、描画する
    pd.DataFrame({   
        'AAPL': aapl_close,
        'SMA20': aapl_sma20,
        'SMA50': aapl_sma50
    }).plot(
        title='AAPL Close Price / SMA Crossover'
    );



.. image:: notebook_files/notebook_5_0.png


ではさっそく次のレッスンで、Researchを使ってQuantopianのデータセットをさわってみましょう。さらに、取引戦略を定義し、過去のデータに基づいてリターンを効果的に予測できるかどうかを検証してみます。最後に、その結果をもとに、インタラクティブ開発環境（IDE）で取引アルゴリズムを開発し、テストを行ってみましょう。

