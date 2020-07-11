EventVestor: Stock Splits
=========================

In this notebook, we’ll take a look at EventVestor’s *Stock Splits*
dataset, available on the `Quantopian
Store <https://www.quantopian.com/store>`__. This dataset spans January
01, 2007 through the current day, and documents stock splits and reverse
stock splits.

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
    from quantopian.interactive.data.eventvestor import stock_splits
    # or if you want to import the free dataset, use:
    # from quantopian.data.eventvestor import stock_splits_free
    
    # import data operations
    from odo import odo
    # import other libraries we will use
    import pandas as pd

.. code:: ipython2

    # Let's use blaze to understand the data a bit using Blaze dshape()
    stock_splits.dshape




.. parsed-literal::

    dshape("""var * {
      event_id: ?float64,
      asof_date: datetime,
      trade_date: ?datetime,
      symbol: ?string,
      event_type: ?string,
      event_headline: ?string,
      split_type: ?string,
      split_factor: ?string,
      new_shares: ?float64,
      old_shares: ?float64,
      effective_date: ?datetime,
      event_rating: ?float64,
      timestamp: datetime,
      sid: ?int64
      }""")



.. code:: ipython2

    # And how many rows are there?
    # N.B. we're using a Blaze function to do this, not len()
    stock_splits.count()




.. raw:: html

    1062



.. code:: ipython2

    # Let's see what the data looks like. We'll grab the first three rows.
    stock_splits[:3]




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
          <th>split_type</th>
          <th>split_factor</th>
          <th>new_shares</th>
          <th>old_shares</th>
          <th>effective_date</th>
          <th>event_rating</th>
          <th>timestamp</th>
          <th>sid</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>61191</td>
          <td>2007-01-09</td>
          <td>2007-01-09</td>
          <td>MDCI</td>
          <td>Stock Split</td>
          <td>Medical Action announces 3-for-2 stock split, ...</td>
          <td>Split</td>
          <td>3-for-2</td>
          <td>3</td>
          <td>2</td>
          <td>NaT</td>
          <td>1</td>
          <td>2007-01-10</td>
          <td>4737</td>
        </tr>
        <tr>
          <th>1</th>
          <td>61190</td>
          <td>2007-01-09</td>
          <td>2007-01-09</td>
          <td>SSI</td>
          <td>Stock Split</td>
          <td>Stage Stores announces 3-for-2 stock split, pa...</td>
          <td>Split</td>
          <td>3-for-2</td>
          <td>3</td>
          <td>2</td>
          <td>NaT</td>
          <td>1</td>
          <td>2007-01-10</td>
          <td>23395</td>
        </tr>
        <tr>
          <th>2</th>
          <td>61189</td>
          <td>2007-01-17</td>
          <td>2007-01-17</td>
          <td>APH</td>
          <td>Stock Split</td>
          <td>Amphenol announces 2-for-1 stock split, payabl...</td>
          <td>Split</td>
          <td>2-for-1</td>
          <td>2</td>
          <td>1</td>
          <td>NaT</td>
          <td>1</td>
          <td>2007-01-18</td>
          <td>465</td>
        </tr>
      </tbody>
    </table>



Let’s go over the columns: - **event_id**: the unique identifier for
this event. - **asof_date**: EventVestor’s timestamp of event capture. -
**trade_date**: for event announcements made before trading ends,
trade_date is the same as event_date. For announcements issued after
market close, trade_date is next market open day. - **symbol**: stock
ticker symbol of the affected company. - **event_type**: this should
always be *Stock Split*. - **event_headline**: a brief description of
the event - **split_type**: *stock split* or *reverse split* -
**split_factor**: the ``x-for-y`` split factor. This is equivalently
expressed by ``new_shares`` and ``old_shares``. - **new_shares**: number
of new shares for ``x`` number of old shares - **old_shares**: number of
old shares exchanged for the number of new shares. - **effective_date**:
effective date of stock split. - **event_rating**: this is always 1. The
meaning of this is uncertain. - **timestamp**: this is our timestamp on
when we registered the data. - **sid**: the equity’s unique identifier.
Use this instead of the symbol.

