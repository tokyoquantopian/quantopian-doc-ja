Quandl: S&P 500 Volatility Index (VIX)
======================================

In this notebook, we’ll take a look at data set , available on
`Quantopian <https://www.quantopian.com/data>`__. This dataset spans
from January, 1990 through the current day. It contains the value for
the index VIX, a measure of volatility in the S&P 500. We access this
data via the API provided by `Quandl <https://www.quandl.com>`__. `More
details <https://www.quandl.com/data/YAHOO/INDEX_VIX-VIX-S-P-500-Volatility-Index>`__
on this dataset can be found on Quandl’s website.

To be clear, this is a single value for VIX each day.

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

Limits
~~~~~~

One key caveat: we limit the number of results returned from any given
expression to 10,000 to protect against runaway memory usage. To be
clear, you have access to all the data server side. We are limiting the
size of the responses back from Blaze.

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

    # import the dataset
    from quantopian.interactive.data.quandl import yahoo_index_vix as dataset
    # Since this data is provided by Quandl for free, there is no _free version of this
    # data set, as found in the premium sets. This import gets you the entirety of this data set.
    
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
      open_: float64,
      high: float64,
      low: float64,
      close: float64,
      volume: float64,
      adjusted_close: float64,
      asof_date: datetime,
      timestamp: datetime
      }""")



.. code:: ipython2

    # And how many rows are there?
    # N.B. we're using a Blaze function to do this, not len()
    dataset.count()




.. raw:: html

    6651



.. code:: ipython2

    # Let's see what the data looks like. We'll grab the first three rows.
    dataset[:3]




.. raw:: html

    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>open_</th>
          <th>high</th>
          <th>low</th>
          <th>close</th>
          <th>volume</th>
          <th>adjusted_close</th>
          <th>asof_date</th>
          <th>timestamp</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>19.750000</td>
          <td>21.160000</td>
          <td>19.540001</td>
          <td>20.980000</td>
          <td>0</td>
          <td>20.980000</td>
          <td>2016-02-23</td>
          <td>2016-02-24 08:01:58.351899</td>
        </tr>
        <tr>
          <th>1</th>
          <td>22.280001</td>
          <td>22.870001</td>
          <td>20.260000</td>
          <td>20.719999</td>
          <td>0</td>
          <td>20.719999</td>
          <td>2016-02-24</td>
          <td>2016-02-25 08:02:33.397136</td>
        </tr>
        <tr>
          <th>2</th>
          <td>20.540001</td>
          <td>21.260000</td>
          <td>19.100000</td>
          <td>19.110001</td>
          <td>0</td>
          <td>19.110001</td>
          <td>2016-02-25</td>
          <td>2016-02-26 05:01:23.226761</td>
        </tr>
      </tbody>
    </table>



Let’s go over the columns: - **asof_date**: the timeframe to which this
data applies - **timestamp**: the simulated date upon which this data
point is available to a backtest - **open**: opening price for the day
indicated on asof_date - **high**: high price for the day indicated on
asof_date - **low**: lowest price for the day indicated by asof_date -
**close**: closing price for asof_date

We’ve done much of the data processing for you. Fields like
``timestamp`` and ``sid`` are standardized across all our Store
Datasets, so the datasets are easy to combine. We have standardized the
``sid`` across all our equity databases.

We can select columns and rows with ease. Let’s go plot it for fun
below. 6500 rows is small enough to just convert right over to Pandas.

.. code:: ipython2

    # Convert it over to a Pandas dataframe for easy charting
    vix_df = odo(dataset, pd.DataFrame)
    
    vix_df.plot(x='asof_date', y='close')
    plt.xlabel("As of Date (asof_date)")
    plt.ylabel("Close Price")
    plt.axis([None, None, 0, 100])
    plt.title("VIX")
    plt.legend().set_visible(False)



.. image:: notebook_files/notebook_6_0.png


#Pipeline Overview

Accessing the data in your algorithms & research
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The only method for accessing partner data within algorithms running on
Quantopian is via the pipeline API. Different data sets work differently
but in the case of this data, you can add this data to your pipeline as
follows:

Import the data set here >
``from quantopian.pipeline.data.quandl import yahoo_index_vix``

Then in intialize() you could do something simple like adding the raw
value of one of the fields to your pipeline: >
``pipe.add(yahoo_index_vix.close, 'close')``

.. code:: ipython2

    # Import necessary Pipeline modules
    from quantopian.pipeline import Pipeline
    from quantopian.research import run_pipeline
    from quantopian.pipeline.factors import AverageDollarVolume

.. code:: ipython2

    # For use in your algorithms
    # Using the full dataset in your pipeline algo
    from quantopian.pipeline.data.quandl import yahoo_index_vix

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
    
    for data in (yahoo_index_vix,):
        _print_fields(data)
    
    
    print "---------------------------------------------------\n"


.. parsed-literal::

    Here are the list of available fields per dataset:
    ---------------------------------------------------
    
    Dataset: yahoo_index_vix
    
    Fields:
    low - float64
    high - float64
    adjusted_close - float64
    volume - float64
    close - float64
    open_ - float64
    
    
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
           
    pipe.add(yahoo_index_vix.open_.latest, 'open')
    pipe.add(yahoo_index_vix.close.latest, 'close')
    pipe.add(yahoo_index_vix.adjusted_close.latest, 'adjusted_close')
    pipe.add(yahoo_index_vix.high.latest, 'high')
    pipe.add(yahoo_index_vix.low.latest, 'low')
    pipe.add(yahoo_index_vix.volume.latest, 'volume')

.. code:: ipython2

    # The show_graph() method of pipeline objects produces a graph to show how it is being calculated.
    pipe.show_graph(format='png')




.. image:: notebook_files/notebook_14_0.png



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
          <th>adjusted_close</th>
          <th>close</th>
          <th>high</th>
          <th>low</th>
          <th>open</th>
          <th>volume</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th rowspan="30" valign="top">2013-11-01 00:00:00+00:00</th>
          <th>Equity(2 [AA])</th>
          <td>13.75</td>
          <td>13.75</td>
          <td>14.02</td>
          <td>13.28</td>
          <td>13.83</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(21 [AAME])</th>
          <td>13.75</td>
          <td>13.75</td>
          <td>14.02</td>
          <td>13.28</td>
          <td>13.83</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(24 [AAPL])</th>
          <td>13.75</td>
          <td>13.75</td>
          <td>14.02</td>
          <td>13.28</td>
          <td>13.83</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(25 [AA_PR])</th>
          <td>13.75</td>
          <td>13.75</td>
          <td>14.02</td>
          <td>13.28</td>
          <td>13.83</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(31 [ABAX])</th>
          <td>13.75</td>
          <td>13.75</td>
          <td>14.02</td>
          <td>13.28</td>
          <td>13.83</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(39 [DDC])</th>
          <td>13.75</td>
          <td>13.75</td>
          <td>14.02</td>
          <td>13.28</td>
          <td>13.83</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(41 [ARCB])</th>
          <td>13.75</td>
          <td>13.75</td>
          <td>14.02</td>
          <td>13.28</td>
          <td>13.83</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(52 [ABM])</th>
          <td>13.75</td>
          <td>13.75</td>
          <td>14.02</td>
          <td>13.28</td>
          <td>13.83</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(53 [ABMD])</th>
          <td>13.75</td>
          <td>13.75</td>
          <td>14.02</td>
          <td>13.28</td>
          <td>13.83</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(62 [ABT])</th>
          <td>13.75</td>
          <td>13.75</td>
          <td>14.02</td>
          <td>13.28</td>
          <td>13.83</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(64 [ABX])</th>
          <td>13.75</td>
          <td>13.75</td>
          <td>14.02</td>
          <td>13.28</td>
          <td>13.83</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(66 [AB])</th>
          <td>13.75</td>
          <td>13.75</td>
          <td>14.02</td>
          <td>13.28</td>
          <td>13.83</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(67 [ADSK])</th>
          <td>13.75</td>
          <td>13.75</td>
          <td>14.02</td>
          <td>13.28</td>
          <td>13.83</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(69 [ACAT])</th>
          <td>13.75</td>
          <td>13.75</td>
          <td>14.02</td>
          <td>13.28</td>
          <td>13.83</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(70 [VBF])</th>
          <td>13.75</td>
          <td>13.75</td>
          <td>14.02</td>
          <td>13.28</td>
          <td>13.83</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(76 [TAP])</th>
          <td>13.75</td>
          <td>13.75</td>
          <td>14.02</td>
          <td>13.28</td>
          <td>13.83</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(84 [ACET])</th>
          <td>13.75</td>
          <td>13.75</td>
          <td>14.02</td>
          <td>13.28</td>
          <td>13.83</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(86 [ACG])</th>
          <td>13.75</td>
          <td>13.75</td>
          <td>14.02</td>
          <td>13.28</td>
          <td>13.83</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(88 [ACI])</th>
          <td>13.75</td>
          <td>13.75</td>
          <td>14.02</td>
          <td>13.28</td>
          <td>13.83</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(99 [ACO])</th>
          <td>13.75</td>
          <td>13.75</td>
          <td>14.02</td>
          <td>13.28</td>
          <td>13.83</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(100 [IEP])</th>
          <td>13.75</td>
          <td>13.75</td>
          <td>14.02</td>
          <td>13.28</td>
          <td>13.83</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(106 [ACU])</th>
          <td>13.75</td>
          <td>13.75</td>
          <td>14.02</td>
          <td>13.28</td>
          <td>13.83</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(110 [ACXM])</th>
          <td>13.75</td>
          <td>13.75</td>
          <td>14.02</td>
          <td>13.28</td>
          <td>13.83</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(112 [ACY])</th>
          <td>13.75</td>
          <td>13.75</td>
          <td>14.02</td>
          <td>13.28</td>
          <td>13.83</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(114 [ADBE])</th>
          <td>13.75</td>
          <td>13.75</td>
          <td>14.02</td>
          <td>13.28</td>
          <td>13.83</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(117 [AEY])</th>
          <td>13.75</td>
          <td>13.75</td>
          <td>14.02</td>
          <td>13.28</td>
          <td>13.83</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(122 [ADI])</th>
          <td>13.75</td>
          <td>13.75</td>
          <td>14.02</td>
          <td>13.28</td>
          <td>13.83</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(128 [ADM])</th>
          <td>13.75</td>
          <td>13.75</td>
          <td>14.02</td>
          <td>13.28</td>
          <td>13.83</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(134 [SXCL])</th>
          <td>13.75</td>
          <td>13.75</td>
          <td>14.02</td>
          <td>13.28</td>
          <td>13.83</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(149 [ADX])</th>
          <td>13.75</td>
          <td>13.75</td>
          <td>14.02</td>
          <td>13.28</td>
          <td>13.83</td>
          <td>0</td>
        </tr>
        <tr>
          <th>...</th>
          <th>...</th>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
        </tr>
        <tr>
          <th rowspan="30" valign="top">2013-11-25 00:00:00+00:00</th>
          <th>Equity(45864 [CDX])</th>
          <td>12.26</td>
          <td>12.26</td>
          <td>12.91</td>
          <td>12.24</td>
          <td>12.69</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(45865 [XNCR])</th>
          <td>12.26</td>
          <td>12.26</td>
          <td>12.91</td>
          <td>12.24</td>
          <td>12.69</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(45866 [ZU])</th>
          <td>12.26</td>
          <td>12.26</td>
          <td>12.91</td>
          <td>12.24</td>
          <td>12.69</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(45867 [EROS])</th>
          <td>12.26</td>
          <td>12.26</td>
          <td>12.91</td>
          <td>12.24</td>
          <td>12.69</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(45873 [IR_WI])</th>
          <td>12.26</td>
          <td>12.26</td>
          <td>12.91</td>
          <td>12.24</td>
          <td>12.69</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(45874 [ALLE])</th>
          <td>12.26</td>
          <td>12.26</td>
          <td>12.91</td>
          <td>12.24</td>
          <td>12.69</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(45875 [HFIN])</th>
          <td>12.26</td>
          <td>12.26</td>
          <td>12.91</td>
          <td>12.24</td>
          <td>12.69</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(45880 [CACQ])</th>
          <td>12.26</td>
          <td>12.26</td>
          <td>12.91</td>
          <td>12.24</td>
          <td>12.69</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(45882 [TKF_WD])</th>
          <td>12.26</td>
          <td>12.26</td>
          <td>12.91</td>
          <td>12.24</td>
          <td>12.69</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(45883 [IIF_WD])</th>
          <td>12.26</td>
          <td>12.26</td>
          <td>12.91</td>
          <td>12.24</td>
          <td>12.69</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(45885 [EGF_WD])</th>
          <td>12.26</td>
          <td>12.26</td>
          <td>12.91</td>
          <td>12.24</td>
          <td>12.69</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(45891 [OXFD])</th>
          <td>12.26</td>
          <td>12.26</td>
          <td>12.91</td>
          <td>12.24</td>
          <td>12.69</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(45892 [TLOG])</th>
          <td>12.26</td>
          <td>12.26</td>
          <td>12.91</td>
          <td>12.24</td>
          <td>12.69</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(45893 [VTL])</th>
          <td>12.26</td>
          <td>12.26</td>
          <td>12.91</td>
          <td>12.24</td>
          <td>12.69</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(45894 [RTGN])</th>
          <td>12.26</td>
          <td>12.26</td>
          <td>12.91</td>
          <td>12.24</td>
          <td>12.69</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(45895 [EMSH])</th>
          <td>12.26</td>
          <td>12.26</td>
          <td>12.91</td>
          <td>12.24</td>
          <td>12.69</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(45896 [AMZG])</th>
          <td>12.26</td>
          <td>12.26</td>
          <td>12.91</td>
          <td>12.24</td>
          <td>12.69</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(45902 [WBAI])</th>
          <td>12.26</td>
          <td>12.26</td>
          <td>12.91</td>
          <td>12.24</td>
          <td>12.69</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(45903 [GOMO])</th>
          <td>12.26</td>
          <td>12.26</td>
          <td>12.91</td>
          <td>12.24</td>
          <td>12.69</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(45904 [IPWR])</th>
          <td>12.26</td>
          <td>12.26</td>
          <td>12.91</td>
          <td>12.24</td>
          <td>12.69</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(45905 [GFIS])</th>
          <td>12.26</td>
          <td>12.26</td>
          <td>12.91</td>
          <td>12.24</td>
          <td>12.69</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(45906 [VNCE])</th>
          <td>12.26</td>
          <td>12.26</td>
          <td>12.91</td>
          <td>12.24</td>
          <td>12.69</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(45907 [RITT_W])</th>
          <td>12.26</td>
          <td>12.26</td>
          <td>12.91</td>
          <td>12.24</td>
          <td>12.69</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(45914 [EVGN])</th>
          <td>12.26</td>
          <td>12.26</td>
          <td>12.91</td>
          <td>12.24</td>
          <td>12.69</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(45915 [NVGS])</th>
          <td>12.26</td>
          <td>12.26</td>
          <td>12.91</td>
          <td>12.24</td>
          <td>12.69</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(48504 [ERUS])</th>
          <td>12.26</td>
          <td>12.26</td>
          <td>12.91</td>
          <td>12.24</td>
          <td>12.69</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(49010 [TBRA])</th>
          <td>12.26</td>
          <td>12.26</td>
          <td>12.91</td>
          <td>12.24</td>
          <td>12.69</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(49131 [OESX])</th>
          <td>12.26</td>
          <td>12.26</td>
          <td>12.91</td>
          <td>12.24</td>
          <td>12.69</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(49259 [ITUS])</th>
          <td>12.26</td>
          <td>12.26</td>
          <td>12.91</td>
          <td>12.24</td>
          <td>12.69</td>
          <td>0</td>
        </tr>
        <tr>
          <th>Equity(49523 [TLGT])</th>
          <td>12.26</td>
          <td>12.26</td>
          <td>12.91</td>
          <td>12.24</td>
          <td>12.69</td>
          <td>0</td>
        </tr>
      </tbody>
    </table>
    <p>134806 rows × 6 columns</p>
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
    # Using the full dataset in your pipeline algo
    from quantopian.pipeline.data.quandl import yahoo_index_vix
    
    def make_pipeline():
        # Create our pipeline
        pipe = Pipeline()
    
        # Add pipeline factors
        pipe.add(yahoo_index_vix.open_.latest, 'open')
        pipe.add(yahoo_index_vix.close.latest, 'close')
        pipe.add(yahoo_index_vix.adjusted_close.latest, 'adjusted_close')
        pipe.add(yahoo_index_vix.high.latest, 'high')
        pipe.add(yahoo_index_vix.low.latest, 'low')
        pipe.add(yahoo_index_vix.volume.latest, 'volume')
    
        return pipe
    
    def initialize(context):
        attach_pipeline(make_pipeline(), "pipeline")
        
    def before_trading_start(context, data):
        results = pipeline_output('pipeline')

Now you can take that and begin to use it as a building block for your
algorithms, for more examples on how to do that you can visit our data
pipeline factor library
