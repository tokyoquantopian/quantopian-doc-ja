##Factors A factor is a function from an asset and a moment in time to a
number.

::

   F(asset, timestamp) -> float

In Pipeline,
`Factors <https://www.quantopian.com/help#quantopian_pipeline_factors_Factor>`__
are the most commonly-used term, representing the result of any
computation producing a numerical result. Factors require a column of
data and a window length as input.

The simplest factors in Pipeline are `built-in
Factors <https://www.quantopian.com/help#built-in-factors>`__. Built-in
Factors are pre-built to perform common computations. As a first
example, let’s make a factor to compute the average close price over the
last 10 days. We can use the ``SimpleMovingAverage`` built-in factor
which computes the average value of the input data (close price) over
the specified window length (10 days). To do this, we need to import our
built-in ``SimpleMovingAverage`` factor and the `USEquityPricing
dataset <https://www.quantopian.com/help#importing-datasets>`__.

.. code:: ipython2

    from quantopian.pipeline import Pipeline
    from quantopian.research import run_pipeline
    
    # New from the last lesson, import the USEquityPricing dataset.
    from quantopian.pipeline.data.builtin import USEquityPricing
    
    # New from the last lesson, import the built-in SimpleMovingAverage factor.
    from quantopian.pipeline.factors import SimpleMovingAverage

###Creating a Factor Let’s go back to our ``make_pipeline`` function
from the previous lesson and instantiate a ``SimpleMovingAverage``
factor. To create a ``SimpleMovingAverage`` factor, we can call the
``SimpleMovingAverage`` constructor with two arguments: inputs, which
must be a list of ``BoundColumn`` objects, and window_length, which must
be an integer indicating how many days worth of data our moving average
calculation should receive. (We’ll discuss ``BoundColumn`` in more depth
later; for now we just need to know that a ``BoundColumn`` is an object
indicating what kind of data should be passed to our Factor.).

The following line creates a ``Factor`` for computing the 10-day mean
close price of securities.

.. code:: ipython2

    mean_close_10 = SimpleMovingAverage(inputs=[USEquityPricing.close], window_length=10)

It’s important to note that creating the factor does not actually
perform a computation. Creating a factor is like defining the function.
To perform a computation, we need to add the factor to our pipeline and
run it.

###Adding a Factor to a Pipeline Let’s update our original empty
pipeline to make it compute our new moving average factor. To start,
let’s move our factor instantatiation into ``make_pipeline``. Next, we
can tell our pipeline to compute our factor by passing it a ``columns``
argument, which should be a dictionary mapping column names to factors,
filters, or classifiers. Our updated ``make_pipeline`` function should
look something like this:

.. code:: ipython2

    def make_pipeline():
        
        mean_close_10 = SimpleMovingAverage(inputs=[USEquityPricing.close], window_length=10)
        
        return Pipeline(
            columns={
                '10_day_mean_close': mean_close_10
            }
        )

To see what this looks like, let’s make our pipeline, run it, and
display the result.

.. code:: ipython2

    result = run_pipeline(make_pipeline(), '2015-05-05', '2015-05-05')
    result




