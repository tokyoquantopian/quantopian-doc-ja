Lesson 3: The ``history`` Function
----------------------------------

The ``history`` function allows us to get trailing windows of historical
pricing and volume data in Research. It requires 5 arguments: the
contracts for which we want data, the data fields, a lookback frequency,
and the start and end dates for the lookback window. Possible fields
include ``'price'``, ``'open_price'``, ``'high'``, ``'low'``,
``'close_price'``, ``'volume'`` and ``'contract'``. Possible frequencies
are ``'daily'`` and ``'minute'``.

Let’s start by importing the ``history`` function from research’s
experimental library:

.. code:: ipython2

    from quantopian.research.experimental import history

The return type of ``history`` depends on the input types of the
``symbols`` and ``fields`` parameters. If a single ``Future`` reference
and a single field are specified, ``history`` returns a pandas Series:

.. code:: ipython2

    clf16_price = history(
        'CLF16', 
        fields='price', 
        frequency='daily', 
        start='2015-10-21', 
        end='2015-12-21'
    )
    
    clf16_price.head()




.. parsed-literal::

    2015-10-21 00:00:00+00:00    45.97
    2015-10-22 00:00:00+00:00    46.27
    2015-10-23 00:00:00+00:00    45.65
    2015-10-26 00:00:00+00:00    44.63
    2015-10-27 00:00:00+00:00    44.30
    Freq: C, Name: Future(1058201601 [CLF16]), dtype: float64



When a list of ``Futures`` and a single field are specified, the return
type is a pandas DataFrame indexed by date, with assets as columns:

.. code:: ipython2

    cl_fgh_16_volume = history(
        symbols=['CLF16', 'CLG16', 'CLH16'], 
        fields='volume', 
        frequency='daily', 
        start='2015-10-21', 
        end='2015-12-21'
    )
    
    cl_fgh_16_volume.head()




.. raw:: html

    <div>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>Future(1058201601 [CLF16])</th>
          <th>Future(1058201602 [CLG16])</th>
          <th>Future(1058201603 [CLH16])</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>2015-10-21 00:00:00+00:00</th>
          <td>21833.0</td>
          <td>2949.0</td>
          <td>1725.0</td>
        </tr>
        <tr>
          <th>2015-10-22 00:00:00+00:00</th>
          <td>17198.0</td>
          <td>2150.0</td>
          <td>1544.0</td>
        </tr>
        <tr>
          <th>2015-10-23 00:00:00+00:00</th>
          <td>27287.0</td>
          <td>3544.0</td>
          <td>1849.0</td>
        </tr>
        <tr>
          <th>2015-10-26 00:00:00+00:00</th>
          <td>18940.0</td>
          <td>2745.0</td>
          <td>1457.0</td>
        </tr>
        <tr>
          <th>2015-10-27 00:00:00+00:00</th>
          <td>22071.0</td>
          <td>3211.0</td>
          <td>2638.0</td>
        </tr>
      </tbody>
    </table>
    </div>



And if we pass a list of Futures and a list of fields, we get a pandas
Panel indexed by field, having date as the major axis, and assets as the
minor axis.

To see the full doc string for ``history``, run the cell at the end of
this notebook.

Futures Pricing & Volume
~~~~~~~~~~~~~~~~~~~~~~~~

Now, let’s use ``history`` to get the close price and volume for the
Crude Oil contract with delivery date in January 2016 (``CLF16``), for
the 2 months leading up to its delivery date (``2015-12-21``).

.. code:: ipython2

    clf16 = symbols('CLF16')
    
    clf16_data = history(
        clf16, 
        fields=['price', 'volume'], 
        frequency='daily', 
        start='2015-10-21', 
        end='2015-12-21'
    )
    
    clf16_data.head()




