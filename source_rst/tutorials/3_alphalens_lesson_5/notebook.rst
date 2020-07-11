Alphalens Quickstart Template
=============================

.. code:: ipython3

    from quantopian.pipeline import Pipeline
    from quantopian.research import run_pipeline
    from quantopian.pipeline.filters import QTradableStocksUS
    from quantopian.pipeline.data import factset, USEquityPricing
    from quantopian.pipeline.classifiers.fundamentals import Sector
    from quantopian.pipeline.factors import Returns, SimpleMovingAverage, CustomFactor, RSI
    
    from alphalens.performance import mean_information_coefficient
    from alphalens.utils import get_clean_factor_and_forward_returns
    from alphalens.tears import create_information_tear_sheet, create_returns_tear_sheet

Define Your Alpha Factor Here
-----------------------------

Spend your time in this cell, creating good factors. Then simply run the
rest of the notebook to analyze ``factor_to_analyze``!

.. code:: ipython3

    def make_pipeline():
        
        assets_moving_average = SimpleMovingAverage(inputs=[factset.Fundamentals.assets], window_length=252)
        current_assets = factset.Fundamentals.assets.latest
        
        factor_to_analyze = (current_assets - assets_moving_average)
        
        sector = Sector()
        
        return Pipeline(
            columns = {'factor_to_analyze': factor_to_analyze, 'sector': sector},
            screen = QTradableStocksUS() & factor_to_analyze.notnull() & sector.notnull()
        )
    
    factor_data = run_pipeline(make_pipeline(), '2015-1-1', '2016-1-1')
    pricing_data = get_pricing(factor_data.index.levels[1], '2015-1-1', '2016-6-1', fields='open_price')

Determine The Decay Rate Of Your Alpha Factor.
----------------------------------------------

.. code:: ipython3

    longest_look_forward_period = 63 # week = 5, month = 21, quarter = 63, year = 252
    range_step = 5
    
    merged_data = get_clean_factor_and_forward_returns(
        factor = factor_data['factor_to_analyze'],
        prices = pricing_data,
        periods = range(1, longest_look_forward_period, range_step)
    )
    
    mean_information_coefficient(merged_data).plot(title="IC Decay")

Create Group Neutral Tear Sheets
--------------------------------

.. code:: ipython3

    sector_labels, sector_labels[-1] = dict(Sector.SECTOR_NAMES), "Unknown"
    
    merged_data = get_clean_factor_and_forward_returns(
        factor = factor_data['factor_to_analyze'],
        prices = pricing_data,
        groupby = factor_data['sector'],
        groupby_labels = sector_labels,
        binning_by_group = True,
        periods = (1,5,10)
    )
    
    create_information_tear_sheet(merged_data, by_group=True, group_neutral=True)
    create_returns_tear_sheet(merged_data, by_group=True, group_neutral=True)