.. raw:: html

    <div style="max-height:1000px;max-width:1500px;overflow:auto;">
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th></th>
          <th>10_day_mean_close</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th rowspan="61" valign="top">2015-05-05 00:00:00+00:00</th>
          <th>Equity(2 [AA])</th>
          <td>13.559500</td>
        </tr>
        <tr>
          <th>Equity(21 [AAME])</th>
          <td>3.962500</td>
        </tr>
        <tr>
          <th>Equity(24 [AAPL])</th>
          <td>129.025700</td>
        </tr>
        <tr>
          <th>Equity(25 [AA_PR])</th>
          <td>88.362500</td>
        </tr>
        <tr>
          <th>Equity(31 [ABAX])</th>
          <td>61.920900</td>
        </tr>
        <tr>
          <th>Equity(39 [DDC])</th>
          <td>19.287072</td>
        </tr>
        <tr>
          <th>Equity(41 [ARCB])</th>
          <td>37.880000</td>
        </tr>
        <tr>
          <th>Equity(52 [ABM])</th>
          <td>32.083400</td>
        </tr>
        <tr>
          <th>Equity(53 [ABMD])</th>
          <td>66.795000</td>
        </tr>
        <tr>
          <th>Equity(62 [ABT])</th>
          <td>47.466000</td>
        </tr>
        <tr>
          <th>Equity(64 [ABX])</th>
          <td>12.919000</td>
        </tr>
        <tr>
          <th>Equity(66 [AB])</th>
          <td>31.547000</td>
        </tr>
        <tr>
          <th>Equity(67 [ADSK])</th>
          <td>60.212000</td>
        </tr>
        <tr>
          <th>Equity(69 [ACAT])</th>
          <td>36.331000</td>
        </tr>
        <tr>
          <th>Equity(70 [VBF])</th>
          <td>18.767000</td>
        </tr>
        <tr>
          <th>Equity(76 [TAP])</th>
          <td>74.632000</td>
        </tr>
        <tr>
          <th>Equity(84 [ACET])</th>
          <td>19.873000</td>
        </tr>
        <tr>
          <th>Equity(86 [ACG])</th>
          <td>7.810000</td>
        </tr>
        <tr>
          <th>Equity(88 [ACI])</th>
          <td>0.996100</td>
        </tr>
        <tr>
          <th>Equity(100 [IEP])</th>
          <td>91.821200</td>
        </tr>
        <tr>
          <th>Equity(106 [ACU])</th>
          <td>18.641000</td>
        </tr>
        <tr>
          <th>Equity(110 [ACXM])</th>
          <td>18.045500</td>
        </tr>
        <tr>
          <th>Equity(112 [ACY])</th>
          <td>11.571000</td>
        </tr>
        <tr>
          <th>Equity(114 [ADBE])</th>
          <td>76.072000</td>
        </tr>
        <tr>
          <th>Equity(117 [AEY])</th>
          <td>2.423400</td>
        </tr>
        <tr>
          <th>Equity(122 [ADI])</th>
          <td>63.205900</td>
        </tr>
        <tr>
          <th>Equity(128 [ADM])</th>
          <td>48.788500</td>
        </tr>
        <tr>
          <th>Equity(134 [SXCL])</th>
          <td>NaN</td>
        </tr>
        <tr>
          <th>Equity(149 [ADX])</th>
          <td>14.150500</td>
        </tr>
        <tr>
          <th>Equity(153 [AE])</th>
          <td>54.099000</td>
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
          <td>29.992000</td>
        </tr>
        <tr>
          <th>Equity(48963 [PAK])</th>
          <td>15.531875</td>
        </tr>
        <tr>
          <th>Equity(48969 [NSA])</th>
          <td>13.045000</td>
        </tr>
        <tr>
          <th>Equity(48971 [BSM])</th>
          <td>17.995000</td>
        </tr>
        <tr>
          <th>Equity(48972 [EVA])</th>
          <td>21.413250</td>
        </tr>
        <tr>
          <th>Equity(48981 [APIC])</th>
          <td>14.814000</td>
        </tr>
        <tr>
          <th>Equity(48989 [UK])</th>
          <td>24.946667</td>
        </tr>
        <tr>
          <th>Equity(48990 [ACWF])</th>
          <td>25.250000</td>
        </tr>
        <tr>
          <th>Equity(48991 [ISCF])</th>
          <td>24.985000</td>
        </tr>
        <tr>
          <th>Equity(48992 [INTF])</th>
          <td>25.030000</td>
        </tr>
        <tr>
          <th>Equity(48993 [JETS])</th>
          <td>24.579333</td>
        </tr>
        <tr>
          <th>Equity(48994 [ACTX])</th>
          <td>15.097333</td>
        </tr>
        <tr>
          <th>Equity(48995 [LRGF])</th>
          <td>24.890000</td>
        </tr>
        <tr>
          <th>Equity(48996 [SMLF])</th>
          <td>29.456667</td>
        </tr>
        <tr>
          <th>Equity(48997 [VKTX])</th>
          <td>9.115000</td>
        </tr>
        <tr>
          <th>Equity(48998 [OPGN])</th>
          <td>NaN</td>
        </tr>
        <tr>
          <th>Equity(48999 [AAPC])</th>
          <td>10.144000</td>
        </tr>
        <tr>
          <th>Equity(49000 [BPMC])</th>
          <td>20.810000</td>
        </tr>
        <tr>
          <th>Equity(49001 [CLCD])</th>
          <td>NaN</td>
        </tr>
        <tr>
          <th>Equity(49004 [TNP_PRD])</th>
          <td>24.750000</td>
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



Now we have a column in our pipeline output with the 10-day average
close price for all 8000+ securities (display truncated). Note that each
row corresponds to the result of our computation for a given security on
a given date stored. The ``DataFrame`` has a
`MultiIndex <http://pandas.pydata.org/pandas-docs/version/0.16.2/advanced.html>`__
where the first level is a datetime representing the date of the
computation and the second level is an
`Equity <http://localhost:3000/help#api-sidinfo>`__ object corresponding
to the security. For example, the first row
(``2015-05-05 00:00:00+00:00``, ``Equity(2 [AA])``) will contain the
result of our ``mean_close_10`` factor for AA on May 5th, 2015.

