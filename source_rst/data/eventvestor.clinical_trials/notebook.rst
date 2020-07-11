EventVestor: Clinical Trials
============================

In this notebook, we’ll take a look at EventVestor’s *Clinical Trials*
dataset, available on the `Quantopian
Store <https://www.quantopian.com/store>`__. This dataset spans January
01, 2007 through the current day, and documents announcements of key
phases of clinical trials by biotech/pharmaceutical companies.

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
    from quantopian.interactive.data.eventvestor import clinical_trials
    # or if you want to import the free dataset, use:
    # from quantopian.data.eventvestor import clinical_trials_free
    
    # import data operations
    from odo import odo
    # import other libraries we will use
    import pandas as pd

.. code:: ipython2

    # Let's use blaze to understand the data a bit using Blaze dshape()
    clinical_trials.dshape




.. parsed-literal::

    dshape("""var * {
      event_id: ?float64,
      asof_date: datetime,
      trade_date: ?datetime,
      symbol: ?string,
      event_type: ?string,
      event_headline: ?string,
      clinical_phase: ?string,
      clinical_scope: ?string,
      clinical_result: ?string,
      product_name: ?string,
      event_rating: ?float64,
      timestamp: datetime,
      sid: ?int64
      }""")



.. code:: ipython2

    # And how many rows are there?
    # N.B. we're using a Blaze function to do this, not len()
    clinical_trials.count()




.. raw:: html

    9118



.. code:: ipython2

    # Let's see what the data looks like. We'll grab the first three rows.
    clinical_trials[:3]




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
          <th>clinical_phase</th>
          <th>clinical_scope</th>
          <th>clinical_result</th>
          <th>product_name</th>
          <th>event_rating</th>
          <th>timestamp</th>
          <th>sid</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>138303</td>
          <td>2007-01-03</td>
          <td>2007-01-03</td>
          <td>IMCL</td>
          <td>Clinical Trials</td>
          <td>ImClone Systems Commences Patient Treatment in...</td>
          <td>Phase I</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>IMC-3G3</td>
          <td>1</td>
          <td>2007-01-04</td>
          <td>3871</td>
        </tr>
        <tr>
          <th>1</th>
          <td>138180</td>
          <td>2007-01-04</td>
          <td>2007-01-04</td>
          <td>DNA</td>
          <td>Clinical Trials</td>
          <td>Genentech Announces Positive Results From Rand...</td>
          <td>Phase II</td>
          <td>NaN</td>
          <td>Positive</td>
          <td>Pertuzumab</td>
          <td>1</td>
          <td>2007-01-05</td>
          <td>24847</td>
        </tr>
        <tr>
          <th>2</th>
          <td>952759</td>
          <td>2007-01-04</td>
          <td>2007-01-04</td>
          <td>VICL</td>
          <td>Clinical Trials</td>
          <td>Vical Initiates Pivotal Phase 3 Trial of Allov...</td>
          <td>Phase III</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>Allovectin-7</td>
          <td>1</td>
          <td>2007-01-05</td>
          <td>8763</td>
        </tr>
      </tbody>
    </table>



Let’s go over the columns: - **event_id**: the unique identifier for
this clinical trial. - **asof_date**: EventVestor’s timestamp of event
capture. - **trade_date**: for event announcements made before trading
ends, trade_date is the same as event_date. For announcements issued
after market close, trade_date is next market open day. - **symbol**:
stock ticker symbol of the affected company. - **event_type**: this
should always be *Clinical Trials*. - **event_headline**: a short
description of the event. - **clinical_phase**: phases include *0, I,
II, III, IV, Pre-Clinical* - **clinical_scope**: types of scope include
*additional indications, all indications, limited indications* -
**clinical_result**: result types include *negative, partial, positive*
- **product_name**: name of the drug being investigated. -
**event_rating**: this is always 1. The meaning of this is uncertain. -
**timestamp**: this is our timestamp on when we registered the data. -
**sid**: the equity’s unique identifier. Use this instead of the symbol.

