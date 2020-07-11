Sentdex
=======

In this notebook, we’ll take a look at Sentdex’s *Sentiment* dataset,
available through the `Quantopian partner
program <https://www.quantopian.com/data>`__. This dataset spans 2012
through the current day, and documents the mood of traders based on
their messages.

Notebook Contents
-----------------

There are two ways to access the data and you’ll find both of them
listed below. Just click on the section you’d like to read through.

-  Interactive overview: This is only available on Research and uses
   blaze to give you access to large amounts of data. Recommended for
   exploration and plotting.
-  Pipeline overview: Data is made available through pipeline which is
   available on both the Research & Backtesting environment. Recommended
   for custom factor development and moving back & forth between
   research/backtesting.

Free samples and limits
~~~~~~~~~~~~~~~~~~~~~~~

One key caveat: we limit the number of results returned from any given
expression to 10,000 to protect against runaway memory usage. To be
clear, you have access to all the data server side. We are limiting the
size of the responses back from Blaze.

There is a *free* version of this dataset as well as a paid one. The
free sample includes data until 2 months prior to the current date.

To access the most up-to-date values for this data set for trading a
live algorithm (as with other partner sets), you need to purchase acess
to the full set.

With preamble in place, let’s get started:

#Interactive Overview ### Accessing the data with Blaze and Interactive
on Research Partner datasets are available on Quantopian Research
through an API service known as `Blaze <http://blaze.pydata.org>`__.
Blaze provides the Quantopian user with a convenient interface to access
very large datasets, in an interactive, generic manner.

Blaze provides an important function for accessing these datasets. Some
of these sets are many millions of records. Bringing that data directly
into Quantopian Research directly just is not viable. So Blaze allows us
to provide a simple querying interface and shift the burden over to the
server side.

It is common to use Blaze to reduce your dataset in size, convert it
over to Pandas and then to use Pandas for further computation,
manipulation and visualization.

Helpful links: \* `Query building for
Blaze <http://blaze.readthedocs.io/en/latest/queries.html>`__ \*
`Pandas-to-Blaze
dictionary <http://blaze.readthedocs.io/en/latest/rosetta-pandas.html>`__
\* `SQL-to-Blaze
dictionary <http://blaze.readthedocs.io/en/latest/rosetta-sql.html>`__.

| Once you’ve limited the size of your Blaze object, you can convert it
  to a Pandas DataFrames using: > ``from odo import odo``
| > ``odo(expr, pandas.DataFrame)``

###To see how this data can be used in your algorithm, search for the
``Pipeline Overview`` section of this notebook or head straight to
Pipeline Overview

.. code:: ipython2

    # import the free sample of the dataset
    from quantopian.interactive.data.sentdex import sentiment_free as dataset
    
    # or if you want to import the full dataset, use:
    # from quantopian.interactive.data.sentdex import sentiment
    
    # import data operations
    from odo import odo
    # import other libraries we will use
    import pandas as pd
    import matplotlib.pyplot as plt

.. code:: ipython2

    # Let's use blaze to understand the data a bit using Blaze dshape()
    dataset.dshape




.. parsed-literal::

    dshape("""var * {
      symbol: string,
      sentiment_signal: float64,
      sid: int64,
      asof_date: datetime,
      timestamp: datetime
      }""")



.. code:: ipython2

    # And how many rows are there?
    # N.B. we're using a Blaze function to do this, not len()
    dataset.count()




.. raw:: html

    644635



.. code:: ipython2

    # Let's see what the data looks like. We'll grab the first three rows.
    dataset[:3]




.. raw:: html

    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>symbol</th>
          <th>sentiment_signal</th>
          <th>sid</th>
          <th>asof_date</th>
          <th>timestamp</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>AAPL</td>
          <td>6</td>
          <td>24</td>
          <td>2012-10-15</td>
          <td>2012-10-16</td>
        </tr>
        <tr>
          <th>1</th>
          <td>AAPL</td>
          <td>2</td>
          <td>24</td>
          <td>2012-10-16</td>
          <td>2012-10-17</td>
        </tr>
        <tr>
          <th>2</th>
          <td>AAPL</td>
          <td>6</td>
          <td>24</td>
          <td>2012-10-17</td>
          <td>2012-10-18</td>
        </tr>
      </tbody>
    </table>



The Sentdex Sentiment data feed is elegant and simple. Just a few
fields:

Let’s go over the columns: - **asof_date**: The date to which this data
applies. - **symbol**: stock ticker symbol of the affected company. -
**timestamp**: the datetime at which the data is available to the
Quantopian system. For historical data loaded, we have simulated a lag.
For data we have loaded since the advent of loading this data set, the
timestamp is an actual recorded value. - **sentiment_signal**: A
standalone sentiment score from -3 to 6 for stocks - **sid**: the
equity’s unique identifier. Use this instead of the symbol.

