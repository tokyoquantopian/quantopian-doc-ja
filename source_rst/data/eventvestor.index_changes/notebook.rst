EventVestor: Index Changes
==========================

In this notebook, we’ll take a look at EventVestor’s *Index Changes*
dataset, available on the `Quantopian
Store <https://www.quantopian.com/store>`__. This dataset spans January
01, 2007 through the current day, and documents index additions and
deletions to major S&P, Russell, and Nasdaq 100 indexes.

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
    from quantopian.interactive.data.eventvestor import index_changes
    # or if you want to import the free dataset, use:
    # from quantopian.interactive.data.eventvestor import index_changes_free
    
    # import data operations
    from odo import odo
    # import other libraries we will use
    import pandas as pd

.. code:: ipython2

    # Let's use blaze to understand the data a bit using Blaze dshape()
    index_changes.dshape




.. parsed-literal::

    dshape("""var * {
      event_id: ?float64,
      asof_date: datetime,
      trade_date: ?datetime,
      symbol: ?string,
      event_type: ?string,
      event_headline: ?string,
      index_name: ?string,
      change_type: ?string,
      change_reason: ?string,
      event_rating: ?float64,
      timestamp: datetime,
      sid: ?int64
      }""")



.. code:: ipython2

    # And how many rows are there?
    # N.B. we're using a Blaze function to do this, not len()
    index_changes.count()




.. raw:: html

    2510



.. code:: ipython2

    # Let's see what the data looks like. We'll grab the first three rows.
    index_changes[:3]




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
          <th>index_name</th>
          <th>change_type</th>
          <th>change_reason</th>
          <th>event_rating</th>
          <th>timestamp</th>
          <th>sid</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>174074</td>
          <td>2007-01-02</td>
          <td>2007-01-03</td>
          <td>BLS</td>
          <td>Index Change</td>
          <td>BellSouth Corp. (BLS) removed from S&amp;P 500</td>
          <td>S&amp;P 500</td>
          <td>Deletion</td>
          <td>NaN</td>
          <td>1</td>
          <td>2007-01-03</td>
          <td>948</td>
        </tr>
        <tr>
          <th>1</th>
          <td>174076</td>
          <td>2007-01-02</td>
          <td>2007-01-03</td>
          <td>ESV</td>
          <td>Index Change</td>
          <td>ENSCO, Int'l (ESV) removed from S&amp;P 400</td>
          <td>S&amp;P 400</td>
          <td>Deletion</td>
          <td>NaN</td>
          <td>1</td>
          <td>2007-01-03</td>
          <td>2621</td>
        </tr>
        <tr>
          <th>2</th>
          <td>174071</td>
          <td>2007-01-02</td>
          <td>2007-01-03</td>
          <td>ESV</td>
          <td>Index Change</td>
          <td>ENSCO International (ESV) added to S&amp;P 500</td>
          <td>S&amp;P 500</td>
          <td>Addition</td>
          <td>NaN</td>
          <td>1</td>
          <td>2007-01-03</td>
          <td>2621</td>
        </tr>
      </tbody>
    </table>



Let’s go over the columns: - **event_id**: the unique identifier for
this event. - **asof_date**: EventVestor’s timestamp of event capture. -
**trade_date**: for event announcements made before trading ends,
trade_date is the same as event_date. For announcements issued after
market close, trade_date is next market open day. - **symbol**: stock
ticker symbol of the affected company. - **event_type**: this should
always be *Index Change*. - **event_headline**: a brief description of
the event - **index_name**: name of the index affected. Values include
*S&P 400, S&P 500, S&P 600* - **change_type**: Addition/Deletion of
equity - **change_reason**: reason for addition/deletion of the equity
from the index. Reasons include *Acquired, Market Cap, Other*. -
**event_rating**: this is always 1. The meaning of this is uncertain. -
**timestamp**: this is our timestamp on when we registered the data. -
**sid**: the equity’s unique identifier. Use this instead of the symbol.
Note: this sid represents the company the shares of which are being
purchased, not the acquiring entity.