We’ve done much of the data processing for you. Fields like
``timestamp`` and ``sid`` are standardized across all our Store
Datasets, so the datasets are easy to combine. We have standardized the
``sid`` across all our equity databases.

We can select columns and rows with ease. Below, we’ll fetch all phase-3
announcements. We’ll only display the columns for the sid and the drug
name.

.. code:: ipython2

    phase_three = clinical_trials[clinical_trials.clinical_phase == "Phase III"][['timestamp', 'sid','product_name']].sort('timestamp')
    # When displaying a Blaze Data Object, the printout is automatically truncated to ten rows.
    phase_three




.. raw:: html

    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>timestamp</th>
          <th>sid</th>
          <th>product_name</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>2007-01-05</td>
          <td>8763</td>
          <td>Allovectin-7</td>
        </tr>
        <tr>
          <th>1</th>
          <td>2007-01-09</td>
          <td>1416</td>
          <td>FENTORA</td>
        </tr>
        <tr>
          <th>2</th>
          <td>2007-01-11</td>
          <td>3871</td>
          <td>ERBITUX</td>
        </tr>
        <tr>
          <th>3</th>
          <td>2007-01-25</td>
          <td>8763</td>
          <td>Allovectin-7</td>
        </tr>
        <tr>
          <th>4</th>
          <td>2007-02-09</td>
          <td>24415</td>
          <td>Xibrom</td>
        </tr>
        <tr>
          <th>5</th>
          <td>2007-02-23</td>
          <td>24847</td>
          <td>Avastin</td>
        </tr>
        <tr>
          <th>6</th>
          <td>2007-04-05</td>
          <td>3871</td>
          <td>ERBITUX (Cetuximab)</td>
        </tr>
        <tr>
          <th>7</th>
          <td>2007-04-11</td>
          <td>3871</td>
          <td>ERBITUX</td>
        </tr>
        <tr>
          <th>8</th>
          <td>2007-04-17</td>
          <td>3871</td>
          <td>ERBITUX (Cetuximab)</td>
        </tr>
        <tr>
          <th>9</th>
          <td>2007-04-26</td>
          <td>23846</td>
          <td>BEMA Fentanyl</td>
        </tr>
        <tr>
          <th>10</th>
          <td>2007-04-27</td>
          <td>5847</td>
          <td>Nuvion</td>
        </tr>
      </tbody>
    </table>



Finally, suppose we want a DataFrame of GlaxoSmithKline Phase-III
announcements, sorted in descending order by date:

.. code:: ipython2

    gsk_sid = symbols('GSK').sid

