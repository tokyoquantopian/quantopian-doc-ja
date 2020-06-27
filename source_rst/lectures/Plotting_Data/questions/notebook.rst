Exercises: Plotting
===================

By Christopher van Hoecke, Max Margenot, and Delaney Mackenzie

Lecture Link:
-------------

https://www.quantopian.com/lectures/plotting-data

IMPORTANT NOTE:
~~~~~~~~~~~~~~~

This lecture corresponds to the Plotting Data lecture, which is part of
the Quantopian lecture series. This homework expects you to rely heavily
on the code presented in the corresponding lecture. Please copy and
paste regularly from that lecture when starting to work on the problems,
as trying to do them from scratch will likely be too difficult.

Part of the Quantopian Lecture Series:

-  `www.quantopian.com/lectures <https://www.quantopian.com/lectures>`__
-  `github.com/quantopian/research_public <https://github.com/quantopian/research_public>`__

--------------

.. code:: 

    # Useful Functions
    import numpy as np
    import matplotlib.pyplot as plt

--------------

Exercise 1: Histograms
======================

a. Returns
----------

Find the daily returns for SPY over a 7-year window.

.. code:: 

    ## Your code goes here

b. Graphing
-----------

Using the techniques laid out in lecture, plot a histogram of the
returns

.. code:: 

    ## Your code goes here

c. Cumulative distribution
--------------------------

Plot the cumulative distribution histogram for your returns

.. code:: 

    ## Your code goes here

--------------

Exercise 2 : Scatter Plots
==========================

a. Data
-------

Start by collecting the close prices of McDonalds Corp. (MCD) and
Starbucks (SBUX) for the last 5 years with daily frequency.

.. code:: 

    ## Your code goes here

b. Plotting
-----------

Graph a scatter plot of SPY and Starbucks.

.. code:: 

    ## Your code goes here

c. Plotting Returns
-------------------

Graph a scatter plot of the returns of SPY and Starbucks.

.. code:: 

    ## Your code goes here

*Remember a scatter plot must have the same number of values for each
parameter. If SPY and SBUX did not have the same number of data points,
your graph will return an error*

--------------

Exercise 3 : Linear Plots
=========================

a. Getting Data
---------------

Use the techniques laid out in lecture to find the open price over a
2-year period for Starbucks (SBUX) and Dunkin Brands Group (DNKN). Print
them out in a table.

.. code:: 

    ## Your code goes here

b. Data Structure
-----------------

The data is returned to us as a pandas dataframe object. Index your data
to convert them into simple strings.

.. code:: 

    data.columns = ## Your code goes here
    ## Your code goes here

c. Plotting
-----------

Plot the data for SBUX stock price as a function of time. Remember to
label your axis and title the graph.

.. code:: 

    ## Your code goes here
    plt.xlabel();## Your code goes here
    plt.ylabel();## Your code goes here
    plt.title(); ## Your code goes here

--------------

Exercise 4 : Best fits plots
============================

Here we have a scatter plot of two data sets. Vary the ``a`` and ``b``
parameter in the code to try to draw a line that ‘fits’ our data nicely.
The line should seem as if it is describing a pattern in the data.
*While quantitative methods exist to do this automatically, we would
like you to try to get an intuition for what this feels like.*

.. code:: 

    data1 = get_pricing('SBUX', fields='open_price', start_date='2013-01-01', end_date='2014-01-01')
    data2 = get_pricing('SPY', fields='open_price', start_date = '2013-01-01', end_date='2014-01-01')
    
    rdata1= data1.pct_change()[1:]
    rdata2= data2.pct_change()[1:]
    plt.scatter(rdata2, rdata1);

.. code:: 

    plt.scatter(rdata2, rdata1)
    
    a = ## Your code goes here
    b = ## Your code goes here
    
    x = np.arange(-0.02, 0.03, 0.01)
    y = a + (b*x)
    plt.plot(x,y, color='r');

--------------

Congratulations on completing the Plotting exercises!

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
