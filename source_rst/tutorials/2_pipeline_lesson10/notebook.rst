.. code:: ipython2

    from quantopian.pipeline import Pipeline
    from quantopian.research import run_pipeline
    from quantopian.pipeline.data.builtin import USEquityPricing
    from quantopian.pipeline.factors import SimpleMovingAverage, AverageDollarVolume

Custom Factors
--------------

When we first looked at factors, we explored the set of built-in
factors. Frequently, a desired computation isn’t included as a built-in
factor. One of the most powerful features of the Pipeline API is that it
allows us to define our own custom factors. When a desired computation
doesn’t exist as a built-in, we define a custom factor.

Conceptually, a custom factor is identical to a built-in factor. It
accepts ``inputs``, ``window_length``, and ``mask`` as constructor
arguments, and returns a ``Factor`` object each day.

Let’s take an example of a computation that doesn’t exist as a built-in:
standard deviation. To create a factor that computes the `standard
deviation <https://en.wikipedia.org/wiki/Standard_deviation>`__ over a
trailing window, we can subclass ``quantopian.pipeline.CustomFactor``
and implement a compute method whose signature is:

::

   def compute(self, today, asset_ids, out, *inputs):
       ...

-  ``*inputs`` are M x N `numpy
   arrays <http://docs.scipy.org/doc/numpy-1.10.1/reference/arrays.ndarray.html>`__,
   where M is the ``window_length`` and N is the number of securities
   (usually around ~8000 unless a ``mask`` is provided). ``*inputs`` are
   trailing data windows. Note that there will be one M x N array for
   each ``BoundColumn`` provided in the factor’s ``inputs`` list. The
   data type of each array will be the ``dtype`` of the corresponding
   ``BoundColumn``.
-  ``out`` is an empty array of length N. ``out`` will be the output of
   our custom factor each day. The job of compute is to write output
   values into out.
-  ``asset_ids`` will be an integer
   `array <http://docs.scipy.org/doc/numpy-1.10.0/reference/generated/numpy.array.html>`__
   of length N containing security ids corresponding to the columns in
   our ``*inputs`` arrays.
-  ``today`` will be a `pandas
   Timestamp <http://pandas.pydata.org/pandas-docs/stable/timeseries.html#converting-to-timestamps>`__
   representing the day for which ``compute`` is being called.

Of these, ``*inputs`` and ``out`` are most commonly used.

An instance of ``CustomFactor`` that’s been added to a pipeline will
have its compute method called every day. For example, let’s define a
custom factor that computes the standard deviation of the close price
over the last 5 days. To start, let’s add ``CustomFactor`` and ``numpy``
to our import statements.

.. code:: ipython2

    from quantopian.pipeline import CustomFactor
    import numpy

Next, let’s define our custom factor to calculate the standard deviation
over a trailing window using
`numpy.nanstd <http://docs.scipy.org/doc/numpy-dev/reference/generated/numpy.nanstd.html>`__:

.. code:: ipython2

    class StdDev(CustomFactor):
        def compute(self, today, asset_ids, out, values):
            # Calculates the column-wise standard deviation, ignoring NaNs
            out[:] = numpy.nanstd(values, axis=0)

Finally, let’s instantiate our factor in ``make_pipeline()``:

.. code:: ipython2

    def make_pipeline():
        std_dev = StdDev(inputs=[USEquityPricing.close], window_length=5)
    
        return Pipeline(
            columns={
                'std_dev': std_dev
            }
        )

When this pipeline is run, ``StdDev.compute()`` will be called every day
with data as follows:

-  ``values``: An M x N `numpy
   array <http://docs.scipy.org/doc/numpy-1.10.1/reference/arrays.ndarray.html>`__,
   where M is 5 (``window_length``), and N is ~8000 (the number of
   securities in our database on the day in question).
-  ``out``: An empty array of length N (~8000). In this example, the job
   of ``compute`` is to populate ``out`` with an array storing of 5-day
   close price standard deviations.

.. code:: ipython2

    result = run_pipeline(make_pipeline(), '2015-05-05', '2015-05-05')
    result


.. parsed-literal::

    /usr/local/lib/python2.7/dist-packages/numpy/lib/nanfunctions.py:1057: RuntimeWarning: Degrees of freedom <= 0 for slice.
      warnings.warn("Degrees of freedom <= 0 for slice.", RuntimeWarning)




