EventVestor: Contract Wins
==========================

In this notebook, we’ll take a look at EventVestor’s *Contract Wins*
dataset, available on the `Quantopian
Store <https://www.quantopian.com/store>`__. This dataset spans January
01, 2007 through the current day, and documents major contract wins by
companies.

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
    from quantopian.interactive.data.eventvestor import contract_win
    # or if you want to import the free dataset, use:
    # from quantopian.data.eventvestor import contract_win_free
    
    # import data operations
    from odo import odo
    # import other libraries we will use
    import pandas as pd

.. code:: ipython2

    # Let's use blaze to understand the data a bit using Blaze dshape()
    contract_win.dshape




.. parsed-literal::

    dshape("""var * {
      event_id: ?float64,
      asof_date: datetime,
      trade_date: ?datetime,
      symbol: ?string,
      event_type: ?string,
      event_headline: ?string,
      contract_amount: ?float64,
      amount_units: ?string,
      contract_entity: ?string,
      event_rating: ?float64,
      timestamp: datetime,
      sid: ?int64
      }""")



.. code:: ipython2

    # And how many rows are there?
    # N.B. we're using a Blaze function to do this, not len()
    contract_win.count()




.. raw:: html

    4135



.. code:: ipython2

    # Let's see what the data looks like. We'll grab the first three rows.
    contract_win[:3]




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
          <th>contract_amount</th>
          <th>amount_units</th>
          <th>contract_entity</th>
          <th>event_rating</th>
          <th>timestamp</th>
          <th>sid</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>907471</td>
          <td>2007-01-03</td>
          <td>2007-01-03</td>
          <td>CECE</td>
          <td>Contract Win</td>
          <td>CECO Environmental Gets Two Orders for $55M Plus</td>
          <td>55.0</td>
          <td>$M</td>
          <td>NaN</td>
          <td>1</td>
          <td>2007-01-04</td>
          <td>1396</td>
        </tr>
        <tr>
          <th>1</th>
          <td>148887</td>
          <td>2007-01-04</td>
          <td>2007-01-04</td>
          <td>ATK</td>
          <td>Contract Win</td>
          <td>Alliant Techsystems Gets $90M Contract from U....</td>
          <td>90.0</td>
          <td>$M</td>
          <td>U.S. Department of Homeland Security</td>
          <td>1</td>
          <td>2007-01-05</td>
          <td>NaN</td>
        </tr>
        <tr>
          <th>2</th>
          <td>908341</td>
          <td>2007-01-04</td>
          <td>2007-01-04</td>
          <td>BCRX</td>
          <td>Contract Win</td>
          <td>BioCryst Pharma Gets $102.6M Contract From US ...</td>
          <td>102.6</td>
          <td>$M</td>
          <td>U.S. Department of Health and Human Services</td>
          <td>1</td>
          <td>2007-01-05</td>
          <td>10905</td>
        </tr>
      </tbody>
    </table>



Let’s go over the columns: - **event_id**: the unique identifier for
this contract win. - **asof_date**: EventVestor’s timestamp of event
capture. - **trade_date**: for event announcements made before trading
ends, trade_date is the same as event_date. For announcements issued
after market close, trade_date is next market open day. - **symbol**:
stock ticker symbol of the affected company. - **event_type**: this
should always be *Contract Win*. - **contract_amount**: the amount of
amount_units the contract is for. - **amount_units**: the currency or
other units for the value of the contract. Most commonly in millions of
dollars. - **contract_entity**: name of the customer, if available -
**event_rating**: this is always 1. The meaning of this is uncertain. -
**timestamp**: this is our timestamp on when we registered the data. -
**sid**: the equity’s unique identifier. Use this instead of the symbol.

We’ve done much of the data processing for you. Fields like
``timestamp`` and ``sid`` are standardized across all our Store
Datasets, so the datasets are easy to combine. We have standardized the
``sid`` across all our equity databases.

We can select columns and rows with ease. Below, we’ll fetch all
contract wins by Boeing. We’ll display only the contract_amount,
amount_units, contract_entity, and timestamp. We’ll sort by date.

.. code:: ipython2

    ba_sid = symbols('BA').sid
    wins = contract_win[contract_win.sid == ba_sid][['timestamp', 'contract_amount','amount_units','contract_entity']].sort('timestamp')
    # When displaying a Blaze Data Object, the printout is automatically truncated to ten rows.
    wins