.. raw:: html

    <div>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>price</th>
          <th>volume</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>2015-10-21 00:00:00+00:00</th>
          <td>45.97</td>
          <td>21833.0</td>
        </tr>
        <tr>
          <th>2015-10-22 00:00:00+00:00</th>
          <td>46.27</td>
          <td>17198.0</td>
        </tr>
        <tr>
          <th>2015-10-23 00:00:00+00:00</th>
          <td>45.65</td>
          <td>27287.0</td>
        </tr>
        <tr>
          <th>2015-10-26 00:00:00+00:00</th>
          <td>44.63</td>
          <td>18940.0</td>
        </tr>
        <tr>
          <th>2015-10-27 00:00:00+00:00</th>
          <td>44.30</td>
          <td>22071.0</td>
        </tr>
      </tbody>
    </table>
    </div>



All pricing values for futures contracts correspond to unit prices. For
a ``CL`` contract, this value denotes the price per barrel, and a single
contract represents 1000 barrels (``multiplier``). In backtesting, if
you hold a particular contract, the value of your portfolio will change
by an amount equal to the change in price of the contract (unit price)
\* the multiplier.

Plotting price and volume will give us a better idea of how these values
change over time for this particular contract.

.. code:: ipython2

    clf16_data.plot(subplots=True);



.. image:: notebook_files/notebook_15_0.png


Notice the rise and subsequent drop in trading volume prior to the
delivery date of the contract? This is typical behavior for futures.
Let’s see what the volume looks like for a chain of consecutive
contracts.

.. code:: ipython2

    cl_contracts = symbols(['CLF16', 'CLG16', 'CLH16', 'CLJ16', 'CLK16', 'CLM16'])
    
    cl_consecutive_contract_volume = history(
        cl_contracts, 
        fields='volume', 
        frequency='daily', 
        start='2015-10-21', 
        end='2016-06-01'
    )

.. code:: ipython2

    cl_consecutive_contract_volume.plot(title='Consecutive Contracts Volume Over Time');



.. image:: notebook_files/notebook_18_0.png


Trading activity jumps from one contract to the next, and transitions
happen just prior to the delivery of each contract.

As you might imagine, having to explicitly reference a series of
transient contracts when trading or simulating futures can make it
difficult to work with them. In the next lesson we introduce a solution
to this problem in the form of Continuous Futures.



.. code:: ipython2

    print history.__doc__


.. parsed-literal::

    
        Load a table of historical trade data.
    
        Parameters
        ----------
        symbols : Asset-convertible object, ContinuousFuture, or iterable of same.
            Valid input types are Asset, Integral, basestring, or ContinuousFuture.
            In the case that the passed objects are strings, they are interpreted
            as ticker symbols and resolved relative to the date specified by
            symbol_reference_date.
    
        fields : str or list
            String or list drawn from {'price', 'open_price', 'high', 'low',
            'close_price', 'volume', 'contract'}.
    
        start : str or pd.Timestamp
            String or Timestamp representing a start date or start intraday minute
            for the returned data.
    
        end : str or pd.Timestamp
            String or Timestamp representing an end date or end intraday minute for
            the returned data.
    
        frequency : {'daily', 'minute'}
            Resolution of the data to be returned.
    
        symbol_reference_date : str or pd.Timestamp, optional
            String or Timestamp representing a date used to resolve symbols that
            have been held by multiple companies. Defaults to the current time.
    
        handle_missing : {'raise', 'log', 'ignore'}, optional
            String specifying how to handle unmatched securities. Defaults to
            'raise'.
    
        start_offset : int, optional
            Number of periods before ``start`` to fetch.
            Default is 0. This is most often useful when computing returns.
    
        Returns
        -------
        pandas Panel/DataFrame/Series
            The pricing data that was requested. See note below.
    
        Notes
        -----
        If a list of symbols is provided, data is returned in the form of a pandas
        Panel object with the following indices::
    
            items = fields
            major_axis = TimeSeries (start_date -> end_date)
            minor_axis = symbols
    
        If a string is passed for the value of `symbols` and `fields` is None or a
        list of strings, data is returned as a DataFrame with a DatetimeIndex and
        columns given by the passed fields.
    
        If a list of symbols is provided, and `fields` is a string, data is
        returned as a DataFrame with a DatetimeIndex and a columns given by the
        passed `symbols`.
    
        If both parameters are passed as strings, data is returned as a Series.
        