From the `Sentdex
documentation <http://sentdex.com/blog/back-testing-sentdex-sentiment-analysis-signals-for-stocks>`__:

::

   The signals currently vary from -3 to a positive 6, where -3 is as equally strongly negative of sentiment as a 6 is strongly positive sentiment.

   Sentiment signals:

   6 - Strongest positive sentiment.
   5 - Extremely strong, positive, sentiment.
   4 - Very strong, positive, sentiment.
   3 - Strong, positive sentiment.
   2 - Substantially positive sentiment.
   1 - Barely positive sentiment.
   0 - Neutral sentiment
   -1 - Sentiment trending into negatives.
   -2 - Weak negative sentiment.
   -3 - Strongest negative sentiment.

We’ve done much of the data processing for you. Fields like
``timestamp`` and ``sid`` are standardized across all our Store
Datasets, so the datasets are easy to combine. We have standardized the
``sid`` across all our equity databases.

We can select columns and rows with ease. Below, we’ll fetch all rows
for Apple (sid 24) and explore the scores a bit with a chart.

.. code:: ipython2

    # Filtering for AAPL
    aapl = dataset[dataset.sid == 24]
    aapl_df = odo(aapl.sort('asof_date'), pd.DataFrame)
    plt.plot(aapl_df.asof_date, aapl_df.sentiment_signal, marker='.', linestyle='None', color='r')
    plt.plot(aapl_df.asof_date, pd.rolling_mean(aapl_df.sentiment_signal, 30))
    plt.xlabel("As Of Date (asof_date)")
    plt.ylabel("Sentiment")
    plt.title("Sentdex Sentiment for AAPL")
    plt.legend(["Sentiment - Single Day", "30 Day Rolling Average"], loc=1)
    x1,x2,y1,y2 = plt.axis()
    plt.axis((x1,x2,-4,7.5))




.. parsed-literal::

    (734791.0, 736078.0, -4, 7.5)




.. image:: notebook_files/notebook_6_1.png


Let’s check out Comcast’s sentiment for fun

.. code:: ipython2

    comcast = dataset[dataset.sid == 1637]
    comcast_df = odo(comcast.sort('asof_date'), pd.DataFrame)
    plt.plot(comcast_df.asof_date, comcast_df.sentiment_signal, marker='.', linestyle='None', color='r')
    plt.plot(comcast_df.asof_date, pd.rolling_mean(comcast_df.sentiment_signal, 30))
    plt.xlabel("As Of Date (asof_date)")
    plt.ylabel("Sentiment")
    plt.title("Sentdex Sentiment for Comcast")
    plt.legend(["Sentiment - Single Day", "30 Day Rolling Average"], loc=1)
    x1,x2,y1,y2 = plt.axis()
    plt.axis((x1,x2,-4,7.5))




.. parsed-literal::

    (734843.0, 736039.0, -4, 7.5)




.. image:: notebook_files/notebook_8_1.png


#Pipeline Overview

Accessing the data in your algorithms & research
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The only method for accessing partner data within algorithms running on
Quantopian is via the pipeline API. Different data sets work differently
but in the case of this PsychSignal data, you can add this data to your
pipeline as follows:

Import the data set >
``from quantopian.pipeline.data.sentdex import sentiment``

Then in intialize() you could do something simple like adding the raw
value of one of the fields to your pipeline: >
``pipe.add(sentiment.sentiment_signal.latest, 'sentdex_sentiment')``

.. code:: ipython2

    # Import necessary Pipeline modules
    from quantopian.pipeline import Pipeline
    from quantopian.research import run_pipeline
    from quantopian.pipeline.factors import AverageDollarVolume

.. code:: ipython2

    # For use in your algorithms
    # Using the full paid dataset in your pipeline algo
    # from quantopian.pipeline.data.sentdex import sentiment
    
    # Using the free sample in your pipeline algo
    from quantopian.pipeline.data.sentdex import sentiment_free

Now that we’ve imported the data, let’s take a look at which fields are
available for each dataset.

You’ll find the dataset, the available fields, and the datatypes for
each of those fields.

.. code:: ipython2

    print "Here are the list of available fields per dataset:"
    print "---------------------------------------------------\n"
    
    def _print_fields(dataset):
        print "Dataset: %s\n" % dataset.__name__
        print "Fields:"
        for field in list(dataset.columns):
            print "%s - %s" % (field.name, field.dtype)
        print "\n"
    
    for data in (sentiment_free,):
        _print_fields(data)
    
    
    print "---------------------------------------------------\n"


.. parsed-literal::

    Here are the list of available fields per dataset:
    ---------------------------------------------------
    
    Dataset: sentiment_free
    
    Fields:
    sentiment_signal - float64
    
    
    ---------------------------------------------------
    


