EventVestor: Earnings Releases
==============================

In this notebook, we’ll take a look at EventVestor’s *Earnings Releases*
dataset, available on the `Quantopian
Store <https://www.quantopian.com/store>`__. This dataset spans January
01, 2007 through the current day, and documents quarterly earnings
releases.

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
    from quantopian.interactive.data.eventvestor import earnings_releases
    # or if you want to import the free dataset, use:
    # from quantopian.interactivedata.eventvestor import earnings_releases_free
    
    # import data operations
    from odo import odo
    # import other libraries we will use
    import pandas as pd

.. code:: ipython2

    # Let's use blaze to understand the data a bit using Blaze dshape()
    earnings_releases.dshape




.. parsed-literal::

    dshape("""var * {
      event_id: ?float64,
      asof_date: datetime,
      trade_date: ?datetime,
      symbol: ?string,
      event_type: ?string,
      event_headline: ?string,
      event_phase: ?string,
      fiscal_period: ?string,
      calendar_period: ?string,
      fiscal_periodend: ?datetime,
      currency: ?string,
      revenue: ?float64,
      gross_income: ?float64,
      operating_income: ?float64,
      net_income: ?float64,
      eps: ?float64,
      eps_surprisepct: ?float64,
      event_rating: ?float64,
      timestamp: datetime,
      sid: ?int64
      }""")



.. code:: ipython2

    # And how many rows are there?
    # N.B. we're using a Blaze function to do this, not len()
    earnings_releases.count()




.. raw:: html

    139427



.. code:: ipython2

    # Let's see what the data looks like. We'll grab the first three rows.
    earnings_releases[:3]




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
          <th>event_phase</th>
          <th>fiscal_period</th>
          <th>calendar_period</th>
          <th>fiscal_periodend</th>
          <th>currency</th>
          <th>revenue</th>
          <th>gross_income</th>
          <th>operating_income</th>
          <th>net_income</th>
          <th>eps</th>
          <th>eps_surprisepct</th>
          <th>event_rating</th>
          <th>timestamp</th>
          <th>sid</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>526391</td>
          <td>2007-01-03</td>
          <td>2007-01-04</td>
          <td>ANGO</td>
          <td>Earnings Release</td>
          <td>AngioDynamics 2Q Net up 48%</td>
          <td>NaN</td>
          <td>2Q 07</td>
          <td>4Q 06</td>
          <td>2006-12-02</td>
          <td>$</td>
          <td>24.37</td>
          <td>14.24</td>
          <td>3.0</td>
          <td>2.45</td>
          <td>0.15</td>
          <td>0</td>
          <td>1</td>
          <td>2007-01-04</td>
          <td>26324</td>
        </tr>
        <tr>
          <th>1</th>
          <td>196507</td>
          <td>2007-01-03</td>
          <td>2007-01-04</td>
          <td>BLUD</td>
          <td>Earnings Release</td>
          <td>Immucor Reports 2Q Results</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>NaT</td>
          <td>NaN</td>
          <td>0.00</td>
          <td>0.00</td>
          <td>0.0</td>
          <td>0.00</td>
          <td>0.00</td>
          <td>0</td>
          <td>1</td>
          <td>2007-01-04</td>
          <td>955</td>
        </tr>
        <tr>
          <th>2</th>
          <td>180559</td>
          <td>2007-01-03</td>
          <td>2007-01-03</td>
          <td>CALM</td>
          <td>Earnings Release</td>
          <td>CAL-MAINE FOODS REPORTS 2Q 07 RESULTS</td>
          <td>NaN</td>
          <td>2Q 07</td>
          <td>4Q 06</td>
          <td>2006-12-02</td>
          <td>$</td>
          <td>137.74</td>
          <td>24.96</td>
          <td>10.5</td>
          <td>6.40</td>
          <td>0.27</td>
          <td>0</td>
          <td>1</td>
          <td>2007-01-04</td>
          <td>16169</td>
        </tr>
      </tbody>
    </table>



