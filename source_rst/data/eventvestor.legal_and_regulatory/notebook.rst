EventVestor: Legal and Regulatory
=================================

In this notebook, we’ll take a look at EventVestor’s *Legal and
Regulatory* dataset, available on the `Quantopian
Store <https://www.quantopian.com/store>`__. This dataset spans January
01, 2007 through the current day, and documents major legal and
regulatory events affecting publicly traded companies.

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
    from quantopian.interactive.data.eventvestor import legal_and_regulatory
    # or if you want to import the free dataset, use:
    # from quantopian.interactive.data.eventvestor import legal_and_regulatory_free
    
    # import data operations
    from odo import odo
    # import other libraries we will use
    import pandas as pd

.. code:: ipython2

    # Let's use blaze to understand the data a bit using Blaze dshape()
    legal_and_regulatory.dshape




.. parsed-literal::

    dshape("""var * {
      event_id: ?float64,
      asof_date: datetime,
      trade_date: ?datetime,
      symbol: ?string,
      event_type: ?string,
      event_headline: ?string,
      legal_amount: ?float64,
      legal_units: ?string,
      legal_entity: ?string,
      event_rating: ?float64,
      timestamp: datetime,
      sid: ?int64
      }""")



.. code:: ipython2

    # And how many rows are there?
    # N.B. we're using a Blaze function to do this, not len()
    legal_and_regulatory.count()




.. raw:: html

    8180



.. code:: ipython2

    # Let's see what the data looks like. We'll grab the first three rows.
    legal_and_regulatory[:3]




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
          <th>legal_amount</th>
          <th>legal_units</th>
          <th>legal_entity</th>
          <th>event_rating</th>
          <th>timestamp</th>
          <th>sid</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>77848</td>
          <td>2007-01-05</td>
          <td>2007-01-08</td>
          <td>AMAT</td>
          <td>Legal/Regulatory</td>
          <td>Applied Materials' Review under the HSR Antitr...</td>
          <td>0.0</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>1</td>
          <td>2007-01-06</td>
          <td>337</td>
        </tr>
        <tr>
          <th>1</th>
          <td>148666</td>
          <td>2007-01-05</td>
          <td>2007-01-05</td>
          <td>FCS</td>
          <td>Legal/Regulatory</td>
          <td>Fairchild Semiconductor Appeals Ruling in ZTE ...</td>
          <td>8.4</td>
          <td>$M</td>
          <td>Zhongxing Telecom Ltd</td>
          <td>1</td>
          <td>2007-01-06</td>
          <td>20486</td>
        </tr>
        <tr>
          <th>2</th>
          <td>77994</td>
          <td>2007-01-09</td>
          <td>2007-01-09</td>
          <td>XLNX</td>
          <td>Legal/Regulatory</td>
          <td>Xilinx announces dismissal of shareholder deri...</td>
          <td>0.0</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>1</td>
          <td>2007-01-10</td>
          <td>8344</td>
        </tr>
      </tbody>
    </table>



Let’s go over the columns: - **event_id**: the unique identifier for
this event. - **asof_date**: EventVestor’s timestamp of event capture. -
**trade_date**: for event announcements made before trading ends,
trade_date is the same as event_date. For announcements issued after
market close, trade_date is next market open day. - **symbol**: stock
ticker symbol of the affected company. - **event_type**: this should
always be *Legal/Regulatory*. - **event_headline**: a brief description
of the event - **legal_amount**: amount mentioned in the case, if any. -
**legal_units**: units of the legal_amount: most commonly millions of
dollars. - **legal_entity**: the related entity in the legal case. -
**event_rating**: this is always 1. The meaning of this is uncertain. -
**timestamp**: this is our timestamp on when we registered the data. -
**sid**: the equity’s unique identifier. Use this instead of the symbol.

We’ve done much of the data processing for you. Fields like
``timestamp`` and ``sid`` are standardized across all our Store
Datasets, so the datasets are easy to combine. We have standardized the
``sid`` across all our equity databases.

We can select columns and rows with ease. Below, we’ll fetch all 2014
events involving General Motors.

