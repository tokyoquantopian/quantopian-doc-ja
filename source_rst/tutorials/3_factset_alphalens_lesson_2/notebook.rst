Companion notebook for Alphalens tutorial lesson 2
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Creating Tear Sheets With Alphalens
===================================

In the previous lesson, you learned what Alphalens is. In this lesson,
you will learn a four step process for how to use it:

1. Express an alpha factor and define a trading universe by creating and
   running a Pipeline over a certain time period.
2. Query pricing data for the assets in our universe during that same
   time period with ``get_pricing()``.
3. Align the alpha factor data with the pricing data with
   ``get_clean_factor_and_forward_returns()``.
4. Visualize how well our alpha factor predicts future price movements
   with ``create_full_tear_sheet()``.

Build And Run A Pipeline
------------------------

The following code creates a trading universe and expresses an alpha
factor within a pipeline, then runs it with ``run_pipeline()``

.. code:: ipython2

    from quantopian.pipeline import Pipeline
    from quantopian.research import run_pipeline
    from quantopian.pipeline.factors import AverageDollarVolume
    from quantopian.pipeline.data import factset, USEquityPricing
    
    def make_pipeline():
        # Filter out equities with low market capitalization
        market_cap_filter = factset.Fundamentals.mkt_val.latest > 500000000
    
        # Filter out equities with low volume
        volume_filter = AverageDollarVolume(window_length=200) > 2500000
    
        # Filter out equities with a close price below $5
        price_filter = USEquityPricing.close.latest > 5
    
        # Our final base universe
        base_universe = market_cap_filter & volume_filter & price_filter
    
        # Measures a company's asset growth rate.
        asset_growth = factset.Fundamentals.assets_gr_qf.latest
    
        return Pipeline(
          columns={'asset_growth': asset_growth},
          screen=base_universe & asset_growth.notnull()
        )
    
    pipeline_output = run_pipeline(pipeline=make_pipeline(), start_date='2014-1-1', end_date='2016-1-1')
    
    # Show the first 5 rows of factor data
    pipeline_output.head(5)

Query Pricing Data
------------------

Now that we have factor data, let’s get pricing data for the same time
period. ``get_pricing()`` returns pricing data for a list of assets over
a specified time period. It requires four arguments: - A list of assets
for which we want pricing. - A start date - An end date - Whether to use
open, high, low or close pricing.

Execute the following cell to get pricing data.

.. code:: ipython2

    pricing_data = get_pricing(
        symbols=pipeline_output.index.levels[1], # Finds all assets that appear at least once in "pipeline_output"  
        start_date='2014-1-1',
        end_date='2016-2-1', # must be after run_pipeline()'s end date. Explained more in lesson 4
        fields='open_price' # Generally, you should use open pricing. Explained more in lesson 4
    )
    
    # Show the first 5 rows of pricing_data
    pricing_data.head(5)

Align Data
----------

``get_clean_factor_and_forward_returns()`` aligns the factor data
created by ``run_pipeline()`` with the pricing data created by
``get_pricing()``, and returns an object suitable for analysis with
Alphalens’ charting functions. It requires two arguments: - The factor
data we created with ``run_pipeline()``. - The pricing data we created
with ``get_pricing()``.

Execute the following cell to align the factor data with the pricing
data.

.. code:: ipython2

    from alphalens.utils import get_clean_factor_and_forward_returns
    
    factor_data = get_clean_factor_and_forward_returns(
        factor=pipeline_output, 
        prices=pricing_data
    )
    
    # Show the first 5 rows of merged_data
    factor_data.head(5)

Visualize Results
-----------------

Finally, execute the following cell to pass the output of
``get_clean_factor_and_forward_returns()`` to a function called
``create_full_tear_sheet()``. This will create whats known as a tear
sheet.

.. code:: ipython2

    from alphalens.tears import create_full_tear_sheet
    
    create_full_tear_sheet(factor_data)

That’s It!
----------

In the next lesson, we will show you how to interpret the charts
produced by ``create_full_tear_sheet()``.

