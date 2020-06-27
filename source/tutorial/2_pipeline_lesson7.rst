.. code:: ipython2

    from quantopian.pipeline import Pipeline
    from quantopian.research import run_pipeline
    from quantopian.pipeline.data.builtin import USEquityPricing
    from quantopian.pipeline.factors import SimpleMovingAverage, AverageDollarVolume

マスキング 
------------------

パイプラインを処理するときに特定の資産を除外したいことがあります。資産が除外できると便利な事例が2つあります：

1. 計算コストが高い評価式を計算したいが、特定の資産の結果だけ確認できれば良い場合。そのような計算コストが高い評価式の例として、回帰係数 ( `RollingLinearRegressionOfReturns <https://www.quantopian.com/help#quantopian_pipeline_factors_RollingLinearRegressionOfReturns>`__) を計算する ``ファクター`` が挙げられます。
2. 資産どうしの比較を行いたいが、計算は全資産のうちの一部分だけを対象にして実行したい場合。たとえば ``ファクター`` メソッドのひとつである ``top`` を使って益回り(earnings yield)の高い上位200銘柄を計算したいが、一定の流動性制約を満たさない資産は除外したいと考える場合が挙げられます。

このようなユースケースに対応するため、すべての ``ファクター`` と多くの ``ファクター`` メソッドは、mask引数を受け取ることができます。
ただし、その引数は計算時にどのような資産を対象とすべきかを示す ``Filter`` になってなければなりません。

ファクターに対するマスキング
-----------------------------

売買代金が10,000,000ドルよりも大きい銘柄のみを対象とし、値段変化率が高いまたは低い銘柄を出力するパイプラインを考えてみましょう。
これを実現するためには ``make_pipeline`` 関数を再編集して最初に ``high_dollar_volume`` フィルタを作成します。
そして ``high_dollar_volume`` フィルタを ``SimpleMovingAverage`` の ``mask`` 引数として渡すことで、移動平均を計算するファクターの ``mask`` とすることができます。

.. code:: ipython2

    # 売買代金を計算するファクター
    dollar_volume = AverageDollarVolume(window_length=30)
    
    # 高売買代金フィルタ
    high_dollar_volume = (dollar_volume > 10000000)
    
    # 終値移動平均を計算するファクター
    mean_close_10 = SimpleMovingAverage(inputs=[USEquityPricing.close], window_length=10, mask=high_dollar_volume)
    mean_close_30 = SimpleMovingAverage(inputs=[USEquityPricing.close], window_length=30, mask=high_dollar_volume)
    
    # 変化率を計算するファクター
    percent_difference = (mean_close_10 - mean_close_30) / mean_close_30


``high_dollar_volume`` フィルタによって ``SimpleMovingAverage`` をマスキングすることで、終値平均値段のファクターの計算対象は
マスキング無しで約8,000銘柄となるのに対し、約2,000銘柄に絞り込まれます。
``mean_close_10`` と ``mean_close_30`` を組み合わせた ``percent_defference`` ファクターの計算は同じ約2,000銘柄を対象に行われます。


フィルタに対するマスキング
-----------------------------

マスキングは ``top`` 、 ``bottom`` 、 ``percentile_between`` といったフィルタを返すメソッドに対しても適用可能です。
マスキングは結合処理の早い段階でフィルタを適用したいときに最も役立ちます。
例えば売買代金トップ10%の中で始値が最も高い50銘柄を取り出したいとします。
さらに、それらの銘柄のうち終値が第90分位から第100分位の間に含まれる銘柄を取り出したいとします。
これらは以下のようにして実現できます：


.. code:: ipython2

    # 売買代金を計算するファクター
    dollar_volume = AverageDollarVolume(window_length=30)
    
    # 高売買代金フィルタ
    high_dollar_volume = dollar_volume.percentile_between(90,100)
    
    # 始値上位フィルタ (高売買代金フィルタでマスキング)
    top_open_price = USEquityPricing.open.latest.top(50, mask=high_dollar_volume)
    
    # 終値分位上位フィルタ (高売買代金フィルタと始値上位50銘柄フィルタでマスキング)
    high_close_price = USEquityPricing.close.latest.percentile_between(90, 100, mask=top_open_price)

これを ``make_pipeline`` に追加し、``high_close_price`` でスクリーニングをかけた空のパイプラインを出力します。

.. code:: ipython2

    def make_pipeline():
    
        # 売買代金を計算するファクター
        dollar_volume = AverageDollarVolume(window_length=30)
    
        # 高売買代金フィルタ
        high_dollar_volume = dollar_volume.percentile_between(90,100)
    
        # 始値上位フィルタ (高売買代金フィルタでマスキング)
        top_open_price = USEquityPricing.open.latest.top(50, mask=high_dollar_volume)
    
        # 終値分位上位フィルタ (高売買代金フィルタと始値上位50銘柄フィルタでマスキング)
        high_close_price = USEquityPricing.close.latest.percentile_between(90, 100, mask=top_open_price)
    
        return Pipeline(
            screen=high_close_price
        )

2015年5月15日付のパイプライン実行結果は、5銘柄の出力となります。

.. code:: ipython2

    result = run_pipeline(make_pipeline(), '2015-05-05', '2015-05-05')
    print 'Number of securities that passed the filter: %d' % len(result)

.. todo::

    python2コードの修正どうするか

.. parsed-literal::

    Number of securities that passed the filter: 5


上記で行ったようにレイヤにマスクを適用することは、 「資産の絞り込み作業」 と捉えることができます。
次のレッスンでは、クラシファイアについて見ていきます。