.. raw:: html

    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>timestamp</th>
          <th>contract_amount</th>
          <th>amount_units</th>
          <th>contract_entity</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>2007-04-19</td>
          <td>2500</td>
          <td>$M</td>
          <td>South Korea</td>
        </tr>
        <tr>
          <th>1</th>
          <td>2007-04-20</td>
          <td>295</td>
          <td>$M</td>
          <td>CIT Aerospace</td>
        </tr>
        <tr>
          <th>2</th>
          <td>2007-04-24</td>
          <td>1600</td>
          <td>$M</td>
          <td>Aviation Capital Group</td>
        </tr>
        <tr>
          <th>3</th>
          <td>2007-04-25</td>
          <td>3600</td>
          <td>$M</td>
          <td>Virgin Atlantic</td>
        </tr>
        <tr>
          <th>4</th>
          <td>2007-04-27</td>
          <td>700</td>
          <td>$M</td>
          <td>SpiceJet</td>
        </tr>
        <tr>
          <th>5</th>
          <td>2007-05-17</td>
          <td>4700</td>
          <td>$M</td>
          <td>TUI Group</td>
        </tr>
        <tr>
          <th>6</th>
          <td>2007-05-30</td>
          <td>2400</td>
          <td>$M</td>
          <td>Russian Airline S7</td>
        </tr>
        <tr>
          <th>7</th>
          <td>2007-06-01</td>
          <td>1900</td>
          <td>$M</td>
          <td>Ryanair Holdings PLC</td>
        </tr>
        <tr>
          <th>8</th>
          <td>2007-06-05</td>
          <td>3000</td>
          <td>$M</td>
          <td>Kuwait Airways</td>
        </tr>
        <tr>
          <th>9</th>
          <td>2007-06-07</td>
          <td>500</td>
          <td>$M</td>
          <td>Philippine Airlines</td>
        </tr>
        <tr>
          <th>10</th>
          <td>2007-06-19</td>
          <td>1420</td>
          <td>$M</td>
          <td>GE Commercial Aviation Services</td>
        </tr>
      </tbody>
    </table>



Finally, suppose we want the above as a DataFrame:

.. code:: ipython2

    ba_df = odo(wins, pd.DataFrame)
    # Printing a pandas DataFrame displays the first 30 and last 30 items, and truncates the middle.
    ba_df




