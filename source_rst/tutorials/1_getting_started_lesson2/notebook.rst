Data Exploration
----------------

Research provides utility functions to query pricing, volume, and
returns data for 8000+ US equities, from 2002 up to the most recently
completed trading day. These functions take an asset (or list of assets)
along with a start and end date, and return a pandas
`Series <http://pandas.pydata.org/pandas-docs/version/0.18/generated/pandas.Series.html>`__
(or
`DataFrame <http://pandas.pydata.org/pandas-docs/version/0.18/generated/pandas.DataFrame.html>`__)
indexed by date.

Let’s define the period of time we want to explore and use the
``returns`` function to query data for AAPL:

.. code:: ipython3

    # Research environment functions
    from quantopian.research import returns, symbols
    
    # Select a time range to inspect
    period_start = '2014-01-01'
    period_end = '2017-1-1'
    
    # Query returns data for AAPL
    # over the selected time range
    aapl_returns = returns(
        assets=symbols('AAPL'),
        start=period_start,
        end=period_end,
    )
    
    # Display first 10 rows
    aapl_returns.head(10)




.. parsed-literal::

    2014-01-02 00:00:00+00:00   -0.014142
    2014-01-03 00:00:00+00:00   -0.022028
    2014-01-06 00:00:00+00:00    0.005370
    2014-01-07 00:00:00+00:00   -0.007199
    2014-01-08 00:00:00+00:00    0.006412
    2014-01-09 00:00:00+00:00   -0.012865
    2014-01-10 00:00:00+00:00   -0.006676
    2014-01-13 00:00:00+00:00    0.005047
    2014-01-14 00:00:00+00:00    0.020130
    2014-01-15 00:00:00+00:00    0.020073
    Freq: C, Name: Equity(24 [AAPL]), dtype: float64



Alternative Data
~~~~~~~~~~~~~~~~

In addition to pricing and volume data, Quantopian integrates a number
of alternative datasets that include corporate fundamentals, stock
sentiment analysis, and macroeconomic indicators, to name a few. You can
find the complete list of 50+ datasets on Quantopian’s `data
page <https://www.quantopian.com/data>`__.

Our goal in this tutorial will be to build an algorithm that selects and
trades assets based on sentiment data, so let’s take a look at Sentdex
News Sentiment
`sentiment <https://www.quantopian.com/docs/data-reference/sentdex>`__
dataset. The sentiment scores are generated from four simple moving
average (SMA) factors over the last 100, 250, 500, and 5000 news events
for each stock. News events are pulled from over 20 sources including
The Wall Street Journal, CNBC, Forbes, Business Insider, and Yahoo
Finance.

We can start by inspecting the ``sentiment_signal`` from the
``sentiment`` dataset. We will query the data using Quantopian’s
Pipeline API, which is a powerful tool you will use over and over again
to access and analyze data in Research. You will learn a lot more about
the Pipeline API in the next lesson and a `later
tutorial <https://www.quantopian.com/tutorials/pipeline>`__. For now all
you need to know is that the following code uses a data pipeline to
query ``sentiment`` and returns data, and plots the results for AAPL:

.. code:: ipython3

    # Pipeline imports
    from quantopian.research import run_pipeline
    from quantopian.pipeline import Pipeline
    from quantopian.pipeline.domain import US_EQUITIES
    from quantopian.pipeline.factors import Returns
    from quantopian.pipeline.data.sentdex import sentiment
    
    # Pipeline definition
    def make_pipeline():
    
        returns = Returns(window_length=2)
        sentiment_score = sentiment.sentiment_signal.latest
    
        return Pipeline(
            columns={
                'daily_returns': returns,
                'sentiment': sentiment_score,
            },
            domain=US_EQUITIES,
        )
    
    # Pipeline execution
    data_output = run_pipeline(
        make_pipeline(),
        start_date=period_start,
        end_date=period_end
    )
    
    # Filter results for AAPL
    aapl_output = data_output.xs(
        symbols('AAPL'),
        level=1
    )
    
    # Plot results for AAPL
    aapl_output.plot(subplots=True);



.. parsed-literal::

    



.. raw:: html

    <b>Pipeline Execution Time:</b> 2 Minutes, 52.29 Seconds



.. image:: notebook_files/notebook_5_2.png


When exploring a dataset, try to look for patterns that might serve as
the basis for a trading strategy. For example, the above plot shows some
matching spikes between daily returns and ``sentiment``, and in some
cases, the direction of the spikes in returns match the direction of
AAPL’s sentiment signal. This looks interesting enough that we should
conduct more rigorous statistical tests to confirm our hypotheses.

In the next lesson we will cover the Pipeline API in more depth.
