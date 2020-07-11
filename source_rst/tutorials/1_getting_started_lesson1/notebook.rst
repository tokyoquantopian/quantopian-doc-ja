Welcome to Quantopian! The Getting Started Tutorial will guide you
through researching and developing a quantitative trading strategy on
Quantopian. It covers many of the basics of Quantopian’s API, and is
designed for those who are new to the platform. All you need to get
started on this tutorial is to have some basic
`Python <https://docs.python.org/2.7/>`__ programming skills.

What is a Trading Algorithm?
----------------------------

A trading algorithm is a computer program that defines a set of rules
for buying and selling assets. Most trading algorithms make decisions
based on mathematical or statistical models that are derived from
research conducted on historical data.

Where do I start?
-----------------

The first step to writing a trading algorithm is to find an economic or
statistical relationship on which we can base our strategy. To do this,
we can use Quantopian’s Research environment to access and analyze
historical datasets available in the platform. Research is a `Jupyter
Notebook <http://jupyter-notebook-beginner-guide.readthedocs.io/en/latest/what_is_jupyter.html>`__
environment that allows us to run Python code in units called ‘cells’.

For example, the following code plots the daily closing price for Apple
Inc. (AAPL), along with its 20 and 50 day moving averages:

.. code:: ipython2

    # Research environment functions
    from quantopian.research import prices, symbols
    
    # Pandas library: https://pandas.pydata.org/
    import pandas as pd
    
    # Query historical pricing data for AAPL
    aapl_close = prices(
        assets=symbols('AAPL'),
        start='2013-01-01',
        end='2016-01-01',
    )
    
    # Compute 20 and 50 day moving averages on
    # AAPL's pricing data
    aapl_sma20 = aapl_close.rolling(20).mean()
    aapl_sma50 = aapl_close.rolling(50).mean()
    
    # Combine results into a pandas DataFrame and plot
    pd.DataFrame({   
        'AAPL': aapl_close,
        'SMA20': aapl_sma20,
        'SMA50': aapl_sma50
    }).plot(
        title='AAPL Close Price / SMA Crossover'
    );



.. image:: notebook_files/notebook_5_0.png


In the next lesson we will use Research to explore Quantopian’s
datasets. Then, we will define our trading strategy and test whether it
can effectively predict returns based on historical data. Finally, we
will use our findings to develop and test a trading algorithm in the
Interactive Development Environment (IDE).
