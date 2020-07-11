Measures of Dispersion
======================

By Evgenia “Jenny” Nitishinskaya, Maxwell Margenot, and Delaney
Mackenzie.

Part of the Quantopian Lecture Series:

-  `www.quantopian.com/lectures <https://www.quantopian.com/lectures>`__
-  `github.com/quantopian/research_public <https://github.com/quantopian/research_public>`__

--------------

Dispersion measures how spread out a set of data is. This is especially
important in finance because one of the main ways risk is measured is in
how spread out returns have been historically. If returns have been very
tight around a central value, then we have less reason to worry. If
returns have been all over the place, that is risky.

Data with low dispersion is heavily clustered around the mean, while
data with high dispersion indicates many very large and very small
values.

Let’s generate an array of random integers to work with.

.. code:: ipython2

    # Import libraries
    import numpy as np
    
    np.random.seed(121)

.. code:: ipython2

    # Generate 20 random integers < 100
    X = np.random.randint(100, size=20)
    
    # Sort them
    X = np.sort(X)
    print 'X: %s' %(X)
    
    mu = np.mean(X)
    print 'Mean of X:', mu


.. parsed-literal::

    X: [ 3  8 34 39 46 52 52 52 54 57 60 65 66 75 83 85 88 94 95 96]
    Mean of X: 60.2


Range
=====

Range is simply the difference between the maximum and minimum values in
a dataset. Not surprisingly, it is very sensitive to outliers. We’ll use
``numpy``\ ’s peak to peak (ptp) function for this.

.. code:: ipython2

    print 'Range of X: %s' %(np.ptp(X))


.. parsed-literal::

    Range of X: 93


Mean Absolute Deviation (MAD)
=============================

The mean absolute deviation is the average of the distances of
observations from the arithmetic mean. We use the absolute value of the
deviation, so that 5 above the mean and 5 below the mean both contribute
5, because otherwise the deviations always sum to 0.

.. math::  MAD = \frac{\sum_{i=1}^n |X_i - \mu|}{n} 

where :math:`n` is the number of observations and :math:`\mu` is their
mean.

.. code:: ipython2

    abs_dispersion = [np.abs(mu - x) for x in X]
    MAD = np.sum(abs_dispersion)/len(abs_dispersion)
    print 'Mean absolute deviation of X:', MAD


.. parsed-literal::

    Mean absolute deviation of X: 20.52


Variance and standard deviation
===============================

The variance :math:`\sigma^2` is defined as the average of the squared
deviations around the mean:

.. math::  \sigma^2 = \frac{\sum_{i=1}^n (X_i - \mu)^2}{n} 

This is sometimes more convenient than the mean absolute deviation
because absolute value is not differentiable, while squaring is smooth,
and some optimization algorithms rely on differentiability.

Standard deviation is defined as the square root of the variance,
:math:`\sigma`, and it is the easier of the two to interpret because it
is in the same units as the observations.

.. code:: ipython2

    print 'Variance of X:', np.var(X)
    print 'Standard deviation of X:', np.std(X)


.. parsed-literal::

    Variance of X: 670.16
    Standard deviation of X: 25.8874486962


One way to interpret standard deviation is by referring to Chebyshev’s
inequality. This tells us that the proportion of samples within
:math:`k` standard deviations (that is, within a distance of
:math:`k \cdot` standard deviation) of the mean is at least
:math:`1 - 1/k^2` for all :math:`k>1`.

Let’s check that this is true for our data set.

.. code:: ipython2

    k = 1.25
    dist = k*np.std(X)
    l = [x for x in X if abs(x - mu) <= dist]
    print 'Observations within', k, 'stds of mean:', l
    print 'Confirming that', float(len(l))/len(X), '>', 1 - 1/k**2


.. parsed-literal::

    Observations within 1.25 stds of mean: [34, 39, 46, 52, 52, 52, 54, 57, 60, 65, 66, 75, 83, 85, 88]
    Confirming that 0.75 > 0.36


The bound given by Chebyshev’s inequality seems fairly loose in this
case. This bound is rarely strict, but it is useful because it holds for
all data sets and distributions.

Semivariance and semideviation
==============================

Although variance and standard deviation tell us how volatile a quantity
is, they do not differentiate between deviations upward and deviations
downward. Often, such as in the case of returns on an asset, we are more
worried about deviations downward. This is addressed by semivariance and
semideviation, which only count the observations that fall below the
mean. Semivariance is defined as

.. math::  \frac{\sum_{X_i < \mu} (X_i - \mu)^2}{n_<} 

where :math:`n_<` is the number of observations which are smaller than
the mean. Semideviation is the square root of the semivariance.

.. code:: ipython2

    # Because there is no built-in semideviation, we'll compute it ourselves
    lows = [e for e in X if e <= mu]
    
    semivar = np.sum( (lows - mu) ** 2 ) / len(lows)
    
    print 'Semivariance of X:', semivar
    print 'Semideviation of X:', np.sqrt(semivar)


.. parsed-literal::

    Semivariance of X: 689.512727273
    Semideviation of X: 26.2585743572


A related notion is target semivariance (and target semideviation),
where we average the distance from a target of values which fall below
that target:

.. math::  \frac{\sum_{X_i < B} (X_i - B)^2}{n_{<B}} 

.. code:: ipython2

    B = 19
    lows_B = [e for e in X if e <= B]
    semivar_B = sum(map(lambda x: (x - B)**2,lows_B))/len(lows_B)
    
    print 'Target semivariance of X:', semivar_B
    print 'Target semideviation of X:', np.sqrt(semivar_B)


.. parsed-literal::

    Target semivariance of X: 188
    Target semideviation of X: 13.7113092008


These are Only Estimates
========================

All of these computations will give you sample statistics, that is
standard deviation of a sample of data. Whether or not this reflects the
current true population standard deviation is not always obvious, and
more effort has to be put into determining that. This is especially
problematic in finance because all data are time series and the mean and
variance may change over time. There are many different techniques and
subtleties here, some of which are addressed in other lectures in the
`Quantopian Lecture Series <https://www.quantopian.com/lectures>`__.

In general do not assume that because something is true of your sample,
it will remain true going forward.

References
----------

-  “Quantitative Investment Analysis”, by DeFusco, McLeavey, Pinto, and
   Runkle

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
