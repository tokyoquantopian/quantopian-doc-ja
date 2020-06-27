#Introduction to the Research Environment

The research environment is powered by IPython notebooks, which allow
one to perform a great deal of data analysis and statistical validation.
We’ll demonstrate a few simple techniques here.

##Code Cells vs. Text Cells

As you can see, each cell can be either code or text. To select between
them, choose from the ‘Cell Type’ dropdown menu on the top left.

##Executing a Command

A code cell will be evaluated when you press play, or when you press the
shortcut, shift-enter. Evaluating a cell evaluates each line of code in
sequence, and prints the results of the last line below the cell.

.. code:: ipython2

    2 + 2

Sometimes there is no result to be printed, as is the case with
assignment.

.. code:: ipython2

    X = 2

Remember that only the result from the last line is printed.

.. code:: ipython2

    2 + 2
    3 + 3

However, you can print whichever lines you want using the ``print``
statement.

.. code:: ipython2

    print 2 + 2
    3 + 3

##Knowing When a Cell is Running

While a cell is running, a ``[*]`` will display on the left. When a cell
has yet to be executed, ``[ ]`` will display. When it has been run, a
number will display indicating the order in which it was run during the
execution of the notebook ``[5]``. Try on this cell and note it
happening.

.. code:: ipython2

    #Take some time to run something
    c = 0
    for i in range(10000000):
        c = c + i
    c

##Importing Libraries

The vast majority of the time, you’ll want to use functions from
pre-built libraries. You can’t import every library on Quantopian due to
security issues, but you can import most of the common scientific ones.
Here I import numpy and pandas, the two most common and useful libraries
in quant finance. I recommend copying this import statement to every new
notebook.

Notice that you can rename libraries to whatever you want after
importing. The ``as`` statement allows this. Here we use ``np`` and
``pd`` as aliases for ``numpy`` and ``pandas``. This is a very common
aliasing and will be found in most code snippets around the web. The
point behind this is to allow you to type fewer characters when you are
frequently accessing these libraries.

.. code:: ipython2

    import numpy as np
    import pandas as pd
    
    # This is a plotting library for pretty pictures.
    import matplotlib.pyplot as plt

##Tab Autocomplete

Pressing tab will give you a list of IPython’s best guesses for what you
might want to type next. This is incredibly valuable and will save you a
lot of time. If there is only one possible option for what you could
type next, IPython will fill that in for you. Try pressing tab very
frequently, it will seldom fill in anything you don’t want, as if there
is ambiguity a list will be shown. This is a great way to see what
functions are available in a library.

Try placing your cursor after the ``.`` and pressing tab.

.. code:: ipython2

    np.random.

##Getting Documentation Help

Placing a question mark after a function and executing that line of code
will give you the documentation IPython has for that function. It’s
often best to do this in a new cell, as you avoid re-executing other
code and running into bugs.

.. code:: ipython2

    np.random.normal?

##Sampling

We’ll sample some random data using a function from ``numpy``.

.. code:: ipython2

    # Sample 100 points with a mean of 0 and an std of 1. This is a standard normal distribution.
    X = np.random.normal(0, 1, 100)

##Plotting

We can use the plotting library we imported as follows.

.. code:: ipython2

    plt.plot(X)

###Squelching Line Output

You might have noticed the annoying line of the form
``[<matplotlib.lines.Line2D at 0x7f72fdbc1710>]`` before the plots. This
is because the ``.plot`` function actually produces output. Sometimes we
wish not to display output, we can accomplish this with the semi-colon
as follows.

.. code:: ipython2

    plt.plot(X);

###Adding Axis Labels

No self-respecting quant leaves a graph without labeled axes. Here are
some commands to help with that.

.. code:: ipython2

    X = np.random.normal(0, 1, 100)
    X2 = np.random.normal(0, 1, 100)
    
    plt.plot(X);
    plt.plot(X2);
    plt.xlabel('Time') # The data we generated is unitless, but don't forget units in general.
    plt.ylabel('Returns')
    plt.legend(['X', 'X2']);

##Generating Statistics

Let’s use ``numpy`` to take some simple statistics.

.. code:: ipython2

    np.mean(X)

.. code:: ipython2

    np.std(X)

##Getting Real Pricing Data

Randomly sampled data can be great for testing ideas, but let’s get some
real data. We can use ``get_pricing`` to do that. You can use the ``?``
syntax as discussed above to get more information on ``get_pricing``\ ’s
arguments.

.. code:: ipython2

    data = get_pricing('MSFT', start_date='2012-1-1', end_date='2015-6-1')

Our data is now a dataframe. You can see the datetime index and the
colums with different pricing data.

.. code:: ipython2

    data

This is a pandas dataframe, so we can index in to just get price like
this. For more info on pandas, please `click
here <http://pandas.pydata.org/pandas-docs/stable/10min.html>`__.

.. code:: ipython2

    X = data['price']

Because there is now also date information in our data, we provide two
series to ``.plot``. ``X.index`` gives us the datetime index, and
``X.values`` gives us the pricing values. These are used as the X and Y
coordinates to make a graph.

.. code:: ipython2

    plt.plot(X.index, X.values)
    plt.ylabel('Price')
    plt.legend(['MSFT']);

We can get statistics again on real data.

.. code:: ipython2

    np.mean(X)

.. code:: ipython2

    np.std(X)

##Getting Returns from Prices

We can use the ``pct_change`` function to get returns. Notice how we
drop the first element after doing this, as it will be ``NaN`` (nothing
-> something results in a NaN percent change).

.. code:: ipython2

    R = X.pct_change()[1:]

We can plot the returns distribution as a histogram.

.. code:: ipython2

    plt.hist(R, bins=20)
    plt.xlabel('Return')
    plt.ylabel('Frequency')
    plt.legend(['MSFT Returns']);

Get statistics again.

.. code:: ipython2

    np.mean(R)

.. code:: ipython2

    np.std(R)

Now let’s go backwards and generate data out of a normal distribution
using the statistics we estimated from Microsoft’s returns. We’ll see
that we have good reason to suspect Microsoft’s returns may not be
normal, as the resulting normal distribution looks far different.

.. code:: ipython2

    plt.hist(np.random.normal(np.mean(R), np.std(R), 10000), bins=20)
    plt.xlabel('Return')
    plt.ylabel('Frequency')
    plt.legend(['Normally Distributed Returns']);

##Generating a Moving Average

``pandas`` has some nice tools to allow us to generate rolling
statistics. Here’s an example. Notice how there’s no moving average for
the first 60 days, as we don’t have 60 days of data on which to generate
the statistic.

.. code:: ipython2

    # Take the average of the last 60 days at each timepoint.
    MAVG = pd.rolling_mean(X, window=60)
    plt.plot(X.index, X.values)
    plt.plot(MAVG.index, MAVG.values)
    plt.ylabel('Price')
    plt.legend(['MSFT', '60-day MAVG']);

This presentation is for informational purposes only and does not
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
including changes in market conditions or economic circumstances.
