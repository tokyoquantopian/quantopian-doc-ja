EventVestor: CEO Changes
========================

In this notebook, we’ll take a look at EventVestor’s *CEO Changes*
dataset, available on the `Quantopian
Store <https://www.quantopian.com/store>`__. This dataset spans January
01, 2007 through the current day.

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
    from quantopian.interactive.data.eventvestor import ceo_change
    # or if you want to import the free dataset, use:
    # from quantopian.data.eventvestor import ceo_change_free
    
    # import data operations
    from odo import odo
    # import other libraries we will use
    import pandas as pd

.. code:: ipython2

    # Let's use blaze to understand the data a bit using Blaze dshape()
    ceo_change.dshape




.. parsed-literal::

    dshape("""var * {
      event_id: ?float64,
      asof_date: datetime,
      trade_date: ?datetime,
      symbol: ?string,
      event_type: ?string,
      event_headline: ?string,
      change_status: ?string,
      change_scenario: ?string,
      change_type: ?string,
      change_source: ?string,
      change_reason: ?string,
      in_ceoname: ?string,
      in_ceogender: ?string,
      out_ceoname: ?string,
      out_ceogender: ?string,
      effective_date: ?datetime,
      event_rating: ?float64,
      timestamp: datetime,
      sid: ?int64
      }""")



.. code:: ipython2

    # And how many rows are there?
    # N.B. we're using a Blaze function to do this, not len()
    ceo_change.count()




.. raw:: html

    4324



.. code:: ipython2

    # Let's see what the data looks like. We'll grab the first three rows.
    ceo_change[:3]




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
          <th>change_status</th>
          <th>change_scenario</th>
          <th>change_type</th>
          <th>change_source</th>
          <th>change_reason</th>
          <th>in_ceoname</th>
          <th>in_ceogender</th>
          <th>out_ceoname</th>
          <th>out_ceogender</th>
          <th>effective_date</th>
          <th>event_rating</th>
          <th>timestamp</th>
          <th>sid</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>134628</td>
          <td>2007-01-03</td>
          <td>2007-01-03</td>
          <td>HD</td>
          <td>CEO Change</td>
          <td>Home Depot CEO Steps Down</td>
          <td>Declaration</td>
          <td>In/Out</td>
          <td>Permanent</td>
          <td>Succession</td>
          <td>Resign</td>
          <td>Frank Blake,</td>
          <td>Male</td>
          <td>Robert Nardelli</td>
          <td>Male</td>
          <td>2007-01-02</td>
          <td>1</td>
          <td>2007-01-04</td>
          <td>3496</td>
        </tr>
        <tr>
          <th>1</th>
          <td>1133605</td>
          <td>2007-01-04</td>
          <td>2007-01-04</td>
          <td>RAIL</td>
          <td>CEO Change</td>
          <td>FreightCar America CEO John E. Carroll to Reti...</td>
          <td>Proposal</td>
          <td>In/Out</td>
          <td>Permanent</td>
          <td>Outsider</td>
          <td>Resign</td>
          <td>Christian Ragot</td>
          <td>Male</td>
          <td>John E. Carroll, Jr.</td>
          <td>Male</td>
          <td>2007-04-30</td>
          <td>1</td>
          <td>2007-01-05</td>
          <td>27161</td>
        </tr>
        <tr>
          <th>2</th>
          <td>950064</td>
          <td>2007-01-04</td>
          <td>2007-01-04</td>
          <td>VIRL</td>
          <td>CEO Change</td>
          <td>Virage Logic CEO Adam Kablanian Resigns; Appoi...</td>
          <td>Declaration</td>
          <td>In/Out</td>
          <td>Permanent</td>
          <td>Succession</td>
          <td>Out + Retained</td>
          <td>Dan McCranie</td>
          <td>Male</td>
          <td>Adam Kablanian</td>
          <td>Male</td>
          <td>2007-01-04</td>
          <td>1</td>
          <td>2007-01-05</td>
          <td>21957</td>
        </tr>
      </tbody>
    </table>



Let’s go over the columns: - **event_id**: the unique identifier for
this CEO Change. - **asof_date**: EventVestor’s timestamp of event
capture. - **trade_date**: for event announcements made before trading
ends, trade_date is the same as event_date. For announcements issued
after market close, trade_date is next market open day. - **symbol**:
stock ticker symbol of the affected company. - **event_type**: this
should always be *CEO Change*. - **event_headline**: a short description
of the event. - **change_status**: indicates whether the change is a
proposal or a confirmation. - **change_scenario**: indicates if the CEO
Change is *in*, *out*, or both. - **change_type**: indicates if the
incoming CEO is interim or permanent. - **change_source**: is the
incoming CEO an internal candidate, or recruited from the outside? -
**change_reason**: reason for the CEO transition - **in_ceoname**: name
of the incoming CEO - **in_ceoname**: gender of the incoming CEO -
**out_ceoname**: name of the outgoing CEO - **out_ceogender**: gender of
the outgoing CEO - **effective_date**: date as of which the CEO change
is effective. - **event_rating**: this is always 1. The meaning of this
is uncertain. - **timestamp**: this is our timestamp on when we
registered the data. - **sid**: the equity’s unique identifier. Use this
instead of the symbol.

We’ve done much of the data processing for you. Fields like
``timestamp`` and ``sid`` are standardized across all our Store
Datasets, so the datasets are easy to combine. We have standardized the
``sid`` across all our equity databases.

We can select columns and rows with ease. Below, we’ll fetch all entries
for Microsoft. We’re really only interested in the CEO coming in, the
CEO going out, and the date, so we’ll display only those columns.

