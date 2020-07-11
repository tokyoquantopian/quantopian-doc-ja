.. code:: ipython2

    from quantopian.pipeline import Pipeline
    from quantopian.research import run_pipeline
    from quantopian.pipeline.data.builtin import USEquityPricing
    from quantopian.pipeline.factors import SimpleMovingAverage

##Filters A Filter is a function from an asset and a moment in time to a
boolean:

::

   F(asset, timestamp) -> boolean

In Pipeline,
`Filters <https://www.quantopian.com/help#quantopian_pipeline_filters_Filter>`__
are used for narrowing down the set of securities included in a
computation or in the final output of a pipeline. There are two common
ways to create a ``Filter``: comparison operators and
``Factor``/``Classifier`` methods.

###Comparison Operators Comparison operators on ``Factors`` and
``Classifiers`` produce Filters. Since we haven’t looked at
``Classifiers`` yet, let’s stick to examples using ``Factors``. The
following example produces a filter that returns ``True`` whenever the
latest close price is above $20.

.. code:: ipython2

    last_close_price = USEquityPricing.close.latest
    close_price_filter = last_close_price > 20

And this example produces a filter that returns True whenever the 10-day
mean is below the 30-day mean.

.. code:: ipython2

    mean_close_10 = SimpleMovingAverage(inputs=[USEquityPricing.close], window_length=10)
    mean_close_30 = SimpleMovingAverage(inputs=[USEquityPricing.close], window_length=30)
    mean_crossover_filter = mean_close_10 < mean_close_30

Remember, each security will get its own ``True`` or ``False`` value
each day.

###Factor/Classifier Methods Various methods of the ``Factor`` and
``Classifier`` classes return ``Filters``. Again, since we haven’t yet
looked at ``Classifiers``, let’s stick to ``Factor`` methods for now
(we’ll look at ``Classifier`` methods later). The ``Factor.top(n)``
method produces a ``Filter`` that returns ``True`` for the top ``n``
securities of a given ``Factor``. The following example produces a
filter that returns ``True`` for exactly 200 securities every day,
indicating that those securities were in the top 200 by last close price
across all known securities.

.. code:: ipython2

    last_close_price = USEquityPricing.close.latest
    top_close_price_filter = last_close_price.top(200)

For a full list of ``Factor`` methods that return ``Filters``, see `this
link <https://www.quantopian.com/help#quantopian_pipeline_factors_Factor>`__.

For a full list of ``Classifier`` methods that return ``Filters``, see
`this
link <https://www.quantopian.com/help#quantopian_pipeline_classifiers_Classifier>`__.

##Dollar Volume Filter As a starting example, let’s create a filter that
returns ``True`` if a security’s 30-day average dollar volume is above
$10,000,000. To do this, we’ll first need to create an
``AverageDollarVolume`` factor to compute the 30-day average dollar
volume. Let’s include the built-in ``AverageDollarVolume`` factor in our
imports:

.. code:: ipython2

    from quantopian.pipeline.factors import AverageDollarVolume

And then, let’s instantiate our average dollar volume factor.

.. code:: ipython2

    dollar_volume = AverageDollarVolume(window_length=30)

By default, ``AverageDollarVolume`` uses ``USEquityPricing.close`` and
``USEquityPricing.volume`` as its ``inputs``, so we don’t specify them.

Now that we have a dollar volume factor, we can create a filter with a
boolean expression. The following line creates a filter returning
``True`` for securities with a ``dollar_volume`` greater than
10,000,000:

.. code:: ipython2

    high_dollar_volume = (dollar_volume > 10000000)

To see what this filter looks like, let’s can add it as a column to the
pipeline we defined in the previous lesson.

.. code:: ipython2

    def make_pipeline():
    
        mean_close_10 = SimpleMovingAverage(inputs=[USEquityPricing.close], window_length=10)
        mean_close_30 = SimpleMovingAverage(inputs=[USEquityPricing.close], window_length=30)
    
        percent_difference = (mean_close_10 - mean_close_30) / mean_close_30
        
        dollar_volume = AverageDollarVolume(window_length=30)
        high_dollar_volume = (dollar_volume > 10000000)
    
        return Pipeline(
            columns={
                'percent_difference': percent_difference,
                'high_dollar_volume': high_dollar_volume
            }
        )

