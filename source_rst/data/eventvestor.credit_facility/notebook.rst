EventVestor: Credit Facility
============================

In this notebook, we’ll take a look at EventVestor’s *Credit Facility*
dataset, available on the `Quantopian
Store <https://www.quantopian.com/store>`__. This dataset spans January
01, 2007 through the current day, and documents financial events
covering new or extended credit facilities.

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
    from quantopian.interactive.data.eventvestor import credit_facility
    # or if you want to import the free dataset, use:
    # from quantopian.data.eventvestor import credit_facility_free
    
    # import data operations
    from odo import odo
    # import other libraries we will use
    import pandas as pd

.. code:: ipython2

    # Let's use blaze to understand the data a bit using Blaze dshape()
    credit_facility.dshape




.. parsed-literal::

    dshape("""var * {
      event_id: ?float64,
      asof_date: datetime,
      trade_date: ?datetime,
      symbol: ?string,
      event_type: ?string,
      event_headline: ?string,
      credit_amount: ?float64,
      credit_units: ?string,
      event_rating: ?float64,
      timestamp: datetime,
      sid: ?int64
      }""")



.. code:: ipython2

    # And how many rows are there?
    # N.B. we're using a Blaze function to do this, not len()
    credit_facility.count()




.. raw:: html

    8000



.. code:: ipython2

    # Let's see what the data looks like. We'll grab the first three rows.
    credit_facility[:3]




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
          <th>credit_amount</th>
          <th>credit_units</th>
          <th>event_rating</th>
          <th>timestamp</th>
          <th>sid</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>78219</td>
          <td>2007-01-03</td>
          <td>2007-01-03</td>
          <td>NVLS</td>
          <td>Credit Facility</td>
          <td>Novellus Signs $150M Credit Facility</td>
          <td>150</td>
          <td>$M</td>
          <td>1</td>
          <td>2007-01-04</td>
          <td>5509</td>
        </tr>
        <tr>
          <th>1</th>
          <td>961784</td>
          <td>2007-01-04</td>
          <td>2007-01-04</td>
          <td>NAV</td>
          <td>Credit Facility</td>
          <td>Navistar International Gets $1.3B Credit Facil...</td>
          <td>1300</td>
          <td>$M</td>
          <td>1</td>
          <td>2007-01-05</td>
          <td>5199</td>
        </tr>
        <tr>
          <th>2</th>
          <td>145867</td>
          <td>2007-01-10</td>
          <td>2007-01-10</td>
          <td>CCI</td>
          <td>Credit Facility</td>
          <td>Crown Castle Signs $250M Revolving Credit Faci...</td>
          <td>250</td>
          <td>$M</td>
          <td>1</td>
          <td>2007-01-11</td>
          <td>19258</td>
        </tr>
      </tbody>
    </table>



Let’s go over the columns: - **event_id**: the unique identifier for
this event. - **asof_date**: EventVestor’s timestamp of event capture. -
**trade_date**: for event announcements made before trading ends,
trade_date is the same as event_date. For announcements issued after
market close, trade_date is next market open day. - **symbol**: stock
ticker symbol of the affected company. - **event_type**: this should
always be *Credit Facility/Credit facility*. - **event_headline**: a
brief description of the event - **credit_amount**: the amount of
credit_units being availed - **credit_units**: the units for
credit_amount: currency or other value. Most commonly in millions of
USD. - **event_rating**: this is always 1. The meaning of this is
uncertain. - **timestamp**: this is our timestamp on when we registered
the data. - **sid**: the equity’s unique identifier. Use this instead of
the symbol.

We’ve done much of the data processing for you. Fields like
``timestamp`` and ``sid`` are standardized across all our Store
Datasets, so the datasets are easy to combine. We have standardized the
``sid`` across all our equity databases.

We can select columns and rows with ease. Below, we’ll fetch all events
in which $ 200M of credit was availed.