Now that we know what fields we have access to, let’s see what this data
looks like when we run it through Pipeline.

This is constructed the same way as you would in the backtester. For
more information on using Pipeline in Research view this thread:
https://www.quantopian.com/posts/pipeline-in-research-build-test-and-visualize-your-factors-and-filters

.. code:: ipython2

    # Let's see what this data looks like when we run it through Pipeline
    # This is constructed the same way as you would in the backtester. For more information
    # on using Pipeline in Research view this thread:
    # https://www.quantopian.com/posts/pipeline-in-research-build-test-and-visualize-your-factors-and-filters
    pipe = Pipeline()
           
    pipe.add(sentiment_free.sentiment_signal.latest, 'sentiment_signal')

.. code:: ipython2

    # Setting some basic liquidity strings (just for good habit)
    dollar_volume = AverageDollarVolume(window_length=20)
    top_1000_most_liquid = dollar_volume.rank(ascending=False) < 1000
    
    pipe.set_screen(top_1000_most_liquid & sentiment_free.sentiment_signal.latest.notnan())

.. code:: ipython2

    # The show_graph() method of pipeline objects produces a graph to show how it is being calculated.
    pipe.show_graph(format='png')




.. image:: notebook_files/notebook_17_0.png



.. code:: ipython2

    # run_pipeline will show the output of your pipeline
    pipe_output = run_pipeline(pipe, start_date='2013-11-01', end_date='2013-11-25')
    pipe_output




.. raw:: html

    <div style="max-height:1000px;max-width:1500px;overflow:auto;">
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th></th>
          <th>sentiment_signal</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th rowspan="30" valign="top">2013-11-01 00:00:00+00:00</th>
          <th>Equity(2 [AA])</th>
          <td>-1</td>
        </tr>
        <tr>
          <th>Equity(24 [AAPL])</th>
          <td>3</td>
        </tr>
        <tr>
          <th>Equity(62 [ABT])</th>
          <td>1</td>
        </tr>
        <tr>
          <th>Equity(67 [ADSK])</th>
          <td>2</td>
        </tr>
        <tr>
          <th>Equity(76 [TAP])</th>
          <td>5</td>
        </tr>
        <tr>
          <th>Equity(88 [ACI])</th>
          <td>-3</td>
        </tr>
        <tr>
          <th>Equity(114 [ADBE])</th>
          <td>-3</td>
        </tr>
        <tr>
          <th>Equity(122 [ADI])</th>
          <td>2</td>
        </tr>
        <tr>
          <th>Equity(128 [ADM])</th>
          <td>1</td>
        </tr>
        <tr>
          <th>Equity(161 [AEP])</th>
          <td>-1</td>
        </tr>
        <tr>
          <th>Equity(166 [AES])</th>
          <td>5</td>
        </tr>
        <tr>
          <th>Equity(168 [AET])</th>
          <td>6</td>
        </tr>
        <tr>
          <th>Equity(185 [AFL])</th>
          <td>5</td>
        </tr>
        <tr>
          <th>Equity(216 [HES])</th>
          <td>4</td>
        </tr>
        <tr>
          <th>Equity(239 [AIG])</th>
          <td>-1</td>
        </tr>
        <tr>
          <th>Equity(328 [ALTR])</th>
          <td>2</td>
        </tr>
        <tr>
          <th>Equity(337 [AMAT])</th>
          <td>-1</td>
        </tr>
        <tr>
          <th>Equity(338 [BEAM])</th>
          <td>3</td>
        </tr>
        <tr>
          <th>Equity(351 [AMD])</th>
          <td>3</td>
        </tr>
        <tr>
          <th>Equity(357 [TWX])</th>
          <td>-1</td>
        </tr>
        <tr>
          <th>Equity(368 [AMGN])</th>
          <td>3</td>
        </tr>
        <tr>
          <th>Equity(410 [AN])</th>
          <td>1</td>
        </tr>
        <tr>
          <th>Equity(438 [AON])</th>
          <td>6</td>
        </tr>
        <tr>
          <th>Equity(448 [APA])</th>
          <td>-1</td>
        </tr>
        <tr>
          <th>Equity(455 [APC])</th>
          <td>-1</td>
        </tr>
        <tr>
          <th>Equity(460 [APD])</th>
          <td>6</td>
        </tr>
        <tr>
          <th>Equity(465 [APH])</th>
          <td>5</td>
        </tr>
        <tr>
          <th>Equity(510 [ARG])</th>
          <td>6</td>
        </tr>
        <tr>
          <th>Equity(630 [ADP])</th>
          <td>-3</td>
        </tr>
        <tr>
          <th>Equity(660 [AVP])</th>
          <td>1</td>
        </tr>
        <tr>
          <th>...</th>
          <th>...</th>
          <td>...</td>
        </tr>
        <tr>
          <th rowspan="30" valign="top">2013-11-25 00:00:00+00:00</th>
          <th>Equity(36346 [LO])</th>
          <td>2</td>
        </tr>
        <tr>
          <th>Equity(36372 [SNI])</th>
          <td>3</td>
        </tr>
        <tr>
          <th>Equity(36930 [DISC_A])</th>
          <td>2</td>
        </tr>
        <tr>
          <th>Equity(38084 [MJN])</th>
          <td>-1</td>
        </tr>
        <tr>
          <th>Equity(38691 [CFN])</th>
          <td>6</td>
        </tr>
        <tr>
          <th>Equity(38936 [DG])</th>
          <td>6</td>
        </tr>
        <tr>
          <th>Equity(39546 [LYB])</th>
          <td>-1</td>
        </tr>
        <tr>
          <th>Equity(39778 [QEP])</th>
          <td>4</td>
        </tr>
        <tr>
          <th>Equity(39840 [TSLA])</th>
          <td>4</td>
        </tr>
        <tr>
          <th>Equity(40430 [GM])</th>
          <td>2</td>
        </tr>
        <tr>
          <th>Equity(40852 [KMI])</th>
          <td>-1</td>
        </tr>
        <tr>
          <th>Equity(41451 [LNKD])</th>
          <td>-1</td>
        </tr>
        <tr>
          <th>Equity(41462 [MOS])</th>
          <td>-1</td>
        </tr>
        <tr>
          <th>Equity(41579 [P])</th>
          <td>2</td>
        </tr>
        <tr>
          <th>Equity(41636 [MPC])</th>
          <td>-1</td>
        </tr>
        <tr>
          <th>Equity(42023 [XYL])</th>
          <td>-1</td>
        </tr>
        <tr>
          <th>Equity(42118 [GRPN])</th>
          <td>2</td>
        </tr>
        <tr>
          <th>Equity(42173 [DLPH])</th>
          <td>3</td>
        </tr>
        <tr>
          <th>Equity(42230 [TRIP])</th>
          <td>6</td>
        </tr>
        <tr>
          <th>Equity(42251 [WPX])</th>
          <td>-1</td>
        </tr>
        <tr>
          <th>Equity(42270 [KORS])</th>
          <td>2</td>
        </tr>
        <tr>
          <th>Equity(42277 [ZNGA])</th>
          <td>6</td>
        </tr>
        <tr>
          <th>Equity(42788 [PSX])</th>
          <td>5</td>
        </tr>
        <tr>
          <th>Equity(42950 [FB])</th>
          <td>5</td>
        </tr>
        <tr>
          <th>Equity(43399 [ADT])</th>
          <td>6</td>
        </tr>
        <tr>
          <th>Equity(43405 [KRFT])</th>
          <td>3</td>
        </tr>
        <tr>
          <th>Equity(43694 [ABBV])</th>
          <td>3</td>
        </tr>
        <tr>
          <th>Equity(43721 [SCTY])</th>
          <td>2</td>
        </tr>
        <tr>
          <th>Equity(44931 [NWSA])</th>
          <td>2</td>
        </tr>
        <tr>
          <th>Equity(45815 [TWTR])</th>
          <td>-1</td>
        </tr>
      </tbody>
    </table>
    <p>8214 rows × 1 columns</p>
    </div>



