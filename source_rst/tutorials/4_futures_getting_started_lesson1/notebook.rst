Lesson 1: Hello Futures
-----------------------

The Getting Started with Futures Tutorial will walk you through the
process of researching a quantitative strategy using futures,
implementing that strategy in an algorithm, and backtesting it on
Quantopian. It covers many of the basics of the Quantopian API, and is
designed for those who are new to the platform. Each lesson focuses on a
different part of the API and by the end, we will work up to a simple
pairs trading algorithm.

If you are unfamiliar with Futures trading, check out the Introduction
to Futures Contracts lecture from our `Lecture
Series <https://quanto-playground.herokuapp.com/lectures>`__.

What is a Trading Algorithm?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

On Quantopian, a trading algorithm is a Python program that defines a
specific set of instructions on how to analyze, order, and manage
assets. Most trading algorithms make desicions based on mathematical or
statistical hypotheses that are derived by conducting research on
historical data.

Coming Up With a Strategy
~~~~~~~~~~~~~~~~~~~~~~~~~

The first step to writing a trading algorithm is to find an economic
relationship on which we can base our strategy. To do this, we can use
Research to inspect and analyze pricing and volume data for 72 different
US futures going as far back as 2002. Research is an IPython Notebook
environment that allows us to run python code in units called ‘cells’.
The code below gets pricing and volume data for the Light Sweet Crude
Oil contract with delivery in January 2016 and plots it. To run it,
click on the grey box and press Shift + Enter to run the cell.

.. code:: ipython2

    # Click somewhere in this box and press Shift + Enter to run the code.
    
    from quantopian.research.experimental import history
    
    # Create a reference to the Light Sweet Crude Oil contract
    # with delivery in January 2016
    clf16 = symbols('CLF16')
    
    # Query historical pricing and volume data for 
    # the contract from October 21st, 2015 to December 21st, 2015
    clf16_data = history(
        clf16, 
        fields=['price', 'volume'], 
        frequency='daily', 
        start='2015-10-21', 
        end='2015-12-21'
    )
    
    # Plot the data
    clf16_data.plot(subplots=True);



.. image:: notebook_files/notebook_1_0.png


Research is the perfect tool to test a hypothesis, so let’s come up with
an idea to test.

**Strategy:** If a pair of commodities exist on the same supply chain,
we expect the spread between the prices of their futures to remain
consistent. As the spread increases or decreases, we can trade on the
expectation that it will revert to the steady state.

In the next few lessons we will learn how to use Research to test our
hypothesis. If we find our hypothesis to be valid (spoiler alert: we
will), we can use it as a starting point for implementing and
backetsting an algorithm in the Interactive Development Environment
(IDE).
