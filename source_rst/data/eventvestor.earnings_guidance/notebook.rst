EventVestor: Earnings Guidance
==============================

In this notebook, we’ll take a look at EventVestor’s *Earnings Guidance*
dataset, available on the `Quantopian
Store <https://www.quantopian.com/store>`__. This dataset spans January
01, 2007 through the current day, and documents forward looking earnings
guidance provided by companies.

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
    from quantopian.interactive.data.eventvestor import earnings_guidance
    # or if you want to import the free dataset, use:
    # from quantopian.interactive.data.eventvestor import earnings_guidance_free
    
    # import data operations
    from odo import odo
    # import other libraries we will use
    import pandas as pd

.. code:: ipython2

    # Let's use blaze to understand the data a bit using Blaze dshape()
    earnings_guidance.dshape




.. parsed-literal::

    dshape("""var * {
      event_id: ?float64,
      asof_date: datetime,
      trade_date: ?datetime,
      symbol: ?string,
      event_type: ?string,
      event_headline: ?string,
      event_phase: ?string,
      guidance_content: ?string,
      guidance_gaap: ?string,
      guidance_trend: ?string,
      guidance_quality: ?string,
      fiscal_quarter: ?string,
      eps_low: ?float64,
      eps_high: ?float64,
      revenue_low: ?float64,
      revenue_high: ?float64,
      netincome_low: ?float64,
      netincome_high: ?float64,
      fiscal_year: ?string,
      annual_trend: ?string,
      event_rating: ?float64,
      timestamp: datetime,
      sid: ?int64
      }""")



.. code:: ipython2

    # And how many rows are there?
    # N.B. we're using a Blaze function to do this, not len()
    earnings_guidance.count()




.. raw:: html

    97769



.. code:: ipython2

    # Let's see what the data looks like. We'll grab the first three rows.
    earnings_guidance[:3]




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
          <th>guidance_content</th>
          <th>guidance_gaap</th>
          <th>guidance_trend</th>
          <th>guidance_quality</th>
          <th>fiscal_quarter</th>
          <th>eps_low</th>
          <th>eps_high</th>
          <th>revenue_low</th>
          <th>revenue_high</th>
          <th>netincome_low</th>
          <th>netincome_high</th>
          <th>fiscal_year</th>
          <th>annual_trend</th>
          <th>event_rating</th>
          <th>timestamp</th>
          <th>sid</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>933903</td>
          <td>2007-01-02</td>
          <td>2007-01-02</td>
          <td>INOD</td>
          <td>Guidance</td>
          <td>Numerex Raises 4Q and FY 06 Guidance</td>
          <td>NaN</td>
          <td>Other Financial</td>
          <td>GAAP</td>
          <td>Higher</td>
          <td>Open Ended</td>
          <td>4Q 06</td>
          <td>0.00</td>
          <td>0.00</td>
          <td>0</td>
          <td>0</td>
          <td>0</td>
          <td>0</td>
          <td>FY 06</td>
          <td>Higher</td>
          <td>1</td>
          <td>2007-01-03</td>
          <td>9581</td>
        </tr>
        <tr>
          <th>1</th>
          <td>138379</td>
          <td>2007-01-02</td>
          <td>2007-01-03</td>
          <td>SLG</td>
          <td>Guidance</td>
          <td>SL Green Realty Issues FY 07 FFO Guidance</td>
          <td>NaN</td>
          <td>Other Financial</td>
          <td>Non-GAAP</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>0.00</td>
          <td>0.00</td>
          <td>0</td>
          <td>0</td>
          <td>0</td>
          <td>0</td>
          <td>FY 07</td>
          <td>New</td>
          <td>1</td>
          <td>2007-01-03</td>
          <td>17448</td>
        </tr>
        <tr>
          <th>2</th>
          <td>137809</td>
          <td>2007-01-03</td>
          <td>2007-01-03</td>
          <td>AEO</td>
          <td>Guidance</td>
          <td>American Eagle Raises 4Q 06 EPS Guidance</td>
          <td>NaN</td>
          <td>EPS</td>
          <td>GAAP</td>
          <td>Higher</td>
          <td>Range</td>
          <td>4Q 06</td>
          <td>0.64</td>
          <td>0.65</td>
          <td>0</td>
          <td>0</td>
          <td>0</td>
          <td>0</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>1</td>
          <td>2007-01-04</td>
          <td>11086</td>
        </tr>
      </tbody>
    </table>



Let’s go over the columns: - **event_id**: the unique identifier for
this event. - **asof_date**: EventVestor’s timestamp of event capture. -
**trade_date**: for event announcements made before trading ends,
trade_date is the same as event_date. For announcements issued after
market close, trade_date is next market open day. - **symbol**: stock
ticker symbol of the affected company. - **event_type**: this should
always be *Guidance*. - **event_headline**: a brief description of the
event - **event_phase**: the inclusion of this field is likely an error
on the part of the data vendor. We’re currently attempting to resolve
this. - **guidance_content**: values include *EPS, EPS & Financial,
Operational, Other Financial* - **guidance_gaap**: values include *GAAP,
Non-GAAP* - **guidance_trend**: values include *Higher, Lower, Narrows,
New, Reiterate, Withdrawal* - **guidance_quality**: values include *Open
Ended, Point, Range* - **fiscal_quarter**: fiscal quarter for which
guidance is provided - **eps_low**: low end of the quarterly EPS
guidance - **eps_high**: high end of the quarterly EPS guidance -
**revenue_low**: low end of the quarterly revenue guidance -
**revenue_high**: high end of the quarterly revenue guidance -
**netincome_low**: low end of the quarterly net income guidance -
**netincome_high**: high end of the quarterly net income guidance -
**fiscal_year**: fiscal year for which the quarterly guidance is
provided - **annual_trend**: the annual guidance trend. Values include
*Higher, Lower, Narrows, New, Reiterate, Withdrawal* - **event_rating**:
this is always 1. The meaning of this is uncertain. - **timestamp**:
this is our timestamp on when we registered the data. - **sid**: the
equity’s unique identifier. Use this instead of the symbol.

