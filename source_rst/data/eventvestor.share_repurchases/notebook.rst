EventVestor: Share Repurchases
==============================

In this notebook, we’ll take a look at EventVestor’s *Share Repurchases*
dataset, available on the `Quantopian
Store <https://www.quantopian.com/store>`__. This dataset spans January
01, 2007 through the current day, and documents actual share repurchase
announcements by companies. Note that this is **different** from `Share
Buyback
Authorizations <https://www.quantopian.com/store/eventvestor/buyback_auth>`__.

Blaze
~~~~~

Before we dig into the data, we want to tell you about how you generally
access Quantopian Store data sets. These datasets are available through
an API service known as `Blaze <http://blaze.pydata.org>`__. Blaze
provides the Quantopian user with a convenient interface to access very
large datasets.

Blaze provides an important function for accessing these datasets. Some
of these sets are many millions of records. Bringing that data directly
into Quantopian Research directly just is not viable. So Blaze allows us
to provide a simple querying interface and shift the burden over to the
server side.

It is common to use Blaze to reduce your dataset in size, convert it
over to Pandas and then to use Pandas for further computation,
manipulation and visualization.

Helpful links: \* `Query building for
Blaze <http://blaze.pydata.org/en/latest/queries.html>`__ \*
`Pandas-to-Blaze
dictionary <http://blaze.pydata.org/en/latest/rosetta-pandas.html>`__ \*
`SQL-to-Blaze
dictionary <http://blaze.pydata.org/en/latest/rosetta-sql.html>`__.

| Once you’ve limited the size of your Blaze object, you can convert it
  to a Pandas DataFrames using: > ``from odo import odo``
| > ``odo(expr, pandas.DataFrame)``

Free samples and limits
~~~~~~~~~~~~~~~~~~~~~~~

One other key caveat: we limit the number of results returned from any
given expression to 10,000 to protect against runaway memory usage. To
be clear, you have access to all the data server side. We are limiting
the size of the responses back from Blaze.

There is a *free* version of this dataset as well as a paid one. The
free one includes about three years of historical data, though not up to
the current day.

With preamble in place, let’s get started:

.. code:: ipython2

    # import the dataset
    from quantopian.interactive.data.eventvestor import share_repurchases
    # or if you want to import the free dataset, use:
    # from quantopian.interactive.data.eventvestor import share_repurchases_free
    
    # import data operations
    from odo import odo
    # import other libraries we will use
    import pandas as pd

.. code:: ipython2

    # Let's use blaze to understand the data a bit using Blaze dshape()
    share_repurchases.dshape




.. parsed-literal::

    dshape("""var * {
      event_id: ?float64,
      asof_date: datetime,
      trade_date: ?datetime,
      symbol: ?string,
      event_type: ?string,
      event_headline: ?string,
      repurchase_amount: ?float64,
      repurchase_units: ?string,
      event_rating: ?float64,
      timestamp: datetime,
      sid: ?int64
      }""")



.. code:: ipython2

    # And how many rows are there?
    # N.B. we're using a Blaze function to do this, not len()
    share_repurchases.count()




.. raw:: html

    15509



.. code:: ipython2

    # Let's see what the data looks like. We'll grab the first three rows.
    share_repurchases[:3]




.. raw:: html

    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>event_id</th>
          <th>asof_date</th>
          <th>trade_date</th>
          <th>symbol</th>
          <th>event_type</th>
          <th>event_headline</th>
          <th>repurchase_amount</th>
          <th>repurchase_units</th>
          <th>event_rating</th>
          <th>timestamp</th>
          <th>sid</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>1113050</td>
          <td>2007-01-17</td>
          <td>2007-01-17</td>
          <td>TESS</td>
          <td>Buyback Update</td>
          <td>TESSCO Tech Repurchases $1.7M Shares in 3Q 07 ...</td>
          <td>1.7</td>
          <td>$M</td>
          <td>1</td>
          <td>2007-01-18</td>
          <td>11968</td>
        </tr>
        <tr>
          <th>1</th>
          <td>131345</td>
          <td>2007-01-17</td>
          <td>2007-01-18</td>
          <td>WM</td>
          <td>Buyback Update</td>
          <td>Washington Mutual Announces $2.7B Accelerated ...</td>
          <td>2700.0</td>
          <td>$M</td>
          <td>1</td>
          <td>2007-01-18</td>
          <td>19181</td>
        </tr>
        <tr>
          <th>2</th>
          <td>137183</td>
          <td>2007-01-23</td>
          <td>2007-01-23</td>
          <td>RDN</td>
          <td>Buyback Update</td>
          <td>Radian Group Repurchased 1.5M shares for $81.1...</td>
          <td>81.1</td>
          <td>$M</td>
          <td>1</td>
          <td>2007-01-24</td>
          <td>20276</td>
        </tr>
      </tbody>
    </table>