.. code:: ipython2

    # get the sid for MSFT
    symbols('MSFT')




.. parsed-literal::

    Equity(5061, symbol=u'MSFT', asset_name=u'MICROSOFT CORP', exchange=u'NASDAQ GLOBAL SELECT MARKET', start_date=u'Mon, 04 Jan 1993 00:00:00 GMT', end_date=u'Tue, 29 Sep 2015 00:00:00 GMT', first_traded=None)



.. code:: ipython2

    # knowing that the MSFT sid is 5061:
    msft = ceo_change[ceo_change.sid==5061][['timestamp','in_ceoname', 'out_ceoname','change_status']].sort('timestamp')
    msft




.. raw:: html

    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>timestamp</th>
          <th>in_ceoname</th>
          <th>out_ceoname</th>
          <th>change_status</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>2013-08-24</td>
          <td>NaN</td>
          <td>Steve Ballmer</td>
          <td>Declaration</td>
        </tr>
        <tr>
          <th>1</th>
          <td>2014-02-05</td>
          <td>Satya Nadella</td>
          <td>NaN</td>
          <td>Declaration</td>
        </tr>
      </tbody>
    </table>



Note that the ``in_ceoname`` and ``out_ceoname`` in these cases were
NaNs because there was a long transition period. Steve Ballmer announced
his resignation on 2013-08-24, and formally stepped down on 2014-02-05.

Let’s try another one:

.. code:: ipython2

    # get the sid for AMD
    sid_amd = symbols('AMD').sid
    amd = ceo_change[ceo_change.sid==sid_amd][['timestamp','in_ceoname', 'out_ceoname','change_status']].sort('timestamp')
    amd




.. raw:: html

    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>timestamp</th>
          <th>in_ceoname</th>
          <th>out_ceoname</th>
          <th>change_status</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>2008-07-18</td>
          <td>Dirk Meyer</td>
          <td>Hector Ruiz</td>
          <td>Declaration</td>
        </tr>
        <tr>
          <th>1</th>
          <td>2011-01-11</td>
          <td>Thomas Seifert</td>
          <td>Dirk Meyer</td>
          <td>Declaration</td>
        </tr>
        <tr>
          <th>2</th>
          <td>2011-08-26</td>
          <td>Rory P. Read</td>
          <td>NaN</td>
          <td>Declaration</td>
        </tr>
        <tr>
          <th>3</th>
          <td>2014-10-09</td>
          <td>Lisa Su</td>
          <td>Rory Read</td>
          <td>Declaration</td>
        </tr>
      </tbody>
    </table>



Now suppose want to know how many CEO changes there were in the past
year in which a female CEO was incoming.

.. code:: ipython2

    females_in = ceo_change[ceo_change['in_ceogender']=='Female']
    # Note that whenever you print a Blaze Data Object here, it will be automatically truncated to ten rows.
    females_in = females_in[females_in.asof_date > '2014-09-17']
    len(females_in)




.. parsed-literal::

    27



Finally, suppose want this as a DataFrame:

.. code:: ipython2

    females_in_df = odo(females_in, pd.DataFrame)
    females_in_df.sort('symbol', inplace=True)
    # let's get the first three rows
    females_in_df[:3]




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
          <th>change_status</th>
          <th>change_scenario</th>
          <th>change_type</th>
          <th>change_source</th>
          <th>change_reason</th>
          <th>in_ceoname</th>
          <th>in_ceogender</th>
          <th>out_ceoname</th>
          <th>out_ceogender</th>
          <th>effective_date</th>
          <th>event_rating</th>
          <th>timestamp</th>
          <th>sid</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>17</th>
          <td>1890286</td>
          <td>2015-06-01</td>
          <td>2015-06-01</td>
          <td>AJRD</td>
          <td>CEO Change</td>
          <td>Aerojet Rocketdynes Holdings CEO Scott Seymour...</td>
          <td>Declaration</td>
          <td>In/Out</td>
          <td>Permanent</td>
          <td>Succession</td>
          <td>Retire</td>
          <td>Eileen Drake</td>
          <td>Female</td>
          <td>Scott Seymour</td>
          <td>Male</td>
          <td>NaT</td>
          <td>1</td>
          <td>2015-06-02</td>
          <td>3424</td>
        </tr>
        <tr>
          <th>2</th>
          <td>1783327</td>
          <td>2014-10-08</td>
          <td>2014-10-09</td>
          <td>AMD</td>
          <td>CEO Change</td>
          <td>Advanced Micro Devices CEO Rory Read Steps Dow...</td>
          <td>Declaration</td>
          <td>In/Out</td>
          <td>Permanent</td>
          <td>Succession</td>
          <td>Out + Retained</td>
          <td>Lisa Su</td>
          <td>Female</td>
          <td>Rory Read</td>
          <td>Male</td>
          <td>2014-10-08</td>
          <td>1</td>
          <td>2014-10-09</td>
          <td>351</td>
        </tr>
        <tr>
          <th>15</th>
          <td>1846426</td>
          <td>2015-03-04</td>
          <td>2015-03-05</td>
          <td>AMSF</td>
          <td>CEO Change</td>
          <td>AMERISAFE Promotes COO, G. Janelle Frost to CE...</td>
          <td>Declaration</td>
          <td>In/Out</td>
          <td>Permanent</td>
          <td>Succession</td>
          <td>Out + Retained</td>
          <td>G. Janelle Frost</td>
          <td>Female</td>
          <td>C. Allen Bradley Jr.</td>
          <td>Male</td>
          <td>2015-04-01</td>
          <td>1</td>
          <td>2015-03-05</td>
          <td>27819</td>
        </tr>
      </tbody>
    </table>
    </div>