.. raw:: html

    <div style="max-height: 1000px; max-width: 1500px; overflow: auto;">
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th></th>
          <th>std_dev</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th rowspan="61" valign="top">2015-05-05 00:00:00+00:00</th>
          <th>Equity(2 [AA])</th>
          <td>0.293428</td>
        </tr>
        <tr>
          <th>Equity(21 [AAME])</th>
          <td>0.004714</td>
        </tr>
        <tr>
          <th>Equity(24 [AAPL])</th>
          <td>1.737677</td>
        </tr>
        <tr>
          <th>Equity(25 [AA_PR])</th>
          <td>0.275000</td>
        </tr>
        <tr>
          <th>Equity(31 [ABAX])</th>
          <td>4.402971</td>
        </tr>
        <tr>
          <th>Equity(39 [DDC])</th>
          <td>0.138939</td>
        </tr>
        <tr>
          <th>Equity(41 [ARCB])</th>
          <td>0.826109</td>
        </tr>
        <tr>
          <th>Equity(52 [ABM])</th>
          <td>0.093680</td>
        </tr>
        <tr>
          <th>Equity(53 [ABMD])</th>
          <td>1.293058</td>
        </tr>
        <tr>
          <th>Equity(62 [ABT])</th>
          <td>0.406546</td>
        </tr>
        <tr>
          <th>Equity(64 [ABX])</th>
          <td>0.178034</td>
        </tr>
        <tr>
          <th>Equity(66 [AB])</th>
          <td>0.510427</td>
        </tr>
        <tr>
          <th>Equity(67 [ADSK])</th>
          <td>1.405754</td>
        </tr>
        <tr>
          <th>Equity(69 [ACAT])</th>
          <td>0.561413</td>
        </tr>
        <tr>
          <th>Equity(70 [VBF])</th>
          <td>0.054626</td>
        </tr>
        <tr>
          <th>Equity(76 [TAP])</th>
          <td>0.411757</td>
        </tr>
        <tr>
          <th>Equity(84 [ACET])</th>
          <td>0.320624</td>
        </tr>
        <tr>
          <th>Equity(86 [ACG])</th>
          <td>0.012806</td>
        </tr>
        <tr>
          <th>Equity(88 [ACI])</th>
          <td>0.026447</td>
        </tr>
        <tr>
          <th>Equity(100 [IEP])</th>
          <td>0.444189</td>
        </tr>
        <tr>
          <th>Equity(106 [ACU])</th>
          <td>0.060531</td>
        </tr>
        <tr>
          <th>Equity(110 [ACXM])</th>
          <td>0.485444</td>
        </tr>
        <tr>
          <th>Equity(112 [ACY])</th>
          <td>0.207107</td>
        </tr>
        <tr>
          <th>Equity(114 [ADBE])</th>
          <td>0.280385</td>
        </tr>
        <tr>
          <th>Equity(117 [AEY])</th>
          <td>0.022471</td>
        </tr>
        <tr>
          <th>Equity(122 [ADI])</th>
          <td>0.549778</td>
        </tr>
        <tr>
          <th>Equity(128 [ADM])</th>
          <td>0.605495</td>
        </tr>
        <tr>
          <th>Equity(134 [SXCL])</th>
          <td>NaN</td>
        </tr>
        <tr>
          <th>Equity(149 [ADX])</th>
          <td>0.072153</td>
        </tr>
        <tr>
          <th>Equity(153 [AE])</th>
          <td>3.676240</td>
        </tr>
        <tr>
          <th>...</th>
          <td>...</td>
        </tr>
        <tr>
          <th>Equity(48961 [NYMT_O])</th>
          <td>NaN</td>
        </tr>
        <tr>
          <th>Equity(48962 [CSAL])</th>
          <td>0.285755</td>
        </tr>
        <tr>
          <th>Equity(48963 [PAK])</th>
          <td>0.034871</td>
        </tr>
        <tr>
          <th>Equity(48969 [NSA])</th>
          <td>0.144305</td>
        </tr>
        <tr>
          <th>Equity(48971 [BSM])</th>
          <td>0.245000</td>
        </tr>
        <tr>
          <th>Equity(48972 [EVA])</th>
          <td>0.207175</td>
        </tr>
        <tr>
          <th>Equity(48981 [APIC])</th>
          <td>0.364560</td>
        </tr>
        <tr>
          <th>Equity(48989 [UK])</th>
          <td>0.148399</td>
        </tr>
        <tr>
          <th>Equity(48990 [ACWF])</th>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(48991 [ISCF])</th>
          <td>0.035000</td>
        </tr>
        <tr>
          <th>Equity(48992 [INTF])</th>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(48993 [JETS])</th>
          <td>0.294937</td>
        </tr>
        <tr>
          <th>Equity(48994 [ACTX])</th>
          <td>0.091365</td>
        </tr>
        <tr>
          <th>Equity(48995 [LRGF])</th>
          <td>0.172047</td>
        </tr>
        <tr>
          <th>Equity(48996 [SMLF])</th>
          <td>0.245130</td>
        </tr>
        <tr>
          <th>Equity(48997 [VKTX])</th>
          <td>0.065000</td>
        </tr>
        <tr>
          <th>Equity(48998 [OPGN])</th>
          <td>NaN</td>
        </tr>
        <tr>
          <th>Equity(48999 [AAPC])</th>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(49000 [BPMC])</th>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(49001 [CLCD])</th>
          <td>NaN</td>
        </tr>
        <tr>
          <th>Equity(49004 [TNP_PRD])</th>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(49005 [ARWA_U])</th>
          <td>NaN</td>
        </tr>
        <tr>
          <th>Equity(49006 [BVXV])</th>
          <td>NaN</td>
        </tr>
        <tr>
          <th>Equity(49007 [BVXV_W])</th>
          <td>NaN</td>
        </tr>
        <tr>
          <th>Equity(49008 [OPGN_W])</th>
          <td>NaN</td>
        </tr>
        <tr>
          <th>Equity(49009 [PRKU])</th>
          <td>NaN</td>
        </tr>
        <tr>
          <th>Equity(49010 [TBRA])</th>
          <td>NaN</td>
        </tr>
        <tr>
          <th>Equity(49131 [OESX])</th>
          <td>NaN</td>
        </tr>
        <tr>
          <th>Equity(49259 [ITUS])</th>
          <td>NaN</td>
        </tr>
        <tr>
          <th>Equity(49523 [TLGT])</th>
          <td>NaN</td>
        </tr>
      </tbody>
    </table>
    <p>8236 rows × 1 columns</p>
    </div>



