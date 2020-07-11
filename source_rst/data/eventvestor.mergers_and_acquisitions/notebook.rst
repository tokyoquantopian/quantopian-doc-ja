EventVestor: Mergers and Acquisitions
=====================================

In this notebook, we’ll take a look at EventVestor’s *Mergers and
Acquisitions* dataset, available on the `Quantopian
Store <https://www.quantopian.com/store>`__. This dataset spans January
01, 2007 through the current day.

Notebook Contents
-----------------

There are two ways to access the data and you’ll find both of them
listed below. Just click on the section you’d like to read through.

-  Interactive overview: This is only available on Research and uses
   blaze to give you access to large amounts of data. Recommended for
   exploration and plotting.
-  Pipeline overview: Data is made available through pipeline which is
   available on both the Research & Backtesting environment. Recommended
   for custom factor development and moving back & forth between
   research/backtesting.

Free samples and limits
~~~~~~~~~~~~~~~~~~~~~~~

One key caveat: we limit the number of results returned from any given
expression to 10,000 to protect against runaway memory usage. To be
clear, you have access to all the data server side. We are limiting the
size of the responses back from Blaze.

There is a *free* version of this dataset as well as a paid one. The
free sample includes data until 2 months prior to the current date.

To access the most up-to-date values for this data set for trading a
live algorithm (as with other partner sets), you need to purchase acess
to the full set.

With preamble in place, let’s get started:

#Interactive Overview ### Accessing the data with Blaze and Interactive
on Research Partner datasets are available on Quantopian Research
through an API service known as `Blaze <http://blaze.pydata.org>`__.
Blaze provides the Quantopian user with a convenient interface to access
very large datasets, in an interactive, generic manner.

Blaze provides an important function for accessing these datasets. Some
of these sets are many millions of records. Bringing that data directly
into Quantopian Research directly just is not viable. So Blaze allows us
to provide a simple querying interface and shift the burden over to the
server side.

It is common to use Blaze to reduce your dataset in size, convert it
over to Pandas and then to use Pandas for further computation,
manipulation and visualization.

Helpful links: \* `Query building for
Blaze <http://blaze.readthedocs.io/en/latest/queries.html>`__ \*
`Pandas-to-Blaze
dictionary <http://blaze.readthedocs.io/en/latest/rosetta-pandas.html>`__
\* `SQL-to-Blaze
dictionary <http://blaze.readthedocs.io/en/latest/rosetta-sql.html>`__.

| Once you’ve limited the size of your Blaze object, you can convert it
  to a Pandas DataFrames using: > ``from odo import odo``
| > ``odo(expr, pandas.DataFrame)``

###To see how this data can be used in your algorithm, search for the
``Pipeline Overview`` section of this notebook or head straight to
Pipeline Overview

.. code:: ipython2

    # import the dataset
    from quantopian.interactive.data.eventvestor import mergers_and_acquisitions_free as dataset
    
    # or if you want to import the free dataset, use:
    #from quantopian.data.eventvestor import buyback_auth_free
    
    # import data operations
    from odo import odo
    # import other libraries we will use
    import pandas as pd
    import matplotlib.pyplot as plt

.. code:: ipython2

    # Let's use blaze to understand the data a bit using Blaze dshape()
    dataset.dshape




.. parsed-literal::

    dshape("""var * {
      event_id: float64,
      mna_type: ?string,
      trade_date: ?datetime,
      symbol: string,
      event_type: ?string,
      event_headline: ?string,
      news_type: ?string,
      firm_type: ?string,
      payment_mode: ?string,
      target_type: ?string,
      is_crossboarder: ?string,
      deal_amount: float64,
      deal_currency: ?string,
      related_ticker: ?string,
      related_entity: ?string,
      event_rating: float64,
      price_pershare: float64,
      premium_pct: float64,
      sid: int64,
      asof_date: datetime,
      timestamp: datetime
      }""")



.. code:: ipython2

    # And how many rows are there?
    # N.B. we're using a Blaze function to do this, not len()
    dataset.count()




.. raw:: html

    16875



.. code:: ipython2

    dataset.asof_date.min()




.. raw:: html

    Timestamp('2007-01-01 00:00:00')



.. code:: ipython2

    # Let's see what the data looks like. We'll grab the first three rows.
    dataset[:3]




