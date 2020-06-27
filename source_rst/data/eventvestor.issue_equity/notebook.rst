EventVestor: Issue Equity
=========================

In this notebook, we’ll take a look at EventVestor’s *Issue Equity*
dataset, available on the `Quantopian
Store <https://www.quantopian.com/store>`__. This dataset spans January
01, 2007 through the current day, and documents events and announcements
covering secondary equity issues by companies.

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
    from quantopian.interactive.data.eventvestor import issue_equity
    # or if you want to import the free dataset, use:
    # from quantopian.interactive.data.eventvestor import issue_equity_free
    
    # import data operations
    from odo import odo
    # import other libraries we will use
    import pandas as pd

.. code:: ipython2

    # Let's use blaze to understand the data a bit using Blaze dshape()
    issue_equity.dshape




.. parsed-literal::

    dshape("""var * {
      event_id: ?float64,
      asof_date: datetime,
      trade_date: ?datetime,
      symbol: ?string,
      event_type: ?string,
      event_headline: ?string,
      issue_amount: ?float64,
      issue_units: ?string,
      issue_stage: ?string,
      event_rating: ?float64,
      timestamp: datetime,
      sid: ?int64
      }""")



.. code:: ipython2

    # And how many rows are there?
    # N.B. we're using a Blaze function to do this, not len()
    issue_equity.count()




.. raw:: html

    15985



.. code:: ipython2

    # Let's see what the data looks like. We'll grab the first three rows.
    issue_equity[:3]




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
          <th>issue_amount</th>
          <th>issue_units</th>
          <th>issue_stage</th>
          <th>event_rating</th>
          <th>timestamp</th>
          <th>sid</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>131337</td>
          <td>2007-01-03</td>
          <td>2007-01-03</td>
          <td>WM</td>
          <td>Issue Equity</td>
          <td>Washington Mutual to convert  its 4% convertib...</td>
          <td>0.0000</td>
          <td>NaN</td>
          <td>NaN</td>
          <td>1</td>
          <td>2007-01-04</td>
          <td>19181</td>
        </tr>
        <tr>
          <th>1</th>
          <td>132940</td>
          <td>2007-01-04</td>
          <td>2007-01-04</td>
          <td>PSA</td>
          <td>Issue Equity</td>
          <td>Public Storage Prices 20M depositary shares at...</td>
          <td>500.0000</td>
          <td>$M</td>
          <td>NaN</td>
          <td>1</td>
          <td>2007-01-05</td>
          <td>24962</td>
        </tr>
        <tr>
          <th>2</th>
          <td>1158828</td>
          <td>2007-01-06</td>
          <td>2007-01-06</td>
          <td>AXTI</td>
          <td>Issue Equity</td>
          <td>AXT Issues 0.9M Shares to Underwriters</td>
          <td>0.8625</td>
          <td>MShares</td>
          <td>Underwriters Exercise</td>
          <td>1</td>
          <td>2007-01-07</td>
          <td>18661</td>
        </tr>
      </tbody>
    </table>



Let’s go over the columns: - **event_id**: the unique identifier for
this event. - **asof_date**: EventVestor’s timestamp of event capture. -
**trade_date**: for event announcements made before trading ends,
trade_date is the same as event_date. For announcements issued after
market close, trade_date is next market open day. - **symbol**: stock
ticker symbol of the affected company. - **event_type**: this should
always be *Issue Equity*. - **event_headline**: a brief description of
the event - **issue_amount**: value of the equity issued in issue_units
- **issue_units**: units of the issue_amount: most commonly millions of
dollars or millions of shares - **issue_stage**: phase of the issue
process: *announcement, closing, pricing, etc.* Note: currently, there
appear to be unrelated entries in this column. We are speaking with the
data vendor to amend this. - **event_rating**: this is always 1. The
meaning of this is uncertain. - **timestamp**: this is our timestamp on
when we registered the data. - **sid**: the equity’s unique identifier.
Use this instead of the symbol.

We’ve done much of the data processing for you. Fields like
``timestamp`` and ``sid`` are standardized across all our Store
Datasets, so the datasets are easy to combine. We have standardized the
``sid`` across all our equity databases.

We can select columns and rows with ease. Below, we’ll fetch all 2015
equity issues smaller than $20M.