If we make and run our pipeline, we now have a column
``high_dollar_volume`` with a boolean value corresponding to the result
of the expression for each security.

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
          <th>high_dollar_volume</th>
          <th>percent_difference</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th rowspan="61" valign="top">2015-05-05 00:00:00+00:00</th>
          <th>Equity(2 [AA])</th>
          <td>True</td>
          <td>0.017975</td>
        </tr>
        <tr>
          <th>Equity(21 [AAME])</th>
          <td>False</td>
          <td>-0.002325</td>
        </tr>
        <tr>
          <th>Equity(24 [AAPL])</th>
          <td>True</td>
          <td>0.016905</td>
        </tr>
        <tr>
          <th>Equity(25 [AA_PR])</th>
          <td>False</td>
          <td>0.021544</td>
        </tr>
        <tr>
          <th>Equity(31 [ABAX])</th>
          <td>False</td>
          <td>-0.019639</td>
        </tr>
        <tr>
          <th>Equity(39 [DDC])</th>
          <td>False</td>
          <td>0.074730</td>
        </tr>
        <tr>
          <th>Equity(41 [ARCB])</th>
          <td>False</td>
          <td>0.007067</td>
        </tr>
        <tr>
          <th>Equity(52 [ABM])</th>
          <td>False</td>
          <td>0.003340</td>
        </tr>
        <tr>
          <th>Equity(53 [ABMD])</th>
          <td>True</td>
          <td>-0.024682</td>
        </tr>
        <tr>
          <th>Equity(62 [ABT])</th>
          <td>True</td>
          <td>0.014385</td>
        </tr>
        <tr>
          <th>Equity(64 [ABX])</th>
          <td>True</td>
          <td>0.046963</td>
        </tr>
        <tr>
          <th>Equity(66 [AB])</th>
          <td>False</td>
          <td>0.013488</td>
        </tr>
        <tr>
          <th>Equity(67 [ADSK])</th>
          <td>True</td>
          <td>-0.003921</td>
        </tr>
        <tr>
          <th>Equity(69 [ACAT])</th>
          <td>False</td>
          <td>-0.007079</td>
        </tr>
        <tr>
          <th>Equity(70 [VBF])</th>
          <td>False</td>
          <td>0.005507</td>
        </tr>
        <tr>
          <th>Equity(76 [TAP])</th>
          <td>True</td>
          <td>-0.008759</td>
        </tr>
        <tr>
          <th>Equity(84 [ACET])</th>
          <td>False</td>
          <td>-0.056139</td>
        </tr>
        <tr>
          <th>Equity(86 [ACG])</th>
          <td>False</td>
          <td>0.010096</td>
        </tr>
        <tr>
          <th>Equity(88 [ACI])</th>
          <td>False</td>
          <td>-0.022089</td>
        </tr>
        <tr>
          <th>Equity(100 [IEP])</th>
          <td>False</td>
          <td>0.011293</td>
        </tr>
        <tr>
          <th>Equity(106 [ACU])</th>
          <td>False</td>
          <td>0.003306</td>
        </tr>
        <tr>
          <th>Equity(110 [ACXM])</th>
          <td>False</td>
          <td>-0.029551</td>
        </tr>
        <tr>
          <th>Equity(112 [ACY])</th>
          <td>False</td>
          <td>-0.057763</td>
        </tr>
        <tr>
          <th>Equity(114 [ADBE])</th>
          <td>True</td>
          <td>0.009499</td>
        </tr>
        <tr>
          <th>Equity(117 [AEY])</th>
          <td>False</td>
          <td>0.012543</td>
        </tr>
        <tr>
          <th>Equity(122 [ADI])</th>
          <td>True</td>
          <td>0.009271</td>
        </tr>
        <tr>
          <th>Equity(128 [ADM])</th>
          <td>True</td>
          <td>0.015760</td>
        </tr>
        <tr>
          <th>Equity(134 [SXCL])</th>
          <td>False</td>
          <td>NaN</td>
        </tr>
        <tr>
          <th>Equity(149 [ADX])</th>
          <td>False</td>
          <td>0.007232</td>
        </tr>
        <tr>
          <th>Equity(153 [AE])</th>
          <td>False</td>
          <td>-0.112999</td>
        </tr>
        <tr>
          <th>...</th>
          <td>...</td>
          <td>...</td>
        </tr>
        <tr>
          <th>Equity(48961 [NYMT_O])</th>
          <td>False</td>
          <td>NaN</td>
        </tr>
        <tr>
          <th>Equity(48962 [CSAL])</th>
          <td>True</td>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(48963 [PAK])</th>
          <td>False</td>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(48969 [NSA])</th>
          <td>True</td>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(48971 [BSM])</th>
          <td>True</td>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(48972 [EVA])</th>
          <td>True</td>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(48981 [APIC])</th>
          <td>False</td>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(48989 [UK])</th>
          <td>False</td>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(48990 [ACWF])</th>
          <td>False</td>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(48991 [ISCF])</th>
          <td>False</td>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(48992 [INTF])</th>
          <td>False</td>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(48993 [JETS])</th>
          <td>False</td>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(48994 [ACTX])</th>
          <td>False</td>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(48995 [LRGF])</th>
          <td>False</td>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(48996 [SMLF])</th>
          <td>False</td>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(48997 [VKTX])</th>
          <td>False</td>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(48998 [OPGN])</th>
          <td>False</td>
          <td>NaN</td>
        </tr>
        <tr>
          <th>Equity(48999 [AAPC])</th>
          <td>False</td>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(49000 [BPMC])</th>
          <td>False</td>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(49001 [CLCD])</th>
          <td>False</td>
          <td>NaN</td>
        </tr>
        <tr>
          <th>Equity(49004 [TNP_PRD])</th>
          <td>False</td>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(49005 [ARWA_U])</th>
          <td>False</td>
          <td>NaN</td>
        </tr>
        <tr>
          <th>Equity(49006 [BVXV])</th>
          <td>False</td>
          <td>NaN</td>
        </tr>
        <tr>
          <th>Equity(49007 [BVXV_W])</th>
          <td>False</td>
          <td>NaN</td>
        </tr>
        <tr>
          <th>Equity(49008 [OPGN_W])</th>
          <td>False</td>
          <td>NaN</td>
        </tr>
        <tr>
          <th>Equity(49009 [PRKU])</th>
          <td>False</td>
          <td>NaN</td>
        </tr>
        <tr>
          <th>Equity(49010 [TBRA])</th>
          <td>False</td>
          <td>NaN</td>
        </tr>
        <tr>
          <th>Equity(49131 [OESX])</th>
          <td>False</td>
          <td>NaN</td>
        </tr>
        <tr>
          <th>Equity(49259 [ITUS])</th>
          <td>False</td>
          <td>NaN</td>
        </tr>
        <tr>
          <th>Equity(49523 [TLGT])</th>
          <td>False</td>
          <td>NaN</td>
        </tr>
      </tbody>
    </table>
    <p>8236 rows × 2 columns</p>
    </div>



