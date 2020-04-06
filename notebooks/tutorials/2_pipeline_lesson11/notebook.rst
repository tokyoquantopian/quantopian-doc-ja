.. code:: ipython2

    from quantopian.pipeline import Pipeline
    from quantopian.research import run_pipeline
    from quantopian.pipeline.data.builtin import USEquityPricing
    from quantopian.pipeline.factors import SimpleMovingAverage, AverageDollarVolume

##Putting It All Together Now that we’ve covered the basic components of
the Pipeline API, let’s construct a pipeline that we might want to use
in an algorithm.

To start, let’s first create a filter to narrow down the types of
securities coming out of our pipeline. In this example, we will create a
filter to select for securities that meet all of the following criteria:
- Is a primary share - Is listed as a common stock - Is not a
`depositary
receipt <http://www.investopedia.com/terms/d/depositaryreceipt.asp>`__
(ADR/GDR) - Is not trading
`over-the-counter <http://www.investopedia.com/terms/o/otc.asp>`__ (OTC)
- Is not `when-issued <http://www.investopedia.com/terms/w/wi.asp>`__
(WI) - Doesn’t have a name indicating it’s a `limited
partnership <http://www.investopedia.com/terms/l/limitedpartnership.asp>`__
(LP) - Doesn’t have a company reference entry indicating it’s a LP - Is
not an `ETF <http://www.investopedia.com/terms/e/etf.asp>`__ (has
Morningstar fundamental data)

####Why These Criteria? Selecting for primary shares and common stock
helps us to select only a single security for each company. In general,
primary shares are a good representative asset of a company so we will
select for these in our pipeline.

ADRs and GDRs are issuances in the US equity market for stocks that
trade on other exchanges. Frequently, there is inherent risk associated
with depositary receipts due to currency fluctuations so we exclude them
from our pipeline.

OTC, WI, and LP securities are not tradeable with most brokers. As a
result, we exclude them from our pipeline.

###Creating Our Pipeline Let’s create a filter for each criterion and
combine them together to create a ``tradeable_stocks`` filter. First, we
need to import the Morningstar ``DataSet`` as well as the
``IsPrimaryShare`` builtin filter.

.. code:: ipython2

    from quantopian.pipeline.data import Fundamentals
    from quantopian.pipeline.filters.fundamentals import IsPrimaryShare

Now we can define our filters:

.. code:: ipython2

    # Filter for primary share equities. IsPrimaryShare is a built-in filter.
    primary_share = IsPrimaryShare()
    
    # Equities listed as common stock (as opposed to, say, preferred stock).
    # 'ST00000001' indicates common stock.
    common_stock = Fundamentals.security_type.latest.eq('ST00000001')
    
    # Non-depositary receipts. Recall that the ~ operator inverts filters,
    # turning Trues into Falses and vice versa
    not_depositary = ~Fundamentals.is_depositary_receipt.latest
    
    # Equities not trading over-the-counter.
    not_otc = ~Fundamentals.exchange_id.latest.startswith('OTC')
    
    # Not when-issued equities.
    not_wi = ~Fundamentals.symbol.latest.endswith('.WI')
    
    # Equities without LP in their name, .matches does a match using a regular
    # expression
    not_lp_name = ~Fundamentals.standard_name.latest.matches('.* L[. ]?P.?$')
    
    # Equities with a null value in the limited_partnership Morningstar
    # fundamental field.
    not_lp_balance_sheet = Fundamentals.limited_partnership.latest.isnull()
    
    # Equities whose most recent Morningstar market cap is not null have
    # fundamental data and therefore are not ETFs.
    have_market_cap = Fundamentals.market_cap.latest.notnull()
    
    # Filter for stocks that pass all of our previous filters.
    tradeable_stocks = (
        primary_share
        & common_stock
        & not_depositary
        & not_otc
        & not_wi
        & not_lp_name
        & not_lp_balance_sheet
        & have_market_cap
    )

Note that when defining our filters, we used several ``Classifier``
methods that we haven’t yet seen including ``notnull``, ``startswith``,
``endswith``, and ``matches``. Documentation on these methods is
available
`here <https://www.quantopian.com/help#quantopian_pipeline_classifiers_Classifier>`__.

Next, let’s create a filter for the top 30% of tradeable stocks by
20-day average dollar volume. We’ll call this our ``base_universe``.

.. code:: ipython2

    base_universe = AverageDollarVolume(window_length=20, mask=tradeable_stocks).percentile_between(70, 100)

####Built-in Base Universe

We have just defined our own base universe to select ‘tradeable’
securities with high dollar volume. However, Quantopian has several
built-in filters that do something similar, the best and newest of which
is the
`QTradableStocksUS <https://www.quantopian.com/help#quantopian_pipeline_filters_QTradableStocksUS>`__.
The QTradableStocksUS is a built-in pipeline filter that selects a daily
universe of stocks that are filtered in three passes and adhere to a set
of criteria to yield the most liquid universe possible without any size
constraints. The QTradableStocksUS therefore has no size cutoff unlike
its predecessors, the
`Q500US <https://www.quantopian.com/help#quantopian_pipeline_filters_Q500US>`__
and the
`Q1500US <https://www.quantopian.com/help#quantopian_pipeline_filters_Q1500US>`__.
More detail on the selection criteria of the QTradableStocksUS can be
found
`here <https://www.quantopian.com/posts/working-on-our-best-universe-yet-qtradablestocksus>`__.

To simplify our pipeline, let’s replace what we’ve already written for
our ``base_universe`` with the ``QTradableStocksUS`` built-in filter.
First, we need to import it.

.. code:: ipython2

    from quantopian.pipeline.filters import QTradableStocksUS

Then, let’s set our base_universe to the ``QTradableStocksUS``.

.. code:: ipython2

    base_universe = QTradableStocksUS()

Now that we have a filter ``base_universe`` that we can use to select a
subset of securities, let’s focus on creating factors for this subset.
For this example, let’s create a pipeline for a mean reversion strategy.
In this strategy, we’ll look at the 10-day and 30-day moving averages
(close price). Let’s plan to open equally weighted long positions in the
75 securities with the least (most negative) percent difference and
equally weighted short positions in the 75 with the greatest percent
difference. To do this, let’s create two moving average factors using
our ``base_universe`` filter as a mask. Then let’s combine them into a
factor computing the percent difference.

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
        
        # Base universe filter.
        base_universe = QTradableStocksUS()
        
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
                'shorts': shorts
            },
            screen=securities_to_trade
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
          <th>Equity(39 [DDC])</th>
          <td>False</td>
          <td>True</td>
        </tr>
        <tr>
          <th>Equity(351 [AMD])</th>
          <td>True</td>
          <td>False</td>
        </tr>
        <tr>
          <th>Equity(371 [TVTY])</th>
          <td>True</td>
          <td>False</td>
        </tr>
        <tr>
          <th>Equity(474 [APOG])</th>
          <td>False</td>
          <td>True</td>
        </tr>
        <tr>
          <th>Equity(523 [AAN])</th>
          <td>False</td>
          <td>True</td>
        </tr>
      </tbody>
    </table>
    </div>



In the next lesson, we’ll add this pipeline to an algorithm.