.. raw:: html

    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>event_id</th>
          <th>mna_type</th>
          <th>trade_date</th>
          <th>symbol</th>
          <th>event_type</th>
          <th>event_headline</th>
          <th>news_type</th>
          <th>firm_type</th>
          <th>payment_mode</th>
          <th>target_type</th>
          <th>is_crossboarder</th>
          <th>deal_amount</th>
          <th>deal_currency</th>
          <th>related_ticker</th>
          <th>related_entity</th>
          <th>event_rating</th>
          <th>price_pershare</th>
          <th>premium_pct</th>
          <th>sid</th>
          <th>asof_date</th>
          <th>timestamp</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>148664</td>
          <td>Acquisition</td>
          <td>2007-01-03</td>
          <td>FCS</td>
          <td>M&amp;A Announcement</td>
          <td>Fairchild Semiconductor to Acquire System Gene...</td>
          <td>Announcement</td>
          <td>Acquirer</td>
          <td>Mixed Offer</td>
          <td>Public</td>
          <td>Cross Border</td>
          <td>200</td>
          <td>$M</td>
          <td>None</td>
          <td>System General Corporatio</td>
          <td>1</td>
          <td>0</td>
          <td>0</td>
          <td>20486</td>
          <td>2007-01-01</td>
          <td>2007-01-02</td>
        </tr>
        <tr>
          <th>1</th>
          <td>132422</td>
          <td>Acquisition</td>
          <td>2007-01-03</td>
          <td>NUE</td>
          <td>M&amp;A Announcement</td>
          <td>Nucor to Acquire Harris Steel Group for $1B</td>
          <td>Announcement</td>
          <td>Acquirer</td>
          <td>Cash Offer</td>
          <td>Public</td>
          <td>Cross Border</td>
          <td>1070</td>
          <td>$M</td>
          <td>None</td>
          <td>Harris Steel Group</td>
          <td>1</td>
          <td>0</td>
          <td>0</td>
          <td>5488</td>
          <td>2007-01-02</td>
          <td>2007-01-03</td>
        </tr>
        <tr>
          <th>2</th>
          <td>134823</td>
          <td>Acquisition</td>
          <td>2007-01-03</td>
          <td>BNI</td>
          <td>M&amp;A Announcement</td>
          <td>Burlington Northern Unit Acquires Pro-Am Trans...</td>
          <td>Announcement</td>
          <td>Acquirer</td>
          <td>None</td>
          <td>Private</td>
          <td>National</td>
          <td>0</td>
          <td>None</td>
          <td>None</td>
          <td>Pro-Am Transportation Ser</td>
          <td>1</td>
          <td>0</td>
          <td>0</td>
          <td>995</td>
          <td>2007-01-02</td>
          <td>2007-01-03</td>
        </tr>
      </tbody>
    </table>



.. code:: ipython2

    dataset.is_crossboarder.distinct()




.. raw:: html

    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>is_crossboarder</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>None</td>
        </tr>
        <tr>
          <th>1</th>
          <td>National</td>
        </tr>
        <tr>
          <th>2</th>
          <td>Other</td>
        </tr>
        <tr>
          <th>3</th>
          <td>Cross Border</td>
        </tr>
      </tbody>
    </table>



Let’s go over the columns: - **event_id**: the unique identifier for
this buyback authorization. - **asof_date**: EventVestor’s timestamp of
event capture. - **trade_date**: for event announcements made before
trading ends, trade_date is the same as event_date. For announcements
issued after market close, trade_date is next market open day. -
**symbol**: stock ticker symbol of the affected company. -
**event_type**: this should always be *Buyback*. - **event_headline**: a
short description of the event. - **timestamp**: this is our timestamp
on when we registered the data. - **sid**: the equity’s unique
identifier. Use this instead of the symbol. - **news_type**: the type of
news -
``Announcement, Close, Proposal, Termination, Rumor, Rejection, None`` -
**firm_type**: either ``Target`` or ``Acquirer`` - **payment_mode**: the
type of offer made -
``Mixed Offer, Cash Offer, Other, Stock Offer, None`` - **target_type**:
``Public, Private, PE Holding, VC Funded, None`` - **is_crossboarder**:
``None, National, Other, Cross Border`` - **deal_amount,
deal_currency**: the amount of the deal and its corresponding currency -
**related_ticker**: if present, this indicates the ticker being acquired
or that is acquiring - **price_pershare, premium_pct**: the price per
share and the premium paid

We’ve done much of the data processing for you. Fields like
``timestamp`` and ``sid`` are standardized across all our Store
Datasets, so the datasets are easy to combine. We have standardized the
``sid`` across all our equity databases.