Let’s go over the columns: - **event_id**: the unique identifier for
this event. - **asof_date**: EventVestor’s timestamp of event capture. -
**trade_date**: for event announcements made before trading ends,
trade_date is the same as event_date. For announcements issued after
market close, trade_date is next market open day. - **symbol**: stock
ticker symbol of the affected company. - **event_type**: this should
always be *Earnings Release/Earnings release*. - **event_headline**: a
brief description of the event - **event_phase**: the inclusion of this
field is likely an error on the part of the data vendor. We’re currently
attempting to resolve this. - **fiscal_period**: fiscal period for the
reported earnings, such as 1Q 15, 2Q 15, etc. - **calendar_period**:
identifies the calendar period based on the fiscal period end date. E.g.
if the fiscal period ends any time after the middle of a given calendar
quarter, like 1Q 15, that calendar quarter will be assigned regardless
of the fiscal quarter. - **fiscal_periodend**: the last date for the
reported earnings period. - **currency**: currency used for reporting
earnings. - **revenue**: revenue in millions - **gross_income**: gross
income in millions - **operating_income**: operating income in millions
- **net_income**: net income in millions - **eps**: earnings per share,
in the reported currency - **eps_surprisepct**: the meaning of this
column is presently uncertain. We’re working with our data vendor to
resolve this issue. - **event_rating**: this is always 1. The meaning of
this is uncertain. - **timestamp**: this is our timestamp on when we
registered the data. - **sid**: the equity’s unique identifier. Use this
instead of the symbol.

We’ve done much of the data processing for you. Fields like
``timestamp`` and ``sid`` are standardized across all our Store
Datasets, so the datasets are easy to combine. We have standardized the
``sid`` across all our equity databases.

We can select columns and rows with ease. Below, we’ll fetch all of
Apple’s entries from 2012.

.. code:: ipython2

    # get apple's sid first
    aapl_sid = symbols('AAPL').sid
    aapl_earnings = earnings_releases[('2011-12-31' < earnings_releases['asof_date']) & (earnings_releases['asof_date'] <'2013-01-01') & (earnings_releases.sid==aapl_sid)]
    # When displaying a Blaze Data Object, the printout is automatically truncated to ten rows.
    aapl_earnings.sort('asof_date')




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
          <th>event_phase</th>
          <th>fiscal_period</th>
          <th>calendar_period</th>
          <th>fiscal_periodend</th>
          <th>currency</th>
          <th>revenue</th>
          <th>gross_income</th>
          <th>operating_income</th>
          <th>net_income</th>
          <th>eps</th>
          <th>eps_surprisepct</th>
          <th>event_rating</th>
          <th>timestamp</th>
          <th>sid</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>1385939</td>
          <td>2012-01-24</td>
          <td>2012-01-25</td>
          <td>AAPL</td>
          <td>Earnings Release</td>
          <td>Apple 1Q 12 Net Jumps to $13B on Higher Revenues</td>
          <td>NaN</td>
          <td>1Q 12</td>
          <td>4Q 11</td>
          <td>2011-12-31</td>
          <td>$</td>
          <td>46333</td>
          <td>20703</td>
          <td>17340</td>
          <td>13064</td>
          <td>13.87</td>
          <td>38.29</td>
          <td>1</td>
          <td>2012-01-25</td>
          <td>24</td>
        </tr>
        <tr>
          <th>1</th>
          <td>1421108</td>
          <td>2012-04-24</td>
          <td>2012-04-25</td>
          <td>AAPL</td>
          <td>Earnings Release</td>
          <td>Apple 2Q 12 Net Up 94% on Higher Revenues</td>
          <td>NaN</td>
          <td>2Q 12</td>
          <td>1Q 12</td>
          <td>2012-03-31</td>
          <td>$</td>
          <td>39186</td>
          <td>18564</td>
          <td>15384</td>
          <td>11622</td>
          <td>12.30</td>
          <td>23.74</td>
          <td>1</td>
          <td>2012-04-25</td>
          <td>24</td>
        </tr>
        <tr>
          <th>2</th>
          <td>1456685</td>
          <td>2012-07-24</td>
          <td>2012-07-25</td>
          <td>AAPL</td>
          <td>Earnings Release</td>
          <td>Apple 3Q 12 Net Up 21%</td>
          <td>NaN</td>
          <td>3Q 12</td>
          <td>2Q 12</td>
          <td>2012-06-30</td>
          <td>$</td>
          <td>35023</td>
          <td>14994</td>
          <td>11573</td>
          <td>8824</td>
          <td>9.32</td>
          <td>-10.21</td>
          <td>1</td>
          <td>2012-07-25</td>
          <td>24</td>
        </tr>
        <tr>
          <th>3</th>
          <td>1496807</td>
          <td>2012-10-25</td>
          <td>2012-10-26</td>
          <td>AAPL</td>
          <td>Earnings Release</td>
          <td>Apple 4Q 12 Net Up 24%</td>
          <td>NaN</td>
          <td>4Q 12</td>
          <td>3Q 12</td>
          <td>2012-09-29</td>
          <td>$</td>
          <td>35966</td>
          <td>14401</td>
          <td>10944</td>
          <td>8223</td>
          <td>8.67</td>
          <td>-2.03</td>
          <td>1</td>
          <td>2012-10-26</td>
          <td>24</td>
        </tr>
      </tbody>
    </table>



