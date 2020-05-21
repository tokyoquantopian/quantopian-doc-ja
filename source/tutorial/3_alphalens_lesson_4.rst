Advanced Alphalens concepts
===========================

You've learned the basics of using Alphalens. This lesson explores the following advanced Alphalens concepts:
Alphalensの基本的な使い方を学びました。このレッスンでは、より高度な Alphalensの概念を学びます。

1. Determining how far an alpha factor's predictive value stretches into the future.アルファファクターがどのくらい長く将来を予測できるか見極める
2. Dealing with a common Alphalens error named MaxLossExceededError. ``MaxLossExceededError``という Alphalensのエラーに対応する
3. Grouping assets by sector, then analyzing each sector individually. セクターごとに資産をグループにまとめ、各セクターを分析。
4. Writing group neutral strategies.どのグループにも偏らないストラテジーを考える

The following code creates an alpha factor in a pipeline. The rest of this lesson will discuss advanced Alphalens concepts using the data created by the pipeline.
次のコードは、パイプラインでアルファファクターを作成します。このレッスンでは、パイプラインが算出する、より高度なアルファレンズの概念を学んでいきましょう。

**重要な注意**：
これまでのレッスンでは、``get_clean_factor_and_forward_returns()`` に何の変更も加えずに、 ``run_pipeline()`` の出力結果を渡していました。これは、パイプラインが返す結果が１つのカラムだけだったからです。このレッスンではパイプラインは２つのカラムを持つ結果を返すので、ファクターデータのカラムを指定する必要があります。指定方法は、下記のコードの ``get_clean_factor_and_forward_returns()`` の中でコメントを見て下さい。

.. code:: ipython3

    from quantopian.pipeline import Pipeline
    from quantopian.pipeline.data import factset
    from quantopian.research import run_pipeline
    from quantopian.pipeline.filters import QTradableStocksUS
    from quantopian.pipeline.classifiers.fundamentals import Sector
    from alphalens.utils import get_clean_factor_and_forward_returns


    def make_pipeline():
        
        change_in_working_capital = factset.Fundamentals.wkcap_chg_qf.latest
        ciwc_processed = change_in_working_capital.winsorize(.2, .98).zscore()
        
        sales_per_working_capital = factset.Fundamentals.sales_wkcap_qf.latest
        spwc_processed = sales_per_working_capital.winsorize(.2, .98).zscore()

        factor_to_analyze = (ciwc_processed + spwc_processed).zscore()

        sector = Sector()

        return Pipeline(
            columns = {
                'factor_to_analyze': factor_to_analyze,
                'sector': sector,
            },
            screen = (
                QTradableStocksUS()
                & factor_to_analyze.notnull()
                & sector.notnull()
            )
        )


    pipeline_output = run_pipeline(make_pipeline(), '2013-1-1', '2014-1-1')
    pricing_data = get_pricing(pipeline_output.index.levels[1], '2013-1-1', '2014-3-1', fields='open_price')


    factor_data = get_clean_factor_and_forward_returns(
        pipeline_output['factor_to_analyze'], # パイプラインの出力結果のどのコラムをAlphalensに分析させるか指定
        pricing_data, 
        periods=range(1,32,3)
    )


Visualizing an alpha factor's decay rate
------------------------------------------

A lot of fundamental data only comes out 4 times a year in quarterly reports. Because of this low frequency, it can be useful to increase the amount of time get_clean_factor_and_forward_returns() looks into the future to calculate returns.
多くのファンダメンタルデータは、年4回の四半期レポートでしか出てきません。頻度が低いので、

Tip: A month usually has 21 trading days, a quarter usually has 63 trading days, and a year usually has 252 trading days.
**ヒント**：1ヶ月の取引日は通常21日、四半期はは通常63日、1年は通常252日です。

たとえば、利益を上げ続けている会社の株式を買うという戦略を立てたとしましょう（データは６３日取引日ごとにリリースされます）。
利益を上げ続けているというファクターをみて、あなたは今後１０日間だけを株式保有期間として分析するでしょうか？たぶん違うでしょう。もっと長い期間を想定するはずです。しかし、どのくらい先の期間を考えるべきでしょうか？

**次のコードは、アルファファクターの平均ICを描画します。**

.. code:: ipython3

    from alphalens.performance import mean_information_coefficient
    mean_information_coefficient(factor_data).plot(title="IC Decay");

0を下回ったポイントが、アルファファクターの予測が有用ではなくなった時を表現しています。

.. image:: notebook_files/alphalens_l4_screenshot1.png


What do you think the chart will look like if we calculate the IC a full year into the future?
1年先のICを計算すると、チャートはどうなると思いますか？

*メモ*: This is a setup for section two of this lesson.

.. code:: ipython3

    factor_data = get_clean_factor_and_forward_returns(
        pipeline_output['factor_to_analyze'], 
        pricing_data,
        periods=range(1,252,20) # この第三引数は、The third argument to the range statement changes the "step" of the range
    )

    mean_information_coefficient(factor_data).plot()

