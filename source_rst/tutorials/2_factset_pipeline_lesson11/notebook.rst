.. code:: ipython2

    from quantopian.pipeline import Pipeline
    from quantopian.pipeline.data.builtin import USEquityPricing
    from quantopian.pipeline.factors import AverageDollarVolume, SimpleMovingAverage
    from quantopian.research import run_pipeline

##Putting It All Together Now that we’ve covered the basic components of
the Pipeline API, let’s construct a pipeline that we might want to use
in an algorithm.

To start, let’s first create a filter to narrow down the types of
securities coming out of our pipeline. In this example, we will create a
filter to select assets that we can reliably trade, by excluding
securities such as microcaps and penny stocks. We’ll select for
securities that meet the following criteria: - Has a sufficiently high
market cap. - Trades at a sufficiently high volume. - Has a daily close
price above $5.00.

###Creating Our Pipeline Let’s create a filter for each criterion and
combine them together to create a ``tradeable_stocks`` filter. First we
need to import the ``USEquityPricing`` and FactSet ``Fundamentals`` data
sets, as well as the ``AverageDollarVolume`` factor we’ll be using.

.. code:: ipython2

    from quantopian.pipeline.data.builtin import USEquityPricing
    from quantopian.pipeline.data.factset import Fundamentals
    from quantopian.pipeline.factors import AverageDollarVolume

Now we can define our filters:

.. code:: ipython2

    # Filter for high market cap.
    market_cap = Fundamentals.mkt_val.latest
    market_cap_filter = market_cap.notnull() & (market_cap > 500000000)
    
    # Equities with a high dollar volume.
    high_dollar_volume = AverageDollarVolume(window_length=200) > 2500000
    
    # Equities with a close price above $5.00.
    price_filter = USEquityPricing.close.latest > 5
    
    # Filter for stocks that pass all of our previous filters.
    base_universe = market_cap_filter & high_dollar_volume & price_filter

Note that when defining our filters, we used several ``Classifier``
methods that we haven’t yet seen including ``notnull``, ``startswith``,
``endswith``, and ``matches``. Documentation on these methods is
available
`here <https://www.quantopian.com/help#quantopian_pipeline_classifiers_Classifier>`__.

Note that when defining our filters we used ``notnull``, a
``Classifier`` method that we haven’t seen before. Documentation on this
and other methods is available
`here <https://www.quantopian.com/help#quantopian_pipeline_classifiers_Classifier>`__.

###Mean Reversion Factors Now that we have a filter ``base_universe``
that we can use to select a subset of securities, let’s focus on
creating factors for this subset. For this example, let’s create a
pipeline for a mean reversion strategy. In this strategy, we’ll look at
the 10-day and 30-day moving averages (close price). Let’s plan to open
equally weighted long positions in the 75 securities with the least
(most negative) percent difference and equally weighted short positions
in the 75 with the greatest percent difference. To do this, let’s create
two moving average factors using our ``base_universe`` filter as a mask.
Then let’s combine them into a factor computing the percent difference.

.. code:: ipython2

    # 10-day close price average.
    mean_10 = SimpleMovingAverage(inputs=[USEquityPricing.close], window_length=10, mask=base_universe)
    
    # 30-day close price average.
    mean_30 = SimpleMovingAverage(inputs=[USEquityPricing.close], window_length=30, mask=base_universe)
    
    percent_difference = (mean_10 - mean_30) / mean_30

Next, let’s create filters for the top 75 and bottom 75 equities by
``percent_difference``.

.. code:: ipython2

    # Create a filter to select securities to short.
    shorts = percent_difference.top(75)
    
    # Create a filter to select securities to long.
    longs = percent_difference.bottom(75)

Let’s then combine ``shorts`` and ``longs`` to create a new filter that
we can use as the screen of our pipeline:

.. code:: ipython2

    securities_to_trade = (shorts | longs)

Since our earlier filters were used as masks as we built up to this
final filter, when we use ``securities_to_trade`` as a screen, the
output securities will meet the criteria outlined at the beginning of
the lesson (primary shares, non-ETFs, etc.). They will also have high
dollar volume.

Finally, let’s instantiate our pipeline. Since we are planning on
opening equally weighted long and short positions later, the only
information that we actually need from our pipeline is which securities
we want to trade (the pipeline index) and whether or not to open a long
or a short position. Let’s add our ``longs`` and ``shorts`` filters to
our pipeline and set our screen to be ``securities_to_trade``.

.. code:: ipython2

    def make_pipeline():
        
        # 10-day close price average.
        mean_10 = SimpleMovingAverage(inputs=[USEquityPricing.close], window_length=10, mask=base_universe)
    
        # 30-day close price average.
        mean_30 = SimpleMovingAverage(inputs=[USEquityPricing.close], window_length=30, mask=base_universe)
    
        # Percent difference factor.
        percent_difference = (mean_10 - mean_30) / mean_30
        
        # Create a filter to select securities to short.
        shorts = percent_difference.top(75)
    
        # Create a filter to select securities to long.
        longs = percent_difference.bottom(75)
        
        # Filter for the securities that we want to trade.
        securities_to_trade = (shorts | longs)
        
        return Pipeline(
            columns={
                'longs': longs,
                'shorts': shorts,
            },
            screen=securities_to_trade,
        )

Running this pipeline will result in a DataFrame containing 2 columns.
Each day, the columns will contain boolean values that we can use to
decide whether we want to open a long or short position in each
security.

.. code:: ipython2

    result = run_pipeline(make_pipeline(), '2015-05-05', '2015-05-05')
    result.head()




.. raw:: html

    <div>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th></th>
          <th>longs</th>
          <th>shorts</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th rowspan="5" valign="top">2015-05-05 00:00:00+00:00</th>
          <th>Equity(371 [TVTY])</th>
          <td>True</td>
          <td>False</td>
        </tr>
        <tr>
          <th>Equity(523 [AAN])</th>
          <td>False</td>
          <td>True</td>
        </tr>
        <tr>
          <th>Equity(1068 [BPT])</th>
          <td>False</td>
          <td>True</td>
        </tr>
        <tr>
          <th>Equity(1244 [CAMP])</th>
          <td>False</td>
          <td>True</td>
        </tr>
        <tr>
          <th>Equity(1595 [CLF])</th>
          <td>False</td>
          <td>True</td>
        </tr>
      </tbody>
    </table>
    </div>



In the next lesson, we’ll add this pipeline to an algorithm.