.. code:: ipython2

    issues = issue_equity[('2014-12-31' < issue_equity['asof_date']) & 
                            (issue_equity['asof_date'] <'2016-01-01') & 
                            (issue_equity.issue_amount < 20)&
                            (issue_equity.issue_units  == "$M")]
    # When displaying a Blaze Data Object, the printout is automatically truncated to ten rows.
    issues.sort('asof_date')




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
          <th>issue_amount</th>
          <th>issue_units</th>
          <th>issue_stage</th>
          <th>event_rating</th>
          <th>timestamp</th>
          <th>sid</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>1820118</td>
          <td>2015-01-05</td>
          <td>2015-01-05</td>
          <td>STAG</td>
          <td>Issue Equity</td>
          <td>STAG Industrial Issues $18.5 Stock</td>
          <td>18.500</td>
          <td>$M</td>
          <td>Announcement</td>
          <td>1</td>
          <td>2015-01-06</td>
          <td>41271</td>
        </tr>
        <tr>
          <th>1</th>
          <td>1821470</td>
          <td>2015-01-08</td>
          <td>2015-01-09</td>
          <td>CERU</td>
          <td>Issue Equity</td>
          <td>Cerulean Pharma Issues $1M Common Stock in Pri...</td>
          <td>1.000</td>
          <td>$M</td>
          <td>Announcement</td>
          <td>1</td>
          <td>2015-01-09</td>
          <td>46730</td>
        </tr>
        <tr>
          <th>2</th>
          <td>1821647</td>
          <td>2015-01-09</td>
          <td>2015-01-09</td>
          <td>ADMP</td>
          <td>Issue Equity</td>
          <td>Adamis Pharmaceuticals Corp. Prices $10M Commo...</td>
          <td>10.000</td>
          <td>$M</td>
          <td>Pricing</td>
          <td>1</td>
          <td>2015-01-10</td>
          <td>13331</td>
        </tr>
        <tr>
          <th>3</th>
          <td>1822765</td>
          <td>2015-01-13</td>
          <td>2015-01-13</td>
          <td>ALDX</td>
          <td>Issue Equity</td>
          <td>Aldeyra Therapeutics to Issue $7.79M Common Stock</td>
          <td>7.790</td>
          <td>$M</td>
          <td>Announcement</td>
          <td>1</td>
          <td>2015-01-14</td>
          <td>46746</td>
        </tr>
        <tr>
          <th>4</th>
          <td>1823486</td>
          <td>2015-01-15</td>
          <td>2015-01-15</td>
          <td>ALDX</td>
          <td>Issue Equity</td>
          <td>Aldeyra Therapeutics Completes $7.79M Common S...</td>
          <td>7.790</td>
          <td>$M</td>
          <td>Closure</td>
          <td>1</td>
          <td>2015-01-16</td>
          <td>46746</td>
        </tr>
        <tr>
          <th>5</th>
          <td>1823866</td>
          <td>2015-01-16</td>
          <td>2015-01-16</td>
          <td>PRKR</td>
          <td>Issue Equity</td>
          <td>ParkerVision Closes Sale of $1.3M Warrants</td>
          <td>1.300</td>
          <td>$M</td>
          <td>Closure</td>
          <td>1</td>
          <td>2015-01-17</td>
          <td>10485</td>
        </tr>
        <tr>
          <th>6</th>
          <td>1824453</td>
          <td>2015-01-20</td>
          <td>2015-01-20</td>
          <td>ALDX</td>
          <td>Issue Equity</td>
          <td>Aldeyra Therapeutics to Issue $2M Common Stock...</td>
          <td>2.000</td>
          <td>$M</td>
          <td>Announcement</td>
          <td>1</td>
          <td>2015-01-21</td>
          <td>46746</td>
        </tr>
        <tr>
          <th>7</th>
          <td>1824465</td>
          <td>2015-01-20</td>
          <td>2015-01-20</td>
          <td>ANH</td>
          <td>Issue Equity</td>
          <td>Anworth Mortgage Asset Corp. Prices $7.35M Pre...</td>
          <td>7.350</td>
          <td>$M</td>
          <td>Pricing</td>
          <td>1</td>
          <td>2015-01-21</td>
          <td>18380</td>
        </tr>
        <tr>
          <th>8</th>
          <td>1825591</td>
          <td>2015-01-22</td>
          <td>2015-01-22</td>
          <td>ALDX</td>
          <td>Issue Equity</td>
          <td>Aldeyra Therapeutics Completes $2M Common Stoc...</td>
          <td>2.000</td>
          <td>$M</td>
          <td>Closure</td>
          <td>1</td>
          <td>2015-01-23</td>
          <td>46746</td>
        </tr>
        <tr>
          <th>9</th>
          <td>1826474</td>
          <td>2015-01-26</td>
          <td>2015-01-26</td>
          <td>CTP</td>
          <td>Issue Equity</td>
          <td>CTPartners Executive Search Announces $12.5M C...</td>
          <td>12.500</td>
          <td>$M</td>
          <td>Announcement</td>
          <td>1</td>
          <td>2015-01-27</td>
          <td>40551</td>
        </tr>
        <tr>
          <th>10</th>
          <td>1827204</td>
          <td>2015-01-27</td>
          <td>2015-01-27</td>
          <td>WAVX</td>
          <td>Issue Equity</td>
          <td>Wave Systems Corp. Closes $3.6M Common Stock O...</td>
          <td>3.583</td>
          <td>$M</td>
          <td>Announcement</td>
          <td>1</td>
          <td>2015-01-28</td>
          <td>11869</td>
        </tr>
      </tbody>
    </table>