Let’s go over the columns: - **event_id**: the unique identifier for
this event. - **asof_date**: EventVestor’s timestamp of event capture. -
**trade_date**: for event announcements made before trading ends,
trade_date is the same as event_date. For announcements issued after
market close, trade_date is next market open day. - **symbol**: stock
ticker symbol of the affected company. - **event_type**: this should
always be *Buyback Update*. - **event_headline**: a brief description of
the event - **repurchase_amount**: amount of shares (in
repurchase_units) repurchased during the reported period -
**repurchase_units**: millions of dollars or percent of total shares
outstanding. - **event_rating**: this is always 1. The meaning of this
is uncertain. - **timestamp**: this is our timestamp on when we
registered the data. - **sid**: the equity’s unique identifier. Use this
instead of the symbol.

We’ve done much of the data processing for you. Fields like
``timestamp`` and ``sid`` are standardized across all our Store
Datasets, so the datasets are easy to combine. We have standardized the
``sid`` across all our equity databases.

We can select columns and rows with ease. Below, we’ll fetch Apple’s
2014 share repurchases.

.. code:: ipython2

    # get apple's sid first
    apple_sid = symbols('AAPL').sid
    buybacks = share_repurchases[('2013-12-31' < share_repurchases['asof_date']) & 
                                    (share_repurchases['asof_date'] <'2015-01-01') & 
                                    (share_repurchases.sid == apple_sid)]
    # When displaying a Blaze Data Object, the printout is automatically truncated to ten rows.
    buybacks.sort('asof_date')




.. raw:: html

    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>event_id</th>
          <th>asof_date</th>
          <th>trade_date</th>
          <th>symbol</th>
          <th>event_type</th>
          <th>event_headline</th>
          <th>repurchase_amount</th>
          <th>repurchase_units</th>
          <th>event_rating</th>
          <th>timestamp</th>
          <th>sid</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>1918241</td>
          <td>2014-01-27</td>
          <td>2014-01-27</td>
          <td>AAPL</td>
          <td>Buyback Update</td>
          <td>Apple Repurchases $5.03B Common Stock in 1Q 14</td>
          <td>5029</td>
          <td>$M</td>
          <td>1</td>
          <td>2014-01-28</td>
          <td>24</td>
        </tr>
        <tr>
          <th>1</th>
          <td>1674141</td>
          <td>2014-02-07</td>
          <td>2014-02-07</td>
          <td>AAPL</td>
          <td>Buyback Update</td>
          <td>Apple Repurchases $14B Common Stock Since 1Q 1...</td>
          <td>14000</td>
          <td>$M</td>
          <td>1</td>
          <td>2014-02-08</td>
          <td>24</td>
        </tr>
        <tr>
          <th>2</th>
          <td>1918254</td>
          <td>2014-04-23</td>
          <td>2014-04-23</td>
          <td>AAPL</td>
          <td>Buyback Update</td>
          <td>Apple Repurchases $23B Common Stock in FY 14 YTD</td>
          <td>23000</td>
          <td>$M</td>
          <td>1</td>
          <td>2014-04-24</td>
          <td>24</td>
        </tr>
        <tr>
          <th>3</th>
          <td>1918258</td>
          <td>2014-07-22</td>
          <td>2014-07-22</td>
          <td>AAPL</td>
          <td>Buyback Update</td>
          <td>Apple Repurchases $28B Common Stock in FY 14 YTD</td>
          <td>5000</td>
          <td>$M</td>
          <td>1</td>
          <td>2014-07-23</td>
          <td>24</td>
        </tr>
        <tr>
          <th>4</th>
          <td>1918275</td>
          <td>2014-10-20</td>
          <td>2014-10-20</td>
          <td>AAPL</td>
          <td>Buyback Update</td>
          <td>Apple Repurchases $45B Common Stock in FY 14</td>
          <td>17000</td>
          <td>$M</td>
          <td>1</td>
          <td>2014-10-21</td>
          <td>24</td>
        </tr>
      </tbody>
    </table>



Now suppose we want a DataFrame of the Blaze Data Object above, but only
want the ``asof_date, repurchase_units``, and the ``repurchase_amount``.

.. code:: ipython2

    df = odo(buybacks, pd.DataFrame)
    df = df[['asof_date','repurchase_amount','repurchase_units']]
    df




.. raw:: html

    <div style="max-height:1000px;max-width:1500px;overflow:auto;">
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>asof_date</th>
          <th>repurchase_amount</th>
          <th>repurchase_units</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>2014-01-27</td>
          <td>5029</td>
          <td>$M</td>
        </tr>
        <tr>
          <th>1</th>
          <td>2014-02-07</td>
          <td>14000</td>
          <td>$M</td>
        </tr>
        <tr>
          <th>2</th>
          <td>2014-04-23</td>
          <td>23000</td>
          <td>$M</td>
        </tr>
        <tr>
          <th>3</th>
          <td>2014-07-22</td>
          <td>5000</td>
          <td>$M</td>
        </tr>
        <tr>
          <th>4</th>
          <td>2014-10-20</td>
          <td>17000</td>
          <td>$M</td>
        </tr>
      </tbody>
    </table>
    </div>


