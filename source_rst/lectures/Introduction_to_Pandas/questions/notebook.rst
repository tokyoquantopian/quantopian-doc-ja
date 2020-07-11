Exercises: Introduction to pandas
=================================

By Christopher van Hoecke, Maxwell Margenot

Lecture Link :
--------------

https://www.quantopian.com/lectures/introduction-to-pandas

IMPORTANT NOTE:
~~~~~~~~~~~~~~~

This lecture corresponds to the Introduction to Pandas lecture, which is
part of the Quantopian lecture series. This homework expects you to rely
heavily on the code presented in the corresponding lecture. Please copy
and paste regularly from that lecture when starting to work on the
problems, as trying to do them from scratch will likely be too
difficult.

Part of the Quantopian Lecture Series:

-  `www.quantopian.com/lectures <https://www.quantopian.com/lectures>`__
-  `github.com/quantopian/research_public <https://github.com/quantopian/research_public>`__

--------------

.. code:: ipython2

    # Useful Functions
    import numpy as np
    import pandas as pd
    import matplotlib.pyplot as plt

--------------

Exercise 1
==========

a. Series
---------

Given an array of data, please create a pandas Series ``s`` with a
datetime index starting ``2016-01-01``. The index should be daily
frequency and should be the same length as the data.

.. code:: ipython2

    l = np.random.randint(1,100, size=1000)
    s = pd.Series(l)
    
    ## Your code goes here

b. Accessing Series Elements.
-----------------------------

-  Print every other element of the first 50 elements of series ``s``.
-  Find the value associated with the index ``2017-02-20``.

.. code:: ipython2

    ## Your code goes here
    ## Your code goes here

c. Boolean Indexing.
--------------------

In the series ``s``, print all the values between 1 and 3.

.. code:: ipython2

    ## Your code goes here

--------------

#Exercise 2 : Indexing and time series. ###a. Display Print the first
and last 5 elements of the series ``s``.

.. code:: ipython2

    ## Your code goes here
    ## Your code goes here

b. Resampling
~~~~~~~~~~~~~

-  Using the resample method, upsample the daily data to monthly
   frequency. Use the median method so that each monthly value is the
   median price of all the days in that month.
-  Take the daily data and fill in every day, including weekends and
   holidays, using forward-fills.

.. code:: ipython2

    symbol = "CMG"
    start = "2012-01-01"
    end = "2016-01-01"
    prices = get_pricing(symbol, start_date=start, end_date=end, fields="price")
    
    ## Your code goes here

.. code:: ipython2

    ## Your code goes here

--------------

#Exercise 3 : Missing Data - Replace all instances of ``NaN`` using the
forward fill method. - Instead of filling, remove all instances of
``NaN`` from the data.

.. code:: ipython2

    ## Your code goes here

.. code:: ipython2

    ## Your code goes here

--------------

Exercise 4 : Time Series Analysis with pandas
=============================================

a. General Information
----------------------

Print the count, mean, standard deviation, minimum, 25th, 50th, and 75th
percentiles, and the max of our series s.

.. code:: ipython2

    print "Summary Statistics"
    ## Your code goes here

b. Series Operations
--------------------

-  Get the additive and multiplicative returns of this series.
-  Calculate the rolling mean with a 60 day window.
-  Calculate the standard deviation with a 60 day window.

.. code:: ipython2

    data = get_pricing('GE', fields='open_price', start_date='2016-01-01', end_date='2017-01-01')
    
    ## Your code goes here
    ## Your code goes here

.. code:: ipython2

    # Rolling mean
    
    ## Your code goes here
    ## Your code goes here

.. code:: ipython2

    # Rolling Standard Deviation
    
    ## Your code goes here
    ## Your code goes here

--------------

Exercise 5 : DataFrames
=======================

a. Indexing
-----------

Form a DataFrame out of ``dict_data`` with ``l`` as its index.

.. code:: ipython2

    l = {'fifth','fourth', 'third', 'second', 'first'}
    dict_data = {'a' : [1, 2, 3, 4, 5], 'b' : ['L', 'K', 'J', 'M', 'Z'],'c' : np.random.normal(0, 1, 5)}
    
    ## Your code goes here