Default Inputs
~~~~~~~~~~~~~~

When writing a custom factor, we can set default ``inputs`` and
``window_length`` in our ``CustomFactor`` subclass. For example, let’s
define the ``TenDayMeanDifference`` custom factor to compute the mean
difference between two data columns over a trailing window using
`numpy.nanmean <http://docs.scipy.org/doc/numpy-dev/reference/generated/numpy.nanmean.html>`__.
Let’s set the default ``inputs`` to
``[USEquityPricing.close, USEquityPricing.open]`` and the default
``window_length`` to 10:

.. code:: ipython2

    class TenDayMeanDifference(CustomFactor):
        # Default inputs.
        inputs = [USEquityPricing.close, USEquityPricing.open]
        window_length = 10
        def compute(self, today, asset_ids, out, close, open):
            # Calculates the column-wise mean difference, ignoring NaNs
            out[:] = numpy.nanmean(close - open, axis=0)

Remember in this case that ``close`` and ``open`` are each 10 x ~8000 2D
`numpy
arrays. <http://docs.scipy.org/doc/numpy-1.10.1/reference/arrays.ndarray.html>`__\ 

If we call ``TenDayMeanDifference`` without providing any arguments, it
will use the defaults.

.. code:: ipython2

    # Computes the 10-day mean difference between the daily open and close prices.
    close_open_diff = TenDayMeanDifference()

The defaults can be manually overridden by specifying arguments in the
constructor call.

.. code:: ipython2

    # Computes the 10-day mean difference between the daily high and low prices.
    high_low_diff = TenDayMeanDifference(inputs=[USEquityPricing.high, USEquityPricing.low])

Further Example
~~~~~~~~~~~~~~~

Let’s take another example where we build a
`momentum <http://www.investopedia.com/terms/m/momentum.asp>`__ custom
factor and use it to create a filter. We will then use that filter as a
``screen`` for our pipeline.

Let’s start by defining a ``Momentum`` factor to be the division of the
most recent close price by the close price from ``n`` days ago where
``n`` is the ``window_length``.