We can select columns and rows with ease. Below, we’ll fetch all entries
for Microsoft. We’re really only interested in the buyback amount, the
units, and the date, so we’ll display only those columns.

.. code:: ipython2

    # get the sid for MSFT
    symbols('MSFT')




.. parsed-literal::

    Equity(5061, symbol=u'MSFT', asset_name=u'MICROSOFT CORP', exchange=u'NASDAQ', start_date=Timestamp('2002-01-01 00:00:00+0000', tz='UTC'), end_date=Timestamp('2016-09-06 00:00:00+0000', tz='UTC'), first_traded=None, auto_close_date=Timestamp('2016-09-09 00:00:00+0000', tz='UTC'), exchange_full=u'NASDAQ GLOBAL SELECT MARKET')



.. code:: ipython2

    # knowing that the MSFT sid is 5061:
    msft = dataset[dataset.sid==5061]
    msft[:5]




.. raw:: html

    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>event_id</th>
          <th>mna_type</th>
          <th>trade_date</th>
          <th>symbol</th>
          <th>event_type</th>
          <th>event_headline</th>
          <th>news_type</th>
          <th>firm_type</th>
          <th>payment_mode</th>
          <th>target_type</th>
          <th>is_crossboarder</th>
          <th>deal_amount</th>
          <th>deal_currency</th>
          <th>related_ticker</th>
          <th>related_entity</th>
          <th>event_rating</th>
          <th>price_pershare</th>
          <th>premium_pct</th>
          <th>sid</th>
          <th>asof_date</th>
          <th>timestamp</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>78373</td>
          <td>Acquisition</td>
          <td>2007-05-25</td>
          <td>MSFT</td>
          <td>M&amp;A Announcement</td>
          <td>Microsoft to Acquire aQuantive for $6B</td>
          <td>Announcement</td>
          <td>Acquirer</td>
          <td>Cash Offer</td>
          <td>Public</td>
          <td>National</td>
          <td>6000</td>
          <td>$M</td>
          <td>AQNT</td>
          <td>None</td>
          <td>1</td>
          <td>0.00</td>
          <td>0</td>
          <td>5061</td>
          <td>2007-05-24</td>
          <td>2007-05-25</td>
        </tr>
        <tr>
          <th>1</th>
          <td>78370</td>
          <td>None</td>
          <td>2007-08-13</td>
          <td>MSFT</td>
          <td>M&amp;A Announcement</td>
          <td>Microsoft Completes Acquisition of aQuantive, ...</td>
          <td>Close</td>
          <td>None</td>
          <td>None</td>
          <td>None</td>
          <td>None</td>
          <td>0</td>
          <td>None</td>
          <td>None</td>
          <td>None</td>
          <td>1</td>
          <td>0.00</td>
          <td>0</td>
          <td>5061</td>
          <td>2007-08-13</td>
          <td>2007-08-14</td>
        </tr>
        <tr>
          <th>2</th>
          <td>125294</td>
          <td>Acquisition</td>
          <td>2007-11-12</td>
          <td>MSFT</td>
          <td>M&amp;A Announcement</td>
          <td>Microsoft Intends to Acquire Musiwave</td>
          <td>Announcement</td>
          <td>Acquirer</td>
          <td>Cash Offer</td>
          <td>Private</td>
          <td>Cross Border</td>
          <td>0</td>
          <td>None</td>
          <td>None</td>
          <td>Musiwave SA</td>
          <td>1</td>
          <td>0.00</td>
          <td>0</td>
          <td>5061</td>
          <td>2007-11-12</td>
          <td>2007-11-13</td>
        </tr>
        <tr>
          <th>3</th>
          <td>134224</td>
          <td>None</td>
          <td>2007-12-12</td>
          <td>MSFT</td>
          <td>M&amp;A Announcement</td>
          <td>Microsoft Completes Acquisition of UK based Mu...</td>
          <td>Close</td>
          <td>None</td>
          <td>None</td>
          <td>None</td>
          <td>None</td>
          <td>0</td>
          <td>None</td>
          <td>None</td>
          <td>Multimap</td>
          <td>1</td>
          <td>0.00</td>
          <td>0</td>
          <td>5061</td>
          <td>2007-12-12</td>
          <td>2007-12-13</td>
        </tr>
        <tr>
          <th>4</th>
          <td>137589</td>
          <td>Acquisition</td>
          <td>2008-01-08</td>
          <td>MSFT</td>
          <td>M&amp;A Announcement</td>
          <td>Microsoft to Acquire Norway's Fast Search for ...</td>
          <td>Announcement</td>
          <td>Acquirer</td>
          <td>Cash Offer</td>
          <td>Public</td>
          <td>Cross Border</td>
          <td>1200</td>
          <td>$M</td>
          <td>FAST</td>
          <td>Fast Search &amp; Transfer AS</td>
          <td>1</td>
          <td>3.54</td>
          <td>42</td>
          <td>5061</td>
          <td>2008-01-08</td>
          <td>2008-01-09</td>
        </tr>
      </tbody>
    </table>



