Exercises: Means
================

By Christopher van Hoecke and Max Margenot

Lecture Link :
--------------

https://www.quantopian.com/lectures/means

IMPORTANT NOTE:
~~~~~~~~~~~~~~~

This lecture corresponds to the Means lecture, which is part of the
Quantopian lecture series. This homework expects you to rely heavily on
the code presented in the corresponding lecture. Please copy and paste
regularly from that lecture when starting to work on the problems, as
trying to do them from scratch will likely be too difficult.

Part of the Quantopian Lecture Series:

-  `www.quantopian.com/lectures <https://www.quantopian.com/lectures>`__
-  `github.com/quantopian/research_public <https://github.com/quantopian/research_public>`__

--------------

Key Concepts
------------

.. code:: ipython3

    # Useful Functions
    def mode(l):
        # Count the number of times each element appears in the list
        counts = {}
        for e in l:
            if e in counts:
                counts[e] += 1
            else:
                counts[e] = 1
                
        # Return the elements that appear the most times
        maxcount = 0
        modes = {}
        for (key, value) in counts.items():
            if value > maxcount:
                maxcount = value
                modes = {key}
            elif value == maxcount:
                modes.add(key)
                
        if maxcount > 1 or len(l) == 1:
            return list(modes)
        return 'No mode'

.. code:: ipython3

    # Useful Libraries
    import scipy.stats as stats
    import numpy as np

Data:
^^^^^

.. code:: ipython3

    l=[]
    for x in range(1,100):
        x=np.random.randint(1,100)
        l.append(x)

--------------

Exercise 1 : Arithmetic mean.
=============================

a. Mean of random data set.
---------------------------

Find the mean of the randomly generated data set ``l``.

.. code:: ipython3

    ## Your code goes here       

b. Mean of returns
------------------

Find the mean of the returns of Iteris Inc. (ITI).

.. code:: ipython3

    price = get_pricing('ITI', fields='price', start_date='2005-01-01', end_date='2010-01-01')
    returns = price.pct_change()[1:]
    
    ## Your code goes here.

--------------

Exercise 2 : Median
===================

a. Median of random data set
----------------------------

Find the median of the randomly generated data set ``l``.

.. code:: ipython3

    ## Your code goes here   

b. Median of the returns.
-------------------------

Find the median associated with the returns of Bank of America
Corp. (BAC).

.. code:: ipython3

    price = get_pricing('BAC', fields='open_price', start_date='2005-01-01', end_date='2010-01-01')
    returns = price.pct_change()[1:]
    
    ## Your code goes here

--------------

Exercise 3 : Mode
=================

a. Mode of a random data set.
-----------------------------

Find the mode of the random generated data set ``l``.

.. code:: ipython3

    ## Your code goes here

b. Mode of the returns.
-----------------------

Find the mode associated with the returns of Goldman Sachs Corp. (GS).
*Recall with returns, there may not be any values that appear more than
once.*

.. code:: ipython3

    start = '2014-01-01'
    end = '2015-01-01'
    pricing = get_pricing('GS', fields='price', start_date=start, end_date=end)
    returns = pricing.pct_change()[1:]
    
    ## Your code goes here

--------------

Exercise 4 : Geometric mean
===========================

a. Geometric Mean of random data set.
-------------------------------------

Find the Geometric mean of the random generated data set.

.. code:: ipython3

    ## Your code goes here

b. Geometric Mean of returns.
-----------------------------

Find the Geometric Mean of the price of Citi bank (C) for the last 5
years.

.. code:: ipython3

    price = get_pricing('C', fields='open_price', start_date='2005-01-01', end_date='2010-01-01')
    ## Your code goes here

--------------

Exercise 5 : Harmonic mean.
===========================

a. Harmonic Mean of random data set.
------------------------------------

Find the harmonic mean of the randomly generated data set ``l``.

.. code:: ipython3

    ## Your code goes here 

b. Harmonic Mean of stock returns.
----------------------------------

Find the Harmonic Mean of the financial ETF (XLF) over the last 2 years.

.. code:: ipython3

    ## Your code goes here

--------------

Exercise 6 : Skewness and why it matters.
=========================================

Skewness in a probability distribution is the measure of asymmetry.
Negative skew has fewer low values and a longer left tail, whereas
positive skew has fewer high values and a longer right tail. In asset
pricing, skewness is an important information, naimly in risk
assessment. Knowledge that the market has a 60% chance of going down and
a 40% chance of going up apears helpfull but only if we know the market
is obeying a normal distrubtuion. If we are told that the market will go
up 2% but down 18%, we can see how skewness would give us better
information.

Determine if the returns of SPY from 2010 to 2017 is positivly or
negativly skewed. *Recall a data set is positivly skewed if the mode is
smaller than the median, which is smaller than the mean. A data set is
negativly skewed in the event of the reverse (i.e: the mean is greater
than the median, which is greater than the mode)*

.. code:: ipython3

    price = get_pricing('SPY', fields='volume', start_date='2010-01-01', end_date='2017-01-01')
    returns = price.pct_change()[1:]
    
    ## Your code goes here

We can clearly see positive skewing from the histogram of the returns.
We see fewer higher values and a longer right tail. Plot the histograms
of the returns now.

.. code:: ipython3

    ## Your code here

--------------

Congratulations on completing the Means exercises!

As you learn more about writing trading algorithms and the Quantopian
platform, be sure to check out the daily `Quantopian
Contest <https://www.quantopian.com/contest>`__, in which you can
compete for a cash prize every day.

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