b. DataFrames Manipulation
--------------------------

-  Concatenate the following two series to form a dataframe.
-  Rename the columns to ``Good Numbers`` and ``Bad Numbers``.
-  Change the index to be a datetime index starting on ``2016-01-01``.

.. code:: ipython2

    s1 = pd.Series([2, 3, 5, 7, 11, 13], name='prime')
    s2 = pd.Series([1, 4, 6, 8, 9, 10], name='other')
    
    ## Your code goes here
    ## Your code goes here
    ## Your code goes here

--------------

Exercise 6 : Accessing DataFrame elements.
==========================================

a. Columns
----------

-  Check the data type of one of the DataFrame’s columns.
-  Print the values associated with time range ``2013-01-01`` to
   ``2013-01-10``.

.. code:: ipython2

    symbol = ["XOM", "BP", "COP", "TOT"]
    start = "2012-01-01"
    end = "2016-01-01"
    prices = get_pricing(symbol, start_date=start, end_date=end, fields="price")
    if isinstance(symbol, list):
        prices.columns = map(lambda x: x.symbol, prices.columns)
    else:
        prices.name = symbol
    
    # Check Type of Data for these two.    
    prices.XOM.head()
    prices.loc[:, 'XOM'].head()

.. code:: ipython2

    ## Your code goes here
    ## Your code goes here

.. code:: ipython2

    ## Your code goes here

--------------

Exercise 7 : Boolean Indexing
=============================

a. Filtering.
-------------

-  Filter pricing data from the last question (stored in ``prices``) to
   only print values where:

   -  BP > 30
   -  XOM < 100
   -  The intersection of both above conditions (BP > 30 **and** XOM <
      100)
   -  The union of the previous composite condition along with TOT
      having no ``nan`` values ((BP > 30 **and** XOM < 100) **or** TOT
      is non-\ ``NaN``).

-  Add a column for TSLA and drop the column for XOM.

.. code:: ipython2

    # Filter the data for prices to only print out values where
    # BP > 30
    
    # XOM < 100
    
    # BP > 30 AND XOM < 100
    
    # The union of (BP > 30 AND XOM < 100) with TOT being non-nan
    
    ## Your code goes here

.. code:: ipython2

    # Add a column for TSLA and drop the column for XOM
    
    ## Your code goes here

b. DataFrame Manipulation (again)
---------------------------------

-  Concatenate these DataFrames.
-  Fill the missing data with 0s

.. code:: ipython2

    # Concatenate these dataframes
    df_1 = get_pricing(['SPY', 'VXX'], start_date=start, end_date=end, fields='price')
    df_2 = get_pricing(['MSFT', 'AAPL', 'GOOG'], start_date=start, end_date=end, fields='price')
    
    ## Your code goes here

.. code:: ipython2

    # Fill GOOG missing data with 0
    
    ## Your code goes here

--------------

Exercise 8 : Time Series Analysis
=================================

a. Summary
----------

-  Print out a summary of the ``prices`` DataFrame from above.
-  Take the log returns and print the first 10 values.
-  Print the multiplicative returns of each company.
-  Normalize and plot the returns from 2014 to 2015.
-  Plot a 60 day window rolling mean of the prices.
-  Plot a 60 day window rolling standfard deviation of the prices.

.. code:: ipython2

    # Print a summary of the 'prices' times series.
    ## Your code goes here

.. code:: ipython2

    # Print the natural log returns of the first 10 values
    ## Your code goes here

.. code:: ipython2

    # Print the Muliplicative returns 
    ## Your code goes here

.. code:: ipython2

    # Normlalize the returns and plot 
    ## Your code goes here

.. code:: ipython2

    # Rolling mean
    ## Your code goes here
    
    # Rolling standard deviation
    ## Your code goes here
    
    # Plotting 
    ## Your code goes here

--------------

Congratulations on completing the Introduction to pandas exercises!

As you learn more about writing trading algorithms and the Quantopian
platform, be sure to check out the daily `Quantopian
Contest <https://www.quantopian.com/contest>`__, in which you can
compete for a cash prize every day.

Start by going through the `Writing a Contest
Algorithm <https://www.quantopian.com/tutorials/contest>`__ Tutorial.

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
