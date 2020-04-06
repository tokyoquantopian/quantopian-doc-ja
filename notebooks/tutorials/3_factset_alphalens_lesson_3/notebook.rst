Companion notebook for Alphalens tutorial lesson 3
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Interpreting Alphalens Tear Sheets
==================================

In the previous lesson, you learned how to query and process data so
that we can analyze it with Alphalens tear sheets. In this lesson, you
will experience a few iterations of the alpha discovery phase of the
`quant
workflow <https://blog.quantopian.com/a-professional-quant-equity-workflow/>`__
by analyzing those tear sheets.

In this lesson, we will:

1. Analyze how well an alpha factor predicts future price movements with
   ``create_information_tear_sheet()``.
2. Try to improve our original alpha factor by combining it with another
   alpha factor.
3. Preview how profitable our alpha factor might be with
   ``create_returns_tear_sheet()``.

Our Starting Alpha Factor
~~~~~~~~~~~~~~~~~~~~~~~~~

The following code expresses an alpha factor based on a company’s net
income and market cap, then creates an information tear sheet for that
alpha factor. We will start analyzing the alpha factor by looking at
it’s information coefficient (IC). The IC is a number ranging from -1,
to 1, which quantifies the predictiveness of an alpha factor. Any number
above 0 is considered somewhat predictive.

The first number you should look at is the IC mean, which is an alpha
factor’s average IC over a given time period. You want your factor’s IC
Mean to be as high as possible. Generally speaking, a factor is worth
investigating if it has an IC mean over 0. If it has an IC mean close to
.1 (or higher) over a large trading universe, that factor is probably
**exceptionally good**. In fact, you might want to check to make sure
there isn’t some lookahead bias if your alpha factor’s IC mean is over
.1

**Run the cell below to create an information tear sheet for our alpha
factor. Notice how the IC Mean figures (the first numbers on the first
chart) are all positive. That is a good sign!**

.. code:: ipython2

    from quantopian.pipeline import Pipeline
    from quantopian.research import run_pipeline
    from alphalens.tears import create_information_tear_sheet
    from quantopian.pipeline.data import factset, USEquityPricing
    from alphalens.utils import get_clean_factor_and_forward_returns
    from quantopian.pipeline.factors import CustomFactor, SimpleMovingAverage, AverageDollarVolume
    
    
    def make_pipeline():
        # Filter out equities with low market capitalization
        market_cap_filter = factset.Fundamentals.mkt_val.latest > 500000000
    
        # Filter out equities with low volume
        volume_filter = AverageDollarVolume(window_length=200) > 2500000
    
        # Filter out equities with a close price below $5
        price_filter = USEquityPricing.close.latest > 5
    
        # Our final base universe
        base_universe = market_cap_filter & volume_filter & price_filter
    
        # 1 year moving average of year over year net income
        net_income_moving_average = SimpleMovingAverage( 
          inputs=[factset.Fundamentals.net_inc_af], 
          window_length=252
        )
    
        # 1 year moving average of market cap
        market_cap_moving_average = SimpleMovingAverage( 
          inputs=[factset.Fundamentals.mkt_val], 
          window_length=252
        )
    
        average_market_cap_per_net_income = (market_cap_moving_average / net_income_moving_average)
    
        # the last quarter's net income
        net_income = factset.Fundamentals.net_inc_qf.latest 
    
        projected_market_cap = average_market_cap_per_net_income * net_income
    
        return Pipeline(
          columns={'projected_market_cap': projected_market_cap},
          screen=base_universe & projected_market_cap.notnull()
        )
    
    
    pipeline_output = run_pipeline(make_pipeline(), '2010-1-1', '2012-1-1')
    pricing_data = get_pricing(pipeline_output.index.levels[1], '2010-1-1', '2012-2-1', fields='open_price')
    factor_data = get_clean_factor_and_forward_returns(pipeline_output, pricing_data)
    
    create_information_tear_sheet(factor_data)

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

.. code:: ipython2

    def make_pipeline():
        # Filter out equities with low market capitalization
        market_cap_filter = factset.Fundamentals.mkt_val.latest > 500000000
    
        # Filter out equities with low volume
        volume_filter = AverageDollarVolume(window_length=200) > 2500000
    
        # Filter out equities with a close price below $5
        price_filter = USEquityPricing.close.latest > 5
    
        # Our final base universe
        base_universe = market_cap_filter & volume_filter & price_filter
    
        # 1 year moving average of year over year net income
        net_income_moving_average = SimpleMovingAverage( 
            inputs=[factset.Fundamentals.net_inc_af], 
            window_length=252
        )
    
        # 1 year moving average of market cap
        market_cap_moving_average = SimpleMovingAverage( 
            inputs=[factset.Fundamentals.mkt_val], 
            window_length=252
        )
    
        average_market_cap_per_net_income = (market_cap_moving_average / net_income_moving_average)
    
        # The last quarter's net income
        net_income = factset.Fundamentals.net_inc_qf.latest
    
        projected_market_cap = average_market_cap_per_net_income * net_income
    
        # The alpha factor we are adding
        price_to_book = factset.Fundamentals.pbk_qf.latest 
    
        factor_to_analyze = projected_market_cap.zscore() + price_to_book.zscore()
    
        return Pipeline(
            columns={'factor_to_analyze': factor_to_analyze},
            screen=base_universe & factor_to_analyze.notnull()
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

.. code:: ipython2

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

