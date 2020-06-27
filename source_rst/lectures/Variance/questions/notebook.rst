Exercises: Variance
===================

By Christopher van Hoecke, Maxwell Margenot, and Delaney Mackenzie

Lecture Link :
--------------

https://www.quantopian.com/lectures/variance

IMPORTANT NOTE:
~~~~~~~~~~~~~~~

This lecture corresponds to the Variance lecture, which is part of the
Quantopian lecture series. This homework expects you to rely heavily on
the code presented in the corresponding lecture. Please copy and paste
regularly from that lecture when starting to work on the problems, as
trying to do them from scratch will likely be too difficult.

Part of the Quantopian Lecture Series:

-  `www.quantopian.com/lectures <https://www.quantopian.com/lectures>`__
-  `github.com/quantopian/research_public <https://github.com/quantopian/research_public>`__

--------------

.. code:: ipython2

    # Useful Libraries
    import pandas as pd
    import numpy as np
    import matplotlib.pyplot as plt

Data:
^^^^^

.. code:: ipython2

    X = np.random.randint(100, size = 100)

--------------

Exercise 1:
===========

Using the skills aquired in the lecture series, find the following
parameters of the list X above: - Range - Mean Absolute Deviation -
Variance and Standard Deviation - Semivariance and Semideviation -
Target variance (with B = 60)

.. code:: ipython2

    # Range of X
    range_X =
    
    ## Your code goes here
    print 'Range of X: %s' %(range_X)

.. code:: ipython2

    # Mean Absolute Deviation
    # First calculate the value of mu (the mean)
    
    mu = np.mean(X)
    
    ## Your code goes here
    print 'Mean absolute deviation of X:', MAD

.. code:: ipython2

    # Variance and standard deviation
    
    ## Your code goes here
    
    print 'Variance of X:',
    print 'Standard deviation of X:',

.. code:: ipython2

    # Semivariance and semideviation
    
    ## Your code goes here
    
    print 'Semivariance of X:', 
    print 'Semideviation of X:', 

.. code:: ipython2

    # Target variance
    
    ## Your code goes here
    
    print 'Target semivariance of X:',
    print 'Target semideviation of X:', 

--------------

Exercise 2:
===========

Using the skills aquired in the lecture series, find the following
parameters of prices for AT&T stock over a year: - 30 days rolling
variance - 15 days rolling Standard Deviation

.. code:: ipython2

    att = get_pricing('T', fields='open_price', start_date='2016-01-01', end_date='2017-01-01')

.. code:: ipython2

    # Rolling mean
    
    ## Your code goes here

.. code:: ipython2

    # Rolling standard deviation
    
    ## Your code goes here

--------------

Exercise 3 :
============

The portfolio variance is calculated as

.. math:: \text{VAR}_p = \text{VAR}_{s1} (w_1^2) + \text{VAR}_{s2}(w_2^2) + \text{COV}_{S_1, S_2} (2 w_1 w_2)

Where :math:`w_1` and :math:`w_2` are the weights of :math:`S_1` and
:math:`S_2`.

Find values of :math:`w_1` and :math:`w_2` to have a portfolio variance
of 50.

.. code:: ipython2

    asset1 = get_pricing('AAPL', fields='open_price', start_date='2016-01-01', end_date='2017-01-01')
    asset2 = get_pricing('XLF', fields='open_price', start_date='2016-01-01', end_date='2017-01-01')
    
    cov = np.cov(asset1, asset2)[0,1]
    
    w1 = ## Your code goes here.
    w2 = 1 - w1
    
    v1 = np.var(asset1)
    v2 = np.var(asset2)
    
    pvariance = (w1**2)*v1+(w2**2)*v2+(2*w1*w2)*cov
    
    print 'Portfolio variance: ', pvariance

--------------

Congratulations on completing the Variance exercises!

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