Now suppose we want a DataFrame of the Blaze Data Object above, want to
filter it further down to the announcements only, and we only want the
sid, issue_amount, and the asof_date.

.. code:: ipython2

    df = odo(issues, pd.DataFrame)
    df = df[df.issue_stage == "Announcement"]
    df = df[['sid', 'issue_amount', 'asof_date']].dropna()
    # When printing a pandas DataFrame, the head 30 and tail 30 rows are displayed. The middle is truncated.
    df




.. raw:: html

    <div style="max-height:1000px;max-width:1500px;overflow:auto;">
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>sid</th>
          <th>issue_amount</th>
          <th>asof_date</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>1</th>
          <td>41271</td>
          <td>18.500</td>
          <td>2015-01-05</td>
        </tr>
        <tr>
          <th>2</th>
          <td>46730</td>
          <td>1.000</td>
          <td>2015-01-08</td>
        </tr>
        <tr>
          <th>4</th>
          <td>46746</td>
          <td>7.790</td>
          <td>2015-01-13</td>
        </tr>
        <tr>
          <th>7</th>
          <td>46746</td>
          <td>2.000</td>
          <td>2015-01-20</td>
        </tr>
        <tr>
          <th>10</th>
          <td>40551</td>
          <td>12.500</td>
          <td>2015-01-26</td>
        </tr>
        <tr>
          <th>11</th>
          <td>11869</td>
          <td>3.583</td>
          <td>2015-01-27</td>
        </tr>
        <tr>
          <th>15</th>
          <td>40551</td>
          <td>5.000</td>
          <td>2015-01-30</td>
        </tr>
        <tr>
          <th>16</th>
          <td>46498</td>
          <td>10.000</td>
          <td>2015-02-03</td>
        </tr>
        <tr>
          <th>17</th>
          <td>16176</td>
          <td>8.000</td>
          <td>2015-02-04</td>
        </tr>
        <tr>
          <th>18</th>
          <td>32415</td>
          <td>4.200</td>
          <td>2015-02-04</td>
        </tr>
        <tr>
          <th>19</th>
          <td>22702</td>
          <td>4.200</td>
          <td>2015-02-04</td>
        </tr>
        <tr>
          <th>24</th>
          <td>46309</td>
          <td>10.000</td>
          <td>2015-02-09</td>
        </tr>
        <tr>
          <th>27</th>
          <td>21423</td>
          <td>11.000</td>
          <td>2015-02-11</td>
        </tr>
        <tr>
          <th>30</th>
          <td>35335</td>
          <td>15.000</td>
          <td>2015-02-11</td>
        </tr>
        <tr>
          <th>32</th>
          <td>24470</td>
          <td>4.900</td>
          <td>2015-02-13</td>
        </tr>
        <tr>
          <th>33</th>
          <td>8732</td>
          <td>1.830</td>
          <td>2015-02-17</td>
        </tr>
        <tr>
          <th>35</th>
          <td>32481</td>
          <td>2.500</td>
          <td>2015-02-20</td>
        </tr>
        <tr>
          <th>36</th>
          <td>36189</td>
          <td>3.500</td>
          <td>2015-02-23</td>
        </tr>
        <tr>
          <th>41</th>
          <td>36318</td>
          <td>9.500</td>
          <td>2015-02-26</td>
        </tr>
        <tr>
          <th>42</th>
          <td>41717</td>
          <td>1.800</td>
          <td>2015-02-26</td>
        </tr>
        <tr>
          <th>43</th>
          <td>39350</td>
          <td>17.900</td>
          <td>2015-02-27</td>
        </tr>
        <tr>
          <th>45</th>
          <td>31163</td>
          <td>6.000</td>
          <td>2015-03-03</td>
        </tr>
        <tr>
          <th>47</th>
          <td>39287</td>
          <td>13.500</td>
          <td>2015-03-04</td>
        </tr>
        <tr>
          <th>51</th>
          <td>3428</td>
          <td>5.560</td>
          <td>2015-03-10</td>
        </tr>
        <tr>
          <th>55</th>
          <td>5264</td>
          <td>2.500</td>
          <td>2015-03-13</td>
        </tr>
        <tr>
          <th>58</th>
          <td>17902</td>
          <td>0.400</td>
          <td>2015-03-20</td>
        </tr>
        <tr>
          <th>59</th>
          <td>9621</td>
          <td>10.000</td>
          <td>2015-03-20</td>
        </tr>
        <tr>
          <th>60</th>
          <td>8732</td>
          <td>5.000</td>
          <td>2015-03-20</td>
        </tr>
        <tr>
          <th>63</th>
          <td>34779</td>
          <td>3.400</td>
          <td>2015-03-30</td>
        </tr>
        <tr>
          <th>64</th>
          <td>34779</td>
          <td>3.400</td>
          <td>2015-03-30</td>
        </tr>
        <tr>
          <th>...</th>
          <td>...</td>
          <td>...</td>
          <td>...</td>
        </tr>
        <tr>
          <th>172</th>
          <td>46327</td>
          <td>7.500</td>
          <td>2015-07-16</td>
        </tr>
        <tr>
          <th>173</th>
          <td>29117</td>
          <td>12.000</td>
          <td>2015-07-17</td>
        </tr>
        <tr>
          <th>175</th>
          <td>23290</td>
          <td>16.200</td>
          <td>2015-07-23</td>
        </tr>
        <tr>
          <th>176</th>
          <td>48079</td>
          <td>10.000</td>
          <td>2015-07-27</td>
        </tr>
        <tr>
          <th>177</th>
          <td>42536</td>
          <td>1.500</td>
          <td>2015-07-27</td>
        </tr>
        <tr>
          <th>178</th>
          <td>28471</td>
          <td>8.500</td>
          <td>2015-07-28</td>
        </tr>
        <tr>
          <th>181</th>
          <td>27226</td>
          <td>12.000</td>
          <td>2015-07-29</td>
        </tr>
        <tr>
          <th>182</th>
          <td>45218</td>
          <td>3.000</td>
          <td>2015-07-30</td>
        </tr>
        <tr>
          <th>184</th>
          <td>41233</td>
          <td>4.500</td>
          <td>2015-08-03</td>
        </tr>
        <tr>
          <th>185</th>
          <td>45239</td>
          <td>10.000</td>
          <td>2015-08-03</td>
        </tr>
        <tr>
          <th>186</th>
          <td>26855</td>
          <td>0.350</td>
          <td>2015-08-06</td>
        </tr>
        <tr>
          <th>187</th>
          <td>15506</td>
          <td>0.190</td>
          <td>2015-08-07</td>
        </tr>
        <tr>
          <th>189</th>
          <td>37336</td>
          <td>10.000</td>
          <td>2015-08-13</td>
        </tr>
        <tr>
          <th>190</th>
          <td>41717</td>
          <td>5.000</td>
          <td>2015-08-14</td>
        </tr>
        <tr>
          <th>191</th>
          <td>39270</td>
          <td>2.000</td>
          <td>2015-08-17</td>
        </tr>
        <tr>
          <th>195</th>
          <td>28835</td>
          <td>10.000</td>
          <td>2015-08-19</td>
        </tr>
        <tr>
          <th>197</th>
          <td>42536</td>
          <td>2.100</td>
          <td>2015-08-21</td>
        </tr>
        <tr>
          <th>200</th>
          <td>45995</td>
          <td>17.000</td>
          <td>2015-08-24</td>
        </tr>
        <tr>
          <th>204</th>
          <td>30504</td>
          <td>5.000</td>
          <td>2015-09-01</td>
        </tr>
        <tr>
          <th>205</th>
          <td>21254</td>
          <td>4.500</td>
          <td>2015-09-04</td>
        </tr>
        <tr>
          <th>206</th>
          <td>46343</td>
          <td>15.000</td>
          <td>2015-09-08</td>
        </tr>
        <tr>
          <th>207</th>
          <td>30365</td>
          <td>10.000</td>
          <td>2015-09-08</td>
        </tr>
        <tr>
          <th>209</th>
          <td>1144</td>
          <td>8.580</td>
          <td>2015-09-14</td>
        </tr>
        <tr>
          <th>210</th>
          <td>48026</td>
          <td>10.000</td>
          <td>2015-09-14</td>
        </tr>
        <tr>
          <th>211</th>
          <td>28030</td>
          <td>0.500</td>
          <td>2015-09-17</td>
        </tr>
        <tr>
          <th>213</th>
          <td>48545</td>
          <td>5.000</td>
          <td>2015-09-23</td>
        </tr>
        <tr>
          <th>214</th>
          <td>9700</td>
          <td>5.700</td>
          <td>2015-09-24</td>
        </tr>
        <tr>
          <th>215</th>
          <td>16607</td>
          <td>15.700</td>
          <td>2015-09-24</td>
        </tr>
        <tr>
          <th>216</th>
          <td>28030</td>
          <td>0.450</td>
          <td>2015-09-24</td>
        </tr>
        <tr>
          <th>217</th>
          <td>31461</td>
          <td>3.000</td>
          <td>2015-09-25</td>
        </tr>
      </tbody>
    </table>
    <p>118 rows × 3 columns</p>
    </div>



