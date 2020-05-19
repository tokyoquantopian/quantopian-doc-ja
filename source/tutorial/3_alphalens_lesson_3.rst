
.. note:: 

    本Tutorialのコードは、`ドキュメント原作ページ <https://www.quantopian.com/tutorials/alphalens#lesson3>`__ にある ``Get Notebook`` ボタンでクローン出来ます。


Alphalensティアシートを読み解く
==================================

前回のレッスンでは、Alphalensのティアシートを使ってデータを分析できるように、データを取得して処理する方法を学びました。

このレッスンでは、`quant workflow <https://blog.quantopian.com/a-professional-quant-equity-workflow/>`__ のアルファ発見フェーズのいくつかの反復を、ティアシートを分析することで体験していただきます。
このレッスンでは、ティアシートを分析しながらアルファファクターを探す一連の流れをクォントワークフローに沿って経験していきましょう。

このような順で行います：

1. ``create_information_tear_sheet()`` を使って、各アルファファクターがどのくらい適切に将来価格を予測しているかを分析
2. 他のアルファファクターを組み合わせて、自分のアルファファクターを改善
3. ``create_returns_tear_sheet()`` を使って、自分のアルファファクターの収益性を測る。


はじめてのアルファファクター
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
次のコードは、企業の純利益と時価総額からアルファファクターを表現し、そのアルファファクターの情報ティアシート(information tear sheet)を作成します。

まずはアルファファクターの情報係数 (IC) から分析していきます。
ICは、-1から1までの数値で、アルファファクターがどの程度予測できるか定量化するための数値です。
0以上の数値は、予測可能性がやや高いと考えられます。

最初に見るべき数値はICの平均値です。これは与えられた期間におけるアルファファクターの平均ICです。
ファクターのIC平均はできるだけ高いほうがよいです。多くの場合、ファクター平均が0以上であればそのファクターをさらに調査する価値があります。もし大きな取引ユニバースにおいて、IC平均が0.1あたり（もしくはそれ以上）の場合、そのファクターはおそらくとても良いものと言えるでしょう。


.. code:: ipython3

    from quantopian.pipeline.data import factset

    from quantopian.pipeline import Pipeline
    from quantopian.research import run_pipeline
    from quantopian.pipeline.factors import CustomFactor, SimpleMovingAverage
    from quantopian.pipeline.filters import QTradableStocksUS
    from alphalens.tears import create_information_tear_sheet
    from alphalens.utils import get_clean_factor_and_forward_returns


    def make_pipeline():

        # 年次の企業収益を、1年間の移動平均で取得
        net_income_moving_average = SimpleMovingAverage( 
        inputs=[factset.Fundamentals.net_inc_af], 
        window_length=252
        )

        # 時価総額を1年間の移動平均で取得
        market_cap_moving_average = SimpleMovingAverage( 
        inputs=[factset.Fundamentals.mkt_val], 
        window_length=252
        )

        average_market_cap_per_net_income = (market_cap_moving_average / net_income_moving_average)

        # 直近四半期の企業収益を取得
        net_income = factset.Fundamentals.net_inc_qf.latest 

        projected_market_cap = average_market_cap_per_net_income * net_income

        return Pipeline(
        columns={'projected_market_cap': projected_market_cap},
        screen=QTradableStocksUS() & projected_market_cap.notnull()
        )


    factor_data = run_pipeline(make_pipeline(), '2010-1-1', '2012-1-1')
    pricing_data = get_pricing(factor_data.index.levels[1], '2010-1-1', '2012-2-1', fields='open_price')
    merged_data = get_clean_factor_and_forward_returns(factor_data, pricing_data)

    create_information_tear_sheet(merged_data)

Below is the first chart produced by create_information_tear_sheet(). Notice how the IC Mean figures are all positive. That is a good sign!

``create_information_tear_sheet()`` で作成された最初のチャートが出来ました。IC平均値がすべて正の値であることに注目してください。これは良い兆候です。