.. code:: ipython2

    # get GM's sid first
    gm_sid = symbols('GM').sid
    cases = legal_and_regulatory[('2013-12-31' < legal_and_regulatory['asof_date']) & 
                            (legal_and_regulatory['asof_date'] <'2015-01-01') & 
                            (legal_and_regulatory.sid == gm_sid)]
    # When displaying a Blaze Data Object, the printout is automatically truncated to ten rows.
    cases.sort('asof_date')




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
          <th>legal_amount</th>
          <th>legal_units</th>
          <th>legal_entity</th>
          <th>event_rating</th>
          <th>timestamp</th>
          <th>sid</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>1695334</td>
          <td>2014-03-19</td>
          <td>2014-03-20</td>
          <td>GM</td>
          <td>Legal/Regulatory</td>
          <td>General Motors Co. Faces Class Action Lawsuit ...</td>
          <td>0</td>
          <td>NaN</td>
          <td>Hagens Berman</td>
          <td>1</td>
          <td>2014-03-20</td>
          <td>40430</td>
        </tr>
        <tr>
          <th>1</th>
          <td>1696156</td>
          <td>2014-03-21</td>
          <td>2014-03-21</td>
          <td>GM</td>
          <td>Legal/Regulatory</td>
          <td>General Motors Co. Faces Class Action Lawsuit</td>
          <td>0</td>
          <td>NaN</td>
          <td>Pomerantz LLP</td>
          <td>1</td>
          <td>2014-03-22</td>
          <td>40430</td>
        </tr>
        <tr>
          <th>2</th>
          <td>1703401</td>
          <td>2014-04-22</td>
          <td>2014-04-23</td>
          <td>GM</td>
          <td>Legal/Regulatory</td>
          <td>General Motors Company Faces Class Action Laws...</td>
          <td>0</td>
          <td>NaN</td>
          <td>Law Offices of Howard G. Smith; Glancy Binkow ...</td>
          <td>1</td>
          <td>2014-04-23</td>
          <td>40430</td>
        </tr>
        <tr>
          <th>3</th>
          <td>1725393</td>
          <td>2014-05-16</td>
          <td>2014-05-19</td>
          <td>GM</td>
          <td>Legal/Regulatory</td>
          <td>General Motors Co. Faces Class Action Lawsuits</td>
          <td>0</td>
          <td>NaN</td>
          <td>Pomerantz LLP</td>
          <td>1</td>
          <td>2014-05-17</td>
          <td>40430</td>
        </tr>
        <tr>
          <th>4</th>
          <td>1725455</td>
          <td>2014-05-17</td>
          <td>2014-05-19</td>
          <td>GM</td>
          <td>Legal/Regulatory</td>
          <td>General Motors to Pay $35M Fine Over Delay to ...</td>
          <td>35</td>
          <td>$M</td>
          <td>NaN</td>
          <td>1</td>
          <td>2014-05-18</td>
          <td>40430</td>
        </tr>
        <tr>
          <th>5</th>
          <td>1736421</td>
          <td>2014-06-18</td>
          <td>2014-06-18</td>
          <td>GM</td>
          <td>Legal/Regulatory</td>
          <td>General Motors Co. Faces Class Action Lawsuit</td>
          <td>0</td>
          <td>NaN</td>
          <td>Hagens Berman Sobol Shapiro</td>
          <td>1</td>
          <td>2014-06-19</td>
          <td>40430</td>
        </tr>
        <tr>
          <th>6</th>
          <td>1768189</td>
          <td>2014-08-09</td>
          <td>2014-08-09</td>
          <td>GM</td>
          <td>Legal/Regulatory</td>
          <td>Judge Rejects General Motors Co. Motion to Dis...</td>
          <td>0</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>1</td>
          <td>2014-08-10</td>
          <td>40430</td>
        </tr>
      </tbody>
    </table>



Now suppose we want a DataFrame of the Blaze Data Object above, but only
want entries with a non-zero legal_amount. Further, we want to drop the
``event_rating`` and ``event_type``.

.. code:: ipython2

    df = odo(cases, pd.DataFrame)
    df = df[df.legal_amount > 0]
    df.drop(df[['event_type','event_rating']], axis=1, inplace=True)
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
          <th>event_headline</th>
          <th>legal_amount</th>
          <th>legal_units</th>
          <th>legal_entity</th>
          <th>timestamp</th>
          <th>sid</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>4</th>
          <td>1725455</td>
          <td>2014-05-17</td>
          <td>2014-05-19</td>
          <td>GM</td>
          <td>General Motors to Pay $35M Fine Over Delay to ...</td>
          <td>35</td>
          <td>$M</td>
          <td>NaN</td>
          <td>2014-05-18</td>
          <td>40430</td>
        </tr>
      </tbody>
    </table>
    </div>


