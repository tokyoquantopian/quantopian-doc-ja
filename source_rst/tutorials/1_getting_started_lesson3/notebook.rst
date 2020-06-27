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

.. code:: ipython3

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

.. code:: ipython3

    # Import Pipeline class and USEquityPricing dataset
    from quantopian.pipeline import Pipeline
    from quantopian.pipeline.data import EquityPricing
    from quantopian.pipeline.domain import US_EQUITIES
    
    def make_pipeline():
        # Get latest closing price
        close_price = EquityPricing.close.latest
    
        # Return Pipeline containing latest closing price
        return Pipeline(
            columns={
                'close_price': close_price,
            },
            domain=US_EQUITIES,
        )

The Pipeline API also provides a number of built-in calculations, some
of which are computed over trailing windows of data. For example, the
following code imports the sentdex ``sentiment`` dataset and defines an
output as the 3 day moving average of its ``sentiment_signal`` column:

.. code:: ipython3

    # Import Pipeline class and datasets
    from quantopian.pipeline import Pipeline
    from quantopian.pipeline.data import EquityPricing
    from quantopian.pipeline.domain import US_EQUITIES
    from quantopian.pipeline.data.sentdex import sentiment
    
    # Import built-in moving average calculation
    from quantopian.pipeline.factors import SimpleMovingAverage
    
    
    def make_pipeline():
        # Get latest closing price
        close_price = EquityPricing.close.latest
    
        # Calculate 3 day average the sentiment signal
        sentiment_score = SimpleMovingAverage(
            inputs=[sentiment.sentiment_signal],
            window_length=3,
        )
    
        # Return Pipeline containing close_price
        # and sentiment_score
        return Pipeline(
            columns={
                'close_price': close_price,
                'sentiment_score': sentiment_score,
            }
        )

Universe Selection
~~~~~~~~~~~~~~~~~~

An important part of developing a strategy is defining the set of assets
that we want to consider trading in our portfolio. We usually refer to
this set of assets as our trading universe.

A trading universe should be as large as possible, while also excluding
assets that aren’t appropriate for our portfolio. For example, we want
to exclude stocks that are illiquid or difficult to trade. Quantopian’s
``QTradableStocksUS`` universe offers this characteristic. We can set
``QTradableStocksUS`` as our trading universe using the screen parameter
of our pipeline constructor. We can also screen for only those stocks
that have a valid, non nan, sentiment_mean:

.. code:: ipython3

    # Import Pipeline class and datasets
    from quantopian.pipeline import Pipeline
    from quantopian.pipeline.data import EquityPricing
    from quantopian.pipeline.domain import US_EQUITIES
    from quantopian.pipeline.data.sentdex import sentiment
    
    # Import built-in moving average calculation
    from quantopian.pipeline.factors import SimpleMovingAverage
    
    # Import built-in trading universe
    from quantopian.pipeline.filters import QTradableStocksUS
    
    
    def make_pipeline():
        # Create a reference to our trading universe
        base_universe = QTradableStocksUS()
    
        # Get latest closing price
        close_price = EquityPricing.close.latest
    
        # Calculate 3 day average of sentiment scores
        sentiment_score = SimpleMovingAverage(
            inputs=[sentiment.sentiment_signal],
            window_length=3,
        )
    
        # Return Pipeline containing close_price and
        # sentiment_score that has our trading universe as screen
        return Pipeline(
            columns={
                'close_price': close_price,
                'sentiment_score': sentiment_score,
            },
            screen=base_universe & sentiment_score.notnan(),
            domain=US_EQUITIES,
        )

Now that our pipeline definition is complete, we can execute it over a
specific period of time using ``run_pipeline``. The output will be a
pandas DataFrame indexed by date and asset, with columns corresponding
to the outputs we added to our pipeline definition:

.. code:: ipython3

    # Import run_pipeline method
    from quantopian.research import run_pipeline
    
    # Execute pipeline created by make_pipeline
    # between start_date and end_date
    pipeline_output = run_pipeline(
        make_pipeline(),
        start_date='2014-01-01',
        end_date='2017-1-1'
    )
    
    # Display last 10 rows
    pipeline_output.head(10)



.. parsed-literal::

    



.. raw:: html

    <b>Pipeline Execution Time:</b> 2 Minutes, 36.61 Seconds




.. raw:: html

    <div>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th></th>
          <th>close_price</th>
          <th>sentiment_score</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th rowspan="10" valign="top">2014-01-02 00:00:00+00:00</th>
          <th>Equity(2 [HWM])</th>
          <td>10.63</td>
          <td>6.000000</td>
        </tr>
        <tr>
          <th>Equity(24 [AAPL])</th>
          <td>561.16</td>
          <td>-1.666667</td>
        </tr>
        <tr>
          <th>Equity(62 [ABT])</th>
          <td>38.34</td>
          <td>4.000000</td>
        </tr>
        <tr>
          <th>Equity(67 [ADSK])</th>
          <td>50.32</td>
          <td>6.000000</td>
        </tr>
        <tr>
          <th>Equity(76 [TAP])</th>
          <td>56.15</td>
          <td>6.000000</td>
        </tr>
        <tr>
          <th>Equity(88 [ACI])</th>
          <td>4.44</td>
          <td>6.000000</td>
        </tr>
        <tr>
          <th>Equity(114 [ADBE])</th>
          <td>59.87</td>
          <td>5.000000</td>
        </tr>
        <tr>
          <th>Equity(122 [ADI])</th>
          <td>50.93</td>
          <td>6.000000</td>
        </tr>
        <tr>
          <th>Equity(128 [ADM])</th>
          <td>43.41</td>
          <td>4.000000</td>
        </tr>
        <tr>
          <th>Equity(161 [AEP])</th>
          <td>46.74</td>
          <td>6.000000</td>
        </tr>
      </tbody>
    </table>
    </div>



In the next lesson we will formalize the strategy our algorithm will use
to select assets to trade. Then, we will use a factor analysis tool to
evaluate the predictive power of our strategy over historical data.