Running the code above would produce the following error:




.. code:: ipython3

    from alphalens.tears import create_returns_tear_sheet
    
    sector_labels, sector_labels[-1] = dict(Sector.SECTOR_NAMES), "Unknown"
    
    factor_data = get_clean_factor_and_forward_returns(
        factor=pipeline_output['factor_to_analyze'],
        prices=pricing_data,
        groupby=pipeline_output['sector'],
        groupby_labels=sector_labels,
    )
    
    create_returns_tear_sheet(factor_data=factor_data, by_group=True)

Writing Group Neutral Strategies
--------------------------------

Not only does Alphalens allow us to simulate how our alpha factor would
perform in a long/short trading strategy, it also allows us to simulate
how it would do if we went long/short on every group!

Grouping by sector, and going long/short on each sector allows you to
limit exposure to the overall movement of sectors. For example, you may
have noticed in step three of this tutorial, that certain sectors had
all positive returns, or all negative returns. That information isn’t
useful to us, because that just means the sector group outperformed (or
underperformed) the market; it doesn’t give us any insight into how our
factor performs within that sector.

Since we grouped our assets by sector in the previous cell, going group
neutral is easy; just make the two following changes: - Pass
``binning_by_group=True`` as an argument to
``get_clean_factor_and_forward_returns()``. - Pass
``group_neutral=True`` as an argument to ``create_full_tear_sheet()``.

**The following cell has made the approriate changes. Try running it and
notice how the results differ from the previous cell.**

.. code:: ipython3

    factor_data = get_clean_factor_and_forward_returns(
        pipeline_output['factor_to_analyze'],
        prices=pricing_data,
        groupby=pipeline_output['sector'],
        groupby_labels=sector_labels,
        binning_by_group=True,
    )
    
    create_returns_tear_sheet(factor_data, by_group=True, group_neutral=True)

Visualizing An Alpha Factor’s Decay Rate
----------------------------------------

A lot of fundamental data only comes out 4 times a year in quarterly
reports. Because of this low frequency, it can be useful to increase the
amount of time ``get_clean_factor_and_forward_returns()`` looks into the
future to calculate returns.

**Tip:** A month usually has 21 trading days, a quarter usually has 63
trading days, and a year usually has 252 trading days.

Let’s say you’re creating a strategy that buys stock in companies with
rising profits (data that is released every 63 trading days). Would you
only look 10 days into the future to analyze that factor? Probably not!
But how do you decide how far to look forward?

**Run the following cell to chart our alpha factor’s IC mean over time.
The point where the line dips below 0 represents when our alpha factor’s
predictions stop being useful.**

.. code:: ipython3

    from alphalens.performance import mean_information_coefficient
    mean_information_coefficient(factor_data).plot(title="IC Decay");

What do you think the chart will look like if we calculate the IC a full
year into the future?

*Hint*: This is a setup for section two of this lesson.

.. code:: ipython3

    factor_data = get_clean_factor_and_forward_returns(
        pipeline_output['factor_to_analyze'], 
        pricing_data,
        periods=range(1,252,20) # The third argument to the range statement changes the "step" of the range
    )
    
    mean_information_coefficient(factor_data).plot()

Dealing With MaxLossExceededError
---------------------------------

Oh no! What does ``MaxLossExceededError`` mean?

``get_clean_factor_and_forward_returns()`` looks at how alpha factor
data affects pricing data *in the future*. This means we need our
pricing data to go further into the future than our alpha factor data
**by at least as long as our forward looking period.**

In this case, we’ll change ``get_pricing()``\ ’s ``end_date`` to be at
least a year after ``run_pipeline()``\ ’s ``end_date``.

**Run the following cell to make those changes. As you can see, this
alpha factor’s IC decays quickly after a quarter, but comes back even
stronger six months into the future. Interesting!**

.. code:: ipython3

    pipeline_output = run_pipeline(
        make_pipeline(),
        start_date='2013-1-1', 
        end_date='2014-1-1' #  *** NOTE *** Our factor data ends in 2014
    )
    
    pricing_data = get_pricing(
        pipeline_output.index.levels[1], 
        start_date='2013-1-1',
        end_date='2015-2-1', # *** NOTE *** Our pricing data ends in 2015
        fields='open_price'
    )
    
    factor_data = get_clean_factor_and_forward_returns(
        pipeline_output['factor_to_analyze'], 
        pricing_data,
        periods=range(1,252,20) # Change the step to 10 or more for long look forward periods to save time
    )
    
    mean_information_coefficient(factor_data).plot()

*Note: MaxLossExceededError has two possible causes; forward returns
computation and binning. We showed you how to fix forward returns
computation here because it is much more common. Try passing
``quantiles=None`` and ``bins=5`` if you get MaxLossExceededError
because of binning.*

That’s it! This tutorial got you started with Alphalens, but there’s so
much more to it. Check out our `API
docs <http://quantopian.github.io/alphalens/>`__ to see the rest!

