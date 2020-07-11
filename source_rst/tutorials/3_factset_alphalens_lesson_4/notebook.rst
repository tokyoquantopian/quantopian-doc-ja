Companion notebook for Alphalens tutorial lesson 4
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Advanced Alphalens concepts
===========================

You’ve learned the basics of using Alphalens. This lesson explores the
following advanced Alphalens concepts:

1. Grouping assets by market cap, then analyzing each cap type
   individually.
2. Writing group neutral strategies.
3. Determining an alpha factor’s decay rate.
4. Dealing with a common Alphalens error named MaxLossExceededError.

Grouping By Market Cap
----------------------

The following code defines a universe and creates an alpha factor within
a pipeline. It also returns a classifier by using the quantiles()
function. This function is useful for grouping your assets by an
arbitrary column of data. In this example, we will group our assets by
their market cap, and analyze how effective our alpha factor is among
the different cap types (small, medium, and large cap).

.. code:: ipython2

    from quantopian.pipeline import Pipeline
    from quantopian.research import run_pipeline
    from quantopian.pipeline.factors import AverageDollarVolume
    from quantopian.pipeline.data import factset, USEquityPricing
    from alphalens.utils import get_clean_factor_and_forward_returns
    
    
    def make_pipeline():
        # Filter out equities with low market capitalization
        market_cap_filter = factset.Fundamentals.mkt_val.latest > 500000000
    
        # Filter out equities with low volume
        volume_filter = AverageDollarVolume(window_length=200) > 2500000
    
        # Filter out equities with a close price below $5
        price_filter = USEquityPricing.close.latest > 5
    
        # Our final base universe
        base_universe = market_cap_filter & volume_filter & price_filter
        
        change_in_working_capital = factset.Fundamentals.wkcap_chg_qf.latest
        ciwc_processed = change_in_working_capital.winsorize(.2, .98).zscore()
        
        sales_per_working_capital = factset.Fundamentals.sales_wkcap_qf.latest
        spwc_processed = sales_per_working_capital.winsorize(.2, .98).zscore()
    
        factor_to_analyze = (ciwc_processed + spwc_processed).zscore()
    
        market_cap = factset.Fundamentals.mkt_val.latest
        
        # .quantiles(), when supplied with bins=3, tells you which third that the assets value places in.
        # for example, in 2018, Apple is in the third bin, because it has a large market cap.
        # A different asset with a smaller market cap would probably be in the first or second bin.
        cap_type = market_cap.quantiles(bins=3, mask=base_universe)
    
        return Pipeline(
            columns = {
              'factor_to_analyze': factor_to_analyze, 
              'cap_type': cap_type
            },
            screen = (
                base_universe
                & factor_to_analyze.notnull()
                & market_cap.notnull()
            )
        )
    
    # Create the pipeline data
    pipeline_output = run_pipeline(make_pipeline(), '2013-1-1', '2014-1-1')
    
    # Replace the quantile values in the cap_type column for added clarity
    pipeline_output['cap_type'].replace([0, 1, 2], ['small_cap', 'mid_cap', 'large_cap'], inplace=True)
    
    pricing_data = get_pricing(pipeline_output.index.levels[1], '2013-1-1', '2014-3-1', fields='open_price')
    
    pipeline_output.head(5)

Analyzing Alpha Factors By Group
--------------------------------

Alphalens allows you to group assets using a classifier. A common use
case for this is classifying equities by market cap then comparing your
alpha factor’s returns among cap types.

You can group assets by any classifier, but sector and market cap are
most common. The Pipeline in the first cell of this lesson returns a
column named ``cap_type``, whose values represent the assets market
capitalization. All we have to do now is pass that column to the
``groupby`` argument of ``get_clean_factor_and_forward_returns()``

**Run the following cell, and notice the charts at the bottom of the
tear sheet showing how our factor performs among different cap types.**

**Important note**: Until this lesson, we passed the output of
``run_pipeline()`` to ``get_clean_factor_and_forward_returns()`` without
any changes. This was possible because the previous lessons’ Pipelines
only returned one column. This lesson’s Pipeline returns two columns,
which means we need to *specify the column* we’re passing as factor
data. Look for commented code near
``get_clean_factor_and_forward_returns()`` in the following cell to see
how to do this.

.. code:: ipython2

    from alphalens.tears import create_returns_tear_sheet
    
    factor_data = get_clean_factor_and_forward_returns(
        factor=pipeline_output['factor_to_analyze'], # This is how you pass a single column from pipeline_output
        prices=pricing_data,
        groupby=pipeline_output['cap_type'],
    )
    
    create_returns_tear_sheet(factor_data=factor_data, by_group=True)

Writing Group Neutral Strategies
--------------------------------

Not only does Alphalens allow us to simulate how our alpha factor would
perform in a long/short trading strategy, it also allows us to simulate
how it would do if we went long/short on every group!

Grouping by cap type, and going long/short on each cap type allows you
to limit exposure to the overall movement of those market cap groups.
For example, you may have noticed in step three of this tutorial, that
certain cap types had all positive returns, or all negative returns.
That information isn’t useful to us, because that just means the market
cap group outperformed (or underperformed) the market; it doesn’t give
us any insight into how our factor performs within that cap type.

Since we grouped our assets by cap type in the previous cell, going
group neutral is easy; just make the two following changes: - Pass
``binning_by_group=True`` as an argument to
``get_clean_factor_and_forward_returns()``. - Pass
``group_neutral=True`` as an argument to ``create_full_tear_sheet()``.

**The following cell has made the approriate changes. Try running it and
notice how the results differ from the previous cell.**

.. code:: ipython2

    factor_data = get_clean_factor_and_forward_returns(
        pipeline_output['factor_to_analyze'],
        prices=pricing_data,
        groupby=pipeline_output['cap_type'],
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

.. code:: ipython2

    longest_look_forward_period = 63 # week = 5, month = 21, quarter = 63, year = 252
    range_step = 5
    
    factor_data = get_clean_factor_and_forward_returns(
        factor = pipeline_output['factor_to_analyze'],
        prices = pricing_data,
        periods = range(1, longest_look_forward_period, range_step)
    )
    
    from alphalens.performance import mean_information_coefficient
    mean_information_coefficient(factor_data).plot(title="IC Decay");

What do you think the chart will look like if we calculate the IC a full
year into the future?

*Hint*: This is a setup for the next part of this lesson.

.. code:: ipython2

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

.. code:: ipython2

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

