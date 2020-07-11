EventVestor: Issue Debt
=======================

In this notebook, we’ll take a look at EventVestor’s *Issue Debt*
dataset, available on the `Quantopian
Store <https://www.quantopian.com/store>`__. This dataset spans January
01, 2007 through the current day, and documents events and announcements
covering new debt issues by companies.

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
    from quantopian.interactive.data.eventvestor import issue_debt
    # or if you want to import the free dataset, use:
    # from quantopian.interactive.data.eventvestor import issue_debt_free
    
    # import data operations
    from odo import odo
    # import other libraries we will use
    import pandas as pd

.. code:: ipython2

    # Let's use blaze to understand the data a bit using Blaze dshape()
    issue_debt.dshape




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
    issue_debt.count()




.. raw:: html

    13844



.. code:: ipython2

    # Let's see what the data looks like. We'll grab the first three rows.
    issue_debt[:3]




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
          <td>932000</td>
          <td>2007-01-02</td>
          <td>2007-01-02</td>
          <td>NMRX</td>
          <td>Issue Debt</td>
          <td>Numerex to Issue $10M Debt in Private Placement</td>
          <td>10</td>
          <td>$M</td>
          <td>NaN</td>
          <td>1</td>
          <td>2007-01-03</td>
          <td>11002</td>
        </tr>
        <tr>
          <th>1</th>
          <td>149958</td>
          <td>2007-01-02</td>
          <td>2007-01-03</td>
          <td>TECD</td>
          <td>Issue Debt</td>
          <td>Tech Data Completes Sale Of $25M Senior Debent...</td>
          <td>25</td>
          <td>$M</td>
          <td>NaN</td>
          <td>1</td>
          <td>2007-01-03</td>
          <td>7372</td>
        </tr>
        <tr>
          <th>2</th>
          <td>134612</td>
          <td>2007-01-04</td>
          <td>2007-01-04</td>
          <td>RRD</td>
          <td>Issue Debt</td>
          <td>R.R. Donnelley Prices $1.25B Debt Offering</td>
          <td>1250</td>
          <td>$M</td>
          <td>NaN</td>
          <td>1</td>
          <td>2007-01-05</td>
          <td>2248</td>
        </tr>
      </tbody>
    </table>



Let’s go over the columns: - **event_id**: the unique identifier for
this event. - **asof_date**: EventVestor’s timestamp of event capture. -
**trade_date**: for event announcements made before trading ends,
trade_date is the same as event_date. For announcements issued after
market close, trade_date is next market open day. - **symbol**: stock
ticker symbol of the affected company. - **event_type**: this should
always be *Issue Debt*. - **event_headline**: a brief description of the
event - **issue_amount**: value of the debt issued in issue_units -
**issue_units**: units of the issue_amount, most commonly millions of
dollars. - **issue_stage**: phase of the issue process: announcement,
closing, pricing, etc. Note: currently, there appear to be unrelated
entries in this column. We are speaking with the data vendor to amend
this. - **event_rating**: this is always 1. The meaning of this is
uncertain. - **timestamp**: this is our timestamp on when we registered
the data. - **sid**: the equity’s unique identifier. Use this instead of
the symbol.

We’ve done much of the data processing for you. Fields like
``timestamp`` and ``sid`` are standardized across all our Store
Datasets, so the datasets are easy to combine. We have standardized the
``sid`` across all our equity databases.

We can select columns and rows with ease. Below, we’ll fetch all 2015
debt issues smaller than $20M.

