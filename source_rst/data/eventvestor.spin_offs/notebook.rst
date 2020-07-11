EventVestor: Spin-Offs
======================

In this notebook, we’ll take a look at EventVestor’s *Spin-Offs*
dataset, available on the `Quantopian
Store <https://www.quantopian.com/store>`__. This dataset spans January
01, 2007 through the current day, and documents corporate spin-off
events.

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
    from quantopian.interactive.data.eventvestor import spin_offs
    # or if you want to import the free dataset, use:
    # from quantopian.data.eventvestor import spin_offs_free
    
    # import data operations
    from odo import odo
    # import other libraries we will use
    import pandas as pd

.. code:: ipython2

    # Let's use blaze to understand the data a bit using Blaze dshape()
    spin_offs.dshape




.. parsed-literal::

    dshape("""var * {
      event_id: ?float64,
      asof_date: datetime,
      trade_date: ?datetime,
      symbol: ?string,
      event_type: ?string,
      event_headline: ?string,
      spinoff_phase: ?string,
      spinoff_name: ?string,
      event_rating: ?float64,
      timestamp: datetime,
      sid: ?int64
      }""")



.. code:: ipython2

    # And how many rows are there?
    # N.B. we're using a Blaze function to do this, not len()
    spin_offs.count()




.. raw:: html

    1189



.. code:: ipython2

    # Let's see what the data looks like. We'll grab the first three rows.
    spin_offs[:3]




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
          <th>spinoff_phase</th>
          <th>spinoff_name</th>
          <th>event_rating</th>
          <th>timestamp</th>
          <th>sid</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>127421</td>
          <td>2007-01-08</td>
          <td>2007-01-09</td>
          <td>DUK</td>
          <td>Spin-off</td>
          <td>Duke Energy completes Natural Gas business spi...</td>
          <td>Completes</td>
          <td>NaN</td>
          <td>1</td>
          <td>2007-01-09</td>
          <td>2351</td>
        </tr>
        <tr>
          <th>1</th>
          <td>134268</td>
          <td>2007-01-08</td>
          <td>2007-01-08</td>
          <td>NCR</td>
          <td>Spin-off</td>
          <td>NCR To Separate Into Two Independent Companies</td>
          <td>Proposal</td>
          <td>NaN</td>
          <td>1</td>
          <td>2007-01-09</td>
          <td>16389</td>
        </tr>
        <tr>
          <th>2</th>
          <td>77960</td>
          <td>2007-01-16</td>
          <td>2007-01-16</td>
          <td>VZ</td>
          <td>Spin-off</td>
          <td>Verizon to spin off and merge local exchange a...</td>
          <td>Board Approval</td>
          <td>NaN</td>
          <td>1</td>
          <td>2007-01-17</td>
          <td>21839</td>
        </tr>
      </tbody>
    </table>



Let’s go over the columns: - **event_id**: the unique identifier for
this event. - **asof_date**: EventVestor’s timestamp of event capture. -
**trade_date**: for event announcements made before trading ends,
trade_date is the same as event_date. For announcements issued after
market close, trade_date is next market open day. - **symbol**: stock
ticker symbol of the affected company. - **event_type**: this should
always be *Spin-off*. - **event_headline**: a brief description of the
event - **spinoff_phase**: values include *proposal, approval,
completes*. - **spinoff_name**: name of the entity being spun off. -
**event_rating**: this is always 1. The meaning of this is uncertain. -
**timestamp**: this is our timestamp on when we registered the data. -
**sid**: the equity’s unique identifier. Use this instead of the symbol.

We’ve done much of the data processing for you. Fields like
``timestamp`` and ``sid`` are standardized across all our Store
Datasets, so the datasets are easy to combine. We have standardized the
``sid`` across all our equity databases.

We can select columns and rows with ease. Below, we’ll fetch Yahoo’s
2015 spin-offs.

.. code:: ipython2

    # get yahoo's sid first
    yahoo_sid = symbols('YHOO').sid
    spinoffs = spin_offs[('2014-12-31' < spin_offs['asof_date']) & 
                                    (spin_offs['asof_date'] <'2016-01-01') & 
                                    (spin_offs.sid == yahoo_sid)]
    # When displaying a Blaze Data Object, the printout is automatically truncated to ten rows.
    spinoffs.sort('asof_date')




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
          <th>spinoff_phase</th>
          <th>spinoff_name</th>
          <th>event_rating</th>
          <th>timestamp</th>
          <th>sid</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>1827542</td>
          <td>2015-01-27</td>
          <td>2015-01-28</td>
          <td>YHOO</td>
          <td>Spin-off</td>
          <td>Yahoo to Spin-Off its Alibaba Stake into Newly...</td>
          <td>Board Approval</td>
          <td>NaN</td>
          <td>1</td>
          <td>2015-01-28 00:00:00</td>
          <td>14848</td>
        </tr>
        <tr>
          <th>1</th>
          <td>1903562</td>
          <td>2015-07-17</td>
          <td>2015-07-20</td>
          <td>YHOO</td>
          <td>Spin-off</td>
          <td>Yahoo! Announces SEC Filing for Planned Spin-O...</td>
          <td>Updates</td>
          <td>Aabaco Holdings Inc.</td>
          <td>1</td>
          <td>2015-07-18 00:00:00</td>
          <td>14848</td>
        </tr>
        <tr>
          <th>2</th>
          <td>1937451</td>
          <td>2015-09-28</td>
          <td>2015-09-29</td>
          <td>YHOO</td>
          <td>Spin-off</td>
          <td>Yahoo! to Proceed Alibaba Stake Spinoff withou...</td>
          <td>Updates</td>
          <td>Alibaba Holding Group Ltd.</td>
          <td>1</td>
          <td>2015-09-29 11:14:35.314487</td>
          <td>14848</td>
        </tr>
      </tbody>
    </table>