##Applying a Screen By default, a pipeline produces computed values each
day for every asset in the Quantopian database. Very often however, we
only care about a subset of securities that meet specific criteria (for
example, we might only care about securities that have enough daily
trading volume to fill our orders quickly). We can tell our Pipeline to
ignore securities for which a filter produces ``False`` by passing that
filter to our Pipeline via the ``screen`` keyword.

To screen our pipeline output for securities with a 30-day average
dollar volume greater than $10,000,000, we can simply pass our
``high_dollar_volume`` filter as the ``screen`` argument. This is what
our ``make_pipeline`` function now looks like:

.. code:: ipython2

    def make_pipeline():
    
        mean_close_10 = SimpleMovingAverage(inputs=[USEquityPricing.close], window_length=10)
        mean_close_30 = SimpleMovingAverage(inputs=[USEquityPricing.close], window_length=30)
    
        percent_difference = (mean_close_10 - mean_close_30) / mean_close_30
    
        dollar_volume = AverageDollarVolume(window_length=30)
        high_dollar_volume = dollar_volume > 10000000
    
        return Pipeline(
            columns={
                'percent_difference': percent_difference
            },
            screen=high_dollar_volume
        )

When we run this, the pipeline output only includes securities that pass
the ``high_dollar_volume`` filter on a given day. For example, running
this pipeline on May 5th, 2015 results in an output for ~2,100
securities

.. code:: ipython2

    result = run_pipeline(make_pipeline(), '2015-05-05', '2015-05-05')
    print 'Number of securities that passed the filter: %d' % len(result)
    result