.. raw:: html

    <div style="max-height:1000px;max-width:1500px;overflow:auto;">
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>timestamp</th>
          <th>contract_amount</th>
          <th>amount_units</th>
          <th>contract_entity</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>2007-04-19</td>
          <td>2500.0</td>
          <td>$M</td>
          <td>South Korea</td>
        </tr>
        <tr>
          <th>1</th>
          <td>2007-04-20</td>
          <td>295.0</td>
          <td>$M</td>
          <td>CIT Aerospace</td>
        </tr>
        <tr>
          <th>2</th>
          <td>2007-04-24</td>
          <td>1600.0</td>
          <td>$M</td>
          <td>Aviation Capital Group</td>
        </tr>
        <tr>
          <th>3</th>
          <td>2007-04-25</td>
          <td>3600.0</td>
          <td>$M</td>
          <td>Virgin Atlantic</td>
        </tr>
        <tr>
          <th>4</th>
          <td>2007-04-27</td>
          <td>700.0</td>
          <td>$M</td>
          <td>SpiceJet</td>
        </tr>
        <tr>
          <th>5</th>
          <td>2007-05-17</td>
          <td>4700.0</td>
          <td>$M</td>
          <td>TUI Group</td>
        </tr>
        <tr>
          <th>6</th>
          <td>2007-05-30</td>
          <td>2400.0</td>
          <td>$M</td>
          <td>Russian Airline S7</td>
        </tr>
        <tr>
          <th>7</th>
          <td>2007-06-01</td>
          <td>1900.0</td>
          <td>$M</td>
          <td>Ryanair Holdings PLC</td>
        </tr>
        <tr>
          <th>8</th>
          <td>2007-06-05</td>
          <td>3000.0</td>
          <td>$M</td>
          <td>Kuwait Airways</td>
        </tr>
        <tr>
          <th>9</th>
          <td>2007-06-07</td>
          <td>500.0</td>
          <td>$M</td>
          <td>Philippine Airlines</td>
        </tr>
        <tr>
          <th>10</th>
          <td>2007-06-19</td>
          <td>1420.0</td>
          <td>$M</td>
          <td>GE Commercial Aviation Services</td>
        </tr>
        <tr>
          <th>11</th>
          <td>2007-06-20</td>
          <td>8800.0</td>
          <td>$M</td>
          <td>International Lease Finance Corp</td>
        </tr>
        <tr>
          <th>12</th>
          <td>2007-06-21</td>
          <td>2700.0</td>
          <td>$M</td>
          <td>Air France KLM</td>
        </tr>
        <tr>
          <th>13</th>
          <td>2007-07-01</td>
          <td>2000.0</td>
          <td>$M</td>
          <td>U.S. Air Force</td>
        </tr>
        <tr>
          <th>14</th>
          <td>2007-07-06</td>
          <td>810.0</td>
          <td>$M</td>
          <td>CIT Aerospace</td>
        </tr>
        <tr>
          <th>15</th>
          <td>2007-07-09</td>
          <td>4000.0</td>
          <td>$M</td>
          <td>Air Berlin</td>
        </tr>
        <tr>
          <th>16</th>
          <td>2007-08-03</td>
          <td>523.0</td>
          <td>$M</td>
          <td>AeroSvit</td>
        </tr>
        <tr>
          <th>17</th>
          <td>2007-08-04</td>
          <td>1100.0</td>
          <td>$M</td>
          <td>Air New Zealand</td>
        </tr>
        <tr>
          <th>18</th>
          <td>2007-08-09</td>
          <td>1400.0</td>
          <td>$M</td>
          <td>Cathay Pacific Airways</td>
        </tr>
        <tr>
          <th>19</th>
          <td>2007-08-31</td>
          <td>3100.0</td>
          <td>$M</td>
          <td>Norwegian Air Shuttle ASA</td>
        </tr>
        <tr>
          <th>20</th>
          <td>2007-09-07</td>
          <td>3800.0</td>
          <td>$M</td>
          <td>China Southern Airlines</td>
        </tr>
        <tr>
          <th>21</th>
          <td>2007-09-12</td>
          <td>1100.0</td>
          <td>$M</td>
          <td>US Air Force</td>
        </tr>
        <tr>
          <th>22</th>
          <td>2007-10-17</td>
          <td>1500.0</td>
          <td>$M</td>
          <td>NaN</td>
        </tr>
        <tr>
          <th>23</th>
          <td>2007-11-06</td>
          <td>5000.0</td>
          <td>$M</td>
          <td>LAN Airlines</td>
        </tr>
        <tr>
          <th>24</th>
          <td>2007-11-09</td>
          <td>5200.0</td>
          <td>$M</td>
          <td>Cathay Pacific</td>
        </tr>
        <tr>
          <th>25</th>
          <td>2007-11-12</td>
          <td>3200.0</td>
          <td>$M</td>
          <td>Emirates</td>
        </tr>
        <tr>
          <th>26</th>
          <td>2007-11-14</td>
          <td>523.0</td>
          <td>$M</td>
          <td>transavia.com</td>
        </tr>
        <tr>
          <th>27</th>
          <td>2007-11-23</td>
          <td>716.0</td>
          <td>$M</td>
          <td>KLM Royal Dutch Airlines</td>
        </tr>
        <tr>
          <th>28</th>
          <td>2007-12-05</td>
          <td>1700.0</td>
          <td>$M</td>
          <td>Lion Air</td>
        </tr>
        <tr>
          <th>29</th>
          <td>2007-12-11</td>
          <td>1500.0</td>
          <td>$M</td>
          <td>Babcock &amp; Brown</td>
        </tr>
        <tr>
          <th>...</th>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
        </tr>
        <tr>
          <th>191</th>
          <td>2014-01-07</td>
          <td>8800.0</td>
          <td>$M</td>
          <td>flydubai</td>
        </tr>
        <tr>
          <th>192</th>
          <td>2014-01-21</td>
          <td>3900.0</td>
          <td>$M</td>
          <td>GE Capital Aviation Services</td>
        </tr>
        <tr>
          <th>193</th>
          <td>2014-02-06</td>
          <td>228.0</td>
          <td>$M</td>
          <td>Linhas Aereas de Mocambique</td>
        </tr>
        <tr>
          <th>194</th>
          <td>2014-02-15</td>
          <td>357.5</td>
          <td>$M</td>
          <td>Cargolux Airlines</td>
        </tr>
        <tr>
          <th>195</th>
          <td>2014-03-13</td>
          <td>4400.0</td>
          <td>$M</td>
          <td>SpiceJet Ltd.</td>
        </tr>
        <tr>
          <th>196</th>
          <td>2014-03-20</td>
          <td>830.0</td>
          <td>$M</td>
          <td>Comair Limited</td>
        </tr>
        <tr>
          <th>197</th>
          <td>2014-05-01</td>
          <td>452.0</td>
          <td>$M</td>
          <td>Ryanair</td>
        </tr>
        <tr>
          <th>198</th>
          <td>2014-05-31</td>
          <td>1100.0</td>
          <td>$M</td>
          <td>Japan Transocean Air</td>
        </tr>
        <tr>
          <th>199</th>
          <td>2014-06-17</td>
          <td>1600.0</td>
          <td>$M</td>
          <td>Turkish Airlines</td>
        </tr>
        <tr>
          <th>200</th>
          <td>2014-06-27</td>
          <td>272.0</td>
          <td>$M</td>
          <td>Belarus flag carrier Belavia Airlines</td>
        </tr>
        <tr>
          <th>201</th>
          <td>2014-07-10</td>
          <td>56000.0</td>
          <td>$M</td>
          <td>Emirates Airline</td>
        </tr>
        <tr>
          <th>202</th>
          <td>2014-07-15</td>
          <td>980.0</td>
          <td>$M</td>
          <td>Okay Airways Company</td>
        </tr>
        <tr>
          <th>203</th>
          <td>2014-07-15</td>
          <td>2000.0</td>
          <td>$M</td>
          <td>Avolon</td>
        </tr>
        <tr>
          <th>204</th>
          <td>2014-07-16</td>
          <td>1900.0</td>
          <td>$M</td>
          <td>Intrepid Aviation</td>
        </tr>
        <tr>
          <th>205</th>
          <td>2014-07-17</td>
          <td>1890.0</td>
          <td>$M</td>
          <td>Qatar Airways</td>
        </tr>
        <tr>
          <th>206</th>
          <td>2014-07-17</td>
          <td>152.0</td>
          <td>$M</td>
          <td>NaN</td>
        </tr>
        <tr>
          <th>207</th>
          <td>2014-08-26</td>
          <td>8800.0</td>
          <td>$M</td>
          <td>BOC Aviation</td>
        </tr>
        <tr>
          <th>208</th>
          <td>2014-09-18</td>
          <td>2100.0</td>
          <td>$M</td>
          <td>Avolon</td>
        </tr>
        <tr>
          <th>209</th>
          <td>2014-09-21</td>
          <td>2100.0</td>
          <td>$M</td>
          <td>Ethiopian Airlines</td>
        </tr>
        <tr>
          <th>210</th>
          <td>2014-10-07</td>
          <td>990.0</td>
          <td>$M</td>
          <td>Alaska Airlines</td>
        </tr>
        <tr>
          <th>211</th>
          <td>2014-12-02</td>
          <td>11000.0</td>
          <td>$M</td>
          <td>Ryanair</td>
        </tr>
        <tr>
          <th>212</th>
          <td>2014-12-16</td>
          <td>438.0</td>
          <td>$M</td>
          <td>Jetlines</td>
        </tr>
        <tr>
          <th>213</th>
          <td>2014-12-19</td>
          <td>186.0</td>
          <td>$M</td>
          <td>BOC Aviation</td>
        </tr>
        <tr>
          <th>214</th>
          <td>2014-12-24</td>
          <td>3300.0</td>
          <td>$M</td>
          <td>Kuwait Airways</td>
        </tr>
        <tr>
          <th>215</th>
          <td>2015-01-07</td>
          <td>514.0</td>
          <td>$M</td>
          <td>Air New Zealand</td>
        </tr>
        <tr>
          <th>216</th>
          <td>2015-01-07</td>
          <td>1240.0</td>
          <td>$M</td>
          <td>Qatar Airways</td>
        </tr>
        <tr>
          <th>217</th>
          <td>2015-01-16</td>
          <td>3600.0</td>
          <td>$M</td>
          <td>Air Europa</td>
        </tr>
        <tr>
          <th>218</th>
          <td>2015-02-13</td>
          <td>1600.0</td>
          <td>$M</td>
          <td>Transavia Company</td>
        </tr>
        <tr>
          <th>219</th>
          <td>2015-02-13</td>
          <td>1500.0</td>
          <td>$M</td>
          <td>Korean Air</td>
        </tr>
        <tr>
          <th>220</th>
          <td>2015-03-28</td>
          <td>900.0</td>
          <td>$M</td>
          <td>All Nippon Airways</td>
        </tr>
      </tbody>
    </table>
    <p>221 rows × 4 columns</p>
    </div>


