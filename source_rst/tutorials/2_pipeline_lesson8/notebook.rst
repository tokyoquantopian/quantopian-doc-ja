.. code:: ipython2

    from quantopian.pipeline import Pipeline
    from quantopian.research import run_pipeline
    from quantopian.pipeline.factors import SimpleMovingAverage, AverageDollarVolume

##Classifiers A classifier is a function from an asset and a moment in
time to a `categorical
output <https://en.wikipedia.org/wiki/Categorical_variable>`__ such as a
``string`` or ``integer`` label:

::

   F(asset, timestamp) -> category

An example of a classifier producing a string output is the exchange ID
of a security. To create this classifier, we’ll have to import
``Fundamentals.exchange_id`` and use the
`latest <https://www.quantopian.com/tutorials/pipeline#lesson3>`__
attribute to instantiate our classifier:

.. code:: ipython2

    from quantopian.pipeline.data import Fundamentals
    
    # Since the underlying data of Fundamentals.exchange_id
    # is of type string, .latest returns a Classifier
    exchange = Fundamentals.exchange_id.latest

Previously, we saw that the ``latest`` attribute produced an instance of
a ``Factor``. In this case, since the underlying data is of type
``string``, ``latest`` produces a ``Classifier``.

Similarly, a computation producing the latest Morningstar sector code of
a security is a ``Classifier``. In this case, the underlying type is an
``int``, but the integer doesn’t represent a numerical value (it’s a
category) so it produces a classifier. To get the latest sector code, we
can use the built-in ``Sector`` classifier.

.. code:: ipython2

    from quantopian.pipeline.classifiers.fundamentals import Sector  
    morningstar_sector = Sector()

Using ``Sector`` is equivalent to
``Fundamentals.morningstar_sector_code.latest``.

###Building Filters from Classifiers Classifiers can also be used to
produce filters with methods like ``isnull``, ``eq``, and
``startswith``. The full list of ``Classifier`` methods producing
``Filters`` can be found
`here <https://www.quantopian.com/help#quantopian_pipeline_classifiers_Classifier>`__.

As an example, if we wanted a filter to select for securities trading on
the New York Stock Exchange, we can use the ``eq`` method of our
``exchange`` classifier.

.. code:: ipython2

    nyse_filter = exchange.eq('NYS')

This filter will return ``True`` for securities having ``'NYS'`` as
their most recent ``exchange_id``.

###Quantiles Classifiers can also be produced from various ``Factor``
methods. The most general of these is the ``quantiles`` method which
accepts a bin count as an argument. The ``quantiles`` method assigns a
label from 0 to (bins - 1) to every non-NaN data point in the factor
output and returns a ``Classifier`` with these labels. ``NaN``\ s are
labeled with -1. Aliases are available for
`quartiles <https://www.quantopian.com/help/#quantopian_pipeline_factors_Factor_quartiles>`__
(``quantiles(4)``),
`quintiles <https://www.quantopian.com/help/#quantopian_pipeline_factors_Factor_quintiles>`__
(``quantiles(5)``), and
`deciles <https://www.quantopian.com/help/#quantopian_pipeline_factors_Factor_deciles>`__
(``quantiles(10)``). As an example, this is what a filter for the top
decile of a factor might look like:

.. code:: ipython2

    dollar_volume_decile = AverageDollarVolume(window_length=10).deciles()
    top_decile = (dollar_volume_decile.eq(9))

Let’s put each of our classifiers into a pipeline and run it to see what
they look like.

.. code:: ipython2

    def make_pipeline():
        exchange = Fundamentals.exchange_id.latest
        nyse_filter = exchange.eq('NYS')
    
        morningstar_sector = Sector()
    
        dollar_volume_decile = AverageDollarVolume(window_length=10).deciles()
        top_decile = (dollar_volume_decile.eq(9))
    
        return Pipeline(
            columns={
                'exchange': exchange,
                'sector_code': morningstar_sector,
                'dollar_volume_decile': dollar_volume_decile
            },
            screen=(nyse_filter & top_decile)
        )

.. code:: ipython2

    result = run_pipeline(make_pipeline(), '2015-05-05', '2015-05-05')
    print 'Number of securities that passed the filter: %d' % len(result)
    result.head(5)


.. parsed-literal::

    Number of securities that passed the filter: 513




.. raw:: html

    <div>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th></th>
          <th>dollar_volume_decile</th>
          <th>exchange</th>
          <th>sector_code</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th rowspan="5" valign="top">2015-05-05 00:00:00+00:00</th>
          <th>Equity(2 [ARNC])</th>
          <td>9</td>
          <td>NYS</td>
          <td>101</td>
        </tr>
        <tr>
          <th>Equity(62 [ABT])</th>
          <td>9</td>
          <td>NYS</td>
          <td>206</td>
        </tr>
        <tr>
          <th>Equity(64 [ABX])</th>
          <td>9</td>
          <td>NYS</td>
          <td>101</td>
        </tr>
        <tr>
          <th>Equity(76 [TAP])</th>
          <td>9</td>
          <td>NYS</td>
          <td>205</td>
        </tr>
        <tr>
          <th>Equity(128 [ADM])</th>
          <td>9</td>
          <td>NYS</td>
          <td>205</td>
        </tr>
      </tbody>
    </table>
    </div>



Classifiers are also useful for describing grouping keys for complex
transformations on Factor outputs. Grouping operations such as
`demean <https://www.quantopian.com/help#quantopian_pipeline_factors_Factor_demean>`__
and
`groupby <https://www.quantopian.com/help#quantopian_pipeline_factors_Factor_groupby>`__
are outside the scope of this tutorial. A future tutorial will cover
more advanced uses for classifiers.

In the next lesson, we’ll look at the different datasets that we can use
in pipeline.