.. code:: ipython2

    issues = issue_debt[('2014-12-31' < issue_debt['asof_date']) & 
                                            (issue_debt['asof_date'] <'2016-01-01') & 
                                            (issue_debt.issue_amount < 20)&
                                            (issue_debt.issue_units  == "$M")]
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
          <td>1820589</td>
          <td>2015-01-06</td>
          <td>2015-01-07</td>
          <td>MVC</td>
          <td>Issue Debt</td>
          <td>MVC Capital Draws $15.9M from Credit Facility</td>
          <td>15.900</td>
          <td>$M</td>
          <td>Announcement</td>
          <td>1</td>
          <td>2015-01-07 00:00:00</td>
          <td>21667</td>
        </tr>
        <tr>
          <th>1</th>
          <td>1821690</td>
          <td>2015-01-09</td>
          <td>2015-01-09</td>
          <td>MNTX</td>
          <td>Issue Debt</td>
          <td>Manitex International Issues $15M Notes</td>
          <td>15.000</td>
          <td>$M</td>
          <td>Announcement</td>
          <td>1</td>
          <td>2015-01-10 00:00:00</td>
          <td>27042</td>
        </tr>
        <tr>
          <th>2</th>
          <td>1823391</td>
          <td>2015-01-14</td>
          <td>2015-01-15</td>
          <td>TAT</td>
          <td>Issue Debt</td>
          <td>TransAtlantic Petroluem Issues Additional $1.3...</td>
          <td>1.300</td>
          <td>$M</td>
          <td>NaN</td>
          <td>1</td>
          <td>2015-01-15 00:00:00</td>
          <td>38304</td>
        </tr>
        <tr>
          <th>3</th>
          <td>1830977</td>
          <td>2015-02-03</td>
          <td>2015-02-03</td>
          <td>EBTC</td>
          <td>Issue Debt</td>
          <td>Enterprise Bancorp Issues $15M Notes Due 2030</td>
          <td>15.000</td>
          <td>$M</td>
          <td>Announcement</td>
          <td>1</td>
          <td>2015-02-04 00:00:00</td>
          <td>27032</td>
        </tr>
        <tr>
          <th>4</th>
          <td>1859691</td>
          <td>2015-02-10</td>
          <td>2015-02-11</td>
          <td>TAT</td>
          <td>Issue Debt</td>
          <td>TransAtlantic Petroluem Sells Additional $1M C...</td>
          <td>1.000</td>
          <td>$M</td>
          <td>Announcement</td>
          <td>1</td>
          <td>2015-02-11 00:00:00</td>
          <td>38304</td>
        </tr>
        <tr>
          <th>5</th>
          <td>1860242</td>
          <td>2015-02-12</td>
          <td>2015-02-12</td>
          <td>T</td>
          <td>Issue Debt</td>
          <td>AT&amp;T Issues $2.62B Notes Due 2045</td>
          <td>2.619</td>
          <td>$M</td>
          <td>Announcement</td>
          <td>1</td>
          <td>2015-02-13 00:00:00</td>
          <td>6653</td>
        </tr>
        <tr>
          <th>6</th>
          <td>1860506</td>
          <td>2015-02-20</td>
          <td>2015-02-23</td>
          <td>VGGL</td>
          <td>Issue Debt</td>
          <td>Viggle Gets $0.75M Unsecured Demand Loan from ...</td>
          <td>0.750</td>
          <td>$M</td>
          <td>Announcement</td>
          <td>1</td>
          <td>2015-02-21 00:00:00</td>
          <td>31350</td>
        </tr>
        <tr>
          <th>7</th>
          <td>1852384</td>
          <td>2015-03-03</td>
          <td>2015-03-03</td>
          <td>HPT</td>
          <td>Issue Debt</td>
          <td>Mobi724 Global Solutions Completes Third Tranc...</td>
          <td>0.120</td>
          <td>$M</td>
          <td>NaN</td>
          <td>1</td>
          <td>2015-03-04 00:00:00</td>
          <td>13373</td>
        </tr>
        <tr>
          <th>8</th>
          <td>1845300</td>
          <td>2015-03-03</td>
          <td>2015-03-03</td>
          <td>CNDO</td>
          <td>Issue Debt</td>
          <td>Coronado Biosciences Closes $10M Notes Offering</td>
          <td>10.000</td>
          <td>$M</td>
          <td>Announcement</td>
          <td>1</td>
          <td>2015-03-04 00:00:00</td>
          <td>NaN</td>
        </tr>
        <tr>
          <th>9</th>
          <td>1845300</td>
          <td>2015-03-03</td>
          <td>2015-03-03</td>
          <td>CNDO</td>
          <td>Issue Debt</td>
          <td>Coronado Biosciences Closes $10M Notes Offering</td>
          <td>10.000</td>
          <td>$M</td>
          <td>Announcement</td>
          <td>1</td>
          <td>2015-09-29 10:53:30.635231</td>
          <td>NaN</td>
        </tr>
        <tr>
          <th>10</th>
          <td>1845309</td>
          <td>2015-03-03</td>
          <td>2015-03-03</td>
          <td>MOS</td>
          <td>Issue Debt</td>
          <td>Mobi724 Global Solutions Completes Third Tranc...</td>
          <td>0.120</td>
          <td>$M</td>
          <td>NaN</td>
          <td>1</td>
          <td>2015-03-04 00:00:00</td>
          <td>41462</td>
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
          <th>0</th>
          <td>21667</td>
          <td>15.9000</td>
          <td>2015-01-06</td>
        </tr>
        <tr>
          <th>1</th>
          <td>27042</td>
          <td>15.0000</td>
          <td>2015-01-09</td>
        </tr>
        <tr>
          <th>3</th>
          <td>27032</td>
          <td>15.0000</td>
          <td>2015-02-03</td>
        </tr>
        <tr>
          <th>4</th>
          <td>38304</td>
          <td>1.0000</td>
          <td>2015-02-10</td>
        </tr>
        <tr>
          <th>5</th>
          <td>6653</td>
          <td>2.6190</td>
          <td>2015-02-12</td>
        </tr>
        <tr>
          <th>6</th>
          <td>31350</td>
          <td>0.7500</td>
          <td>2015-02-20</td>
        </tr>
        <tr>
          <th>10</th>
          <td>8151</td>
          <td>0.7900</td>
          <td>2015-03-04</td>
        </tr>
        <tr>
          <th>11</th>
          <td>8151</td>
          <td>9.9540</td>
          <td>2015-03-04</td>
        </tr>
        <tr>
          <th>12</th>
          <td>26723</td>
          <td>9.3200</td>
          <td>2015-03-10</td>
        </tr>
        <tr>
          <th>14</th>
          <td>8151</td>
          <td>6.9710</td>
          <td>2015-03-27</td>
        </tr>
        <tr>
          <th>15</th>
          <td>8151</td>
          <td>13.2290</td>
          <td>2015-03-31</td>
        </tr>
        <tr>
          <th>16</th>
          <td>46731</td>
          <td>14.9000</td>
          <td>2015-04-01</td>
        </tr>
        <tr>
          <th>18</th>
          <td>28632</td>
          <td>15.0000</td>
          <td>2015-04-06</td>
        </tr>
        <tr>
          <th>19</th>
          <td>8151</td>
          <td>7.1730</td>
          <td>2015-04-06</td>
        </tr>
        <tr>
          <th>20</th>
          <td>40551</td>
          <td>12.5000</td>
          <td>2015-04-08</td>
        </tr>
        <tr>
          <th>21</th>
          <td>43721</td>
          <td>0.3200</td>
          <td>2015-04-09</td>
        </tr>
        <tr>
          <th>22</th>
          <td>46731</td>
          <td>11.2000</td>
          <td>2015-04-13</td>
        </tr>
        <tr>
          <th>24</th>
          <td>40197</td>
          <td>5.0000</td>
          <td>2015-04-24</td>
        </tr>
        <tr>
          <th>25</th>
          <td>43552</td>
          <td>5.0000</td>
          <td>2015-05-01</td>
        </tr>
        <tr>
          <th>26</th>
          <td>43367</td>
          <td>1.5500</td>
          <td>2015-05-01</td>
        </tr>
        <tr>
          <th>27</th>
          <td>39803</td>
          <td>12.5000</td>
          <td>2015-05-04</td>
        </tr>
        <tr>
          <th>28</th>
          <td>8151</td>
          <td>2.3840</td>
          <td>2015-05-07</td>
        </tr>
        <tr>
          <th>30</th>
          <td>40197</td>
          <td>3.8000</td>
          <td>2015-05-18</td>
        </tr>
        <tr>
          <th>31</th>
          <td>13046</td>
          <td>15.0000</td>
          <td>2015-05-18</td>
        </tr>
        <tr>
          <th>32</th>
          <td>41233</td>
          <td>5.0000</td>
          <td>2015-05-18</td>
        </tr>
        <tr>
          <th>33</th>
          <td>43721</td>
          <td>1.5000</td>
          <td>2015-05-26</td>
        </tr>
        <tr>
          <th>34</th>
          <td>41717</td>
          <td>1.0500</td>
          <td>2015-06-12</td>
        </tr>
        <tr>
          <th>35</th>
          <td>39627</td>
          <td>15.0000</td>
          <td>2015-06-15</td>
        </tr>
        <tr>
          <th>36</th>
          <td>38327</td>
          <td>11.6000</td>
          <td>2015-06-17</td>
        </tr>
        <tr>
          <th>37</th>
          <td>29204</td>
          <td>10.0000</td>
          <td>2015-06-19</td>
        </tr>
        <tr>
          <th>38</th>
          <td>24475</td>
          <td>1.5000</td>
          <td>2015-06-29</td>
        </tr>
        <tr>
          <th>40</th>
          <td>48506</td>
          <td>11.7000</td>
          <td>2015-06-29</td>
        </tr>
        <tr>
          <th>41</th>
          <td>8151</td>
          <td>13.9100</td>
          <td>2015-06-30</td>
        </tr>
        <tr>
          <th>42</th>
          <td>23901</td>
          <td>2.0000</td>
          <td>2015-07-01</td>
        </tr>
        <tr>
          <th>43</th>
          <td>26904</td>
          <td>18.5000</td>
          <td>2015-07-13</td>
        </tr>
        <tr>
          <th>44</th>
          <td>48998</td>
          <td>1.0000</td>
          <td>2015-07-14</td>
        </tr>
        <tr>
          <th>45</th>
          <td>8151</td>
          <td>14.4270</td>
          <td>2015-07-23</td>
        </tr>
        <tr>
          <th>46</th>
          <td>35352</td>
          <td>10.0000</td>
          <td>2015-07-24</td>
        </tr>
        <tr>
          <th>48</th>
          <td>23714</td>
          <td>5.0000</td>
          <td>2015-08-05</td>
        </tr>
        <tr>
          <th>49</th>
          <td>38327</td>
          <td>2.6316</td>
          <td>2015-08-12</td>
        </tr>
        <tr>
          <th>50</th>
          <td>28414</td>
          <td>6.0000</td>
          <td>2015-08-19</td>
        </tr>
        <tr>
          <th>52</th>
          <td>44040</td>
          <td>6.6000</td>
          <td>2015-08-24</td>
        </tr>
        <tr>
          <th>54</th>
          <td>3343</td>
          <td>14.1000</td>
          <td>2015-09-02</td>
        </tr>
        <tr>
          <th>55</th>
          <td>32481</td>
          <td>1.5000</td>
          <td>2015-09-08</td>
        </tr>
        <tr>
          <th>56</th>
          <td>22039</td>
          <td>10.0000</td>
          <td>2015-09-08</td>
        </tr>
      </tbody>
    </table>
    </div>