Now suppose we want a DataFrame of ``spin_offs``, but only want the
``asof_date, spinoff_phase``, and the ``sid``.

.. code:: ipython2

    #len(spin_offs) = ~10000, so we can convert it to a dataframe without a worry -- it's a small dataset.
    df = odo(spin_offs, pd.DataFrame)
    df = df[['asof_date','spinoff_phase','sid']]
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
          <th>spinoff_phase</th>
          <th>sid</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>2007-01-08</td>
          <td>Completes</td>
          <td>2351</td>
        </tr>
        <tr>
          <th>1</th>
          <td>2007-01-08</td>
          <td>Proposal</td>
          <td>16389</td>
        </tr>
        <tr>
          <th>2</th>
          <td>2007-01-16</td>
          <td>Board Approval</td>
          <td>21839</td>
        </tr>
        <tr>
          <th>3</th>
          <td>2007-01-17</td>
          <td>Updates</td>
          <td>4758</td>
        </tr>
        <tr>
          <th>4</th>
          <td>2007-01-19</td>
          <td>Updates</td>
          <td>13373</td>
        </tr>
        <tr>
          <th>5</th>
          <td>2007-01-31</td>
          <td>Completes</td>
          <td>13373</td>
        </tr>
        <tr>
          <th>6</th>
          <td>2007-01-31</td>
          <td>Board Approval</td>
          <td>4954</td>
        </tr>
        <tr>
          <th>8</th>
          <td>2007-02-02</td>
          <td>Updates</td>
          <td>8326</td>
        </tr>
        <tr>
          <th>9</th>
          <td>2007-02-06</td>
          <td>Updates</td>
          <td>22954</td>
        </tr>
        <tr>
          <th>10</th>
          <td>2007-02-27</td>
          <td>Proposal</td>
          <td>22983</td>
        </tr>
        <tr>
          <th>11</th>
          <td>2007-02-27</td>
          <td>Proposal</td>
          <td>7449</td>
        </tr>
        <tr>
          <th>12</th>
          <td>2007-03-02</td>
          <td>Updates</td>
          <td>3443</td>
        </tr>
        <tr>
          <th>13</th>
          <td>2007-03-02</td>
          <td>Updates</td>
          <td>32880</td>
        </tr>
        <tr>
          <th>14</th>
          <td>2007-03-06</td>
          <td>Updates</td>
          <td>22983</td>
        </tr>
        <tr>
          <th>15</th>
          <td>2007-03-07</td>
          <td>Completes</td>
          <td>8326</td>
        </tr>
        <tr>
          <th>16</th>
          <td>2007-03-20</td>
          <td>Updates</td>
          <td>4954</td>
        </tr>
        <tr>
          <th>17</th>
          <td>2007-03-21</td>
          <td>Proposal</td>
          <td>630</td>
        </tr>
        <tr>
          <th>18</th>
          <td>2007-03-26</td>
          <td>Updates</td>
          <td>22954</td>
        </tr>
        <tr>
          <th>19</th>
          <td>2007-03-30</td>
          <td>Completes</td>
          <td>4954</td>
        </tr>
        <tr>
          <th>20</th>
          <td>2007-04-03</td>
          <td>Updates</td>
          <td>3443</td>
        </tr>
        <tr>
          <th>21</th>
          <td>2007-04-03</td>
          <td>Updates</td>
          <td>32880</td>
        </tr>
        <tr>
          <th>22</th>
          <td>2007-04-04</td>
          <td>Completes</td>
          <td>630</td>
        </tr>
        <tr>
          <th>23</th>
          <td>2007-04-04</td>
          <td>Proposal</td>
          <td>5025</td>
        </tr>
        <tr>
          <th>24</th>
          <td>2007-04-05</td>
          <td>Completes</td>
          <td>3443</td>
        </tr>
        <tr>
          <th>25</th>
          <td>2007-04-05</td>
          <td>Completes</td>
          <td>32880</td>
        </tr>
        <tr>
          <th>26</th>
          <td>2007-04-10</td>
          <td>Proposal</td>
          <td>18027</td>
        </tr>
        <tr>
          <th>28</th>
          <td>2007-05-15</td>
          <td>Proposal</td>
          <td>4010</td>
        </tr>
        <tr>
          <th>29</th>
          <td>2007-05-26</td>
          <td>Updates</td>
          <td>2190</td>
        </tr>
        <tr>
          <th>30</th>
          <td>2007-06-01</td>
          <td>Board Approval</td>
          <td>17080</td>
        </tr>
        <tr>
          <th>31</th>
          <td>2007-06-08</td>
          <td>Proposal</td>
          <td>7679</td>
        </tr>
        <tr>
          <th>...</th>
          <td>...</td>
          <td>...</td>
          <td>...</td>
        </tr>
        <tr>
          <th>1028</th>
          <td>2015-07-13</td>
          <td>Updates</td>
          <td>34575</td>
        </tr>
        <tr>
          <th>1029</th>
          <td>2015-07-14</td>
          <td>Board Approval</td>
          <td>9693</td>
        </tr>
        <tr>
          <th>1030</th>
          <td>2015-07-17</td>
          <td>Updates</td>
          <td>14848</td>
        </tr>
        <tr>
          <th>1031</th>
          <td>2015-07-20</td>
          <td>Completes</td>
          <td>24819</td>
        </tr>
        <tr>
          <th>1032</th>
          <td>2015-07-22</td>
          <td>Updates</td>
          <td>34575</td>
        </tr>
        <tr>
          <th>1034</th>
          <td>2015-07-24</td>
          <td>Updates</td>
          <td>34575</td>
        </tr>
        <tr>
          <th>1035</th>
          <td>2015-07-24</td>
          <td>Updates</td>
          <td>4117</td>
        </tr>
        <tr>
          <th>1036</th>
          <td>2015-07-30</td>
          <td>Updates</td>
          <td>22015</td>
        </tr>
        <tr>
          <th>1037</th>
          <td>2015-07-30</td>
          <td>Board Approval</td>
          <td>18821</td>
        </tr>
        <tr>
          <th>1038</th>
          <td>2015-07-31</td>
          <td>Updates</td>
          <td>13306</td>
        </tr>
        <tr>
          <th>1039</th>
          <td>2015-08-03</td>
          <td>Completes</td>
          <td>9693</td>
        </tr>
        <tr>
          <th>1040</th>
          <td>2015-08-03</td>
          <td>Proposal</td>
          <td>21608</td>
        </tr>
        <tr>
          <th>1041</th>
          <td>2015-08-04</td>
          <td>Proposal</td>
          <td>2248</td>
        </tr>
        <tr>
          <th>1042</th>
          <td>2015-08-06</td>
          <td>Proposal</td>
          <td>32878</td>
        </tr>
        <tr>
          <th>1043</th>
          <td>2015-08-06</td>
          <td>Updates</td>
          <td>47812</td>
        </tr>
        <tr>
          <th>1044</th>
          <td>2015-08-18</td>
          <td>Completes</td>
          <td>18821</td>
        </tr>
        <tr>
          <th>1045</th>
          <td>2015-08-27</td>
          <td>NaN</td>
          <td>22689</td>
        </tr>
        <tr>
          <th>1046</th>
          <td>2015-08-31</td>
          <td>Updates</td>
          <td>1898</td>
        </tr>
        <tr>
          <th>1047</th>
          <td>2015-09-01</td>
          <td>Updates</td>
          <td>4656</td>
        </tr>
        <tr>
          <th>1048</th>
          <td>2015-09-04</td>
          <td>Updates</td>
          <td>21608</td>
        </tr>
        <tr>
          <th>1049</th>
          <td>2015-09-08</td>
          <td>Board Approval</td>
          <td>1936</td>
        </tr>
        <tr>
          <th>1050</th>
          <td>2015-09-08</td>
          <td>NaN</td>
          <td>42176</td>
        </tr>
        <tr>
          <th>1051</th>
          <td>2015-09-11</td>
          <td>Updates</td>
          <td>34067</td>
        </tr>
        <tr>
          <th>1052</th>
          <td>2015-09-15</td>
          <td>Updates</td>
          <td>11498</td>
        </tr>
        <tr>
          <th>1053</th>
          <td>2015-09-16</td>
          <td>Board Approval</td>
          <td>460</td>
        </tr>
        <tr>
          <th>1054</th>
          <td>2015-09-22</td>
          <td>Proposal</td>
          <td>559</td>
        </tr>
        <tr>
          <th>1055</th>
          <td>2015-09-28</td>
          <td>NaN</td>
          <td>2</td>
        </tr>
        <tr>
          <th>1160</th>
          <td>2014-07-11</td>
          <td>Board Approval</td>
          <td>5249</td>
        </tr>
        <tr>
          <th>1187</th>
          <td>2015-09-28</td>
          <td>Completes</td>
          <td>7086</td>
        </tr>
        <tr>
          <th>1188</th>
          <td>2015-09-28</td>
          <td>Updates</td>
          <td>14848</td>
        </tr>
      </tbody>
    </table>
    <p>929 rows × 3 columns</p>
    </div>



