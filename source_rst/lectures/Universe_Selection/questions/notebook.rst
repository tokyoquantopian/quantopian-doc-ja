#Exercises: Universe Selection

Lecture Link
------------

This exercise notebook refers to this lecture. Please use the lecture
for explanations and sample code.

https://www.quantopian.com/lectures#Universe-Selection

Part of the Quantopian Lecture Series:

-  `www.quantopian.com/lectures <https://www.quantopian.com/lectures>`__
-  `github.com/quantopian/research_public <https://github.com/quantopian/research_public>`__

.. code:: ipython2

    import numpy as np
    import pandas as pd
    import matplotlib.pyplot as plt
    from quantopian.pipeline.classifiers.morningstar import Sector
    from quantopian.pipeline import Pipeline
    from quantopian.pipeline.data.builtin import USEquityPricing
    from quantopian.pipeline.filters import QTradableStocksUS, AtLeastN
    from quantopian.research import run_pipeline
    from quantopian.pipeline.data import morningstar
    from quantopian.pipeline.factors import CustomFactor, AverageDollarVolume

Helper Functions
----------------

.. code:: ipython2

    def calculate_daily_turnover(unstacked):
        return (unstacked
                .diff()        # Get True/False showing where values changed from previous day.
                .iloc[1:]      # Drop first row, which is meaningless after diff().
                .astype(bool)  # diff() coerces from bool -> object :(.  Undo that.
                .groupby(axis=1, level=0)  
                .sum())  

#Exercise 1: Examining the QTradableStocksUS Universe

##a. Initializing the Universe

Set the QTradableStocksUS as your universe by using the
``QTradableStocksUS()`` function.

.. code:: ipython2

    # Your code goes here

b. Finding Asset Composition
----------------------------

Use the pipeline API with the QTradableStocksUS as a screen to find and
print the list of equities included in the QTradableStocksUS on
2016-07-01.

.. code:: ipython2

    # Your code goes here

c. Sector Exposure
------------------

Use the pipeline API with the QtradableStocksUS as a screen to find and
print the sector composition of the universe on 2016-07-01.

.. code:: ipython2

    SECTOR_CODE_NAMES = {
        Sector.BASIC_MATERIALS: 'Basic Materials',
        Sector.CONSUMER_CYCLICAL: 'Consumer Cyclical',
        Sector.FINANCIAL_SERVICES: 'Financial Services',
        Sector.REAL_ESTATE: 'Real Estate',
        Sector.CONSUMER_DEFENSIVE: 'Consumer Defensive',
        Sector.HEALTHCARE: 'Healthcare',
        Sector.UTILITIES: 'Utilities',
        Sector.COMMUNICATION_SERVICES: 'Communication Services',
        Sector.ENERGY: 'Energy',
        Sector.INDUSTRIALS: 'Industrials',
        Sector.TECHNOLOGY: 'Technology',
        -1 : 'Misc'
    }
    
    # Your code goes here

d. Turnover Rate
----------------

Use the pipeline API with the QtradableStocksUS as a screen and the
``calculate_daily_turnover`` helper function to find and plot the
turnover of the universe during 2016.

.. code:: ipython2

    # Your code goes here

Exercise 2: Examining Tradability
=================================

a. NetIncome 1500
-----------------

Create a universe consisting of the top 1500 equities by net income then
find and print the list of equities included in the universe on
2016-07-01.

.. code:: ipython2

    # Your code goes here

b. Measuring Tradability
------------------------

Find the average 200 day average dollar volume of the NetIncome 1500
universe using the ``AverageDollarVolume`` built in factor and compare
to that of the QTradableStocksUS.

.. code:: ipython2

    # Your code goes here

Exercise 3: Sector Balance
==========================

a. Dividend 1500
----------------

Create a universe consisting of the top 1500 equities by dividend yield
then find and print the list of equities included in the this universe
on 2016-07-01.

.. code:: ipython2

    # Your code goes here

b. Dividend 1500 Sector Composition
-----------------------------------

Find and print the sector composition of the universe on 2016-07-01.

.. code:: ipython2

    SECTOR_CODE_NAMES = {
        Sector.BASIC_MATERIALS: 'Basic Materials',
        Sector.CONSUMER_CYCLICAL: 'Consumer Cyclical',
        Sector.FINANCIAL_SERVICES: 'Financial Services',
        Sector.REAL_ESTATE: 'Real Estate',
        Sector.CONSUMER_DEFENSIVE: 'Consumer Defensive',
        Sector.HEALTHCARE: 'Healthcare',
        Sector.UTILITIES: 'Utilities',
        Sector.COMMUNICATION_SERVICES: 'Communication Services',
        Sector.ENERGY: 'Energy',
        Sector.INDUSTRIALS: 'Industrials',
        Sector.TECHNOLOGY: 'Technology',
        -1 : 'Misc'
    }
    
    # Your code goes here

Exercise 4: Turnover Smoothing
==============================

a. PE 1500
----------

Create a universe consisting of the top 1500 equities by price to
earnings ratio then find and print the list of equities included in the
this universe on 2016-07-01.

.. code:: ipython2

    # Your code goes here

b. PE 1500 Turnover
-------------------

Use the ``calculate_daily_turnover`` helper function to find and plot
the turnover of the PE 1500 universe during 2016. Compare the average to
that of the QTradableStocksUS.

.. code:: ipython2

    # Your code goes here

c. Smoothing the PE 1500
------------------------

Using ``AtLeastN``, apply a smoothing function to the PE 1500 to reduce
turnover noise and find the new mean turnover.

.. code:: ipython2

    # Your code goes here

--------------

Congratulations on completing the Universe Selection exercises!

As you learn more about writing trading models and the Quantopian
platform, enter a daily `Quantopian
Contest <https://www.quantopian.com/contest>`__. Your strategy will be
evaluated for a cash prize every day.

Start by going through the `Writing a Contest
Algorithm <https://www.quantopian.com/tutorials/contest>`__ tutorial.

*This presentation is for informational purposes only and does not
constitute an offer to sell, a solicitation to buy, or a recommendation
for any security; nor does it constitute an offer to provide investment
advisory or other services by Quantopian, Inc. (“Quantopian”). Nothing
contained herein constitutes investment advice or offers any opinion
with respect to the suitability of any security, and any views expressed
herein should not be taken as advice to buy, sell, or hold any security
or as an endorsement of any security or company. In preparing the
information contained herein, Quantopian, Inc. has not taken into
account the investment needs, objectives, and financial circumstances of
any particular investor. Any views expressed and data illustrated herein
were prepared based upon information, believed to be reliable, available
to Quantopian, Inc. at the time of publication. Quantopian makes no
guarantees as to their accuracy or completeness. All information is
subject to change and may quickly become unreliable for various reasons,
including changes in market conditions or economic circumstances.*
