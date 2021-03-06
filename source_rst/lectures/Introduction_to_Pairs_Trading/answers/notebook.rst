Exercises: Introduction to Pairs Trading - Answer Key
=====================================================

By Chris Fenaroli, Delaney Mackenzie, and Maxwell Margenot

Lecture Link :
--------------

https://www.quantopian.com/lectures/introduction-to-pairs-trading

IMPORTANT NOTE:
~~~~~~~~~~~~~~~

These exercises correspond to the Introduction to Pairs Trading lecture,
which is part of the Quantopian Lecture Series. It is meant to cement a
few concepts presented in the lecture, specifically cointegration and
construction of cointegrated pair spreads.

We expect you to rely heavily on the code presented in the corresponding
lecture. Please copy and paste regularly from that lecture when starting
to work on the problems, as trying to do them from scratch will likely
be too difficult. There is an answer key available for this problem set.
Please email us at feedback@quantopian.com for results.

Part of the Quantopian Lecture Series:

-  `www.quantopian.com/lectures <https://www.quantopian.com/lectures>`__
-  `github.com/quantopian/research_public <https://github.com/quantopian/research_public>`__

--------------

Key Concepts
------------

.. code:: ipython2

    # Useful Functions
    def find_cointegrated_pairs(data):
        n = data.shape[1]
        score_matrix = np.zeros((n, n))
        pvalue_matrix = np.ones((n, n))
        keys = data.keys()
        pairs = []
        for i in range(n):
            for j in range(i+1, n):
                S1 = data[keys[i]]
                S2 = data[keys[j]]
                result = coint(S1, S2)
                score = result[0]
                pvalue = result[1]
                score_matrix[i, j] = score
                pvalue_matrix[i, j] = pvalue
                if pvalue < 0.05:
                    pairs.append((keys[i], keys[j]))
        return score_matrix, pvalue_matrix, pairs

.. code:: ipython2

    # Useful Libraries
    import numpy as np
    import pandas as pd
    
    import statsmodels
    import statsmodels.api as sm
    from statsmodels.tsa.stattools import coint, adfuller
    # just set the seed for the random number generator
    np.random.seed(107)
    
    import matplotlib.pyplot as plt

--------------

#Exercise 1: Testing Artificial Examples

We’ll use some artificially generated series first as they are much
cleaner and easier to work with. In general when learning or developing
a new technique, use simulated data to provide a clean environment.
Simulated data also allows you to control the level of noise and
difficulty level for your model.

##a. Cointegration Test I

Determine whether the following two artificial series :math:`A` and
:math:`B` are cointegrated using the ``coint()`` function and a
reasonable confidence level.

.. code:: ipython2

    A_returns = np.random.normal(0, 1, 100)
    A = pd.Series(np.cumsum(A_returns), name='X') + 50
    
    some_noise = np.random.exponential(1, 100)
     
    B = A - 7 + some_noise
    
    #Your code goes here

.. code:: ipython2

    ## answer key ##
    score, pvalue, _ = coint(A,B)
    
    confidence_level = 0.05
    
    if pvalue < confidence_level:
        print ("A and B are cointegrated")
        print pvalue
    else:
        print ("A and B are not cointegrated")
        print pvalue
        
    A.name = "A"
    B.name = "B"
    pd.concat([A, B], axis=1).plot();


.. parsed-literal::

    A and B are cointegrated
    6.96867595624e-15



.. image:: notebook_files/notebook_7_1.png


##b. Cointegration Test II

Determine whether the following two artificial series :math:`C` and
:math:`D` are cointegrated using the ``coint()`` function and a
reasonable confidence level.

.. code:: ipython2

    C_returns = np.random.normal(1, 1, 100) 
    C = pd.Series(np.cumsum(C_returns), name='X') + 100
    
    D_returns = np.random.normal(2, 1, 100)
    D = pd.Series(np.cumsum(D_returns), name='X') + 100
    
    #Your code goes here

.. code:: ipython2

    ## answer key ##
    score, pvalue, _ = coint(C,D)
    
    confidence_level = 0.05
    
    if pvalue < confidence_level:
        print ("C and D are cointegrated")
        print pvalue
    else:
        print ("C and D are not cointegrated")
        print pvalue
    
    C.name = "C"
    D.name = "D"
    pd.concat([C, D], axis=1).plot();


.. parsed-literal::

    C and D are not cointegrated
    0.487261538359



.. image:: notebook_files/notebook_10_1.png


--------------

#Exercise 2: Testing Real Examples

##a. Real Cointegration Test I

Determine whether the following two assets ``UAL`` and ``AAL`` were
cointegrated during 2015 using the ``coint()`` function and a reasonable
confidence level.

.. code:: ipython2

    ual = get_pricing('UAL', fields=['price'], 
                            start_date='2015-01-01', end_date='2016-01-01')['price']
    aal = get_pricing('AAL', fields=['price'], 
                            start_date='2015-01-01', end_date='2016-01-01')['price']
    
    #Your code goes here

.. code:: ipython2

    ## answer key ##
    score, pvalue, _ = coint(ual, aal)
    
    confidence_level = 0.05
    
    if pvalue < confidence_level:
        print ("UAL and AAL are cointegrated")
        print pvalue
    else:
        print ("UAL and AAL are not cointegrated")
        print pvalue
    
    ual.name = "UAL"
    aal.name = "AAL"
    pd.concat([ual, aal], axis=1).plot();


.. parsed-literal::

    UAL and AAL are not cointegrated
    0.11339061615