If we run our pipeline over more than one day, the output looks like
this.

.. code:: ipython2

    result = run_pipeline(make_pipeline(), '2015-05-05', '2015-05-07')
    result




.. raw:: html

    <div style="max-height:1000px;max-width:1500px;overflow:auto;">
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th></th>
          <th>10_day_mean_close</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th rowspan="30" valign="top">2015-05-05 00:00:00+00:00</th>
          <th>Equity(2 [AA])</th>
          <td>13.559500</td>
        </tr>
        <tr>
          <th>Equity(21 [AAME])</th>
          <td>3.962500</td>
        </tr>
        <tr>
          <th>Equity(24 [AAPL])</th>
          <td>129.025700</td>
        </tr>
        <tr>
          <th>Equity(25 [AA_PR])</th>
          <td>88.362500</td>
        </tr>
        <tr>
          <th>Equity(31 [ABAX])</th>
          <td>61.920900</td>
        </tr>
        <tr>
          <th>Equity(39 [DDC])</th>
          <td>19.287072</td>
        </tr>
        <tr>
          <th>Equity(41 [ARCB])</th>
          <td>37.880000</td>
        </tr>
        <tr>
          <th>Equity(52 [ABM])</th>
          <td>32.083400</td>
        </tr>
        <tr>
          <th>Equity(53 [ABMD])</th>
          <td>66.795000</td>
        </tr>
        <tr>
          <th>Equity(62 [ABT])</th>
          <td>47.466000</td>
        </tr>
        <tr>
          <th>Equity(64 [ABX])</th>
          <td>12.919000</td>
        </tr>
        <tr>
          <th>Equity(66 [AB])</th>
          <td>31.547000</td>
        </tr>
        <tr>
          <th>Equity(67 [ADSK])</th>
          <td>60.212000</td>
        </tr>
        <tr>
          <th>Equity(69 [ACAT])</th>
          <td>36.331000</td>
        </tr>
        <tr>
          <th>Equity(70 [VBF])</th>
          <td>18.767000</td>
        </tr>
        <tr>
          <th>Equity(76 [TAP])</th>
          <td>74.632000</td>
        </tr>
        <tr>
          <th>Equity(84 [ACET])</th>
          <td>19.873000</td>
        </tr>
        <tr>
          <th>Equity(86 [ACG])</th>
          <td>7.810000</td>
        </tr>
        <tr>
          <th>Equity(88 [ACI])</th>
          <td>0.996100</td>
        </tr>
        <tr>
          <th>Equity(100 [IEP])</th>
          <td>91.821200</td>
        </tr>
        <tr>
          <th>Equity(106 [ACU])</th>
          <td>18.641000</td>
        </tr>
        <tr>
          <th>Equity(110 [ACXM])</th>
          <td>18.045500</td>
        </tr>
        <tr>
          <th>Equity(112 [ACY])</th>
          <td>11.571000</td>
        </tr>
        <tr>
          <th>Equity(114 [ADBE])</th>
          <td>76.072000</td>
        </tr>
        <tr>
          <th>Equity(117 [AEY])</th>
          <td>2.423400</td>
        </tr>
        <tr>
          <th>Equity(122 [ADI])</th>
          <td>63.205900</td>
        </tr>
        <tr>
          <th>Equity(128 [ADM])</th>
          <td>48.788500</td>
        </tr>
        <tr>
          <th>Equity(134 [SXCL])</th>
          <td>NaN</td>
        </tr>
        <tr>
          <th>Equity(149 [ADX])</th>
          <td>14.150500</td>
        </tr>
        <tr>
          <th>Equity(153 [AE])</th>
          <td>54.099000</td>
        </tr>
        <tr>
          <th>...</th>
          <th>...</th>
          <td>...</td>
        </tr>
        <tr>
          <th rowspan="30" valign="top">2015-05-07 00:00:00+00:00</th>
          <th>Equity(48981 [APIC])</th>
          <td>14.646000</td>
        </tr>
        <tr>
          <th>Equity(48989 [UK])</th>
          <td>24.878000</td>
        </tr>
        <tr>
          <th>Equity(48990 [ACWF])</th>
          <td>25.036667</td>
        </tr>
        <tr>
          <th>Equity(48991 [ISCF])</th>
          <td>24.875000</td>
        </tr>
        <tr>
          <th>Equity(48992 [INTF])</th>
          <td>24.813000</td>
        </tr>
        <tr>
          <th>Equity(48993 [JETS])</th>
          <td>24.343600</td>
        </tr>
        <tr>
          <th>Equity(48994 [ACTX])</th>
          <td>15.020400</td>
        </tr>
        <tr>
          <th>Equity(48995 [LRGF])</th>
          <td>24.788000</td>
        </tr>
        <tr>
          <th>Equity(48996 [SMLF])</th>
          <td>29.370000</td>
        </tr>
        <tr>
          <th>Equity(48997 [VKTX])</th>
          <td>9.232500</td>
        </tr>
        <tr>
          <th>Equity(48998 [OPGN])</th>
          <td>4.950000</td>
        </tr>
        <tr>
          <th>Equity(48999 [AAPC])</th>
          <td>10.167000</td>
        </tr>
        <tr>
          <th>Equity(49000 [BPMC])</th>
          <td>20.906667</td>
        </tr>
        <tr>
          <th>Equity(49001 [CLCD])</th>
          <td>8.010000</td>
        </tr>
        <tr>
          <th>Equity(49004 [TNP_PRD])</th>
          <td>24.633333</td>
        </tr>
        <tr>
          <th>Equity(49005 [ARWA_U])</th>
          <td>10.010000</td>
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
          <td>0.817500</td>
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
          <th>Equity(49015 [ADAP])</th>
          <td>NaN</td>
        </tr>
        <tr>
          <th>Equity(49016 [COLL])</th>
          <td>NaN</td>
        </tr>
        <tr>
          <th>Equity(49017 [GLSS])</th>
          <td>NaN</td>
        </tr>
        <tr>
          <th>Equity(49018 [HTGM])</th>
          <td>NaN</td>
        </tr>
        <tr>
          <th>Equity(49019 [LRET])</th>
          <td>NaN</td>
        </tr>
        <tr>
          <th>Equity(49020 [MVIR])</th>
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
    <p>24705 rows × 1 columns</p>
    </div>