We’ve done much of the data processing for you. Fields like
``timestamp`` and ``sid`` are standardized across all our Store
Datasets, so the datasets are easy to combine. We have standardized the
``sid`` across all our equity databases.

We can select columns and rows with ease. Below, we’ll fetch all 2015
deletions due to market cap.

.. code:: ipython2

    deletions = index_changes[('2014-12-31' < index_changes['asof_date']) & 
                                            (index_changes['asof_date'] <'2016-01-01') & 
                                            (index_changes.change_type == "Deletion")&
                                            (index_changes.change_reason  == "Market Cap")]
    # When displaying a Blaze Data Object, the printout is automatically truncated to ten rows.
    deletions.sort('asof_date')




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
          <th>index_name</th>
          <th>change_type</th>
          <th>change_reason</th>
          <th>event_rating</th>
          <th>timestamp</th>
          <th>sid</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>1885908</td>
          <td>2015-05-21</td>
          <td>2015-05-22</td>
          <td>WIN</td>
          <td>Index Change</td>
          <td>Windstream Holdings to be Removed from S&amp;P Mid...</td>
          <td>S&amp;P 400</td>
          <td>Deletion</td>
          <td>Market Cap</td>
          <td>1</td>
          <td>2015-05-22</td>
          <td>27019</td>
        </tr>
        <tr>
          <th>1</th>
          <td>1894211</td>
          <td>2015-06-24</td>
          <td>2015-06-24</td>
          <td>ATI</td>
          <td>Index Change</td>
          <td>Allegheny Technologies to be Removed from S&amp;P ...</td>
          <td>S&amp;P 500</td>
          <td>Deletion</td>
          <td>Market Cap</td>
          <td>1</td>
          <td>2015-06-25</td>
          <td>24840</td>
        </tr>
        <tr>
          <th>2</th>
          <td>1894270</td>
          <td>2015-06-24</td>
          <td>2015-06-24</td>
          <td>SMTC</td>
          <td>Index Change</td>
          <td>Semtech Corp. to be Removed from S&amp;P MidCap 40...</td>
          <td>S&amp;P 400</td>
          <td>Deletion</td>
          <td>Market Cap</td>
          <td>1</td>
          <td>2015-06-25</td>
          <td>6961</td>
        </tr>
        <tr>
          <th>3</th>
          <td>1894266</td>
          <td>2015-06-24</td>
          <td>2015-06-24</td>
          <td>BTU</td>
          <td>Index Change</td>
          <td>Peabody Energy Corp. to be Removed from S&amp;P Mi...</td>
          <td>S&amp;P 400</td>
          <td>Deletion</td>
          <td>Market Cap</td>
          <td>1</td>
          <td>2015-06-25</td>
          <td>22660</td>
        </tr>
        <tr>
          <th>4</th>
          <td>1894278</td>
          <td>2015-06-24</td>
          <td>2015-06-24</td>
          <td>HSC</td>
          <td>Index Change</td>
          <td>Harsco Corp. to be Removed from S&amp;P MidCap 400...</td>
          <td>S&amp;P 400</td>
          <td>Deletion</td>
          <td>Market Cap</td>
          <td>1</td>
          <td>2015-06-25</td>
          <td>3686</td>
        </tr>
        <tr>
          <th>5</th>
          <td>1894221</td>
          <td>2015-06-24</td>
          <td>2015-06-24</td>
          <td>PQ</td>
          <td>Index Change</td>
          <td>PetroQuest Energy to be Removed from S&amp;P Small...</td>
          <td>S&amp;P 600</td>
          <td>Deletion</td>
          <td>Market Cap</td>
          <td>1</td>
          <td>2015-06-25</td>
          <td>19326</td>
        </tr>
        <tr>
          <th>6</th>
          <td>1894247</td>
          <td>2015-06-24</td>
          <td>2015-06-24</td>
          <td>ARO</td>
          <td>Index Change</td>
          <td>Aeropostale to be Removed from S&amp;P SmallCap 60...</td>
          <td>S&amp;P 600</td>
          <td>Deletion</td>
          <td>Market Cap</td>
          <td>1</td>
          <td>2015-06-25</td>
          <td>23650</td>
        </tr>
        <tr>
          <th>7</th>
          <td>1894217</td>
          <td>2015-06-24</td>
          <td>2015-06-24</td>
          <td>UNT</td>
          <td>Index Change</td>
          <td>Unit Corp. to be Removed from S&amp;P MidCap 400 I...</td>
          <td>S&amp;P 400</td>
          <td>Deletion</td>
          <td>Market Cap</td>
          <td>1</td>
          <td>2015-06-25</td>
          <td>7806</td>
        </tr>
        <tr>
          <th>8</th>
          <td>1894258</td>
          <td>2015-06-24</td>
          <td>2015-06-24</td>
          <td>ZQK</td>
          <td>Index Change</td>
          <td>Quiksilver to be Removed from S&amp;P SmallCap 600...</td>
          <td>S&amp;P 600</td>
          <td>Deletion</td>
          <td>Market Cap</td>
          <td>1</td>
          <td>2015-06-25</td>
          <td>6317</td>
        </tr>
        <tr>
          <th>9</th>
          <td>1894293</td>
          <td>2015-06-24</td>
          <td>2015-06-24</td>
          <td>FXCM</td>
          <td>Index Change</td>
          <td>FXCM to be Removed from S&amp;P SmallCap 600 Index</td>
          <td>S&amp;P 600</td>
          <td>Deletion</td>
          <td>Market Cap</td>
          <td>1</td>
          <td>2015-06-25</td>
          <td>40531</td>
        </tr>
        <tr>
          <th>10</th>
          <td>1895235</td>
          <td>2015-06-26</td>
          <td>2015-06-29</td>
          <td>JBHT</td>
          <td>Index Change</td>
          <td>J.B. Hunt Transport Services to be Removed fro...</td>
          <td>S&amp;P 400</td>
          <td>Deletion</td>
          <td>Market Cap</td>
          <td>1</td>
          <td>2015-06-27</td>
          <td>4108</td>
        </tr>
      </tbody>
    </table>