#Pipeline Overview

Accessing the data in your algorithms & research
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The only method for accessing partner data within algorithms running on
Quantopian is via the pipeline API.

There are a few factors available using the M&A dataset through
Pipeline. **They allow you to identify securities that are the current
target of an acquisition.** You can also view the payment mode used in
the offer as well as the number of business days since the offer was
made.

.. code:: ipython2

    # Import necessary Pipeline modules
    from quantopian.pipeline import Pipeline
    from quantopian.research import run_pipeline
    from quantopian.pipeline.factors import AverageDollarVolume

Filtering out ANNOUNCED targets
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The following code below shows you how to filter out targets of
acquisitions.

.. code:: ipython2

    from quantopian.pipeline.classifiers.eventvestor import (
        AnnouncedAcqTargetType,
        ProposedAcqTargetType,
    )
    from quantopian.pipeline.factors.eventvestor import (
        BusinessDaysSinceAnnouncedAcquisition,
        BusinessDaysSinceProposedAcquisition
    )
    from quantopian.pipeline.filters.eventvestor import (
        IsAnnouncedAcqTarget
    )
    
    from quantopian.pipeline import Pipeline
    from quantopian.research import run_pipeline
        
    def screen_ma_targets_by_type(target_type='cash'):
        """
        target_type:
            (string) Available options are 'cash', 'stock', 'mixed', 'all'.
            This will filter all offers of type target_type.
        """
        if target_type == 'all':
            return (~IsAnnouncedAcqTarget())
        else:
            if target_type == 'cash':
                filter_offer = 'Cash Offer'
            elif target_type == 'stock':
                filter_offer = 'Stock Offer'
            elif target_type == 'mixed':
                filter_offer = 'Mixed Offer'
            return (~AnnouncedAcqTargetType().eq(filter_offer))
        
    def screen_ma_targets_by_days(days=200):
        """
        days:
            (int) Filters out securities that have had an announcement
            less than X days. So if days is 200, all securities
            that have had an announcement less than 200 days ago will be
            filtered out.
        """
        b_days = BusinessDaysSinceAnnouncedAcquisition()
        return ((b_days > days) | b_days.isnull())
    
    pipe = Pipeline(
        columns={
                'AnnouncedAcqTargetType': AnnouncedAcqTargetType(),
                'BusinessDays': BusinessDaysSinceAnnouncedAcquisition()
                },
        screen=(screen_ma_targets_by_days(60) &
                screen_ma_targets_by_type(target_type='stock'))
    )
    
    output = run_pipeline(pipe, start_date='2016-07-28', end_date='2016-07-28')

Filtering out PROPOSED targets
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you’d also like to filter out proposed targets, please view below

.. code:: ipython2

    """
    Similar functions for M&A Proposals (different from Announcements)
    """
    
    def screen_ma_proposal_targets_by_type(target_type='cash'):
        """
        target_type:
            (string) Available options are 'cash', 'stock', 'mixed', 'all'.
            This will filter all offers of type target_type.
        """
        if target_type == 'all':
            return (ProposedAcqTargetType().isnull() &
                    BusinessDaysSinceProposedAcquisition().isnull())
        if target_type == 'cash':
            filter_offer = 'Cash Offer'
        elif target_type == 'stock':
            filter_offer = 'Stock Offer'
        elif target_type == 'mixed':
            filter_offer = 'Mixed Offer'
        return (~ProposedAcqTargetType().eq(filter_offer))
        
    def screen_ma_proposal_targets_by_days(days=200):
        """
        days:
            (int) Filters out securities that have had an announcement
            less than X days. So if days is 200, all securities
            that have had an announcement less than 200 days ago will be
            filtered out.
        """
        b_days = BusinessDaysSinceProposedAcquisition()
        return ((b_days > days) | b_days.isnull())