We’ve done much of the data processing for you. Fields like
``timestamp`` and ``sid`` are standardized across all our Store
Datasets, so the datasets are easy to combine. We have standardized the
``sid`` across all our equity databases.

We can select columns and rows with ease. Below, we’ll fetch all of
Apple’s entries from 2012.

.. code:: ipython2

    # get apple's sid first
    aapl_sid = symbols('AAPL').sid
    aapl_earnings = earnings_guidance[('2011-12-31' < earnings_guidance['asof_date']) & (earnings_guidance['asof_date'] <'2013-01-01') & (earnings_guidance.sid==aapl_sid)]
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
          <th>guidance_content</th>
          <th>guidance_gaap</th>
          <th>guidance_trend</th>
          <th>guidance_quality</th>
          <th>fiscal_quarter</th>
          <th>eps_low</th>
          <th>eps_high</th>
          <th>revenue_low</th>
          <th>revenue_high</th>
          <th>netincome_low</th>
          <th>netincome_high</th>
          <th>fiscal_year</th>
          <th>annual_trend</th>
          <th>event_rating</th>
          <th>timestamp</th>
          <th>sid</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>1385926</td>
          <td>2012-01-24</td>
          <td>2012-01-25</td>
          <td>AAPL</td>
          <td>Guidance</td>
          <td>Apple Issues 2Q 12 Guidance</td>
          <td>NaN</td>
          <td>EPS &amp; Financial</td>
          <td>GAAP</td>
          <td>New</td>
          <td>Point</td>
          <td>2Q 12</td>
          <td>8.50</td>
          <td>8.50</td>
          <td>3250</td>
          <td>3250</td>
          <td>0</td>
          <td>0</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>1</td>
          <td>2012-01-25</td>
          <td>24</td>
        </tr>
        <tr>
          <th>1</th>
          <td>1421102</td>
          <td>2012-04-24</td>
          <td>2012-04-25</td>
          <td>AAPL</td>
          <td>Guidance</td>
          <td>Apple Issues 3Q 12 Guidance</td>
          <td>NaN</td>
          <td>EPS &amp; Financial</td>
          <td>GAAP</td>
          <td>New</td>
          <td>Point</td>
          <td>3Q 12</td>
          <td>8.68</td>
          <td>8.68</td>
          <td>34000</td>
          <td>34000</td>
          <td>0</td>
          <td>0</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>1</td>
          <td>2012-04-25</td>
          <td>24</td>
        </tr>
        <tr>
          <th>2</th>
          <td>1456501</td>
          <td>2012-07-24</td>
          <td>2012-07-25</td>
          <td>AAPL</td>
          <td>Guidance</td>
          <td>Apple Issues 4Q 12 Guidance</td>
          <td>NaN</td>
          <td>EPS &amp; Financial</td>
          <td>GAAP</td>
          <td>New</td>
          <td>Point</td>
          <td>4Q 12</td>
          <td>7.65</td>
          <td>7.65</td>
          <td>34000</td>
          <td>34000</td>
          <td>0</td>
          <td>0</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>1</td>
          <td>2012-07-25</td>
          <td>24</td>
        </tr>
        <tr>
          <th>3</th>
          <td>1496798</td>
          <td>2012-10-25</td>
          <td>2012-10-26</td>
          <td>AAPL</td>
          <td>Guidance</td>
          <td>Apple Issues 1Q 13 Guidance</td>
          <td>NaN</td>
          <td>EPS &amp; Financial</td>
          <td>GAAP</td>
          <td>New</td>
          <td>Point</td>
          <td>1Q 13</td>
          <td>11.75</td>
          <td>11.75</td>
          <td>5200</td>
          <td>5200</td>
          <td>0</td>
          <td>0</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>1</td>
          <td>2012-10-26</td>
          <td>24</td>
        </tr>
      </tbody>
    </table>



Finally, suppose we want a DataFrame of all earnings guidances releases
in 2012 in which revenue_low and revenue_high differ. Then we’ll compute
by how much they differ!

.. code:: ipython2

    # manipulate with Blaze first:
    twentytwelve = earnings_guidance[(earnings_guidance['asof_date'] < '2012-12-31')&('2012-01-01' <= earnings_guidance['asof_date'])]
    # now that we've got a much smaller object (len: ~39000 rows), we can convert it to a pandas DataFrame
    df = odo(twentytwelve, pd.DataFrame)
    df = df[df.revenue_low != df.revenue_high]
    df['revenue_difference'] = df.revenue_high - df.revenue_low
    df.sort('revenue_difference', ascending=False, inplace=True)
    df.index = range(len(df))
    # When printed: pandas DataFrames display the head(30) and tail(30) rows, and truncate the middle.
    df