Now suppose we want a DataFrame of all earnings releases with revenue
over 30 billion dollars. For those earnings releases, we only want the
sid and the asof_date.

.. code:: ipython2

    # manipulate with Blaze first:
    big_earnings = earnings_releases[earnings_releases.revenue > 40000]
    # now that we've got a much smaller object (len: ~2167 rows), we can convert it to a pandas DataFrame
    df = odo(big_earnings, pd.DataFrame)
    df = df[['sid', 'asof_date','revenue']].dropna()
    df.sort('revenue',ascending=False)




.. raw:: html

    <div style="max-height:1000px;max-width:1500px;overflow:auto;">
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>sid</th>
          <th>asof_date</th>
          <th>revenue</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>510</th>
          <td>26503</td>
          <td>2013-01-24</td>
          <td>8743000</td>
        </tr>
        <tr>
          <th>491</th>
          <td>26503</td>
          <td>2012-10-26</td>
          <td>7593000</td>
        </tr>
        <tr>
          <th>657</th>
          <td>26503</td>
          <td>2015-04-22</td>
          <td>7022000</td>
        </tr>
        <tr>
          <th>474</th>
          <td>26503</td>
          <td>2012-07-26</td>
          <td>6910000</td>
        </tr>
        <tr>
          <th>529</th>
          <td>26503</td>
          <td>2013-04-22</td>
          <td>6803000</td>
        </tr>
        <tr>
          <th>98</th>
          <td>7543</td>
          <td>2008-02-05</td>
          <td>6709983</td>
        </tr>
        <tr>
          <th>342</th>
          <td>26503</td>
          <td>2010-10-21</td>
          <td>6698000</td>
        </tr>
        <tr>
          <th>443</th>
          <td>26503</td>
          <td>2012-01-27</td>
          <td>6610000</td>
        </tr>
        <tr>
          <th>590</th>
          <td>7543</td>
          <td>2014-02-04</td>
          <td>6585044</td>
        </tr>
        <tr>
          <th>563</th>
          <td>26503</td>
          <td>2013-10-17</td>
          <td>6579000</td>
        </tr>
        <tr>
          <th>547</th>
          <td>26503</td>
          <td>2013-07-19</td>
          <td>6572000</td>
        </tr>
        <tr>
          <th>49</th>
          <td>7543</td>
          <td>2007-08-03</td>
          <td>6522637</td>
        </tr>
        <tr>
          <th>362</th>
          <td>26503</td>
          <td>2011-01-21</td>
          <td>6483000</td>
        </tr>
        <tr>
          <th>324</th>
          <td>26503</td>
          <td>2010-07-22</td>
          <td>6454000</td>
        </tr>
        <tr>
          <th>576</th>
          <td>7543</td>
          <td>2013-11-06</td>
          <td>6282166</td>
        </tr>
        <tr>
          <th>417</th>
          <td>26503</td>
          <td>2011-10-20</td>
          <td>6269000</td>
        </tr>
        <tr>
          <th>559</th>
          <td>7543</td>
          <td>2013-08-02</td>
          <td>6255319</td>
        </tr>
        <tr>
          <th>155</th>
          <td>7543</td>
          <td>2008-08-07</td>
          <td>6220000</td>
        </tr>
        <tr>
          <th>457</th>
          <td>26503</td>
          <td>2012-04-24</td>
          <td>6184000</td>
        </tr>
        <tr>
          <th>283</th>
          <td>26503</td>
          <td>2010-01-20</td>
          <td>6082000</td>
        </tr>
        <tr>
          <th>398</th>
          <td>26503</td>
          <td>2011-07-21</td>
          <td>6047000</td>
        </tr>
        <tr>
          <th>182</th>
          <td>7543</td>
          <td>2008-11-06</td>
          <td>5975275</td>
        </tr>
        <tr>
          <th>258</th>
          <td>26503</td>
          <td>2009-10-15</td>
          <td>5974000</td>
        </tr>
        <tr>
          <th>310</th>
          <td>26503</td>
          <td>2010-04-22</td>
          <td>5876000</td>
        </tr>
        <tr>
          <th>485</th>
          <td>7543</td>
          <td>2012-08-03</td>
          <td>5501573</td>
        </tr>
        <tr>
          <th>503</th>
          <td>7543</td>
          <td>2012-11-05</td>
          <td>5406781</td>
        </tr>
        <tr>
          <th>381</th>
          <td>26503</td>
          <td>2011-04-18</td>
          <td>5366000</td>
        </tr>
        <tr>
          <th>519</th>
          <td>7543</td>
          <td>2013-02-05</td>
          <td>5318752</td>
        </tr>
        <tr>
          <th>301</th>
          <td>7543</td>
          <td>2010-02-04</td>
          <td>5292890</td>
        </tr>
        <tr>
          <th>231</th>
          <td>26503</td>
          <td>2009-07-16</td>
          <td>4891000</td>
        </tr>
        <tr>
          <th>...</th>
          <td>...</td>
          <td>...</td>
          <td>...</td>
        </tr>
        <tr>
          <th>21</th>
          <td>3149</td>
          <td>2007-07-13</td>
          <td>42316</td>
        </tr>
        <tr>
          <th>104</th>
          <td>3149</td>
          <td>2008-04-11</td>
          <td>42243</td>
        </tr>
        <tr>
          <th>13</th>
          <td>24074</td>
          <td>2007-04-26</td>
          <td>42156</td>
        </tr>
        <tr>
          <th>625</th>
          <td>24</td>
          <td>2014-10-20</td>
          <td>42123</td>
        </tr>
        <tr>
          <th>629</th>
          <td>23052</td>
          <td>2014-10-28</td>
          <td>42114</td>
        </tr>
        <tr>
          <th>642</th>
          <td>3149</td>
          <td>2015-01-23</td>
          <td>42004</td>
        </tr>
        <tr>
          <th>169</th>
          <td>18711</td>
          <td>2008-10-28</td>
          <td>41723</td>
        </tr>
        <tr>
          <th>391</th>
          <td>7538</td>
          <td>2011-04-29</td>
          <td>41602</td>
        </tr>
        <tr>
          <th>415</th>
          <td>22899</td>
          <td>2011-10-19</td>
          <td>41562</td>
        </tr>
        <tr>
          <th>423</th>
          <td>7538</td>
          <td>2011-10-28</td>
          <td>41525</td>
        </tr>
        <tr>
          <th>135</th>
          <td>2673</td>
          <td>2008-07-24</td>
          <td>41500</td>
        </tr>
        <tr>
          <th>286</th>
          <td>3149</td>
          <td>2010-01-22</td>
          <td>41438</td>
        </tr>
        <tr>
          <th>361</th>
          <td>3149</td>
          <td>2011-01-21</td>
          <td>41377</td>
        </tr>
        <tr>
          <th>266</th>
          <td>23998</td>
          <td>2009-10-28</td>
          <td>41305</td>
        </tr>
        <tr>
          <th>605</th>
          <td>42788</td>
          <td>2014-04-30</td>
          <td>41099</td>
        </tr>
        <tr>
          <th>76</th>
          <td>2673</td>
          <td>2007-11-08</td>
          <td>41078</td>
        </tr>
        <tr>
          <th>499</th>
          <td>1091</td>
          <td>2012-11-02</td>
          <td>41050</td>
        </tr>
        <tr>
          <th>500</th>
          <td>11100</td>
          <td>2012-11-02</td>
          <td>41050</td>
        </tr>
        <tr>
          <th>633</th>
          <td>42788</td>
          <td>2014-10-29</td>
          <td>41048</td>
        </tr>
        <tr>
          <th>645</th>
          <td>23052</td>
          <td>2015-01-29</td>
          <td>40959</td>
        </tr>
        <tr>
          <th>380</th>
          <td>22899</td>
          <td>2011-04-18</td>
          <td>40952</td>
        </tr>
        <tr>
          <th>139</th>
          <td>18711</td>
          <td>2008-07-28</td>
          <td>40569</td>
        </tr>
        <tr>
          <th>591</th>
          <td>40430</td>
          <td>2014-02-06</td>
          <td>40485</td>
        </tr>
        <tr>
          <th>408</th>
          <td>7538</td>
          <td>2011-07-29</td>
          <td>40465</td>
        </tr>
        <tr>
          <th>580</th>
          <td>3149</td>
          <td>2014-01-17</td>
          <td>40382</td>
        </tr>
        <tr>
          <th>673</th>
          <td>23112</td>
          <td>2015-07-31</td>
          <td>40357</td>
        </tr>
        <tr>
          <th>670</th>
          <td>23052</td>
          <td>2015-07-28</td>
          <td>40277</td>
        </tr>
        <tr>
          <th>247</th>
          <td>23112</td>
          <td>2009-07-31</td>
          <td>40205</td>
        </tr>
        <tr>
          <th>9</th>
          <td>3149</td>
          <td>2007-04-13</td>
          <td>40195</td>
        </tr>
        <tr>
          <th>377</th>
          <td>7538</td>
          <td>2011-02-11</td>
          <td>40157</td>
        </tr>
      </tbody>
    </table>
    <p>670 rows × 3 columns</p>
    </div>