.. code:: ipython2

    gsk = clinical_trials[clinical_trials.sid==gsk_sid].sort('timestamp',ascending=False)
    gsk_df = odo(gsk, pd.DataFrame)
    # now filter down to the Phase 4 trials
    gsk_df = gsk_df[gsk_df.clinical_phase=="Phase III"]
    gsk_df




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
          <th>clinical_phase</th>
          <th>clinical_scope</th>
          <th>clinical_result</th>
          <th>product_name</th>
          <th>event_rating</th>
          <th>timestamp</th>
          <th>sid</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>1937202</td>
          <td>2015-09-27</td>
          <td>2015-09-28</td>
          <td>GSK</td>
          <td>Clinical Trials</td>
          <td>GlaxoSmithKline Reports Positive Results From ...</td>
          <td>Phase III</td>
          <td>NaN</td>
          <td>Positive</td>
          <td>Anoro Ellipta</td>
          <td>1</td>
          <td>2015-09-29 11:17:13.121838</td>
          <td>3242</td>
        </tr>
        <tr>
          <th>3</th>
          <td>1836852</td>
          <td>2015-02-09</td>
          <td>2015-02-09</td>
          <td>GSK</td>
          <td>Clinical Trials</td>
          <td>GlaxoSmithKline and Theravance Initiate Phase ...</td>
          <td>Phase III</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>fluticasone furoate/umeclidinium/vilanterol (</td>
          <td>1</td>
          <td>2015-02-10 00:00:00</td>
          <td>3242</td>
        </tr>
        <tr>
          <th>4</th>
          <td>1817331</td>
          <td>2014-12-18</td>
          <td>2014-12-18</td>
          <td>GSK</td>
          <td>Clinical Trials</td>
          <td>GlaxoSmithKline Reports ZOE-50 Phase 3 Study M...</td>
          <td>Phase III</td>
          <td>NaN</td>
          <td>Positive</td>
          <td>ZOE-50</td>
          <td>1</td>
          <td>2014-12-19 00:00:00</td>
          <td>3242</td>
        </tr>
        <tr>
          <th>5</th>
          <td>1745987</td>
          <td>2014-07-16</td>
          <td>2014-07-16</td>
          <td>GSK</td>
          <td>Clinical Trials</td>
          <td>GlaxoSmithKline &amp; Theravance Initiates Phase I...</td>
          <td>Phase III</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>IMPACT</td>
          <td>1</td>
          <td>2014-07-17 00:00:00</td>
          <td>3242</td>
        </tr>
        <tr>
          <th>6</th>
          <td>1738566</td>
          <td>2014-06-25</td>
          <td>2014-06-25</td>
          <td>GSK</td>
          <td>Clinical Trials</td>
          <td>GlaxoSmithKline Initiates Phase 3 Study with E...</td>
          <td>Phase III</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>Eltrombopag</td>
          <td>1</td>
          <td>2014-06-26 00:00:00</td>
          <td>3242</td>
        </tr>
        <tr>
          <th>7</th>
          <td>1735091</td>
          <td>2014-06-13</td>
          <td>2014-06-13</td>
          <td>GSK</td>
          <td>Clinical Trials</td>
          <td>GlaxoSmithKline Reports Phase 3 PETIT2 Study M...</td>
          <td>Phase III</td>
          <td>NaN</td>
          <td>Positive</td>
          <td>PETIT2</td>
          <td>1</td>
          <td>2014-06-14 00:00:00</td>
          <td>3242</td>
        </tr>
        <tr>
          <th>8</th>
          <td>1734216</td>
          <td>2014-06-11</td>
          <td>2014-06-11</td>
          <td>GSK</td>
          <td>Clinical Trials</td>
          <td>GlaxoSmithKline Reports Positive Results from ...</td>
          <td>Phase III</td>
          <td>NaN</td>
          <td>Positive</td>
          <td>Incruse Ellipta</td>
          <td>1</td>
          <td>2014-06-12 00:00:00</td>
          <td>3242</td>
        </tr>
        <tr>
          <th>10</th>
          <td>1707265</td>
          <td>2014-04-22</td>
          <td>2014-04-22</td>
          <td>GSK</td>
          <td>Clinical Trials</td>
          <td>GlaxoSmithKline and Theravance Starts Phase II...</td>
          <td>Phase III</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>FF/VI</td>
          <td>1</td>
          <td>2014-04-23 00:00:00</td>
          <td>3242</td>
        </tr>
        <tr>
          <th>11</th>
          <td>1700157</td>
          <td>2014-04-02</td>
          <td>2014-04-02</td>
          <td>GSK</td>
          <td>Clinical Trials</td>
          <td>GlaxoSmithKline to Stop MAGE-A3 Cancer Immunot...</td>
          <td>Phase III</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>MAGRITi</td>
          <td>1</td>
          <td>2014-04-03 00:00:00</td>
          <td>3242</td>
        </tr>
        <tr>
          <th>12</th>
          <td>1695526</td>
          <td>2014-03-20</td>
          <td>2014-03-20</td>
          <td>GSK</td>
          <td>Clinical Trials</td>
          <td>GlaxoSmithKline's MAGE-A3 Cancer Immunotherape...</td>
          <td>Phase III</td>
          <td>NaN</td>
          <td>Negative</td>
          <td>MAGE-A3</td>
          <td>1</td>
          <td>2014-03-21 00:00:00</td>
          <td>3242</td>
        </tr>
        <tr>
          <th>13</th>
          <td>1693181</td>
          <td>2014-03-14</td>
          <td>2014-03-14</td>
          <td>GSK</td>
          <td>Clinical Trials</td>
          <td>GlaxoSmithKline &amp; Theravance Reports Positve R...</td>
          <td>Phase III</td>
          <td>NaN</td>
          <td>Positive</td>
          <td>Anoro Ellipta</td>
          <td>1</td>
          <td>2014-03-15 00:00:00</td>
          <td>3242</td>
        </tr>
        <tr>
          <th>15</th>
          <td>1653485</td>
          <td>2013-12-06</td>
          <td>2013-12-06</td>
          <td>GSK</td>
          <td>Clinical Trials</td>
          <td>GlaxoSmithKline and Theravance Announces Posit...</td>
          <td>Phase III</td>
          <td>NaN</td>
          <td>Positive</td>
          <td>Fluticasone Furoate</td>
          <td>1</td>
          <td>2013-12-07 00:00:00</td>
          <td>3242</td>
        </tr>
        <tr>
          <th>17</th>
          <td>1647384</td>
          <td>2013-11-12</td>
          <td>2013-11-12</td>
          <td>GSK</td>
          <td>Clinical Trials</td>
          <td>GlaxoSmithKline Announces Phase III Stability ...</td>
          <td>Phase III</td>
          <td>NaN</td>
          <td>Negative</td>
          <td>Darapladib</td>
          <td>1</td>
          <td>2013-11-13 00:00:00</td>
          <td>3242</td>
        </tr>
        <tr>
          <th>18</th>
          <td>1620476</td>
          <td>2013-09-05</td>
          <td>2013-09-05</td>
          <td>GSK</td>
          <td>Clinical Trials</td>
          <td>GlaxoSmithKline's MAGE-A3 Vaccine Fails to Mee...</td>
          <td>Phase III</td>
          <td>NaN</td>
          <td>Negative</td>
          <td>MAGE-A3</td>
          <td>1</td>
          <td>2013-09-06 00:00:00</td>
          <td>3242</td>
        </tr>
        <tr>
          <th>19</th>
          <td>1521603</td>
          <td>2012-12-19</td>
          <td>2012-12-20</td>
          <td>GSK</td>
          <td>Clinical Trials</td>
          <td>GlaxoSmithKline, Amicus Therapeutics Announce ...</td>
          <td>Phase III</td>
          <td>NaN</td>
          <td>Negative</td>
          <td>Migalastat HCl</td>
          <td>1</td>
          <td>2012-12-20 00:00:00</td>
          <td>3242</td>
        </tr>
        <tr>
          <th>21</th>
          <td>1474291</td>
          <td>2012-08-24</td>
          <td>2012-08-24</td>
          <td>GSK</td>
          <td>Clinical Trials</td>
          <td>GlaxoSmithKline, Theravance Complete Phase III...</td>
          <td>Phase III</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>LAMA/LABA</td>
          <td>1</td>
          <td>2012-08-25 00:00:00</td>
          <td>3242</td>
        </tr>
        <tr>
          <th>22</th>
          <td>1451483</td>
          <td>2012-07-11</td>
          <td>2012-07-11</td>
          <td>GSK</td>
          <td>Clinical Trials</td>
          <td>Shionogi-ViiV Healthcare Reports Positive Init...</td>
          <td>Phase III</td>
          <td>NaN</td>
          <td>Positive</td>
          <td>ING114467</td>
          <td>1</td>
          <td>2012-07-12 00:00:00</td>
          <td>3242</td>
        </tr>
        <tr>
          <th>23</th>
          <td>1451624</td>
          <td>2012-07-11</td>
          <td>2012-07-11</td>
          <td>GSK</td>
          <td>Clinical Trials</td>
          <td>GlaxoSmithKline Reports Positive Results in Ph...</td>
          <td>Phase III</td>
          <td>NaN</td>
          <td>Positive</td>
          <td>Albiglutide</td>
          <td>1</td>
          <td>2012-07-12 00:00:00</td>
          <td>3242</td>
        </tr>
        <tr>
          <th>24</th>
          <td>1448947</td>
          <td>2012-07-02</td>
          <td>2012-07-02</td>
          <td>GSK</td>
          <td>Clinical Trials</td>
          <td>Theravance and GlaxoSmithKline Report Positive...</td>
          <td>Phase III</td>
          <td>NaN</td>
          <td>Positive</td>
          <td>LAMA/LABA</td>
          <td>1</td>
          <td>2012-07-03 00:00:00</td>
          <td>3242</td>
        </tr>
        <tr>
          <th>26</th>
          <td>1414886</td>
          <td>2012-04-03</td>
          <td>2012-04-03</td>
          <td>GSK</td>
          <td>Clinical Trials</td>
          <td>GlaxoSmithKline Reports Further Positive Resul...</td>
          <td>Phase III</td>
          <td>NaN</td>
          <td>Positive</td>
          <td>Albiglutide</td>
          <td>1</td>
          <td>2012-04-04 00:00:00</td>
          <td>3242</td>
        </tr>
        <tr>
          <th>27</th>
          <td>1381734</td>
          <td>2012-01-09</td>
          <td>2012-01-09</td>
          <td>GSK</td>
          <td>Clinical Trials</td>
          <td>GlaxoSmithKline, Theravance Report Initial Res...</td>
          <td>Phase III</td>
          <td>NaN</td>
          <td>Partial</td>
          <td>Relovair</td>
          <td>1</td>
          <td>2012-01-10 00:00:00</td>
          <td>3242</td>
        </tr>
        <tr>
          <th>28</th>
          <td>1376242</td>
          <td>2011-12-15</td>
          <td>2011-12-15</td>
          <td>GSK</td>
          <td>Clinical Trials</td>
          <td>GlaxoSmithKline and Human Genome Initiate Phas...</td>
          <td>Phase III</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>BENLYSTA</td>
          <td>1</td>
          <td>2011-12-16 00:00:00</td>
          <td>3242</td>
        </tr>
        <tr>
          <th>29</th>
          <td>1352859</td>
          <td>2011-10-18</td>
          <td>2011-10-18</td>
          <td>GSK</td>
          <td>Clinical Trials</td>
          <td>GlaxoSmithKline Reports Positive Results from ...</td>
          <td>Phase III</td>
          <td>NaN</td>
          <td>Positive</td>
          <td>RTS,S</td>
          <td>1</td>
          <td>2011-10-19 00:00:00</td>
          <td>3242</td>
        </tr>
        <tr>
          <th>30</th>
          <td>1351145</td>
          <td>2011-10-11</td>
          <td>2011-10-11</td>
          <td>GSK</td>
          <td>Clinical Trials</td>
          <td>GlaxoSmithKline and Pfizer JV Initiates Phase ...</td>
          <td>Phase III</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>Celsentri/Selzentry; emtricitabine/tenofovir</td>
          <td>1</td>
          <td>2011-10-12 00:00:00</td>
          <td>3242</td>
        </tr>
        <tr>
          <th>31</th>
          <td>1336573</td>
          <td>2011-09-12</td>
          <td>2011-09-12</td>
          <td>GSK</td>
          <td>Clinical Trials</td>
          <td>GlaxoSmithKline and Amicus Therapeutics Initia...</td>
          <td>Phase III</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>Amigal</td>
          <td>1</td>
          <td>2011-09-13 00:00:00</td>
          <td>3242</td>
        </tr>
        <tr>
          <th>32</th>
          <td>1335427</td>
          <td>2011-08-15</td>
          <td>2011-08-15</td>
          <td>GSK</td>
          <td>Clinical Trials</td>
          <td>GlaxoSmithKline Reports Positive Results for I...</td>
          <td>Phase III</td>
          <td>NaN</td>
          <td>Positive</td>
          <td>IPX066</td>
          <td>1</td>
          <td>2011-08-16 00:00:00</td>
          <td>3242</td>
        </tr>
        <tr>
          <th>33</th>
          <td>1332195</td>
          <td>2011-07-26</td>
          <td>2011-07-26</td>
          <td>GSK</td>
          <td>Clinical Trials</td>
          <td>GlaxoSmithKline Reports Positive Data from Pro...</td>
          <td>Phase III</td>
          <td>NaN</td>
          <td>Positive</td>
          <td>ENABLE-1</td>
          <td>1</td>
          <td>2011-07-27 00:00:00</td>
          <td>3242</td>
        </tr>
        <tr>
          <th>34</th>
          <td>1301739</td>
          <td>2011-06-02</td>
          <td>2011-06-02</td>
          <td>GSK</td>
          <td>Clinical Trials</td>
          <td>GlaxoSmithKline and Theravance Announce Positi...</td>
          <td>Phase III</td>
          <td>NaN</td>
          <td>Positive</td>
          <td>Relovair</td>
          <td>1</td>
          <td>2011-06-03 00:00:00</td>
          <td>3242</td>
        </tr>
        <tr>
          <th>36</th>
          <td>1249526</td>
          <td>2011-02-07</td>
          <td>2011-02-07</td>
          <td>GSK</td>
          <td>Clinical Trials</td>
          <td>GlaxoSmithKline, Human Genome Announce Positiv...</td>
          <td>Phase III</td>
          <td>NaN</td>
          <td>Positive</td>
          <td>BENLYSTA</td>
          <td>1</td>
          <td>2011-02-08 00:00:00</td>
          <td>3242</td>
        </tr>
        <tr>
          <th>37</th>
          <td>1249289</td>
          <td>2011-02-03</td>
          <td>2011-02-03</td>
          <td>GSK</td>
          <td>Clinical Trials</td>
          <td>GSK and Theravance Announce Progression of LAM...</td>
          <td>Phase III</td>
          <td>NaN</td>
          <td>Positive</td>
          <td>GSK573719/vilanterol</td>
          <td>1</td>
          <td>2011-02-04 00:00:00</td>
          <td>3242</td>
        </tr>
        <tr>
          <th>39</th>
          <td>1188282</td>
          <td>2010-10-21</td>
          <td>2010-10-21</td>
          <td>GSK</td>
          <td>Clinical Trials</td>
          <td>GlaxoSmithKline JV Initiates Phase III Trial f...</td>
          <td>Phase III</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>S/GSK1349572</td>
          <td>1</td>
          <td>2010-10-22 00:00:00</td>
          <td>3242</td>
        </tr>
        <tr>
          <th>42</th>
          <td>1126301</td>
          <td>2010-06-17</td>
          <td>2010-06-17</td>
          <td>GSK</td>
          <td>Clinical Trials</td>
          <td>GlaxoSmithKline Announces Positive Result in P...</td>
          <td>Phase III</td>
          <td>NaN</td>
          <td>Partial</td>
          <td>BENLYSTA</td>
          <td>1</td>
          <td>2010-06-18 00:00:00</td>
          <td>3242</td>
        </tr>
        <tr>
          <th>43</th>
          <td>1126332</td>
          <td>2010-06-17</td>
          <td>2010-06-17</td>
          <td>GSK</td>
          <td>Clinical Trials</td>
          <td>GlaxoSmithKline Announces Positive Phase 3 Res...</td>
          <td>Phase III</td>
          <td>NaN</td>
          <td>Positive</td>
          <td>BENLYSTA</td>
          <td>1</td>
          <td>2010-06-18 00:00:00</td>
          <td>3242</td>
        </tr>
        <tr>
          <th>44</th>
          <td>1089424</td>
          <td>2010-04-20</td>
          <td>2010-04-20</td>
          <td>GSK</td>
          <td>Clinical Trials</td>
          <td>GlaxoSmithKline, Human Genome Announce Failure...</td>
          <td>Phase III</td>
          <td>Limited Indications</td>
          <td>Negative</td>
          <td>BENLYSTA</td>
          <td>1</td>
          <td>2010-04-21 00:00:00</td>
          <td>3242</td>
        </tr>
        <tr>
          <th>46</th>
          <td>1004093</td>
          <td>2009-11-02</td>
          <td>2009-11-02</td>
          <td>GSK</td>
          <td>Clinical Trials</td>
          <td>GlaxoSmithKline Reports Positive Results in Se...</td>
          <td>Phase III</td>
          <td>NaN</td>
          <td>Positive</td>
          <td>BENLYSTA</td>
          <td>1</td>
          <td>2009-11-03 00:00:00</td>
          <td>3242</td>
        </tr>
        <tr>
          <th>47</th>
          <td>1000032</td>
          <td>2009-10-27</td>
          <td>2009-10-27</td>
          <td>GSK</td>
          <td>Clinical Trials</td>
          <td>GlaxoSmithKline Commences Phase III Horizon Pr...</td>
          <td>Phase III</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>COPD</td>
          <td>1</td>
          <td>2009-10-28 00:00:00</td>
          <td>3242</td>
        </tr>
        <tr>
          <th>48</th>
          <td>976852</td>
          <td>2009-10-20</td>
          <td>2009-10-20</td>
          <td>GSK</td>
          <td>Clinical Trials</td>
          <td>GlaxoSmithKline Announces Positive Phase3 Resu...</td>
          <td>Phase III</td>
          <td>NaN</td>
          <td>Positive</td>
          <td>Belimumab</td>
          <td>1</td>
          <td>2009-10-21 00:00:00</td>
          <td>3242</td>
        </tr>
        <tr>
          <th>53</th>
          <td>537694</td>
          <td>2009-02-17</td>
          <td>2009-02-17</td>
          <td>GSK</td>
          <td>Clinical Trials</td>
          <td>GSK Initiates Phase III Programme for Novel Ty...</td>
          <td>Phase III</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>GLP-1</td>
          <td>1</td>
          <td>2009-02-18 00:00:00</td>
          <td>3242</td>
        </tr>
        <tr>
          <th>56</th>
          <td>522433</td>
          <td>2008-12-06</td>
          <td>2008-12-08</td>
          <td>GSK</td>
          <td>Clinical Trials</td>
          <td>GSK, Valeant's Retigabine Reduces Seizures in ...</td>
          <td>Phase III</td>
          <td>NaN</td>
          <td>Positive</td>
          <td>Retigabine</td>
          <td>1</td>
          <td>2008-12-07 00:00:00</td>
          <td>3242</td>
        </tr>
        <tr>
          <th>70</th>
          <td>147654</td>
          <td>2008-02-28</td>
          <td>2008-02-28</td>
          <td>GSK</td>
          <td>Clinical Trials</td>
          <td>GlaxoSmithKline and XenoPort Get Positive Resu...</td>
          <td>Phase III</td>
          <td>NaN</td>
          <td>Positive</td>
          <td>XP13512</td>
          <td>1</td>
          <td>2008-02-29 00:00:00</td>
          <td>3242</td>
        </tr>
      </tbody>
    </table>
    </div>