.. parsed-literal::

    Number of securities that passed the filter: 2110




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
          <th>Equity(24 [AAPL])</th>
          <td>0.016905</td>
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
          <th>Equity(67 [ADSK])</th>
          <td>-0.003921</td>
        </tr>
        <tr>
          <th>Equity(76 [TAP])</th>
          <td>-0.008759</td>
        </tr>
        <tr>
          <th>Equity(114 [ADBE])</th>
          <td>0.009499</td>
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
          <th>Equity(154 [AEM])</th>
          <td>0.026035</td>
        </tr>
        <tr>
          <th>Equity(161 [AEP])</th>
          <td>0.010405</td>
        </tr>
        <tr>
          <th>Equity(166 [AES])</th>
          <td>0.022158</td>
        </tr>
        <tr>
          <th>Equity(168 [AET])</th>
          <td>0.005853</td>
        </tr>
        <tr>
          <th>Equity(185 [AFL])</th>
          <td>-0.002239</td>
        </tr>
        <tr>
          <th>Equity(197 [AGCO])</th>
          <td>0.032124</td>
        </tr>
        <tr>
          <th>Equity(216 [HES])</th>
          <td>0.036528</td>
        </tr>
        <tr>
          <th>Equity(239 [AIG])</th>
          <td>0.012322</td>
        </tr>
        <tr>
          <th>Equity(253 [AIR])</th>
          <td>-0.012412</td>
        </tr>
        <tr>
          <th>Equity(266 [AJG])</th>
          <td>0.012267</td>
        </tr>
        <tr>
          <th>Equity(270 [AKRX])</th>
          <td>-0.024963</td>
        </tr>
        <tr>
          <th>Equity(273 [ALU])</th>
          <td>-0.021750</td>
        </tr>
        <tr>
          <th>Equity(300 [ALK])</th>
          <td>0.015147</td>
        </tr>
        <tr>
          <th>Equity(301 [ALKS])</th>
          <td>-0.033228</td>
        </tr>
        <tr>
          <th>Equity(328 [ALTR])</th>
          <td>0.012284</td>
        </tr>
        <tr>
          <th>Equity(337 [AMAT])</th>
          <td>-0.050162</td>
        </tr>
        <tr>
          <th>Equity(351 [AMD])</th>
          <td>-0.101477</td>
        </tr>
        <tr>
          <th>Equity(353 [AME])</th>
          <td>-0.003008</td>
        </tr>
        <tr>
          <th>Equity(357 [TWX])</th>
          <td>0.000365</td>
        </tr>
        <tr>
          <th>Equity(368 [AMGN])</th>
          <td>0.008860</td>
        </tr>
        <tr>
          <th>...</th>
          <td>...</td>
        </tr>
        <tr>
          <th>Equity(48126 [HABT])</th>
          <td>0.063080</td>
        </tr>
        <tr>
          <th>Equity(48129 [UBS])</th>
          <td>0.025888</td>
        </tr>
        <tr>
          <th>Equity(48169 [KLXI])</th>
          <td>0.021062</td>
        </tr>
        <tr>
          <th>Equity(48215 [QSR])</th>
          <td>0.037460</td>
        </tr>
        <tr>
          <th>Equity(48220 [LC])</th>
          <td>-0.035048</td>
        </tr>
        <tr>
          <th>Equity(48317 [JUNO])</th>
          <td>-0.103370</td>
        </tr>
        <tr>
          <th>Equity(48384 [QRVO])</th>
          <td>-0.050578</td>
        </tr>
        <tr>
          <th>Equity(48465 [SWNC])</th>
          <td>0.061669</td>
        </tr>
        <tr>
          <th>Equity(48486 [BOX])</th>
          <td>-0.003837</td>
        </tr>
        <tr>
          <th>Equity(48531 [VSTO])</th>
          <td>0.017196</td>
        </tr>
        <tr>
          <th>Equity(48543 [SHAK])</th>
          <td>0.175877</td>
        </tr>
        <tr>
          <th>Equity(48544 [HIFR])</th>
          <td>0.027339</td>
        </tr>
        <tr>
          <th>Equity(48547 [ONCE])</th>
          <td>-0.112191</td>
        </tr>
        <tr>
          <th>Equity(48575 [XHR])</th>
          <td>-0.008521</td>
        </tr>
        <tr>
          <th>Equity(48629 [INOV])</th>
          <td>-0.068366</td>
        </tr>
        <tr>
          <th>Equity(48662 [JPM_PRF])</th>
          <td>0.002978</td>
        </tr>
        <tr>
          <th>Equity(48672 [TOTL])</th>
          <td>0.000991</td>
        </tr>
        <tr>
          <th>Equity(48730 [AGN_PRA])</th>
          <td>-0.008843</td>
        </tr>
        <tr>
          <th>Equity(48821 [CJES])</th>
          <td>0.099492</td>
        </tr>
        <tr>
          <th>Equity(48823 [SEDG])</th>
          <td>0.056643</td>
        </tr>
        <tr>
          <th>Equity(48863 [GDDY])</th>
          <td>-0.003563</td>
        </tr>
        <tr>
          <th>Equity(48892 [IGT])</th>
          <td>0.005591</td>
        </tr>
        <tr>
          <th>Equity(48925 [ADRO])</th>
          <td>-0.076840</td>
        </tr>
        <tr>
          <th>Equity(48933 [PRTY])</th>
          <td>-0.001741</td>
        </tr>
        <tr>
          <th>Equity(48934 [ETSY])</th>
          <td>-0.030142</td>
        </tr>
        <tr>
          <th>Equity(48943 [VIRT])</th>
          <td>-0.009077</td>
        </tr>
        <tr>
          <th>Equity(48962 [CSAL])</th>
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
      </tbody>
    </table>
    <p>2110 rows × 1 columns</p>
    </div>



##Inverting a Filter The ``~`` operator is used to invert a filter,
swapping all ``True`` values with ``Falses`` and vice-versa. For
example, we can write the following to filter for low dollar volume
securities:

.. code:: ipython2

    low_dollar_volume = ~high_dollar_volume

This will return ``True`` for all securities with an average dollar
volume below or equal to $10,000,000 over the last 30 days.

In the next lesson, we will look at combining filters.
