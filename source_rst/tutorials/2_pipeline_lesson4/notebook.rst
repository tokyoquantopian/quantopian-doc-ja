##Combining Factors Factors can be combined, both with other Factors and
with scalar values, via any of the builtin mathematical operators (+, -,
\*, etc). This makes it easy to write complex expressions that combine
multiple Factors. For example, constructing a Factor that computes the
average of two other Factors is simply:

::

   >>> f1 = SomeFactor(...)
   >>> f2 = SomeOtherFactor(...)
   >>> average = (f1 + f2) / 2.0

In this lesson, we will create a pipeline that creates a
``relative_difference`` factor by combining a 10-day average factor and
a 30-day average factor.

As usual, let’s start with our imports:

.. code:: ipython2

    from quantopian.pipeline import Pipeline
    from quantopian.research import run_pipeline
    from quantopian.pipeline.data.builtin import USEquityPricing
    from quantopian.pipeline.factors import SimpleMovingAverage

For this example, we need two factors: a 10-day mean close price factor,
and a 30-day one:

.. code:: ipython2

    mean_close_10 = SimpleMovingAverage(inputs=[USEquityPricing.close], window_length=10)
    mean_close_30 = SimpleMovingAverage(inputs=[USEquityPricing.close], window_length=30)

Then, let’s create a percent difference factor by combining our
``mean_close_30`` factor with our ``mean_close_10`` factor.

.. code:: ipython2

    percent_difference = (mean_close_10 - mean_close_30) / mean_close_30

In this example, ``percent_difference`` is still a ``Factor`` even
though it’s composed as a combination of more primitive factors. We can
add ``percent_difference`` as a column in our pipeline. Let’s define
``make_pipeline`` to create a pipeline with ``percent_difference`` as a
column (and not the mean close factors):

.. code:: ipython2

    def make_pipeline():
    
        mean_close_10 = SimpleMovingAverage(inputs=[USEquityPricing.close], window_length=10)
        mean_close_30 = SimpleMovingAverage(inputs=[USEquityPricing.close], window_length=30)
    
        percent_difference = (mean_close_10 - mean_close_30) / mean_close_30
    
        return Pipeline(
            columns={
                'percent_difference': percent_difference
            }
        )

Let’s see what the new output looks like:

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
          <th>percent_difference</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th rowspan="61" valign="top">2015-05-05 00:00:00+00:00</th>
          <th>Equity(2 [AA])</th>
          <td>0.017975</td>
        </tr>
        <tr>
          <th>Equity(21 [AAME])</th>
          <td>-0.002325</td>
        </tr>
        <tr>
          <th>Equity(24 [AAPL])</th>
          <td>0.016905</td>
        </tr>
        <tr>
          <th>Equity(25 [AA_PR])</th>
          <td>0.021544</td>
        </tr>
        <tr>
          <th>Equity(31 [ABAX])</th>
          <td>-0.019639</td>
        </tr>
        <tr>
          <th>Equity(39 [DDC])</th>
          <td>0.074730</td>
        </tr>
        <tr>
          <th>Equity(41 [ARCB])</th>
          <td>0.007067</td>
        </tr>
        <tr>
          <th>Equity(52 [ABM])</th>
          <td>0.003340</td>
        </tr>
        <tr>
          <th>Equity(53 [ABMD])</th>
          <td>-0.024682</td>
        </tr>
        <tr>
          <th>Equity(62 [ABT])</th>
          <td>0.014385</td>
        </tr>
        <tr>
          <th>Equity(64 [ABX])</th>
          <td>0.046963</td>
        </tr>
        <tr>
          <th>Equity(66 [AB])</th>
          <td>0.013488</td>
        </tr>
        <tr>
          <th>Equity(67 [ADSK])</th>
          <td>-0.003921</td>
        </tr>
        <tr>
          <th>Equity(69 [ACAT])</th>
          <td>-0.007079</td>
        </tr>
        <tr>
          <th>Equity(70 [VBF])</th>
          <td>0.005507</td>
        </tr>
        <tr>
          <th>Equity(76 [TAP])</th>
          <td>-0.008759</td>
        </tr>
        <tr>
          <th>Equity(84 [ACET])</th>
          <td>-0.056139</td>
        </tr>
        <tr>
          <th>Equity(86 [ACG])</th>
          <td>0.010096</td>
        </tr>
        <tr>
          <th>Equity(88 [ACI])</th>
          <td>-0.022089</td>
        </tr>
        <tr>
          <th>Equity(100 [IEP])</th>
          <td>0.011293</td>
        </tr>
        <tr>
          <th>Equity(106 [ACU])</th>
          <td>0.003306</td>
        </tr>
        <tr>
          <th>Equity(110 [ACXM])</th>
          <td>-0.029551</td>
        </tr>
        <tr>
          <th>Equity(112 [ACY])</th>
          <td>-0.057763</td>
        </tr>
        <tr>
          <th>Equity(114 [ADBE])</th>
          <td>0.009499</td>
        </tr>
        <tr>
          <th>Equity(117 [AEY])</th>
          <td>0.012543</td>
        </tr>
        <tr>
          <th>Equity(122 [ADI])</th>
          <td>0.009271</td>
        </tr>
        <tr>
          <th>Equity(128 [ADM])</th>
          <td>0.015760</td>
        </tr>
        <tr>
          <th>Equity(134 [SXCL])</th>
          <td>NaN</td>
        </tr>
        <tr>
          <th>Equity(149 [ADX])</th>
          <td>0.007232</td>
        </tr>
        <tr>
          <th>Equity(153 [AE])</th>
          <td>-0.112999</td>
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
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(48963 [PAK])</th>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(48969 [NSA])</th>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(48971 [BSM])</th>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(48972 [EVA])</th>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(48981 [APIC])</th>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(48989 [UK])</th>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(48990 [ACWF])</th>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(48991 [ISCF])</th>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(48992 [INTF])</th>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(48993 [JETS])</th>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(48994 [ACTX])</th>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(48995 [LRGF])</th>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(48996 [SMLF])</th>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(48997 [VKTX])</th>
          <td>0.000000</td>
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
    <p>8235 rows × 1 columns</p>
    </div>



In the next lesson, we will learn about filters.