.. code:: ipython2

    twohundreds = credit_facility[(credit_facility.credit_amount==200) & (credit_facility.credit_units=="$M")]
    # When displaying a Blaze Data Object, the printout is automatically truncated to ten rows.
    twohundreds.sort('timestamp')




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
          <th>credit_amount</th>
          <th>credit_units</th>
          <th>event_rating</th>
          <th>timestamp</th>
          <th>sid</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>910312</td>
          <td>2007-01-16</td>
          <td>2007-01-16</td>
          <td>CPNO</td>
          <td>Credit Facility</td>
          <td>Copano Energy Completes Amendment of $200M Cre...</td>
          <td>200</td>
          <td>$M</td>
          <td>1</td>
          <td>2007-01-17</td>
          <td>26783</td>
        </tr>
        <tr>
          <th>1</th>
          <td>965519</td>
          <td>2007-01-22</td>
          <td>2007-01-22</td>
          <td>NAV</td>
          <td>Credit Facility</td>
          <td>Navistar International Raises Credit Facility ...</td>
          <td>200</td>
          <td>$M</td>
          <td>1</td>
          <td>2007-01-23</td>
          <td>5199</td>
        </tr>
        <tr>
          <th>2</th>
          <td>133561</td>
          <td>2007-03-13</td>
          <td>2007-03-13</td>
          <td>SIAL</td>
          <td>Credit Facility</td>
          <td>Sigma-Aldrich Announces New European Credit Fa...</td>
          <td>200</td>
          <td>$M</td>
          <td>1</td>
          <td>2007-03-14</td>
          <td>6872</td>
        </tr>
        <tr>
          <th>3</th>
          <td>131670</td>
          <td>2007-05-04</td>
          <td>2007-05-04</td>
          <td>LEG</td>
          <td>Credit Facility</td>
          <td>Leggett &amp; Platt Increases Multi-Currency Credi...</td>
          <td>200</td>
          <td>$M</td>
          <td>1</td>
          <td>2007-05-05</td>
          <td>4415</td>
        </tr>
        <tr>
          <th>4</th>
          <td>125820</td>
          <td>2007-05-31</td>
          <td>2007-05-31</td>
          <td>ABI</td>
          <td>Credit Facility</td>
          <td>Applera Signs $200M Credit Agreement</td>
          <td>200</td>
          <td>$M</td>
          <td>1</td>
          <td>2007-06-01</td>
          <td>25270</td>
        </tr>
        <tr>
          <th>5</th>
          <td>961810</td>
          <td>2007-06-15</td>
          <td>2007-06-15</td>
          <td>NAV</td>
          <td>Credit Facility</td>
          <td>Navistar International Unit Gets $200M Credit ...</td>
          <td>200</td>
          <td>$M</td>
          <td>1</td>
          <td>2007-06-16</td>
          <td>5199</td>
        </tr>
        <tr>
          <th>6</th>
          <td>78520</td>
          <td>2007-07-25</td>
          <td>2007-07-25</td>
          <td>JBL</td>
          <td>Credit Facility</td>
          <td>Jabil Increases Credit Facility to $1B</td>
          <td>200</td>
          <td>$M</td>
          <td>1</td>
          <td>2007-07-26</td>
          <td>8831</td>
        </tr>
        <tr>
          <th>7</th>
          <td>91869</td>
          <td>2007-09-17</td>
          <td>2007-09-17</td>
          <td>AYE</td>
          <td>Credit Facility</td>
          <td>Allegheny Increases Credit Facility to $400M</td>
          <td>200</td>
          <td>$M</td>
          <td>1</td>
          <td>2007-09-18</td>
          <td>17618</td>
        </tr>
        <tr>
          <th>8</th>
          <td>93042</td>
          <td>2007-09-18</td>
          <td>2007-09-18</td>
          <td>AIV</td>
          <td>Credit Facility</td>
          <td>AIMCO Increases Credit Facility by $200M</td>
          <td>200</td>
          <td>$M</td>
          <td>1</td>
          <td>2007-09-19</td>
          <td>11598</td>
        </tr>
        <tr>
          <th>9</th>
          <td>140305</td>
          <td>2007-10-04</td>
          <td>2007-10-04</td>
          <td>NTRI</td>
          <td>Credit Facility</td>
          <td>Nutri/System Establishes $200M Credit Facility</td>
          <td>200</td>
          <td>$M</td>
          <td>1</td>
          <td>2007-10-05</td>
          <td>21697</td>
        </tr>
        <tr>
          <th>10</th>
          <td>138791</td>
          <td>2007-11-07</td>
          <td>2007-11-07</td>
          <td>WTI</td>
          <td>Credit Facility</td>
          <td>W&amp;T Expands Credit Facility by $200M to $500M</td>
          <td>200</td>
          <td>$M</td>
          <td>1</td>
          <td>2007-11-08</td>
          <td>26986</td>
        </tr>
      </tbody>
    </table>