.. code:: ipython2

    class Momentum(CustomFactor):
        # Default inputs
        inputs = [USEquityPricing.close]
    
        # Compute momentum
        def compute(self, today, assets, out, close):
            out[:] = close[-1] / close[0]

Now, let’s instantiate our ``Momentum`` factor (twice) to create a
10-day momentum factor and a 20-day momentum factor. Let’s also create a
``positive_momentum`` filter returning ``True`` for securities with both
a positive 10-day momentum and a positive 20-day momentum.

.. code:: ipython2

    ten_day_momentum = Momentum(window_length=10)
    twenty_day_momentum = Momentum(window_length=20)
    
    positive_momentum = ((ten_day_momentum > 1) & (twenty_day_momentum > 1))

Next, let’s add our momentum factors and our ``positive_momentum``
filter to ``make_pipeline``. Let’s also pass ``positive_momentum`` as a
``screen`` to our pipeline.

.. code:: ipython2

    def make_pipeline():
    
        ten_day_momentum = Momentum(window_length=10)
        twenty_day_momentum = Momentum(window_length=20)
    
        positive_momentum = ((ten_day_momentum > 1) & (twenty_day_momentum > 1))
    
        std_dev = StdDev(inputs=[USEquityPricing.close], window_length=5)
    
        return Pipeline(
            columns={
                'std_dev': std_dev,
                'ten_day_momentum': ten_day_momentum,
                'twenty_day_momentum': twenty_day_momentum
            },
            screen=positive_momentum
        )

Running this pipeline outputs the standard deviation and each of our
momentum computations for securities with positive 10-day and 20-day
momentum.

.. code:: ipython2

    result = run_pipeline(make_pipeline(), '2015-05-05', '2015-05-05')
    result