We’ve done much of the data processing for you. Fields like
``timestamp`` and ``sid`` are standardized across all our Store
Datasets, so the datasets are easy to combine. We have standardized the
``sid`` across all our equity databases.

We can select columns and rows with ease. Below, we’ll fetch Nike’s
stock splits.

.. code:: ipython2

    # get apple's sid first
    nike_sid = symbols('NKE').sid
    splits = stock_splits[(stock_splits.sid == nike_sid)]
    # When displaying a Blaze Data Object, the printout is automatically truncated to ten rows.
    splits.sort('asof_date')




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
          <th>split_type</th>
          <th>split_factor</th>
          <th>new_shares</th>
          <th>old_shares</th>
          <th>effective_date</th>
          <th>event_rating</th>
          <th>timestamp</th>
          <th>sid</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>61171</td>
          <td>2007-02-15</td>
          <td>2007-02-15</td>
          <td>NKE</td>
          <td>Stock Split</td>
          <td>Nike announces 2-for-1 stock split</td>
          <td>Split</td>
          <td>2-for-1</td>
          <td>2</td>
          <td>1</td>
          <td>NaT</td>
          <td>1</td>
          <td>2007-02-16</td>
          <td>5328</td>
        </tr>
        <tr>
          <th>1</th>
          <td>1509519</td>
          <td>2012-11-15</td>
          <td>2012-11-16</td>
          <td>NKE</td>
          <td>Stock Split</td>
          <td>Nike Announces Two-For-One Stock Split</td>
          <td>Split</td>
          <td>2-for-1</td>
          <td>2</td>
          <td>1</td>
          <td>NaT</td>
          <td>1</td>
          <td>2012-11-16</td>
          <td>5328</td>
        </tr>
      </tbody>
    </table>



Now suppose we want a DataFrame of ``stock_splits``, but limited to
reverse splits only. Of those, we then want to display the
``split_factor``, ``timestamp``, and ``sid``.

.. code:: ipython2

    reverse = stock_splits[stock_splits.split_type == "Reverse Split"]
    df = odo(reverse, pd.DataFrame)
    df = df[['asof_date','split_factor','sid']]
    df = df[df.sid.notnull()]
    # When printing a pandas DataFrame, the head 30 and tail 30 rows are displayed. The middle is truncated.
    df




