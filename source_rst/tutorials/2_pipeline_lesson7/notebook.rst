.. code:: ipython2

    from quantopian.pipeline import Pipeline
    from quantopian.research import run_pipeline
    from quantopian.pipeline.data.builtin import USEquityPricing
    from quantopian.pipeline.factors import SimpleMovingAverage, AverageDollarVolume

##Masking Sometimes we want to ignore certain assets when computing
pipeline expresssions. There are two common cases where ignoring assets
is useful: 1. We want to compute an expression that’s computationally
expensive, and we know we only care about results for certain assets. A
common example of such an expensive expression is a ``Factor`` computing
the coefficients of a regression
(`RollingLinearRegressionOfReturns <https://www.quantopian.com/help#quantopian_pipeline_factors_RollingLinearRegressionOfReturns>`__).
2. We want to compute an expression that performs comparisons between
assets, but we only want those comparisons to be performed against a
subset of all assets. For example, we might want to use the ``Factor``
method ``top`` to compute the top 200 assets by earnings yield, ignoring
assets that don’t meet some liquidity constraint.

To support these two use-cases, all ``Factors`` and many ``Factor``
methods can accept a mask argument, which must be a ``Filter``
indicating which assets to consider when computing.

###Masking Factors Let’s say we want our pipeline to output securities
with a high or low percent difference but we also only want to consider
securities with a dollar volume above $10,000,000. To do this, let’s
rearrange our ``make_pipeline`` function so that we first create the
``high_dollar_volume`` filter. We can then use this filter as a ``mask``
for moving average factors by passing ``high_dollar_volume`` as the
``mask`` argument to ``SimpleMovingAverage``.

.. code:: ipython2

    # Dollar volume factor
    dollar_volume = AverageDollarVolume(window_length=30)
    
    # High dollar volume filter
    high_dollar_volume = (dollar_volume > 10000000)
    
    # Average close price factors
    mean_close_10 = SimpleMovingAverage(inputs=[USEquityPricing.close], window_length=10, mask=high_dollar_volume)
    mean_close_30 = SimpleMovingAverage(inputs=[USEquityPricing.close], window_length=30, mask=high_dollar_volume)
    
    # Relative difference factor
    percent_difference = (mean_close_10 - mean_close_30) / mean_close_30

Applying the mask to ``SimpleMovingAverage`` restricts the average close
price factors to a computation over the ~2000 securities passing the
``high_dollar_volume`` filter, as opposed to ~8000 without a mask. When
we combine ``mean_close_10`` and ``mean_close_30`` to form
``percent_difference``, the computation is performed on the same ~2000
securities.

###Masking Filters Masks can be also be applied to methods that return
filters like ``top``, ``bottom``, and ``percentile_between``.

Masks are most useful when we want to apply a filter in the earlier
steps of a combined computation. For example, suppose we want to get the
50 securities with the highest open price that are also in the top 10%
of dollar volume. Suppose that we then want the 90th-100th percentile of
these securities by close price. We can do this with the following:

.. code:: ipython2

    # Dollar volume factor
    dollar_volume = AverageDollarVolume(window_length=30)
    
    # High dollar volume filter
    high_dollar_volume = dollar_volume.percentile_between(90,100)
    
    # Top open price filter (high dollar volume securities)
    top_open_price = USEquityPricing.open.latest.top(50, mask=high_dollar_volume)
    
    # Top percentile close price filter (high dollar volume, top 50 open price)
    high_close_price = USEquityPricing.close.latest.percentile_between(90, 100, mask=top_open_price)

Let’s put this into ``make_pipeline`` and output an empty pipeline
screened with our ``high_close_price`` filter.

.. code:: ipython2

    def make_pipeline():
    
        # Dollar volume factor
        dollar_volume = AverageDollarVolume(window_length=30)
    
        # High dollar volume filter
        high_dollar_volume = dollar_volume.percentile_between(90,100)
    
        # Top open securities filter (high dollar volume securities)
        top_open_price = USEquityPricing.open.latest.top(50, mask=high_dollar_volume)
    
        # Top percentile close price filter (high dollar volume, top 50 open price)
        high_close_price = USEquityPricing.close.latest.percentile_between(90, 100, mask=top_open_price)
    
        return Pipeline(
            screen=high_close_price
        )

Running this pipeline outputs 5 securities on May 5th, 2015.

.. code:: ipython2

    result = run_pipeline(make_pipeline(), '2015-05-05', '2015-05-05')
    print 'Number of securities that passed the filter: %d' % len(result)


.. parsed-literal::

    Number of securities that passed the filter: 5


Note that applying masks in layers as we did above can be thought of as
an “asset funnel”.

In the next lesson, we’ll look at classifiers.