.. raw:: html

    <div style="max-height:1000px;max-width:1500px;overflow:auto;">
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
          <th>guidance_content</th>
          <th>guidance_gaap</th>
          <th>guidance_trend</th>
          <th>...</th>
          <th>revenue_low</th>
          <th>revenue_high</th>
          <th>netincome_low</th>
          <th>netincome_high</th>
          <th>fiscal_year</th>
          <th>annual_trend</th>
          <th>event_rating</th>
          <th>timestamp</th>
          <th>sid</th>
          <th>revenue_difference</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>1930090</td>
          <td>2012-07-26</td>
          <td>2012-07-26</td>
          <td>ATE</td>
          <td>Guidance</td>
          <td>Advantest Corp. Issues 2Q 12 &amp; Narrows FY 12 O...</td>
          <td>NaN</td>
          <td>Other Financial</td>
          <td>GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>72000.00</td>
          <td>77000.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>FY 12</td>
          <td>Narrows</td>
          <td>1</td>
          <td>2012-07-27</td>
          <td>23052</td>
          <td>5000.00</td>
        </tr>
        <tr>
          <th>1</th>
          <td>1496843</td>
          <td>2012-10-25</td>
          <td>2012-10-26</td>
          <td>AMZN</td>
          <td>Guidance</td>
          <td>Amazon.com Issues 4Q 12 Guidance</td>
          <td>NaN</td>
          <td>Other Financial</td>
          <td>GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>20250.00</td>
          <td>22750.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>1</td>
          <td>2012-10-26</td>
          <td>16841</td>
          <td>2500.00</td>
        </tr>
        <tr>
          <th>2</th>
          <td>1496029</td>
          <td>2012-10-25</td>
          <td>2012-10-25</td>
          <td>TSM</td>
          <td>Guidance</td>
          <td>Taiwan Semiconductor Issues 4Q 12 Guidance</td>
          <td>NaN</td>
          <td>Other Financial</td>
          <td>GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>129000.00</td>
          <td>131000.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>1</td>
          <td>2012-10-26</td>
          <td>17773</td>
          <td>2000.00</td>
        </tr>
        <tr>
          <th>3</th>
          <td>1490534</td>
          <td>2012-09-10</td>
          <td>2012-09-10</td>
          <td>TSM</td>
          <td>Guidance</td>
          <td>Taiwan Semiconductor Raises 3Q 12 Guidance</td>
          <td>NaN</td>
          <td>Other Financial</td>
          <td>GAAP</td>
          <td>Higher</td>
          <td>...</td>
          <td>136000.00</td>
          <td>138000.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>1</td>
          <td>2012-09-11</td>
          <td>17773</td>
          <td>2000.00</td>
        </tr>
        <tr>
          <th>4</th>
          <td>1454839</td>
          <td>2012-07-19</td>
          <td>2012-07-19</td>
          <td>TSM</td>
          <td>Guidance</td>
          <td>Taiwan Semiconductor Issues 3Q 12 Guidance</td>
          <td>NaN</td>
          <td>Other Financial</td>
          <td>GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>136000.00</td>
          <td>138000.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>1</td>
          <td>2012-07-20</td>
          <td>17773</td>
          <td>2000.00</td>
        </tr>
        <tr>
          <th>5</th>
          <td>1384154</td>
          <td>2012-01-18</td>
          <td>2012-01-18</td>
          <td>TSM</td>
          <td>Guidance</td>
          <td>Taiwan Semiconductor Issues 1Q &amp; FY 12 Guidance</td>
          <td>NaN</td>
          <td>Other Financial</td>
          <td>GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>103000.00</td>
          <td>105000.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>FY 12</td>
          <td>New</td>
          <td>1</td>
          <td>2012-01-19</td>
          <td>17773</td>
          <td>2000.00</td>
        </tr>
        <tr>
          <th>6</th>
          <td>1423149</td>
          <td>2012-04-26</td>
          <td>2012-04-26</td>
          <td>TSM</td>
          <td>Guidance</td>
          <td>Taiwan Semiconductor Issues 2Q &amp; Revises FY 12...</td>
          <td>NaN</td>
          <td>Operational</td>
          <td>NaN</td>
          <td>New</td>
          <td>...</td>
          <td>126000.00</td>
          <td>128000.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>FY 12</td>
          <td>NaN</td>
          <td>1</td>
          <td>2012-04-27</td>
          <td>17773</td>
          <td>2000.00</td>
        </tr>
        <tr>
          <th>7</th>
          <td>1419797</td>
          <td>2012-04-20</td>
          <td>2012-04-20</td>
          <td>INTU</td>
          <td>Guidance</td>
          <td>Intuit Reaffirms 3Q &amp; FY 12 Earnings Guidance</td>
          <td>NaN</td>
          <td>EPS &amp; Financial</td>
          <td>GAAP</td>
          <td>Reiterate</td>
          <td>...</td>
          <td>0.00</td>
          <td>1950.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>FY 12</td>
          <td>Reiterate</td>
          <td>1</td>
          <td>2012-04-21</td>
          <td>8655</td>
          <td>1950.00</td>
        </tr>
        <tr>
          <th>8</th>
          <td>1388974</td>
          <td>2012-01-31</td>
          <td>2012-02-01</td>
          <td>AMZN</td>
          <td>Guidance</td>
          <td>Amazon Issues 1Q 12 Guidance</td>
          <td>NaN</td>
          <td>Other Financial</td>
          <td>GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>12000.00</td>
          <td>13400.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>1</td>
          <td>2012-02-01</td>
          <td>16841</td>
          <td>1400.00</td>
        </tr>
        <tr>
          <th>9</th>
          <td>1422959</td>
          <td>2012-04-26</td>
          <td>2012-04-27</td>
          <td>AMZN</td>
          <td>Guidance</td>
          <td>Amazon.com Issues 2Q 12 Guidance</td>
          <td>NaN</td>
          <td>Other Financial</td>
          <td>GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>11900.00</td>
          <td>13300.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>1</td>
          <td>2012-04-27</td>
          <td>16841</td>
          <td>1400.00</td>
        </tr>
        <tr>
          <th>10</th>
          <td>1491411</td>
          <td>2012-10-16</td>
          <td>2012-10-17</td>
          <td>INTC</td>
          <td>Guidance</td>
          <td>Intel Corp. Issues 4Q and Revises FY 12 Guidance</td>
          <td>NaN</td>
          <td>Other Financial</td>
          <td>Non-GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>13100.00</td>
          <td>14100.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>FY 12</td>
          <td>NaN</td>
          <td>1</td>
          <td>2012-10-17</td>
          <td>3951</td>
          <td>1000.00</td>
        </tr>
        <tr>
          <th>11</th>
          <td>1453377</td>
          <td>2012-07-17</td>
          <td>2012-07-18</td>
          <td>INTC</td>
          <td>Guidance</td>
          <td>Intel Corp. Issues 3Q and Lowers FY 12 Revenue...</td>
          <td>NaN</td>
          <td>Other Financial</td>
          <td>GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>13800.00</td>
          <td>14800.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>FY 12</td>
          <td>Lower</td>
          <td>1</td>
          <td>2012-07-18</td>
          <td>3951</td>
          <td>1000.00</td>
        </tr>
        <tr>
          <th>12</th>
          <td>1384551</td>
          <td>2012-01-19</td>
          <td>2012-01-20</td>
          <td>INTC</td>
          <td>Guidance</td>
          <td>Intel Issues 1Q &amp; FY 12 Outlook</td>
          <td>NaN</td>
          <td>Other Financial</td>
          <td>GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>12300.00</td>
          <td>13300.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>FY 12</td>
          <td>New</td>
          <td>1</td>
          <td>2012-01-20</td>
          <td>3951</td>
          <td>1000.00</td>
        </tr>
        <tr>
          <th>13</th>
          <td>1418488</td>
          <td>2012-04-17</td>
          <td>2012-04-18</td>
          <td>INTC</td>
          <td>Guidance</td>
          <td>Intel Issues 2Q and Revises FY 12 Outlook</td>
          <td>NaN</td>
          <td>Other Financial</td>
          <td>GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>13100.00</td>
          <td>14100.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>FY 12</td>
          <td>NaN</td>
          <td>1</td>
          <td>2012-04-18</td>
          <td>3951</td>
          <td>1000.00</td>
        </tr>
        <tr>
          <th>14</th>
          <td>1495917</td>
          <td>2012-10-25</td>
          <td>2012-10-25</td>
          <td>AVT</td>
          <td>Guidance</td>
          <td>Avnet Issues 2Q 13 Guidance</td>
          <td>NaN</td>
          <td>EPS &amp; Financial</td>
          <td>Non-GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>5950.00</td>
          <td>6650.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>1</td>
          <td>2012-10-26</td>
          <td>661</td>
          <td>700.00</td>
        </tr>
        <tr>
          <th>15</th>
          <td>1387009</td>
          <td>2012-01-26</td>
          <td>2012-01-26</td>
          <td>AVT</td>
          <td>Guidance</td>
          <td>Avnet Issues 3Q 12 Guidance</td>
          <td>NaN</td>
          <td>EPS &amp; Financial</td>
          <td>Non-GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>5950.00</td>
          <td>6550.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>1</td>
          <td>2012-01-27</td>
          <td>661</td>
          <td>600.00</td>
        </tr>
        <tr>
          <th>16</th>
          <td>1467054</td>
          <td>2012-08-08</td>
          <td>2012-08-08</td>
          <td>AVT</td>
          <td>Guidance</td>
          <td>Avnet Issues 1Q 13 Guidance</td>
          <td>NaN</td>
          <td>EPS &amp; Financial</td>
          <td>Non-GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>5800.00</td>
          <td>6400.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>1</td>
          <td>2012-08-09</td>
          <td>661</td>
          <td>600.00</td>
        </tr>
        <tr>
          <th>17</th>
          <td>1422463</td>
          <td>2012-04-26</td>
          <td>2012-04-26</td>
          <td>AVT</td>
          <td>Guidance</td>
          <td>Avnet Issues 4Q 12 Guidance</td>
          <td>NaN</td>
          <td>EPS &amp; Financial</td>
          <td>Non-GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>6300.00</td>
          <td>6900.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>1</td>
          <td>2012-04-27</td>
          <td>661</td>
          <td>600.00</td>
        </tr>
        <tr>
          <th>18</th>
          <td>1459661</td>
          <td>2012-07-30</td>
          <td>2012-07-30</td>
          <td>ARW</td>
          <td>Guidance</td>
          <td>Arrow Electronics Issues 3Q 12 Guidance</td>
          <td>NaN</td>
          <td>EPS &amp; Financial</td>
          <td>Non-GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>4800.00</td>
          <td>5200.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>3Q 12</td>
          <td>NaN</td>
          <td>1</td>
          <td>2012-07-31</td>
          <td>538</td>
          <td>400.00</td>
        </tr>
        <tr>
          <th>19</th>
          <td>1492709</td>
          <td>2012-10-18</td>
          <td>2012-10-19</td>
          <td>FLEX</td>
          <td>Guidance</td>
          <td>Flextronics International Issues 3Q 13 Guidance</td>
          <td>NaN</td>
          <td>EPS &amp; Financial</td>
          <td>GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>5800.00</td>
          <td>6200.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>1</td>
          <td>2012-10-19</td>
          <td>10953</td>
          <td>400.00</td>
        </tr>
        <tr>
          <th>20</th>
          <td>1389393</td>
          <td>2012-02-01</td>
          <td>2012-02-01</td>
          <td>ARW</td>
          <td>Guidance</td>
          <td>Arrow Electronics Issues 1Q 12 Guidance</td>
          <td>NaN</td>
          <td>EPS &amp; Financial</td>
          <td>Non-GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>4670.00</td>
          <td>5070.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>1</td>
          <td>2012-02-02</td>
          <td>538</td>
          <td>400.00</td>
        </tr>
        <tr>
          <th>21</th>
          <td>1424909</td>
          <td>2012-05-01</td>
          <td>2012-05-01</td>
          <td>ARW</td>
          <td>Guidance</td>
          <td>Arrow Electronics Issues 2Q 12 Guidance</td>
          <td>NaN</td>
          <td>EPS &amp; Financial</td>
          <td>Non-GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>5040.00</td>
          <td>5440.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>1</td>
          <td>2012-05-02</td>
          <td>538</td>
          <td>400.00</td>
        </tr>
        <tr>
          <th>22</th>
          <td>1418864</td>
          <td>2012-04-18</td>
          <td>2012-04-19</td>
          <td>QCOM</td>
          <td>Guidance</td>
          <td>Qualcomm Issues 3Q &amp; Raises FY 12 Guidance</td>
          <td>NaN</td>
          <td>EPS &amp; Financial</td>
          <td>GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>4450.00</td>
          <td>4850.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>FY 12</td>
          <td>Higher</td>
          <td>1</td>
          <td>2012-04-19</td>
          <td>6295</td>
          <td>400.00</td>
        </tr>
        <tr>
          <th>23</th>
          <td>1454039</td>
          <td>2012-07-18</td>
          <td>2012-07-19</td>
          <td>QCOM</td>
          <td>Guidance</td>
          <td>Qualcomm Issues 4Q and Lowers FY 12 Guidance</td>
          <td>NaN</td>
          <td>EPS &amp; Financial</td>
          <td>GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>4450.00</td>
          <td>4850.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>FY 12</td>
          <td>Lower</td>
          <td>1</td>
          <td>2012-07-19</td>
          <td>6295</td>
          <td>400.00</td>
        </tr>
        <tr>
          <th>24</th>
          <td>1457395</td>
          <td>2012-07-25</td>
          <td>2012-07-26</td>
          <td>FLEX</td>
          <td>Guidance</td>
          <td>Flextronics International Issues 2Q 13 Guidance</td>
          <td>NaN</td>
          <td>EPS &amp; Financial</td>
          <td>GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>5900.00</td>
          <td>6300.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>1</td>
          <td>2012-07-26</td>
          <td>10953</td>
          <td>400.00</td>
        </tr>
        <tr>
          <th>25</th>
          <td>1425487</td>
          <td>2012-05-01</td>
          <td>2012-05-02</td>
          <td>FLEX</td>
          <td>Guidance</td>
          <td>Flextronics International Issues 1Q 13 Guidance</td>
          <td>NaN</td>
          <td>EPS &amp; Financial</td>
          <td>GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>5900.00</td>
          <td>6300.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>1</td>
          <td>2012-05-02</td>
          <td>10953</td>
          <td>400.00</td>
        </tr>
        <tr>
          <th>26</th>
          <td>1389626</td>
          <td>2012-02-01</td>
          <td>2012-02-02</td>
          <td>QCOM</td>
          <td>Guidance</td>
          <td>Qualcomm Issues 2Q and Raises FY 12 Guidance</td>
          <td>NaN</td>
          <td>EPS &amp; Financial</td>
          <td>GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>4600.00</td>
          <td>5000.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>FY 12</td>
          <td>Higher</td>
          <td>1</td>
          <td>2012-02-02</td>
          <td>6295</td>
          <td>400.00</td>
        </tr>
        <tr>
          <th>27</th>
          <td>1380315</td>
          <td>2012-01-04</td>
          <td>2012-01-05</td>
          <td>STX</td>
          <td>Guidance</td>
          <td>Seagate Technology Raises 3Q 12 Outlook</td>
          <td>NaN</td>
          <td>Other Financial</td>
          <td>Non-GAAP</td>
          <td>Higher</td>
          <td>...</td>
          <td>4200.00</td>
          <td>4500.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>1</td>
          <td>2012-01-05</td>
          <td>24518</td>
          <td>300.00</td>
        </tr>
        <tr>
          <th>28</th>
          <td>1384635</td>
          <td>2012-01-19</td>
          <td>2012-01-20</td>
          <td>FLEX</td>
          <td>Guidance</td>
          <td>Flextronics Issues 4Q 12 Guidance</td>
          <td>NaN</td>
          <td>EPS &amp; Financial</td>
          <td>GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>6300.00</td>
          <td>6600.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>1</td>
          <td>2012-01-20</td>
          <td>10953</td>
          <td>300.00</td>
        </tr>
        <tr>
          <th>29</th>
          <td>1419632</td>
          <td>2012-04-19</td>
          <td>2012-04-19</td>
          <td>CVI</td>
          <td>Guidance</td>
          <td>CVR Energy Issues 1Q 12 Guidance</td>
          <td>NaN</td>
          <td>Other Financial</td>
          <td>GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>1800.00</td>
          <td>2100.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>1</td>
          <td>2012-04-20</td>
          <td>22766</td>
          <td>300.00</td>
        </tr>
        <tr>
          <th>...</th>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
        </tr>
        <tr>
          <th>1599</th>
          <td>1398611</td>
          <td>2012-02-21</td>
          <td>2012-02-22</td>
          <td>ZIXI</td>
          <td>Guidance</td>
          <td>Zix Corp. Issues 1Q and FY 12 Guidance</td>
          <td>NaN</td>
          <td>EPS &amp; Financial</td>
          <td>Non-GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>10.00</td>
          <td>10.10</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>FY 12</td>
          <td>New</td>
          <td>1</td>
          <td>2012-02-22</td>
          <td>20822</td>
          <td>0.10</td>
        </tr>
        <tr>
          <th>1600</th>
          <td>1403341</td>
          <td>2012-02-29</td>
          <td>2012-03-01</td>
          <td>MTZ</td>
          <td>Guidance</td>
          <td>MasTec Issues 1Q and FY 12 Outlook</td>
          <td>NaN</td>
          <td>EPS &amp; Financial</td>
          <td>GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>0.15</td>
          <td>0.17</td>
          <td>12.3</td>
          <td>14.1</td>
          <td>FY 12</td>
          <td>New</td>
          <td>1</td>
          <td>2012-03-01</td>
          <td>4667</td>
          <td>0.02</td>
        </tr>
        <tr>
          <th>1601</th>
          <td>1387298</td>
          <td>2012-01-26</td>
          <td>2012-01-27</td>
          <td>CYS</td>
          <td>Guidance</td>
          <td>CYS Investments Issues 4Q 11 Guidance</td>
          <td>NaN</td>
          <td>EPS</td>
          <td>GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>0.55</td>
          <td>0.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>1</td>
          <td>2012-01-27</td>
          <td>38477</td>
          <td>-0.55</td>
        </tr>
        <tr>
          <th>1602</th>
          <td>1439505</td>
          <td>2012-02-14</td>
          <td>2012-02-14</td>
          <td>KORS</td>
          <td>Guidance</td>
          <td>Michael Kors Holdings Issues 4Q &amp; FY 12 Guidance</td>
          <td>NaN</td>
          <td>EPS &amp; Financial</td>
          <td>GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>350.00</td>
          <td>335.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>FY 12</td>
          <td>New</td>
          <td>1</td>
          <td>2012-02-15</td>
          <td>42270</td>
          <td>-15.00</td>
        </tr>
        <tr>
          <th>1603</th>
          <td>1424617</td>
          <td>2012-04-26</td>
          <td>2012-04-26</td>
          <td>INOD</td>
          <td>Guidance</td>
          <td>Innodata Isogen Issues 2Q 12 Revenue Guidance</td>
          <td>NaN</td>
          <td>Other Financial</td>
          <td>GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>21.00</td>
          <td>0.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>1</td>
          <td>2012-04-27</td>
          <td>9581</td>
          <td>-21.00</td>
        </tr>
        <tr>
          <th>1604</th>
          <td>1382294</td>
          <td>2012-01-11</td>
          <td>2012-01-11</td>
          <td>SAAS</td>
          <td>Guidance</td>
          <td>inContact Issues 4Q 11 Guidance</td>
          <td>NaN</td>
          <td>Other Financial</td>
          <td>GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>23.50</td>
          <td>0.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>1</td>
          <td>2012-01-12</td>
          <td>31137</td>
          <td>-23.50</td>
        </tr>
        <tr>
          <th>1605</th>
          <td>1441324</td>
          <td>2012-06-06</td>
          <td>2012-06-06</td>
          <td>AMSC</td>
          <td>Guidance</td>
          <td>American Superconductor Issues 1Q 12 Guidance</td>
          <td>NaN</td>
          <td>EPS &amp; Financial</td>
          <td>GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>26.00</td>
          <td>0.00</td>
          <td>0.0</td>
          <td>10.0</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>1</td>
          <td>2012-06-07</td>
          <td>393</td>
          <td>-26.00</td>
        </tr>
        <tr>
          <th>1606</th>
          <td>1393252</td>
          <td>2012-02-09</td>
          <td>2012-02-09</td>
          <td>AMSC</td>
          <td>Guidance</td>
          <td>American Superconductor Updates 4Q 11 Outlook</td>
          <td>NaN</td>
          <td>EPS &amp; Financial</td>
          <td>GAAP</td>
          <td>NaN</td>
          <td>...</td>
          <td>27.00</td>
          <td>0.00</td>
          <td>-24.0</td>
          <td>0.0</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>1</td>
          <td>2012-02-10</td>
          <td>393</td>
          <td>-27.00</td>
        </tr>
        <tr>
          <th>1607</th>
          <td>1391621</td>
          <td>2012-02-06</td>
          <td>2012-02-07</td>
          <td>BDE</td>
          <td>Guidance</td>
          <td>Black Diamond Issues 4Q and Raises FY 11 Guidance</td>
          <td>NaN</td>
          <td>EPS &amp; Financial</td>
          <td>GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>36.00</td>
          <td>0.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>FY 11</td>
          <td>Higher</td>
          <td>1</td>
          <td>2012-02-07</td>
          <td>18803</td>
          <td>-36.00</td>
        </tr>
        <tr>
          <th>1608</th>
          <td>1544306</td>
          <td>2012-06-04</td>
          <td>2012-06-04</td>
          <td>CALL</td>
          <td>Guidance</td>
          <td>magicJack VocalTec Issues 2Q and Raises FY 12 ...</td>
          <td>NaN</td>
          <td>EPS &amp; Financial</td>
          <td>GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>36.00</td>
          <td>0.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>FY 12</td>
          <td>Higher</td>
          <td>1</td>
          <td>2012-06-05</td>
          <td>14457</td>
          <td>-36.00</td>
        </tr>
        <tr>
          <th>1609</th>
          <td>1411122</td>
          <td>2012-03-20</td>
          <td>2012-03-21</td>
          <td>FSII</td>
          <td>Guidance</td>
          <td>FSI International Issues 3Q 12 Guidance</td>
          <td>NaN</td>
          <td>Other Financial</td>
          <td>GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>50.00</td>
          <td>0.00</td>
          <td>7.0</td>
          <td>9.0</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>1</td>
          <td>2012-03-21</td>
          <td>3029</td>
          <td>-50.00</td>
        </tr>
        <tr>
          <th>1610</th>
          <td>1412014</td>
          <td>2012-03-23</td>
          <td>2012-03-23</td>
          <td>KITD</td>
          <td>Guidance</td>
          <td>KIT digital Reaffirms 1Q and FY 12 Guidance</td>
          <td>NaN</td>
          <td>EPS &amp; Financial</td>
          <td>Non-GAAP</td>
          <td>Reiterate</td>
          <td>...</td>
          <td>72.00</td>
          <td>0.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>FY 12</td>
          <td>Reiterate</td>
          <td>1</td>
          <td>2012-03-24</td>
          <td>30692</td>
          <td>-72.00</td>
        </tr>
        <tr>
          <th>1611</th>
          <td>1401903</td>
          <td>2012-02-27</td>
          <td>2012-02-28</td>
          <td>KITD</td>
          <td>Guidance</td>
          <td>KIT digital Issues 1Q and Lowers FY 12 Guidance</td>
          <td>NaN</td>
          <td>EPS &amp; Financial</td>
          <td>Non-GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>72.00</td>
          <td>0.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>FY 12</td>
          <td>Lower</td>
          <td>1</td>
          <td>2012-02-28</td>
          <td>30692</td>
          <td>-72.00</td>
        </tr>
        <tr>
          <th>1612</th>
          <td>1409726</td>
          <td>2012-03-15</td>
          <td>2012-03-15</td>
          <td>KITD</td>
          <td>Guidance</td>
          <td>KIT digital Reaffirms 1Q and FY 12 Guidance</td>
          <td>NaN</td>
          <td>EPS &amp; Financial</td>
          <td>Non-GAAP</td>
          <td>Reiterate</td>
          <td>...</td>
          <td>72.00</td>
          <td>0.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>FY 12</td>
          <td>Reiterate</td>
          <td>1</td>
          <td>2012-03-16</td>
          <td>30692</td>
          <td>-72.00</td>
        </tr>
        <tr>
          <th>1613</th>
          <td>1490036</td>
          <td>2012-10-11</td>
          <td>2012-10-12</td>
          <td>ET</td>
          <td>Guidance</td>
          <td>ExactTarget Raises 3Q 12 Guidance</td>
          <td>NaN</td>
          <td>Other Financial</td>
          <td>Non-GAAP</td>
          <td>Higher</td>
          <td>...</td>
          <td>72.00</td>
          <td>0.00</td>
          <td>-3.0</td>
          <td>0.0</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>1</td>
          <td>2012-10-12</td>
          <td>42700</td>
          <td>-72.00</td>
        </tr>
        <tr>
          <th>1614</th>
          <td>1391196</td>
          <td>2012-02-02</td>
          <td>2012-02-02</td>
          <td>FTK</td>
          <td>Guidance</td>
          <td>Flotek Industries Issues 4Q and FY 11 Guidance</td>
          <td>NaN</td>
          <td>Other Financial</td>
          <td>GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>74.50</td>
          <td>0.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>FY 11</td>
          <td>New</td>
          <td>1</td>
          <td>2012-02-03</td>
          <td>27496</td>
          <td>-74.50</td>
        </tr>
        <tr>
          <th>1615</th>
          <td>1417909</td>
          <td>2012-04-16</td>
          <td>2012-04-16</td>
          <td>FTK</td>
          <td>Guidance</td>
          <td>Flotek Industries Raises 1Q 12 Revenues Guidance</td>
          <td>NaN</td>
          <td>Other Financial</td>
          <td>GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>78.00</td>
          <td>0.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>1</td>
          <td>2012-04-17</td>
          <td>27496</td>
          <td>-78.00</td>
        </tr>
        <tr>
          <th>1616</th>
          <td>1437795</td>
          <td>2012-01-24</td>
          <td>2012-01-24</td>
          <td>FIO</td>
          <td>Guidance</td>
          <td>Fusion-io Issues 3Q &amp; Raises FY 12 Guidance</td>
          <td>NaN</td>
          <td>Other Financial</td>
          <td>Non-GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>85.00</td>
          <td>0.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>FY 12</td>
          <td>Higher</td>
          <td>1</td>
          <td>2012-01-25</td>
          <td>41554</td>
          <td>-85.00</td>
        </tr>
        <tr>
          <th>1617</th>
          <td>1443898</td>
          <td>2012-05-14</td>
          <td>2012-05-14</td>
          <td>MPAA</td>
          <td>Guidance</td>
          <td>Motorcar Parts of America Issues 4Q and FY 12 ...</td>
          <td>NaN</td>
          <td>Other Financial</td>
          <td>GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>90.00</td>
          <td>0.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>FY 12</td>
          <td>New</td>
          <td>1</td>
          <td>2012-05-15</td>
          <td>10992</td>
          <td>-90.00</td>
        </tr>
        <tr>
          <th>1618</th>
          <td>1400239</td>
          <td>2012-02-23</td>
          <td>2012-02-24</td>
          <td>WBMD</td>
          <td>Guidance</td>
          <td>WebMD Health Issues 1Q and Updates FY 12 Guidance</td>
          <td>NaN</td>
          <td>Other Financial</td>
          <td>Non-GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>105.00</td>
          <td>0.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>FY 12</td>
          <td>NaN</td>
          <td>1</td>
          <td>2012-02-24</td>
          <td>27669</td>
          <td>-105.00</td>
        </tr>
        <tr>
          <th>1619</th>
          <td>1457505</td>
          <td>2012-07-25</td>
          <td>2012-07-26</td>
          <td>CVD</td>
          <td>Guidance</td>
          <td>Covance Issues 3Q and Lowers FY 12 Guidance</td>
          <td>NaN</td>
          <td>EPS &amp; Financial</td>
          <td>Non-GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>538.00</td>
          <td>0.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>FY 12</td>
          <td>Lower</td>
          <td>1</td>
          <td>2012-07-26</td>
          <td>25396</td>
          <td>-538.00</td>
        </tr>
        <tr>
          <th>1620</th>
          <td>1486588</td>
          <td>2012-01-06</td>
          <td>2012-01-06</td>
          <td>VRX</td>
          <td>Guidance</td>
          <td>Valeant Pharmaceuticals Issues 4Q and Updates ...</td>
          <td>NaN</td>
          <td>EPS &amp; Financial</td>
          <td>Non-GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>650.00</td>
          <td>0.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>FY 11</td>
          <td>NaN</td>
          <td>1</td>
          <td>2012-01-07</td>
          <td>10908</td>
          <td>-650.00</td>
        </tr>
        <tr>
          <th>1621</th>
          <td>1416853</td>
          <td>2012-04-11</td>
          <td>2012-04-11</td>
          <td>VMW</td>
          <td>Guidance</td>
          <td>VMware Raises 1Q 12 Guidance</td>
          <td>NaN</td>
          <td>Other Financial</td>
          <td>GAAP</td>
          <td>Higher</td>
          <td>...</td>
          <td>1040.00</td>
          <td>0.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>1</td>
          <td>2012-04-12</td>
          <td>34545</td>
          <td>-1040.00</td>
        </tr>
        <tr>
          <th>1622</th>
          <td>1392473</td>
          <td>2012-02-08</td>
          <td>2012-02-08</td>
          <td>CTSH</td>
          <td>Guidance</td>
          <td>Cognizant Technology Solutions Issues 1Q and F...</td>
          <td>NaN</td>
          <td>EPS &amp; Financial</td>
          <td>GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>1700.00</td>
          <td>0.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>FY 12</td>
          <td>New</td>
          <td>1</td>
          <td>2012-02-09</td>
          <td>18870</td>
          <td>-1700.00</td>
        </tr>
        <tr>
          <th>1623</th>
          <td>1428889</td>
          <td>2012-05-07</td>
          <td>2012-05-07</td>
          <td>CTSH</td>
          <td>Guidance</td>
          <td>Cognizant Technology Solutions Issues 2Q and L...</td>
          <td>NaN</td>
          <td>EPS &amp; Financial</td>
          <td>GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>1790.00</td>
          <td>0.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>FY 12</td>
          <td>Lower</td>
          <td>1</td>
          <td>2012-05-08</td>
          <td>18870</td>
          <td>-1790.00</td>
        </tr>
        <tr>
          <th>1624</th>
          <td>1464692</td>
          <td>2012-08-06</td>
          <td>2012-08-06</td>
          <td>CTSH</td>
          <td>Guidance</td>
          <td>Cognizant Technology Solutions Issues 3Q and R...</td>
          <td>NaN</td>
          <td>EPS &amp; Financial</td>
          <td>GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>1875.00</td>
          <td>0.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>FY 12</td>
          <td>Higher</td>
          <td>1</td>
          <td>2012-08-07</td>
          <td>18870</td>
          <td>-1875.00</td>
        </tr>
        <tr>
          <th>1625</th>
          <td>1388993</td>
          <td>2012-01-31</td>
          <td>2012-02-01</td>
          <td>STX</td>
          <td>Guidance</td>
          <td>Seagate Technology Raises 3Q 12 Outlook</td>
          <td>NaN</td>
          <td>Other Financial</td>
          <td>GAAP</td>
          <td>Higher</td>
          <td>...</td>
          <td>4300.00</td>
          <td>0.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>1</td>
          <td>2012-02-01</td>
          <td>24518</td>
          <td>-4300.00</td>
        </tr>
        <tr>
          <th>1626</th>
          <td>1422045</td>
          <td>2012-04-26</td>
          <td>2012-04-26</td>
          <td>TYC</td>
          <td>Guidance</td>
          <td>Tyco International Issues 3Q and Raises FY 12 ...</td>
          <td>NaN</td>
          <td>EPS &amp; Financial</td>
          <td>Non-GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>4500.00</td>
          <td>0.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>FY 12</td>
          <td>Higher</td>
          <td>1</td>
          <td>2012-04-27</td>
          <td>7679</td>
          <td>-4500.00</td>
        </tr>
        <tr>
          <th>1627</th>
          <td>1433072</td>
          <td>2012-04-17</td>
          <td>2012-04-17</td>
          <td>STX</td>
          <td>Guidance</td>
          <td>Seagate Technology Updates 4Q 12 and Calendar ...</td>
          <td>NaN</td>
          <td>Other Financial</td>
          <td>Non-GAAP</td>
          <td>NaN</td>
          <td>...</td>
          <td>5000.00</td>
          <td>0.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>1</td>
          <td>2012-04-18</td>
          <td>24518</td>
          <td>-5000.00</td>
        </tr>
        <tr>
          <th>1628</th>
          <td>1389165</td>
          <td>2012-01-31</td>
          <td>2012-02-01</td>
          <td>STX</td>
          <td>Guidance</td>
          <td>Seagate Technology Issues 4Q 12 Outlook</td>
          <td>NaN</td>
          <td>Other Financial</td>
          <td>GAAP</td>
          <td>New</td>
          <td>...</td>
          <td>5000.00</td>
          <td>0.00</td>
          <td>0.0</td>
          <td>0.0</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>1</td>
          <td>2012-02-01</td>
          <td>24518</td>
          <td>-5000.00</td>
        </tr>
      </tbody>
    </table>
    <p>1629 rows × 24 columns</p>
    </div>



