Alphalens Quickstart Template
=============================

.. code:: ipython3

    from quantopian.pipeline import Pipeline
    from quantopian.research import run_pipeline
    from quantopian.pipeline.data import factset, USEquityPricing
    from quantopian.pipeline.factors import SimpleMovingAverage, AverageDollarVolume
    
    from alphalens.performance import mean_information_coefficient
    from alphalens.utils import get_clean_factor_and_forward_returns
    from alphalens.tears import create_information_tear_sheet, create_returns_tear_sheet

Define Your Alpha Factor Here
-----------------------------

Spend your time in this cell, creating good factors. Then simply run the
rest of the notebook to analyze ``factor_to_analyze``!

.. code:: ipython3

    def make_pipeline():
        # Filter out equities with low market capitalization
        market_cap_filter = factset.Fundamentals.mkt_val.latest > 500000000
    
        # Filter out equities with low volume
        volume_filter = AverageDollarVolume(window_length=200) > 2500000
    
        # Filter out equities with a close price below $5
        price_filter = USEquityPricing.close.latest > 5
    
        # Our final base universe
        base_universe = market_cap_filter & volume_filter & price_filter
        
        assets_moving_average = SimpleMovingAverage(inputs=[factset.Fundamentals.assets], window_length=252)
        current_assets = factset.Fundamentals.assets.latest
        
        # This is the factor that the rest of the notebook will analyze
        factor_to_analyze = (current_assets - assets_moving_average)
        
        # The following columns will help us group assets by market cap. This will allow us to analyze
        # whether our alpha factor's predictiveness varies among assets with different market caps.
        market_cap = factset.Fundamentals.mkt_val.latest
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
    
    pipeline_output = run_pipeline(make_pipeline(), '2015-1-1', '2016-1-1')
    # rename the 1's, 2's and 3's for clarity
    pipeline_output['cap_type'].replace([1, 2, 3], ['small_cap', 'mid_cap', 'large_cap'], inplace=True)
    
    pricing_data = get_pricing(pipeline_output.index.levels[1], '2015-1-1', '2016-6-1', fields='open_price')

Create Group Neutral Tear Sheets
--------------------------------

.. code:: ipython3

    factor_data = get_clean_factor_and_forward_returns(
        factor = pipeline_output['factor_to_analyze'],
        prices = pricing_data,
        groupby = pipeline_output['cap_type'],
        binning_by_group = True,
        periods = (1,5,10)
    )
    
    create_information_tear_sheet(factor_data, by_group=True, group_neutral=True)
    create_returns_tear_sheet(factor_data, by_group=True, group_neutral=True)

Determine The Decay Rate Of Your Alpha Factor
---------------------------------------------

.. code:: ipython3

    longest_look_forward_period = 63 # week = 5, month = 21, quarter = 63, year = 252
    range_step = 5
    
    factor_data = get_clean_factor_and_forward_returns(
        factor = pipeline_output['factor_to_analyze'],
        prices = pricing_data,
        periods = range(1, longest_look_forward_period, range_step)
    )
    
    mean_information_coefficient(factor_data).plot(title="IC Decay")