Taking what we’ve seen from above, let’s see how we’d move that into the
backtester.

.. code:: ipython2

    # This section is only importable in the backtester
    from quantopian.algorithm import attach_pipeline, pipeline_output
    
    # General pipeline imports
    from quantopian.pipeline import Pipeline
    from quantopian.pipeline.factors import AverageDollarVolume
    
    # Import the datasets available
    # For use in your algorithms
    # Using the full paid dataset in your pipeline algo
    # from quantopian.pipeline.data.sentdex import sentiment
    
    # Using the free sample in your pipeline algo
    from quantopian.pipeline.data.sentdex import sentiment_free
    
    def make_pipeline():
        # Create our pipeline
        pipe = Pipeline()
        
        # Screen out penny stocks and low liquidity securities.
        dollar_volume = AverageDollarVolume(window_length=20)
        is_liquid = dollar_volume.rank(ascending=False) < 1000
        
        # Create the mask that we will use for our percentile methods.
        base_universe = (is_liquid)
    
        # Add pipeline factors
        pipe.add(sentiment_free.sentiment_signal.latest, 'sentiment_signal')
    
        # Set our pipeline screens
        pipe.set_screen(is_liquid)
        return pipe
    
    def initialize(context):
        attach_pipeline(make_pipeline(), "pipeline")
        
    def before_trading_start(context, data):
        results = pipeline_output('pipeline')

Now you can take that and begin to use it as a building block for your
algorithms, for more examples on how to do that you can visit our data
pipeline factor library
