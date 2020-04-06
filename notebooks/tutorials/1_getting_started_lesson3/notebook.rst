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
following code imports Psychsignal’s ``stocktwits`` dataset and defines
an output as the 3 day moving average of its ``bull_minus_bear`` column:

.. code:: ipython2

    # Import Pipeline class and datasets
    from quantopian.pipeline import Pipeline
    from quantopian.pipeline.data import USEquityPricing
    from quantopian.pipeline.data.psychsignal import stocktwits
    
    # Import built-in moving average calculation
    from quantopian.pipeline.factors import SimpleMovingAverage
    
    
    def make_pipeline():
        # Get latest closing price
        close_price = USEquityPricing.close.latest
    
        # Calculate 3 day average of bull_minus_bear scores
        sentiment_score = SimpleMovingAverage(
            inputs=[stocktwits.bull_minus_bear],
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
of our pipeline constructor:

.. code:: ipython2

    # Import Pipeline class and datasets
    from quantopian.pipeline import Pipeline
    from quantopian.pipeline.data import USEquityPricing
    from quantopian.pipeline.data.psychsignal import stocktwits
    
    # Import built-in moving average calculation
    from quantopian.pipeline.factors import SimpleMovingAverage
    
    # Import built-in trading universe
    from quantopian.pipeline.filters import QTradableStocksUS
    
    
    def make_pipeline():
        # Create a reference to our trading universe
        base_universe = QTradableStocksUS()
    
        # Get latest closing price
        close_price = USEquityPricing.close.latest
    
        # Calculate 3 day average of bull_minus_bear scores
        sentiment_score = SimpleMovingAverage(
            inputs=[stocktwits.bull_minus_bear],
            window_length=3,
        )
    
        # Return Pipeline containing close_price and
        # sentiment_score that has our trading universe as screen
        return Pipeline(
            columns={
                'close_price': close_price,
                'sentiment_score': sentiment_score,
            },
            screen=base_universe
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
        start_date='2013-01-01',
        end_date='2013-12-31'
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
          <th>close_price</th>
          <th>sentiment_score</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th rowspan="10" valign="top">2013-12-31 00:00:00+00:00</th>
          <th>Equity(43721 [SCTY])</th>
          <td>57.32</td>
          <td>-0.176667</td>
        </tr>
        <tr>
          <th>Equity(43919 [LMCA])</th>
          <td>146.22</td>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(43981 [NCLH])</th>
          <td>35.25</td>
          <td>-0.700000</td>
        </tr>
        <tr>
          <th>Equity(44053 [TPH])</th>
          <td>19.33</td>
          <td>0.333333</td>
        </tr>
        <tr>
          <th>Equity(44060 [ZTS])</th>
          <td>32.68</td>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(44089 [BCC])</th>
          <td>29.66</td>
          <td>1.000000</td>
        </tr>
        <tr>
          <th>Equity(44102 [XONE])</th>
          <td>60.50</td>
          <td>0.396667</td>
        </tr>
        <tr>
          <th>Equity(44158 [XOOM])</th>
          <td>27.31</td>
          <td>-0.160000</td>
        </tr>
        <tr>
          <th>Equity(44249 [APAM])</th>
          <td>64.53</td>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(44270 [SSNI])</th>
          <td>21.05</td>
          <td>0.423333</td>
        </tr>
      </tbody>
    </table>
    </div>



In the next lesson we will formalize the strategy our algorithm will use
to select assets to trade. Then, we will use a factor analysis tool to
evaluate the predictive power of our strategy over historical data.
