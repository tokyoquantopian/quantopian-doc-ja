クイックスタートテンプレート
=============================

原作 : https://www.quantopian.com/tutorials/alphalens#lesson5

.. note:: 

    ここでのチュートリアルのコードは、`ドキュメント原作ページ <https://www.quantopian.com/tutorials/alphalens#lesson5>`__ にある ``Get Notebook`` ボタンでクローン出来ます。

    また  ``Get Notebook`` でクローンしたコードはこのチュートリアルには記されていないライブラリーのインポート文なども記載されています。

これまでのレッスンで、アルファレンズとは何かとその使い方を学びました。このレッスンのノートをクローンして、自分なりのファクター分析を始めましょう。
以下は、ノートの各コードブロックの説明です。コードの中にはこれらの説明は含まれていません。


アルファファクターの定義
---------------------------

（クローンした）ノートブックにはパイプラインを使って、アルファファクター ``factor_to_analyze`` を分析するコードがセットアップされています。このアルファファクターをご自身の思うアルファファクターに変えて見ましょう。

もしあなたが何をしてよいのかまったくわからない状態だったとしても心配ありません。分析を始めるには一行書き換えるだけです。

この行を 

``factor_to_analyze = (current_assets - assets_moving_average)`` 

`FactSetのファンダメンタルデータ  <https://www.quantopian.com/docs/data-reference/factset_fundamentals>`__ からデータを取得して分析出来るように書き換えます。その為にこのように修正します。

``factor_to_analyze = factset.Fundamentals.assets_gr_qf.latest``

書き換えたらセルを実行しましょう。 この一行を修正するだけで、企業の資産成長が株価にどのように影響するかを分析できます。

.. code:: python
   :caption: 修正前のコード

    def make_pipeline():

        assets_moving_average = SimpleMovingAverage(inputs=[factset.Fundamentals.assets], window_length=252)
        current_assets = factset.Fundamentals.assets.latest

        factor_to_analyze = (current_assets - assets_moving_average) 

        sector = Sector()

        return Pipeline(
            columns={'factor_to_analyze': factor_to_analyze, 'sector': sector},
            screen=QTradableStocksUS() & factor_to_analyze.notnull() & sector.notnull()
        )

    factor_data = run_pipeline(make_pipeline(), '2015-1-1', '2016-1-1')
    pricing_data = get_pricing(factor_data.index.levels[1], '2015-1-1', '2016-6-1', fields='open_price')


アルファファクターがどの程度将来を予測するのかを見極める
---------------------------------------------------------

次のコードを実行して得られるチャートは、アルファファクターの情報係数（Information Coefficient, IC）の時系列変化を示しています。覚えておいたほうが良いこととして、ICは与えられたアルファファクターの予測性を定量化するために最も有用な数値であるということです。


.. code:: python

    longest_look_forward_period = 63 # week = 5, month = 21, quarter = 63, year = 252
    range_step = 5

    merged_data = get_clean_factor_and_forward_returns(
        factor=factor_data['factor_to_analyze'],
        prices=pricing_data,
        periods=range(1, longest_look_forward_period, range_step)
    )

    mean_information_coefficient(merged_data).plot(title="IC Decay")


Create Group Neutral Tear Sheets
-----------------------------------

以下のセルを実行すると、アルファファクターをグループニュートラルにした場合のティアシートを作成します。グループニュートラルに関しては、Lesson4を参照してください。

理想的には、ICの平均線が常に0以上になることが望ましいです。また、分位1では常にリターンが低く、分位5では常にリターンが高くなるようにしたいです。

最後に、IC平均とセクター別のリターンを見て、パフォーマンスの面で大きな外れがあるかどうかを確認してください。

.. code:: python

    sector_labels, sector_labels[-1] = dict(Sector.SECTOR_NAMES), "Unknown"

    merged_data = get_clean_factor_and_forward_returns(
        factor=factor_data['factor_to_analyze'],
        prices=pricing_data,
        groupby=factor_data['sector'],
        groupby_labels=sector_labels,
        binning_by_group=True,
        periods=(1,5,10)
    )

    create_information_tear_sheet(merged_data, by_group=True, group_neutral=True)
    create_returns_tear_sheet(merged_data, by_group=True, group_neutral=True)