.. image:: notebook_files/notebook_14_1.png


##b. Real Cointegration Test II

Determine whether the following two assets ``FCAU`` and ``HMC`` were
cointegrated during 2015 using the ``coint()`` function and a reasonable
confidence level.

.. code:: ipython2

    fcau = get_pricing('FCAU', fields=['price'], 
                            start_date='2015-01-01', end_date='2016-01-01')['price']
    hmc = get_pricing('HMC', fields=['price'], 
                            start_date='2015-01-01', end_date='2016-01-01')['price']
    
    #Your code goes here

.. code:: ipython2

    ## answer key ##
    confidence_level = 0.05
    
    score, pvalue, _ = coint(fcau, hmc)
    
    if pvalue < confidence_level:
        print ("FCAU and HMC are cointegrated")
        print pvalue
    else:
        print ("FCAU and HMC are not cointegrated")
        print pvalue
    
    fcau.name = "FCAU"
    hmc.name = "HMC"
    pd.concat([fcau, hmc], axis=1).plot();


.. parsed-literal::

    FCAU and HMC are cointegrated
    0.013687770819



.. image:: notebook_files/notebook_17_1.png


--------------

Exercise 3: Searching for Cointegrated Pairs
============================================

Use the ``find_cointegrated_pairs`` function, defined in the “Helper
Functions” section above, to find any cointegrated pairs among a set of
metal and mining securities.

Note that not all of these securities in this exercise are within the
`QTradableStocksUS <https://www.quantopian.com/posts/working-on-our-best-universe-yet-qtradablestocksus>`__.
As you continue your development, focus on securities within the QTU to
be eligible for an `allocation <https://quantopian.com/allocation>`__.

.. code:: ipython2

    symbol_list = ['MTRN', 'CMP', 'TRQ', 'SCCO', 'HCLP','SPY']
    prices_df = get_pricing(symbol_list, fields=['price']
                                   , start_date='2015-01-01', end_date='2016-01-01')['price']
    prices_df.columns = map(lambda x: x.symbol, prices_df.columns)
    
    #Your code goes here

.. code:: ipython2

    ## answer key ##
    scores, pvalues, pairs = find_cointegrated_pairs(prices_df)
    import seaborn
    seaborn.heatmap(pvalues, xticklabels=symbol_list, yticklabels=symbol_list, cmap='RdYlGn_r' 
                    , mask = (pvalues >= 0.99)
                    )
    print pairs


.. parsed-literal::

    [(u'MTRN', u'SCCO')]



.. image:: notebook_files/notebook_21_1.png


--------------

#Exercise 4: Out of Sample Validation

##a. Calculating the Spread

Using pricing data from 2015, construct a linear regression to find a
coefficient for the linear combination of ``MTRN`` and ``SCCO`` that
makes their spread stationary.

.. code:: ipython2

    S1 = prices_df['MTRN']
    S2 = prices_df['SCCO']
    
    #Your code goes here

.. code:: ipython2

    ## answer key ##
    S1 = sm.add_constant(S1)
    results = sm.OLS(S2, S1).fit()
    b = results.params['MTRN']
    S1 = S1['MTRN']
    
    print b
    spread = S2 - b * S1
    print "p-value for in-sample stationarity: ", adfuller(spread)[1]
    # The p-value is less than 0.05 so we conclude that this spread calculation is stationary in sample
    spread.plot()
    plt.axhline(spread.mean(), color='black')
    plt.legend(['Spread']);


.. parsed-literal::

    0.451956296234
    p-value for in-sample stationarity:  0.000609697742449



.. image:: notebook_files/notebook_25_1.png


##b. Testing the Coefficient

Use your coefficient from part a to plot the weighted spread using
prices from the first half of 2016, and check whether the result is
still stationary.

.. code:: ipython2

    S1_out = get_pricing('MTRN', fields=['price'], 
                            start_date='2016-01-01', end_date='2016-07-01')['price']
    S2_out = get_pricing('SCCO', fields=['price'], 
                            start_date='2016-01-01', end_date='2016-07-01')['price']
    
    #Your code goes here

.. code:: ipython2

    ## answer key ##
    
    spread = S2_out - b * S1_out
    spread.plot()
    plt.axhline(spread.mean(), color='black')
    plt.legend(['Spread']);
    
    print "p-value for spread stationarity: ", adfuller(spread)[1]
    # Our p-value is greater than 0.05 so we conclude that this calculation of
    # the spread is non-stationary out of sample


.. parsed-literal::

    p-value for spread stationarity:  0.145567411661



.. image:: notebook_files/notebook_28_1.png


--------------

#Extra Credit Exercise: Hurst Exponent

This exercise is more difficult and we will not provide initial
structure.

The Hurst exponent is a statistic between 0 and 1 that provides
information about how much a time series is trending or mean reverting.
We want our spread time series to be mean reverting, so we can use the
Hurst exponent to monitor whether our pair is going out of
cointegration. Effectively as a means of process control to know when
our pair is no longer good to trade.

Please find either an existing Python library that computes, or compute
yourself, the Hurst exponent. Then plot it over time for the spread on
the above pair of stocks.

These links may be helpful:

-  https://en.wikipedia.org/wiki/Hurst_exponent
-  https://www.quantopian.com/posts/pair-trade-with-cointegration-and-mean-reversion-tests

.. code:: ipython2

    # No solution provided for extra credit exercises.

*This presentation is for informational purposes only and does not
constitute an offer to sell, a solic itation to buy, or a recommendation
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