Now suppose we want a DataFrame of the Blaze Data Object above, want to
filter it further down to the S&P 600, and we only want the sid and the
asof_date.

.. code:: ipython2

    df = odo(deletions, pd.DataFrame)
    df = df[df.index_name == "S&P 600"]
    df = df[['sid', 'asof_date']]
    df




.. raw:: html

    <div style="max-height:1000px;max-width:1500px;overflow:auto;">
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>sid</th>
          <th>asof_date</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>1</th>
          <td>23650</td>
          <td>2015-06-24</td>
        </tr>
        <tr>
          <th>4</th>
          <td>40531</td>
          <td>2015-06-24</td>
        </tr>
        <tr>
          <th>6</th>
          <td>19326</td>
          <td>2015-06-24</td>
        </tr>
        <tr>
          <th>9</th>
          <td>6317</td>
          <td>2015-06-24</td>
        </tr>
        <tr>
          <th>12</th>
          <td>1308</td>
          <td>2015-06-29</td>
        </tr>
        <tr>
          <th>15</th>
          <td>20740</td>
          <td>2015-07-06</td>
        </tr>
        <tr>
          <th>16</th>
          <td>8291</td>
          <td>2015-07-06</td>
        </tr>
        <tr>
          <th>19</th>
          <td>6825</td>
          <td>2015-07-14</td>
        </tr>
        <tr>
          <th>20</th>
          <td>20526</td>
          <td>2015-07-17</td>
        </tr>
        <tr>
          <th>22</th>
          <td>1263</td>
          <td>2015-07-24</td>
        </tr>
        <tr>
          <th>23</th>
          <td>1663</td>
          <td>2015-07-24</td>
        </tr>
        <tr>
          <th>24</th>
          <td>13918</td>
          <td>2015-07-24</td>
        </tr>
        <tr>
          <th>27</th>
          <td>24823</td>
          <td>2015-08-19</td>
        </tr>
        <tr>
          <th>28</th>
          <td>21736</td>
          <td>2015-09-21</td>
        </tr>
      </tbody>
    </table>
    </div>