.. image:: notebook_files/alphalens_l3_screenshot1.png



Add Another Alpha Factor
~~~~~~~~~~~~~~~~~~~~~~~~

**Alphalens is useful for identifying alpha factors that aren’t
predictive early in the quant workflow. This allows you to avoid wasting
time running a full backtest on a factor that could have been discarded
earlier in the process.**

Run the following cell to express another alpha factor called
``price_to_book``, combine it with ``projected_market_cap`` using
zscores and winsorization, then creates another information tearsheet
based on our new (and hopefully improved) alpha factor.

Notice how the IC figures are lower than they were in the first chart.
That means the factor we added is making our predictions worse!

.. code:: ipython3

    def make_pipeline():
    
        net_income_moving_average = SimpleMovingAverage( # 1 year moving average of year over year net income
            inputs=[factset.Fundamentals.net_inc_af], 
            window_length=252
        )
        
        market_cap_moving_average = SimpleMovingAverage( # 1 year moving average of market cap
            inputs=[factset.Fundamentals.mkt_val], 
            window_length=252
        )
        
        average_market_cap_per_net_income = (market_cap_moving_average / net_income_moving_average)
        
        net_income = factset.Fundamentals.net_inc_qf.latest # the last quarter's net income
        
        projected_market_cap = average_market_cap_per_net_income * net_income
        
        price_to_book = factset.Fundamentals.pbk_qf.latest
        
        factor_to_analyze = projected_market_cap.zscore() + price_to_book.zscore()
        
        return Pipeline(
            columns = {'factor_to_analyze': factor_to_analyze},
            screen = QTradableStocksUS() & factor_to_analyze.notnull()
        )
    
    
    
    pipeline_output = run_pipeline(make_pipeline(), '2010-1-1', '2012-1-1')
    pricing_data = get_pricing(pipeline_output.index.levels[1], '2010-1-1', '2012-2-1', fields='open_price')
    new_factor_data = get_clean_factor_and_forward_returns(pipeline_output, pricing_data)
    
    create_information_tear_sheet(new_factor_data)

See If Our Alpha Factor Might Be Profitable
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We found that the first iteration of our alpha factor had more
predictive value than the second one. Let’s see if the original alpha
factor might make any money.

``create_returns_tear_sheet()`` splits your universe into quantiles,
then shows the returns generated by each quantile over different time
periods. Quantile 1 is the 20% of assets with the lowest alpha factor
values, and quantile 5 is the highest 20%.

This function creates six types of charts, but the two most important
ones are:

-  **Mean period-wise returns by factor quantile:** This chart shows the
   average return for each quantile in your universe, per time period.
   You want the quantiles on the right to have higher average returns
   than the quantiles on the left.
-  **Cumulative return by quantile:** This chart shows you how each
   quantile performed over time. You want to see quantile 1 consistently
   performing the worst, quantile 5 consistently performing the best,
   and the other quantiles in the middle.

**Run the following cell, and notice how quantile 5 doesn’t have the
highest returns. Ideally, you want quantile 1 to have the lowest
returns, and quantile 5 to have the highest returns. This tear sheet is
telling us we still have work to do!**

.. code:: ipython3

    from alphalens.tears import create_returns_tear_sheet
    
    create_returns_tear_sheet(factor_data)

In this lesson, you experienced a few cycles of the alpha discovery
stage of the quant worfklow. Making good alpha factors isn’t easy, but
Alphalens allows you to iterate through them quickly to find out if
you’re on the right track! You can usually improve existing alpha
factors in some way by getting creative with moving averages, looking
for trend reversals, or any number of other stratgies.

Try looking around `Quantopian’s
forums <https://www.quantopian.com/posts>`__, or reading academic papers
for inspiration. **This is where you get to be creative!** In the next
lesson, we’ll discuss advanced Alphalens concepts.