Note: factors can also be added to an existing ``Pipeline`` instance
using the ``Pipeline.add`` method. Using ``add`` looks something like
this: >>> my_pipe = Pipeline() >>> f1 = SomeFactor(…) >>>
my_pipe.add(f1, ‘f1’)

###Latest The most commonly used built-in ``Factor`` is ``Latest``. The
``Latest`` factor gets the most recent value of a given data column.
This factor is common enough that it is instantiated differently from
other factors. The best way to get the latest value of a data column is
by getting its ``.latest`` attribute. As an example, let’s update
``make_pipeline`` to create a latest close price factor and add it to
our pipeline:

.. code:: ipython2

    def make_pipeline():
    
        mean_close_10 = SimpleMovingAverage(inputs=[USEquityPricing.close], window_length=10)
        latest_close = USEquityPricing.close.latest
    
        return Pipeline(
            columns={
                '10_day_mean_close': mean_close_10,
                'latest_close_price': latest_close
            }
        )

And now, when we make and run our pipeline again, there are two columns
in our output dataframe. One column has the 10-day mean close price of
each security, and the other has the latest close price.

.. code:: ipython2

    result = run_pipeline(make_pipeline(), '2015-05-05', '2015-05-05')
    result.head(5)




.. raw:: html

    <div style="max-height:1000px;max-width:1500px;overflow:auto;">
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th></th>
          <th>10_day_mean_close</th>
          <th>latest_close_price</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th rowspan="5" valign="top">2015-05-05 00:00:00+00:00</th>
          <th>Equity(2 [AA])</th>
          <td>13.5595</td>
          <td>14.015</td>
        </tr>
        <tr>
          <th>Equity(21 [AAME])</th>
          <td>3.9625</td>
          <td>NaN</td>
        </tr>
        <tr>
          <th>Equity(24 [AAPL])</th>
          <td>129.0257</td>
          <td>128.699</td>
        </tr>
        <tr>
          <th>Equity(25 [AA_PR])</th>
          <td>88.3625</td>
          <td>NaN</td>
        </tr>
        <tr>
          <th>Equity(31 [ABAX])</th>
          <td>61.9209</td>
          <td>55.030</td>
        </tr>
      </tbody>
    </table>
    </div>



``.latest`` can sometimes return things other than ``Factors``. We’ll
see examples of other possible return types in later lessons.

Default Inputs
--------------

Some factors have default inputs that should never be changed. For
example the `VWAP built-in
factor <https://www.quantopian.com/help#built-in-factors>`__ is always
calculated from ``USEquityPricing.close`` and
``USEquityPricing.volume``. When a factor is always calculated from the
same ``BoundColumns``, we can call the constructor without specifying
``inputs``.

.. code:: ipython2

    from quantopian.pipeline.factors import VWAP
    vwap = VWAP(window_length=10)

In the next lesson, we will look at combining factors.