Finally, suppose we want a DataFrame of that data, but we only want the
symbol, timestamp, and event headline:

.. code:: ipython2

    twohundred_df = odo(twohundreds, pd.DataFrame)
    reduced = twohundred_df[['symbol','event_headline','timestamp']]
    # When printed: pandas DataFrames display the head(30) and tail(30) rows, and truncate the middle.
    reduced




.. raw:: html

    <div style="max-height:1000px;max-width:1500px;overflow:auto;">
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>symbol</th>
          <th>event_headline</th>
          <th>timestamp</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>CPNO</td>
          <td>Copano Energy Completes Amendment of $200M Cre...</td>
          <td>2007-01-17 00:00:00</td>
        </tr>
        <tr>
          <th>1</th>
          <td>NAV</td>
          <td>Navistar International Raises Credit Facility ...</td>
          <td>2007-01-23 00:00:00</td>
        </tr>
        <tr>
          <th>2</th>
          <td>SIAL</td>
          <td>Sigma-Aldrich Announces New European Credit Fa...</td>
          <td>2007-03-14 00:00:00</td>
        </tr>
        <tr>
          <th>3</th>
          <td>LEG</td>
          <td>Leggett &amp; Platt Increases Multi-Currency Credi...</td>
          <td>2007-05-05 00:00:00</td>
        </tr>
        <tr>
          <th>4</th>
          <td>ABI</td>
          <td>Applera Signs $200M Credit Agreement</td>
          <td>2007-06-01 00:00:00</td>
        </tr>
        <tr>
          <th>5</th>
          <td>NAV</td>
          <td>Navistar International Unit Gets $200M Credit ...</td>
          <td>2007-06-16 00:00:00</td>
        </tr>
        <tr>
          <th>6</th>
          <td>JBL</td>
          <td>Jabil Increases Credit Facility to $1B</td>
          <td>2007-07-26 00:00:00</td>
        </tr>
        <tr>
          <th>7</th>
          <td>AYE</td>
          <td>Allegheny Increases Credit Facility to $400M</td>
          <td>2007-09-18 00:00:00</td>
        </tr>
        <tr>
          <th>8</th>
          <td>AIV</td>
          <td>AIMCO Increases Credit Facility by $200M</td>
          <td>2007-09-19 00:00:00</td>
        </tr>
        <tr>
          <th>9</th>
          <td>NTRI</td>
          <td>Nutri/System Establishes $200M Credit Facility</td>
          <td>2007-10-05 00:00:00</td>
        </tr>
        <tr>
          <th>10</th>
          <td>WTI</td>
          <td>W&amp;T Expands Credit Facility by $200M to $500M</td>
          <td>2007-11-08 00:00:00</td>
        </tr>
        <tr>
          <th>11</th>
          <td>PAA</td>
          <td>Plains All American Raises Credit Facility to ...</td>
          <td>2007-11-21 00:00:00</td>
        </tr>
        <tr>
          <th>12</th>
          <td>WLFC</td>
          <td>Willis Lease Unit Gets $200M Credit Facility</td>
          <td>2007-12-14 00:00:00</td>
        </tr>
        <tr>
          <th>13</th>
          <td>CBR</td>
          <td>Ciber Gets $200M Credit Facility</td>
          <td>2008-02-14 00:00:00</td>
        </tr>
        <tr>
          <th>14</th>
          <td>BHMC</td>
          <td>BHMC Gets $200M Revolving Credit Facility</td>
          <td>2008-03-04 00:00:00</td>
        </tr>
        <tr>
          <th>15</th>
          <td>LABL</td>
          <td>Multi-Color Gets New $200M Credit Facility</td>
          <td>2008-03-07 00:00:00</td>
        </tr>
        <tr>
          <th>16</th>
          <td>HXM</td>
          <td>Homex Signs $200M Credit Facility</td>
          <td>2008-07-03 00:00:00</td>
        </tr>
        <tr>
          <th>17</th>
          <td>IMH</td>
          <td>Impac Mortgage Restructures Credit Facility Wi...</td>
          <td>2008-07-04 00:00:00</td>
        </tr>
        <tr>
          <th>18</th>
          <td>CCRN</td>
          <td>Cross Country Gets $200M Credit Commitment fro...</td>
          <td>2008-07-23 00:00:00</td>
        </tr>
        <tr>
          <th>19</th>
          <td>AVA</td>
          <td>Avista Corp. Closes $200M Line of Credit</td>
          <td>2008-12-02 00:00:00</td>
        </tr>
        <tr>
          <th>20</th>
          <td>TAXI</td>
          <td>Medallion Financial Gets $200M Credit Facility</td>
          <td>2008-12-17 00:00:00</td>
        </tr>
        <tr>
          <th>21</th>
          <td>TLB</td>
          <td>Talbots Gets $200M Term Loan Facility</td>
          <td>2009-02-06 00:00:00</td>
        </tr>
        <tr>
          <th>22</th>
          <td>FL</td>
          <td>Foot Locker Gets $200M Credit Facility</td>
          <td>2009-03-25 00:00:00</td>
        </tr>
        <tr>
          <th>23</th>
          <td>IR</td>
          <td>Ingersoll Rand Gets Additional $200M Credit Fa...</td>
          <td>2009-03-31 00:00:00</td>
        </tr>
        <tr>
          <th>24</th>
          <td>ROSE</td>
          <td>Rosetta Resources Expands and Extends Credit F...</td>
          <td>2009-04-10 00:00:00</td>
        </tr>
        <tr>
          <th>25</th>
          <td>WLL</td>
          <td>Whiting Petroleum Increases Credit Facility by...</td>
          <td>2009-04-29 00:00:00</td>
        </tr>
        <tr>
          <th>26</th>
          <td>GLAD</td>
          <td>Gladstone Capital Gets $200M Credit Facility</td>
          <td>2009-05-19 00:00:00</td>
        </tr>
        <tr>
          <th>27</th>
          <td>FCH</td>
          <td>FelCor's Unit Gets $200M Credit Facility</td>
          <td>2009-06-16 00:00:00</td>
        </tr>
        <tr>
          <th>28</th>
          <td>PHM</td>
          <td>Pulte Homes Gets $200M Credit Facility</td>
          <td>2009-06-27 00:00:00</td>
        </tr>
        <tr>
          <th>29</th>
          <td>OHI</td>
          <td>Omega Gets $200M Credit Facility</td>
          <td>2009-07-03 00:00:00</td>
        </tr>
        <tr>
          <th>...</th>
          <td>...</td>
          <td>...</td>
          <td>...</td>
        </tr>
        <tr>
          <th>228</th>
          <td>CLNY</td>
          <td>Colony Financial Raises Credit Facility to $620M</td>
          <td>2014-12-17 00:00:00</td>
        </tr>
        <tr>
          <th>229</th>
          <td>STX</td>
          <td>Seagate Technology Unit Raises Borrowing Capac...</td>
          <td>2015-01-16 00:00:00</td>
        </tr>
        <tr>
          <th>230</th>
          <td>MORE</td>
          <td>Monogram Residential Trust Gets $200M Credit F...</td>
          <td>2015-01-21 00:00:00</td>
        </tr>
        <tr>
          <th>231</th>
          <td>GPT</td>
          <td>Gramercy Property Trust Raises Revolving Borro...</td>
          <td>2015-01-27 00:00:00</td>
        </tr>
        <tr>
          <th>232</th>
          <td>PKD</td>
          <td>Parker Drilling Co. Gets $200M Revolving Credi...</td>
          <td>2015-01-29 00:00:00</td>
        </tr>
        <tr>
          <th>233</th>
          <td>PKY</td>
          <td>Parkway Properties Amends &amp; Raises Credit Faci...</td>
          <td>2015-02-03 00:00:00</td>
        </tr>
        <tr>
          <th>234</th>
          <td>HEES</td>
          <td>H&amp;E Equipment Services Raises Credit Facility ...</td>
          <td>2015-02-10 00:00:00</td>
        </tr>
        <tr>
          <th>235</th>
          <td>AHH</td>
          <td>Armada Hoffler Properties Gets New $200M Unsec...</td>
          <td>2015-02-26 00:00:00</td>
        </tr>
        <tr>
          <th>236</th>
          <td>WRI</td>
          <td>Weingarten Realty Investors Gets $200M Term Loan</td>
          <td>2015-03-03 00:00:00</td>
        </tr>
        <tr>
          <th>237</th>
          <td>KND</td>
          <td>Kindred Healthcare Closes $200M Incremental Te...</td>
          <td>2015-03-11 00:00:00</td>
        </tr>
        <tr>
          <th>238</th>
          <td>VRSN</td>
          <td>VeriSign Gets $200M Credit Facility</td>
          <td>2015-04-02 00:00:00</td>
        </tr>
        <tr>
          <th>239</th>
          <td>PSA</td>
          <td>Public Storage Raises Credit Facility to $500M</td>
          <td>2015-04-03 00:00:00</td>
        </tr>
        <tr>
          <th>240</th>
          <td>IBP</td>
          <td>Installed Building Products Gets $200M Credit ...</td>
          <td>2015-04-30 00:00:00</td>
        </tr>
        <tr>
          <th>241</th>
          <td>HEP</td>
          <td>Holly Energy Partners Unit Amends &amp; Raises Cre...</td>
          <td>2015-05-01 00:00:00</td>
        </tr>
        <tr>
          <th>242</th>
          <td>RGLD</td>
          <td>Royal Gold Amends &amp; Raises Credit Facility to ...</td>
          <td>2015-05-01 00:00:00</td>
        </tr>
        <tr>
          <th>243</th>
          <td>PERY</td>
          <td>Perry Ellis International Raises Credit Facili...</td>
          <td>2015-05-07 00:00:00</td>
        </tr>
        <tr>
          <th>244</th>
          <td>CHS</td>
          <td>Chicoâs FAS Gets $200M Credit Facility</td>
          <td>2015-05-09 00:00:00</td>
        </tr>
        <tr>
          <th>245</th>
          <td>PKY</td>
          <td>Parkway Properties Gets $200M Unsecured Term Loan</td>
          <td>2015-06-30 00:00:00</td>
        </tr>
        <tr>
          <th>246</th>
          <td>GST</td>
          <td>Gastar Exploration Maintains Borrowing Base Un...</td>
          <td>2015-09-01 00:00:00</td>
        </tr>
        <tr>
          <th>247</th>
          <td>IART</td>
          <td>Integra LifeSciences Holdings Corp. Raises Cre...</td>
          <td>2015-09-02 00:00:00</td>
        </tr>
        <tr>
          <th>248</th>
          <td>BID</td>
          <td>Sotheby's Increases Credit Facility Temporaril...</td>
          <td>2015-09-17 00:00:00</td>
        </tr>
        <tr>
          <th>249</th>
          <td>BHMC</td>
          <td>BHMC Gets $200M Revolving Credit Facility</td>
          <td>2015-09-29 11:04:20.713305</td>
        </tr>
        <tr>
          <th>250</th>
          <td>DBRN</td>
          <td>Dress Barn Gets $200M Credit Facility</td>
          <td>2015-09-29 11:04:20.713305</td>
        </tr>
        <tr>
          <th>251</th>
          <td>SOLR</td>
          <td>GT Solar Gets $200M Credit Facility</td>
          <td>2015-09-29 11:04:20.713305</td>
        </tr>
        <tr>
          <th>252</th>
          <td>SOLR</td>
          <td>GT Solar Gets $200M Credit Facility</td>
          <td>2015-09-29 11:04:20.713305</td>
        </tr>
        <tr>
          <th>253</th>
          <td>WACC</td>
          <td>WCA Waste Gets $200M Revolving Credit Facility</td>
          <td>2015-09-29 11:04:20.713305</td>
        </tr>
        <tr>
          <th>254</th>
          <td>YSI</td>
          <td>U-Store-It Gets $200M Term Loans</td>
          <td>2015-09-29 11:04:20.713305</td>
        </tr>
        <tr>
          <th>255</th>
          <td>TVL</td>
          <td>LIN Television Gets $200M New Credit Facility</td>
          <td>2015-09-29 11:04:20.713305</td>
        </tr>
        <tr>
          <th>256</th>
          <td>GY</td>
          <td>GenCorp Amends and Restates $200M Credit Facility</td>
          <td>2015-09-29 11:04:20.713305</td>
        </tr>
        <tr>
          <th>257</th>
          <td>WFR</td>
          <td>MEMC Electronic Materials Gets $200M Credit Fa...</td>
          <td>2015-09-29 11:04:20.713305</td>
        </tr>
      </tbody>
    </table>
    <p>258 rows × 3 columns</p>
    </div>



