Pipeline API
------------

The Pipeline API is a powerful tool for cross-sectional analysis of
asset data. It allows us to define a set of calculations on multiple
data inputs and analyze a large amount of assets at a time. Some common
uses for the Pipeline API include:

-  Selecting assets based on filtering rules
-  Ranking assets based on a scoring function
-  Calculating portfolio allocations

To begin, let’s import the Pipeline class and create a function that
returns an empty pipeline. Putting our pipeline definition inside a
function helps us keep things organized as our pipeline grows in
complexity. This is particularly helpful when transferring data
pipelines between Research and the IDE.

.. code:: ipython2

    # Pipeline class
    from quantopian.pipeline import Pipeline
    
    def make_pipeline():
        # Create and return an empty Pipeline
        return Pipeline()

To add an output to our pipeline we need to include a reference to a
dataset, and specify the computations we want to carry out on that data.
For example, we will add a reference to the ``close`` column from the
``USEquityPricing`` dataset. Then, we can define our output to be the
latest value from this column as follows:

.. code:: ipython2

    # Import Pipeline class and USEquityPricing dataset
    from quantopian.pipeline import Pipeline
    from quantopian.pipeline.data import USEquityPricing
    
    def make_pipeline():
        # Get latest closing price
        close_price = USEquityPricing.close.latest
    
        # Return Pipeline containing latest closing price
        return Pipeline(
            columns={
                'close_price': close_price,
            }
        )

The Pipeline API also provides a number of built-in calculations, some
of which are computed over trailing windows of data. For example, the
following code defines an output as the 3 day moving average of the
``close`` column we added above:

.. code:: ipython2

    # Import Pipeline class and datasets
    from quantopian.pipeline import Pipeline
    from quantopian.pipeline.data import USEquityPricing
    
    # Import built-in moving average calculation
    from quantopian.pipeline.factors import SimpleMovingAverage
    
    
    def make_pipeline():
        # Get latest closing price
        close_price = USEquityPricing.close.latest
    
        # Calculate 3 day average closing price
        avg_close = SimpleMovingAverage(
            inputs=[USEquityPricing.close],
            window_length=3,
        )
    
        # Return Pipeline containing close_price
        # and avg_close
        return Pipeline(
            columns={
                'close_price': close_price,
                'avg_close': avg_close,
            }
        )

Universe Selection
~~~~~~~~~~~~~~~~~~

An important part of developing a strategy is defining the set of assets
that we want to consider trading in our portfolio. We usually refer to
this set of assets as our trading universe. A trading universe should be
as large as possible, while also excluding assets that aren’t
appropriate for our portfolio. For example, we might want to exclude
stocks that are illiquid or difficult to trade.

The Pipeline API allows us to easily create filters from data columns
and calculation results for this purpose. In the following example we
create filters for price, market value, and average dollar volume. Then,
we set the screen parameter of our pipeline constructor to be the
intersection (``&``) of all 3 filters:

.. code:: ipython2

    # Import Pipeline class and datasets
    from quantopian.pipeline import Pipeline
    from quantopian.pipeline.data import factset
    from quantopian.pipeline.data import USEquityPricing
    
    # Import built-in moving average and
    # average dollar volume calculations
    from quantopian.pipeline.factors import (
        AverageDollarVolume,
        SimpleMovingAverage,
    )
    
    def make_pipeline():
        # Trading universe characteristics
        mcap_filter = factset.Fundamentals.mkt_val.latest > 500000000
        adv_filter = AverageDollarVolume(window_length=200) > 2500000
        price_filter = USEquityPricing.close.latest > 5
    
        # Combine filters
        base_universe = mcap_filter & adv_filter & price_filter
    
        # Get latest closing price
        close_price = USEquityPricing.close.latest
    
        # Calculate 3 day average closing price
        avg_close = SimpleMovingAverage(
            inputs=[USEquityPricing.close],
            window_length=3,
        )
    
        # Return Pipeline containing close_price
        # and avg_close
        return Pipeline(
            columns={
                'close_price': close_price,
                'avg_close': avg_close,
            },
            screen=base_universe,
        )

Now that our pipeline definition is complete, we can execute it over a
specific period of time using ``run_pipeline``. The output will be a
pandas DataFrame indexed by date and asset, with columns corresponding
to the outputs we added to our pipeline definition:

.. code:: ipython2

    # Import run_pipeline method
    from quantopian.research import run_pipeline
    
    # Execute pipeline created by make_pipeline
    # between start_date and end_date
    pipeline_output = run_pipeline(
        make_pipeline(),
        start_date='2014-01-01',
        end_date='2014-12-31'
    )
    
    # Display last 10 rows
    pipeline_output.tail(10)




.. raw:: html

    <div>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th></th>
          <th>avg_close</th>
          <th>close_price</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th rowspan="10" valign="top">2014-12-31 00:00:00+00:00</th>
          <th>Equity(47430 [MBLY])</th>
          <td>42.410000</td>
          <td>41.11</td>
        </tr>
        <tr>
          <th>Equity(47578 [LTRP_A])</th>
          <td>27.300000</td>
          <td>27.15</td>
        </tr>
        <tr>
          <th>Equity(47740 [BABA])</th>
          <td>105.880000</td>
          <td>105.77</td>
        </tr>
        <tr>
          <th>Equity(47752 [CDK])</th>
          <td>40.906667</td>
          <td>40.42</td>
        </tr>
        <tr>
          <th>Equity(47764 [AVAL])</th>
          <td>10.609355</td>
          <td>10.49</td>
        </tr>
        <tr>
          <th>Equity(47777 [CFG])</th>
          <td>25.310000</td>
          <td>25.22</td>
        </tr>
        <tr>
          <th>Equity(47779 [CYBR])</th>
          <td>41.006667</td>
          <td>39.90</td>
        </tr>
        <tr>
          <th>Equity(47785 [CNXM])</th>
          <td>23.610000</td>
          <td>24.31</td>
        </tr>
        <tr>
          <th>Equity(47788 [TVPT])</th>
          <td>18.026667</td>
          <td>18.04</td>
        </tr>
        <tr>
          <th>Equity(47921 [KEYS])</th>
          <td>33.863333</td>
          <td>33.95</td>
        </tr>
      </tbody>
    </table>
    </div>



In the next lesson we will formalize the strategy our algorithm will use
to select assets to trade. Then, we will use a factor analysis tool to
evaluate the predictive power of our strategy over historical data.
