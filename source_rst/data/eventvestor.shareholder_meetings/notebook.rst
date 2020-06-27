EventVestor: Shareholder Meetings
=================================

In this notebook, we’ll take a look at EventVestor’s *Shareholder
Meetings* dataset, available on the `Quantopian
Store <https://www.quantopian.com/store>`__. This dataset spans January
01, 2007 through the current day, and documents companies’ annual and
special shareholder meetings calendars.

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
    from quantopian.interactive.data.eventvestor import shareholder_meetings
    # or if you want to import the free dataset, use:
    # from quantopian.data.eventvestor import shareholder_meetings_free
    
    # import data operations
    from odo import odo
    # import other libraries we will use
    import pandas as pd

.. code:: ipython2

    # Let's use blaze to understand the data a bit using Blaze dshape()
    shareholder_meetings.dshape




.. parsed-literal::

    dshape("""var * {
      event_id: ?float64,
      asof_date: datetime,
      symbol: ?string,
      event_headline: ?string,
      meeting_type: ?string,
      record_date: ?datetime,
      meeting_date: ?datetime,
      timestamp: datetime,
      sid: ?int64
      }""")



.. code:: ipython2

    # And how many rows are there?
    # N.B. we're using a Blaze function to do this, not len()
    shareholder_meetings.count()




.. raw:: html

    8969



.. code:: ipython2

    # Let's see what the data looks like. We'll grab the first three rows.
    shareholder_meetings[:3]




.. raw:: html

    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>event_id</th>
          <th>asof_date</th>
          <th>symbol</th>
          <th>event_headline</th>
          <th>meeting_type</th>
          <th>record_date</th>
          <th>meeting_date</th>
          <th>timestamp</th>
          <th>sid</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>9000012933</td>
          <td>2009-01-02</td>
          <td>CENT</td>
          <td>Central Garden &amp; Pet announces Shareholder Mee...</td>
          <td>Annual Meeting</td>
          <td>2008-12-19</td>
          <td>2009-02-09</td>
          <td>2009-01-03</td>
          <td>18855</td>
        </tr>
        <tr>
          <th>1</th>
          <td>9000016639</td>
          <td>2009-12-21</td>
          <td>PENX</td>
          <td>Penford Corp. announces Shareholder Meeting</td>
          <td>Annual Meeting</td>
          <td>2009-12-04</td>
          <td>2010-01-26</td>
          <td>2009-12-22</td>
          <td>18082</td>
        </tr>
        <tr>
          <th>2</th>
          <td>9000016643</td>
          <td>2009-12-23</td>
          <td>CCF</td>
          <td>Chase announces Shareholder Meeting</td>
          <td>Annual Meeting</td>
          <td>2009-11-30</td>
          <td>2010-01-29</td>
          <td>2009-12-24</td>
          <td>13810</td>
        </tr>
      </tbody>
    </table>



Let’s go over the columns: - **event_id**: the unique identifier for
this event. - **asof_date**: EventVestor’s timestamp of event capture. -
**symbol**: stock ticker symbol of the affected company. -
**event_headline**: a brief description of the event - **meeting_type**:
types include *annual meeting, special meeting, proxy contest*. -
**record_date**: record date to be eligible for proxy vote -
**meeting_date**: shareholder meeting date - **timestamp**: this is our
timestamp on when we registered the data. - **sid**: the equity’s unique
identifier. Use this instead of the symbol.

We’ve done much of the data processing for you. Fields like
``timestamp`` and ``sid`` are standardized across all our Store
Datasets, so the datasets are easy to combine. We have standardized the
``sid`` across all our equity databases.

We can select columns and rows with ease. Below, we’ll fetch Tesla’s
2013 and 2014 meetings.

.. code:: ipython2

    # get tesla's sid first
    tesla_sid = symbols('TSLA').sid
    meetings = shareholder_meetings[('2012-12-31' < shareholder_meetings['asof_date']) & 
                                    (shareholder_meetings['asof_date'] <'2015-01-01') & 
                                    (shareholder_meetings.sid == tesla_sid)]
    # When displaying a Blaze Data Object, the printout is automatically truncated to ten rows.
    meetings.sort('asof_date')




.. raw:: html

    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>event_id</th>
          <th>asof_date</th>
          <th>symbol</th>
          <th>event_headline</th>
          <th>meeting_type</th>
          <th>record_date</th>
          <th>meeting_date</th>
          <th>timestamp</th>
          <th>sid</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>900002592</td>
          <td>2013-04-17</td>
          <td>TSLA</td>
          <td>TESLA MOTORS announces Shareholder Meeting</td>
          <td>Annual Meeting</td>
          <td>2013-04-10</td>
          <td>2013-06-04</td>
          <td>2013-04-18</td>
          <td>39840</td>
        </tr>
        <tr>
          <th>1</th>
          <td>9000012760</td>
          <td>2014-04-24</td>
          <td>TSLA</td>
          <td>Tesla Motors, Inc. announces Shareholder Meeting</td>
          <td>Annual Meeting</td>
          <td>2014-04-10</td>
          <td>2014-06-03</td>
          <td>2014-04-25</td>
          <td>39840</td>
        </tr>
      </tbody>
    </table>



Now suppose we want a DataFrame of the Blaze Data Object above, but only
want the ``record_date, meeting_date``, and ``sid``.

.. code:: ipython2

    df = odo(meetings, pd.DataFrame)
    df = df[['record_date','meeting_date','sid']]
    df




.. raw:: html

    <div style="max-height:1000px;max-width:1500px;overflow:auto;">
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>record_date</th>
          <th>meeting_date</th>
          <th>sid</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>2013-04-10</td>
          <td>2013-06-04</td>
          <td>39840</td>
        </tr>
        <tr>
          <th>1</th>
          <td>2014-04-10</td>
          <td>2014-06-03</td>
          <td>39840</td>
        </tr>
      </tbody>
    </table>
    </div>