.. raw:: html

    <div style="max-height: 1000px; max-width: 1500px; overflow: auto;">
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th></th>
          <th>std_dev</th>
          <th>ten_day_momentum</th>
          <th>twenty_day_momentum</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th rowspan="61" valign="top">2015-05-05 00:00:00+00:00</th>
          <th>Equity(2 [AA])</th>
          <td>0.293428</td>
          <td>1.036612</td>
          <td>1.042783</td>
        </tr>
        <tr>
          <th>Equity(24 [AAPL])</th>
          <td>1.737677</td>
          <td>1.014256</td>
          <td>1.021380</td>
        </tr>
        <tr>
          <th>Equity(39 [DDC])</th>
          <td>0.138939</td>
          <td>1.062261</td>
          <td>1.167319</td>
        </tr>
        <tr>
          <th>Equity(52 [ABM])</th>
          <td>0.093680</td>
          <td>1.009212</td>
          <td>1.015075</td>
        </tr>
        <tr>
          <th>Equity(64 [ABX])</th>
          <td>0.178034</td>
          <td>1.025721</td>
          <td>1.065587</td>
        </tr>
        <tr>
          <th>Equity(66 [AB])</th>
          <td>0.510427</td>
          <td>1.036137</td>
          <td>1.067545</td>
        </tr>
        <tr>
          <th>Equity(100 [IEP])</th>
          <td>0.444189</td>
          <td>1.008820</td>
          <td>1.011385</td>
        </tr>
        <tr>
          <th>Equity(114 [ADBE])</th>
          <td>0.280385</td>
          <td>1.016618</td>
          <td>1.002909</td>
        </tr>
        <tr>
          <th>Equity(117 [AEY])</th>
          <td>0.022471</td>
          <td>1.004167</td>
          <td>1.025532</td>
        </tr>
        <tr>
          <th>Equity(128 [ADM])</th>
          <td>0.605495</td>
          <td>1.049625</td>
          <td>1.044832</td>
        </tr>
        <tr>
          <th>Equity(149 [ADX])</th>
          <td>0.072153</td>
          <td>1.004607</td>
          <td>1.016129</td>
        </tr>
        <tr>
          <th>Equity(154 [AEM])</th>
          <td>0.634920</td>
          <td>1.032690</td>
          <td>1.065071</td>
        </tr>
        <tr>
          <th>Equity(161 [AEP])</th>
          <td>0.458938</td>
          <td>1.024926</td>
          <td>1.017563</td>
        </tr>
        <tr>
          <th>Equity(166 [AES])</th>
          <td>0.164973</td>
          <td>1.031037</td>
          <td>1.045946</td>
        </tr>
        <tr>
          <th>Equity(168 [AET])</th>
          <td>1.166938</td>
          <td>1.007566</td>
          <td>1.022472</td>
        </tr>
        <tr>
          <th>Equity(192 [ATAX])</th>
          <td>0.024819</td>
          <td>1.009025</td>
          <td>1.018215</td>
        </tr>
        <tr>
          <th>Equity(197 [AGCO])</th>
          <td>0.646594</td>
          <td>1.066522</td>
          <td>1.098572</td>
        </tr>
        <tr>
          <th>Equity(239 [AIG])</th>
          <td>0.710307</td>
          <td>1.027189</td>
          <td>1.058588</td>
        </tr>
        <tr>
          <th>Equity(253 [AIR])</th>
          <td>0.156844</td>
          <td>1.007474</td>
          <td>1.003818</td>
        </tr>
        <tr>
          <th>Equity(266 [AJG])</th>
          <td>0.397769</td>
          <td>1.000839</td>
          <td>1.018799</td>
        </tr>
        <tr>
          <th>Equity(312 [ALOT])</th>
          <td>0.182893</td>
          <td>1.031780</td>
          <td>1.021352</td>
        </tr>
        <tr>
          <th>Equity(328 [ALTR])</th>
          <td>2.286573</td>
          <td>1.041397</td>
          <td>1.088996</td>
        </tr>
        <tr>
          <th>Equity(353 [AME])</th>
          <td>0.362513</td>
          <td>1.023622</td>
          <td>1.004902</td>
        </tr>
        <tr>
          <th>Equity(357 [TWX])</th>
          <td>0.502816</td>
          <td>1.022013</td>
          <td>1.006976</td>
        </tr>
        <tr>
          <th>Equity(366 [AVD])</th>
          <td>0.842249</td>
          <td>1.114111</td>
          <td>1.093162</td>
        </tr>
        <tr>
          <th>Equity(438 [AON])</th>
          <td>0.881295</td>
          <td>1.020732</td>
          <td>1.018739</td>
        </tr>
        <tr>
          <th>Equity(448 [APA])</th>
          <td>0.678899</td>
          <td>1.002193</td>
          <td>1.051258</td>
        </tr>
        <tr>
          <th>Equity(451 [APB])</th>
          <td>0.081240</td>
          <td>1.026542</td>
          <td>1.105042</td>
        </tr>
        <tr>
          <th>Equity(455 [APC])</th>
          <td>0.152394</td>
          <td>1.012312</td>
          <td>1.097284</td>
        </tr>
        <tr>
          <th>Equity(474 [APOG])</th>
          <td>0.610410</td>
          <td>1.030843</td>
          <td>1.206232</td>
        </tr>
        <tr>
          <th>...</th>
          <td>...</td>
          <td>...</td>
          <td>...</td>
        </tr>
        <tr>
          <th>Equity(48504 [ERUS])</th>
          <td>0.052688</td>
          <td>1.030893</td>
          <td>1.052812</td>
        </tr>
        <tr>
          <th>Equity(48531 [VSTO])</th>
          <td>0.513443</td>
          <td>1.029164</td>
          <td>1.028110</td>
        </tr>
        <tr>
          <th>Equity(48532 [ENTL])</th>
          <td>0.163756</td>
          <td>1.043708</td>
          <td>1.152246</td>
        </tr>
        <tr>
          <th>Equity(48535 [ANH_PRC])</th>
          <td>0.072388</td>
          <td>1.010656</td>
          <td>1.010656</td>
        </tr>
        <tr>
          <th>Equity(48543 [SHAK])</th>
          <td>2.705316</td>
          <td>1.262727</td>
          <td>1.498020</td>
        </tr>
        <tr>
          <th>Equity(48591 [SPYB])</th>
          <td>0.221848</td>
          <td>1.001279</td>
          <td>1.005801</td>
        </tr>
        <tr>
          <th>Equity(48602 [ITEK])</th>
          <td>0.177042</td>
          <td>1.213693</td>
          <td>1.133721</td>
        </tr>
        <tr>
          <th>Equity(48623 [TCCB])</th>
          <td>0.056148</td>
          <td>1.003641</td>
          <td>1.006349</td>
        </tr>
        <tr>
          <th>Equity(48641 [GDJJ])</th>
          <td>0.530298</td>
          <td>1.041176</td>
          <td>1.111809</td>
        </tr>
        <tr>
          <th>Equity(48644 [GDXX])</th>
          <td>0.401079</td>
          <td>1.042319</td>
          <td>1.120948</td>
        </tr>
        <tr>
          <th>Equity(48680 [RODM])</th>
          <td>0.080455</td>
          <td>1.005037</td>
          <td>1.018853</td>
        </tr>
        <tr>
          <th>Equity(48688 [QVM])</th>
          <td>0.152245</td>
          <td>1.009996</td>
          <td>1.021845</td>
        </tr>
        <tr>
          <th>Equity(48701 [AMT_PRB])</th>
          <td>0.546691</td>
          <td>1.010356</td>
          <td>1.023537</td>
        </tr>
        <tr>
          <th>Equity(48706 [GBSN_U])</th>
          <td>0.442285</td>
          <td>1.214035</td>
          <td>1.272059</td>
        </tr>
        <tr>
          <th>Equity(48730 [AGN_PRA])</th>
          <td>9.614542</td>
          <td>1.000948</td>
          <td>1.001694</td>
        </tr>
        <tr>
          <th>Equity(48746 [SUM])</th>
          <td>0.457585</td>
          <td>1.024112</td>
          <td>1.131837</td>
        </tr>
        <tr>
          <th>Equity(48747 [AFTY])</th>
          <td>0.193080</td>
          <td>1.032030</td>
          <td>1.146784</td>
        </tr>
        <tr>
          <th>Equity(48754 [IBDJ])</th>
          <td>0.048949</td>
          <td>1.000161</td>
          <td>1.000561</td>
        </tr>
        <tr>
          <th>Equity(48768 [SDEM])</th>
          <td>0.102439</td>
          <td>1.068141</td>
          <td>1.103535</td>
        </tr>
        <tr>
          <th>Equity(48783 [CHEK_W])</th>
          <td>0.222528</td>
          <td>1.466667</td>
          <td>1.157895</td>
        </tr>
        <tr>
          <th>Equity(48785 [NCOM])</th>
          <td>0.166885</td>
          <td>1.018349</td>
          <td>1.020221</td>
        </tr>
        <tr>
          <th>Equity(48792 [AFSI_PRD])</th>
          <td>0.062426</td>
          <td>1.001572</td>
          <td>1.008307</td>
        </tr>
        <tr>
          <th>Equity(48804 [TANH])</th>
          <td>0.620471</td>
          <td>1.179510</td>
          <td>1.381538</td>
        </tr>
        <tr>
          <th>Equity(48809 [AIC])</th>
          <td>0.027276</td>
          <td>1.000399</td>
          <td>1.008857</td>
        </tr>
        <tr>
          <th>Equity(48821 [CJES])</th>
          <td>0.851751</td>
          <td>1.220506</td>
          <td>1.335895</td>
        </tr>
        <tr>
          <th>Equity(48822 [CLLS])</th>
          <td>0.230596</td>
          <td>1.014299</td>
          <td>1.023526</td>
        </tr>
        <tr>
          <th>Equity(48823 [SEDG])</th>
          <td>1.228733</td>
          <td>1.207086</td>
          <td>1.234685</td>
        </tr>
        <tr>
          <th>Equity(48853 [SGDJ])</th>
          <td>0.381209</td>
          <td>1.026782</td>
          <td>1.060795</td>
        </tr>
        <tr>
          <th>Equity(48863 [GDDY])</th>
          <td>0.453669</td>
          <td>1.046755</td>
          <td>1.029738</td>
        </tr>
        <tr>
          <th>Equity(48875 [HBHC_L])</th>
          <td>0.025687</td>
          <td>1.001746</td>
          <td>1.005010</td>
        </tr>
      </tbody>
    </table>
    <p>2773 rows × 3 columns</p>
    </div>



Custom factors allow us to define custom computations in a pipeline.
They are frequently the best way to perform computations on `partner
datasets <https://www.quantopian.com/data>`__ or on multiple data
columns. The full documentation for CustomFactors is available
`here <https://www.quantopian.com/help#custom-factors>`__.

In the next lesson, we’ll use everything we’ve learned so far to create
a pipeline for an algorithm.