.. raw:: html

    <div style="max-height:1000px;max-width:1500px;overflow:auto;">
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>asof_date</th>
          <th>split_factor</th>
          <th>sid</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>2007-02-20</td>
          <td>1-for-3</td>
          <td>21120</td>
        </tr>
        <tr>
          <th>3</th>
          <td>2007-03-29</td>
          <td>1-for-4</td>
          <td>16607</td>
        </tr>
        <tr>
          <th>4</th>
          <td>2007-07-17</td>
          <td>8-for-9</td>
          <td>12626</td>
        </tr>
        <tr>
          <th>5</th>
          <td>2007-08-01</td>
          <td>1-for-6</td>
          <td>17914</td>
        </tr>
        <tr>
          <th>7</th>
          <td>2007-08-07</td>
          <td>1-for-10</td>
          <td>6276</td>
        </tr>
        <tr>
          <th>8</th>
          <td>2007-08-20</td>
          <td>1-for-20</td>
          <td>17504</td>
        </tr>
        <tr>
          <th>10</th>
          <td>2007-11-01</td>
          <td>1-for-10</td>
          <td>10583</td>
        </tr>
        <tr>
          <th>11</th>
          <td>2007-11-14</td>
          <td>1-for-4</td>
          <td>17799</td>
        </tr>
        <tr>
          <th>12</th>
          <td>2008-01-31</td>
          <td>0-for-0</td>
          <td>26837</td>
        </tr>
        <tr>
          <th>13</th>
          <td>2008-02-22</td>
          <td>1-for-5</td>
          <td>24074</td>
        </tr>
        <tr>
          <th>14</th>
          <td>2008-03-07</td>
          <td>1-for-12</td>
          <td>19187</td>
        </tr>
        <tr>
          <th>17</th>
          <td>2008-04-11</td>
          <td>1-for-10</td>
          <td>14420</td>
        </tr>
        <tr>
          <th>18</th>
          <td>2008-04-23</td>
          <td>1-for-4</td>
          <td>19635</td>
        </tr>
        <tr>
          <th>20</th>
          <td>2008-04-25</td>
          <td>1-for-10</td>
          <td>1365</td>
        </tr>
        <tr>
          <th>21</th>
          <td>2008-05-07</td>
          <td>0-for-0</td>
          <td>6804</td>
        </tr>
        <tr>
          <th>22</th>
          <td>2008-05-09</td>
          <td>1-for-3</td>
          <td>7121</td>
        </tr>
        <tr>
          <th>23</th>
          <td>2008-05-30</td>
          <td>1-for-5</td>
          <td>24074</td>
        </tr>
        <tr>
          <th>24</th>
          <td>2008-06-02</td>
          <td>1-for-10</td>
          <td>19682</td>
        </tr>
        <tr>
          <th>25</th>
          <td>2008-06-02</td>
          <td>1-for-5</td>
          <td>25206</td>
        </tr>
        <tr>
          <th>26</th>
          <td>2008-06-16</td>
          <td>1-for-5</td>
          <td>15881</td>
        </tr>
        <tr>
          <th>27</th>
          <td>2008-07-01</td>
          <td>1-for-5</td>
          <td>25206</td>
        </tr>
        <tr>
          <th>28</th>
          <td>2008-07-03</td>
          <td>1-for-20</td>
          <td>21291</td>
        </tr>
        <tr>
          <th>29</th>
          <td>2008-07-09</td>
          <td>1-for-10</td>
          <td>1365</td>
        </tr>
        <tr>
          <th>30</th>
          <td>2008-07-11</td>
          <td>1-for-10</td>
          <td>21113</td>
        </tr>
        <tr>
          <th>31</th>
          <td>2008-08-12</td>
          <td>1-for-5</td>
          <td>14469</td>
        </tr>
        <tr>
          <th>32</th>
          <td>2008-08-21</td>
          <td>1-for-2</td>
          <td>26470</td>
        </tr>
        <tr>
          <th>33</th>
          <td>2008-08-29</td>
          <td>1-for-10</td>
          <td>16607</td>
        </tr>
        <tr>
          <th>34</th>
          <td>2008-09-05</td>
          <td>1-for-10</td>
          <td>23635</td>
        </tr>
        <tr>
          <th>35</th>
          <td>2008-09-11</td>
          <td>1-for-20</td>
          <td>9774</td>
        </tr>
        <tr>
          <th>36</th>
          <td>2008-09-16</td>
          <td>1-for-10</td>
          <td>14420</td>
        </tr>
        <tr>
          <th>...</th>
          <td>...</td>
          <td>...</td>
          <td>...</td>
        </tr>
        <tr>
          <th>391</th>
          <td>2015-05-26</td>
          <td>1-for-6</td>
          <td>32867</td>
        </tr>
        <tr>
          <th>392</th>
          <td>2015-05-28</td>
          <td>1-for-10</td>
          <td>12765</td>
        </tr>
        <tr>
          <th>393</th>
          <td>2015-06-01</td>
          <td>1-for-4</td>
          <td>19709</td>
        </tr>
        <tr>
          <th>394</th>
          <td>2015-06-18</td>
          <td>1-for-8</td>
          <td>35162</td>
        </tr>
        <tr>
          <th>395</th>
          <td>2015-06-18</td>
          <td>0-for-0</td>
          <td>39627</td>
        </tr>
        <tr>
          <th>396</th>
          <td>2015-06-18</td>
          <td>1-for-8</td>
          <td>40461</td>
        </tr>
        <tr>
          <th>397</th>
          <td>2015-06-23</td>
          <td>1-for-4</td>
          <td>19709</td>
        </tr>
        <tr>
          <th>398</th>
          <td>2015-06-25</td>
          <td>1-for-7</td>
          <td>39627</td>
        </tr>
        <tr>
          <th>400</th>
          <td>2015-06-26</td>
          <td>1-for-5</td>
          <td>28718</td>
        </tr>
        <tr>
          <th>401</th>
          <td>2015-06-29</td>
          <td>1-for-10</td>
          <td>12765</td>
        </tr>
        <tr>
          <th>402</th>
          <td>2015-07-08</td>
          <td>1-for-4</td>
          <td>40822</td>
        </tr>
        <tr>
          <th>403</th>
          <td>2015-07-10</td>
          <td>1-for-8</td>
          <td>4982</td>
        </tr>
        <tr>
          <th>404</th>
          <td>2015-07-13</td>
          <td>1-for-8</td>
          <td>44063</td>
        </tr>
        <tr>
          <th>405</th>
          <td>2015-07-15</td>
          <td>1-for-10</td>
          <td>42820</td>
        </tr>
        <tr>
          <th>406</th>
          <td>2015-07-20</td>
          <td>1-for-10</td>
          <td>88</td>
        </tr>
        <tr>
          <th>407</th>
          <td>2015-07-20</td>
          <td>1-for-10</td>
          <td>41717</td>
        </tr>
        <tr>
          <th>408</th>
          <td>2015-07-21</td>
          <td>1-for-10</td>
          <td>40531</td>
        </tr>
        <tr>
          <th>409</th>
          <td>2015-07-27</td>
          <td>1-for-10</td>
          <td>88</td>
        </tr>
        <tr>
          <th>410</th>
          <td>2015-07-31</td>
          <td>0-for-0</td>
          <td>35162</td>
        </tr>
        <tr>
          <th>411</th>
          <td>2015-08-03</td>
          <td>1-for-10</td>
          <td>42820</td>
        </tr>
        <tr>
          <th>412</th>
          <td>2015-08-04</td>
          <td>1-for-4</td>
          <td>28076</td>
        </tr>
        <tr>
          <th>413</th>
          <td>2015-08-04</td>
          <td>1-for-10</td>
          <td>6523</td>
        </tr>
        <tr>
          <th>414</th>
          <td>2015-08-06</td>
          <td>1-for-10</td>
          <td>19074</td>
        </tr>
        <tr>
          <th>415</th>
          <td>2015-08-18</td>
          <td>1-for-3</td>
          <td>19350</td>
        </tr>
        <tr>
          <th>416</th>
          <td>2015-08-24</td>
          <td>1-for-5</td>
          <td>28416</td>
        </tr>
        <tr>
          <th>417</th>
          <td>2015-08-25</td>
          <td>1-for-7</td>
          <td>39627</td>
        </tr>
        <tr>
          <th>418</th>
          <td>2015-08-31</td>
          <td>1-for-4</td>
          <td>28076</td>
        </tr>
        <tr>
          <th>419</th>
          <td>2015-09-03</td>
          <td>1-for-7</td>
          <td>24624</td>
        </tr>
        <tr>
          <th>420</th>
          <td>2015-09-09</td>
          <td>0-for-0</td>
          <td>22253</td>
        </tr>
        <tr>
          <th>421</th>
          <td>2015-09-16</td>
          <td>1-for-15</td>
          <td>22660</td>
        </tr>
      </tbody>
    </table>
    <p>361 rows × 3 columns</p>
    </div>



