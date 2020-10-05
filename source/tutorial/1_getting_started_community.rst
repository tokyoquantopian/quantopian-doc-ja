Welcome
-------

`原作ページ <https://www.quantopian.com/tutorials/getting-started>`__

Quantopianへようこそ。このチュートリアルではまずQuantopianについて紹介します。
Quantopianで解決できる問題や、解決するために助けとなるツールの紹介を行います。
レッスンを終えれば、Quantopianを使うのに十分な理解を得ることが出来るでしょう。

チュートリアルでは、Quantopianを使いこなせるようになることを目的としていますが、Quantopianのエキスパートになることを目的としたものではありません。Quantopianの基本的な使い方に慣れている方は、Quantopianのツールをより深く理解するためのリソースがありますのでこちらをお勧めします。

* `Documentation <https://www.quantopian.com/docs/index>`__ 
* `Pipeline Tutorial <https://www.quantopian.com/tutorials/pipeline>`__  (`翻訳 <https://quantopian-doc.readthedocs.io/ja/latest/tutorial/index.html#pipeline>`__  )
* `Alphalens Tutorial <https://www.quantopian.com/tutorials/alphalens>`__ (`未翻訳 <https://quantopian-doc.readthedocs.io/ja/latest/tutorial/index.html>`__ ) 

このチュートリアルを始めるためには、基本的な `Python <https://docs.python.org/3.5/>`__ のプログラミングスキルが必要です。

.. note:: 

   `原作ページ <https://www.quantopian.com/tutorials/getting-started>`__ のチュートリアルはQuantopian Research 環境がすく利用できる形で記載されています。Researchはクラウドホスト型のJupyter notebook です。Pythonコードをインタラクティブに実行することができます。
   
   Quantopianにログイン後、`原作ページ <https://www.quantopian.com/tutorials/getting-started>`__ のノートブックをクローンすることでチュートリアルのコードを実行することができます。クローンするには原作ページの `Get Notebook` ボタンを押して下さい。

   Research の詳細については、 `documentation <https://www.quantopian.com/docs/user-guide/environments/research>`__ を参照してください。



Quantopianとは？
-------------------

Quantopianは、Pythonを使って世界中の先進国および新興国の株式市場における定量的な金融要因を調査できるクラウドベースのソフトウェアプラットフォームです。Quantopiahでは、あらゆる種類の `金融データ <https://www.quantopian.com/docs/data-reference/overview>`__ に加え、高速で統一されたAPIを提供します。それによってあなたは自身の戦略アイデアのテストを簡単にかつ何度も繰り返し実行できます。また、Quantopianには、 `あなた独自の金融データセットをアップロード <https://www.quantopian.com/docs/user-guide/tools/self-serve>`__ したり、ファクターの有効性を分析したり、グローバルなクオンツコミュニティと結果を共有したりするためのツールも用意されています。

さまざまな業種の株式に対してファクターを調査する方法は、Quantopianでの手順はいかのようなステップになります：

1. 取引対象銘柄を定義する
2. 取引対象銘柄に対して、ファクターを定義する
3. ファクターが機能するかどうかテストする
4. 他のクオンツ（訳注：ここでは forum に参加している他のユーザーのことを言う）に結果をシェアしあなたの戦略アプローチについて議論する

ステップ１と２は、  `Pipeline
API <https://www.quantopian.com/docs/user-guide/tools/pipeline>`__ で実行し、
ステップ３は  `Alphalens <https://www.quantopian.com/docs/user-guide/tools/alphalens>`__ を使います。ステップ４は  `Quantopian community
forum <https://www.quantopian.com/posts>`__ で行います。

さてこの後のチュートリアルでは、Quantopianでのリサーチワークフローの簡単な説明を行っていきます。


Research 環境
~~~~~~~~~~~~~~~~~~~~

The code in this tutorial can be run in Quantopian’s **Research**
environment (this notebook is currently running in Research). Research
is a hosted
`Jupyter <https://jupyter-notebook-beginner-guide.readthedocs.io/en/latest/what_is_jupyter.html>`__
notebook environment that allows you to interactively run Python code.
Research comes with a mix of proprietary and open-source Python
libraries pre-installed. To learn more about Research, see the
`documentation <https://www.quantopian.com/docs/user-guide/environments/research>`__.

Press **Shift+Enter** to run each cell of code (grey boxes).


チュートリアル中のコードは、 Quantopian の 


Step 1 - Define a universe of assets.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The first step to researching a cross-sectional equity factor is to
select a “universe” of equities over which our factor will be defined.
In this context, a universe represents the set of equities we want to
consider when performing computations later. On Quantopian, defining a
universe is done using the `the Pipeline
API <https://www.quantopian.com/docs/user-guide/tools/pipeline>`__.
Later on, we will use the same API to compute factors over the equities
in this universe.

The Pipeline API provides a uniform interface to several `built-in
datasets <https://www.quantopian.com/docs/data-reference/overview>`__,
as well as any `custom
datasets <https://www.quantopian.com/custom-datasets>`__ that we upload
to our account. Pipeline makes it easy to define computations or
expressions using built-in and custom data. For example, the following
code snippet imports two built-in datasets, `FactSet
Fundamentals <https://www.quantopian.com/docs/data-reference/factset_fundamentals>`__
and `FactSet Equity
Metadata <https://www.quantopian.com/docs/data-reference/equity_metadata>`__,
and uses them to define an equity universe.

.. code:: ipython3

    from quantopian.pipeline import Pipeline
    from quantopian.pipeline.data.factset import Fundamentals, EquityMetadata
    
    is_share = EquityMetadata.security_type.latest.eq('SHARE')
    is_primary = EquityMetadata.is_primary.latest
    primary_shares = (is_share & is_primary)
    market_cap = Fundamentals.mkt_val.latest
    
    universe = market_cap.top(1000, mask=primary_shares)

The above example defines a universe to be the top 1000 primary issue
common stocks ranked by market cap. Universes can be defined using any
of the data available on Quantopian. Additionally, you can upload your
own data, such as index constituents or another custom universe to the
platform using the Self-Serve Data tool. To learn more about uploading a
custom dataset, see the `Self-Serve Data
documentation <https://www.quantopian.com/docs/user-guide/tools/self-serve>`__.
For now, we will stick with the universe definition above.

Step 2 - Define a factor.
~~~~~~~~~~~~~~~~~~~~~~~~~

After defining a universe, the next step is to define a factor for
testing. On Quantopian, a factor is a computation that produces
numerical values at a regular frequency for all assets in a universe.
Similar to step 1, we will use the `the Pipeline
API <https://www.quantopian.com/docs/user-guide/tools/pipeline>`__ to
define factors. In addition to providing a fast, uniform API on top of
pre-integrated and custom datasets, Pipeline also provides a set of
built-in
`classes <https://www.quantopian.com/docs/api-reference/pipeline-api-reference#built-in-factors>`__
and
`methods <https://www.quantopian.com/docs/api-reference/pipeline-api-reference#methods-that-create-factors>`__
that can be used to quickly define factors. For example, the following
code snippet defines a momentum factor using fast and slow moving
average computations.

.. code:: ipython3

    from quantopian.pipeline import Pipeline
    from quantopian.pipeline.data import EquityPricing
    from quantopian.pipeline.factors import SimpleMovingAverage
    
    # 1-month (21 trading day) moving average factor.
    fast_ma = SimpleMovingAverage(inputs=[EquityPricing.close], window_length=21)
    
    # 6-month (126 trading day) moving average factor.
    slow_ma = SimpleMovingAverage(inputs=[EquityPricing.close], window_length=126)
    
    # Divide fast_ma by slow_ma to get momentum factor and z-score.
    momentum = fast_ma / slow_ma
    momentum_factor = momentum.zscore()

Now that we defined a universe and a factor, we can choose a market and
time period and simulate the factor. One of the defining features of the
Pipeline API is that it allows us to define universes and factors using
high level terms, without having to worry about common data engineering
problems like
`adjustments <https://www.quantopian.com/docs/data-reference/overview#corporate-action-adjustments>`__,
`point-in-time
data <https://www.quantopian.com/docs/data-reference/overview#point-in-time-data>`__,
`symbol
mapping <https://www.quantopian.com/docs/data-reference/overview#asset-identifiers>`__,
delistings, and data alignment. Pipeline does all of that work behind
the scenes and allows us to focus our time on building and testing
factors.

The below code creates a Pipeline instance that adds our factor as a
column and screens down to equities in our universe. The Pipline is then
run over the US equities market from 2016 to 2019.

.. code:: ipython3

    from quantopian.pipeline import Pipeline
    from quantopian.pipeline.data import EquityPricing
    from quantopian.pipeline.data.factset import Fundamentals, EquityMetadata
    from quantopian.pipeline.domain import US_EQUITIES, ES_EQUITIES
    from quantopian.pipeline.factors import SimpleMovingAverage
    
    is_share = EquityMetadata.security_type.latest.eq('SHARE')
    is_primary = EquityMetadata.is_primary.latest
    primary_shares = (is_share & is_primary)
    market_cap = Fundamentals.mkt_val.latest
    
    universe = market_cap.top(1000, mask=primary_shares)
    
    # 1-month moving average factor.
    fast_ma = SimpleMovingAverage(inputs=[EquityPricing.close], window_length=21)
    
    # 6-month moving average factor.
    slow_ma = SimpleMovingAverage(inputs=[EquityPricing.close], window_length=126)
    
    # Divide fast_ma by slow_ma to get momentum factor and z-score.
    momentum = fast_ma / slow_ma
    momentum_factor = momentum.zscore()
    
    
    # Create a US equities pipeline with our momentum factor, screening down to our universe.
    pipe = Pipeline(
        columns={
            'momentum_factor': momentum_factor,
        },
        screen=momentum_factor.percentile_between(50, 100, mask=universe),
        domain=US_EQUITIES,
    )
    
    # Run the pipeline from 2016 to 2019 and display the first few rows of output.
    from quantopian.research import run_pipeline
    factor_data = run_pipeline(pipe, '2016-01-01', '2019-01-01')
    print("Result contains {} rows of output.".format(len(factor_data)))
    factor_data.head()



.. parsed-literal::

    



.. raw:: html

    <b>Pipeline Execution Time:</b> 10.35 Seconds


.. parsed-literal::

    Result contains 376888 rows of output.




.. raw:: html

    <div>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th></th>
          <th>momentum_factor</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th rowspan="5" valign="top">2016-01-04 00:00:00+00:00</th>
          <th>Equity(67 [ADSK])</th>
          <td>1.211037</td>
        </tr>
        <tr>
          <th>Equity(76 [TAP])</th>
          <td>1.252325</td>
        </tr>
        <tr>
          <th>Equity(114 [ADBE])</th>
          <td>0.816440</td>
        </tr>
        <tr>
          <th>Equity(161 [AEP])</th>
          <td>0.407423</td>
        </tr>
        <tr>
          <th>Equity(185 [AFL])</th>
          <td>0.288431</td>
        </tr>
      </tbody>
    </table>
    </div>



Running the above code produces a pandas dataframe, stored in the
variable ``factor_data``, and display the first few rows of its output.
The dataframe contains a momentum factor value per equity per day, for
each equity in our universe, based on the definition we provided. Now
that we have a momentum value for each equity in our universe, and each
day between 2016 and 2019, we can test to see if our factor is
predictive.

**Note:** Due to licensing restrictions, some datasets on Quantopian
have
`holdouts <https://www.quantopian.com/docs/data-reference/overview#holdout-periods>`__
on the most recent year or two of data. Each dataset is documented with
the length of holdout on recent data. For instance, FactSet Fundamentals
has the most recent year of data held out. Holdouts to not apply to
`Quantopian Enterprise <https://factset.quantopian.com>`__.

Step 3 - Test the factor.
~~~~~~~~~~~~~~~~~~~~~~~~~

The next step is to test the predictiveness of the factor we defined in
step 2. In order to determine if our factor is predictive, load returns
data from Pipeline, and then feed the factor and returns data into
`Alphalens <https://www.quantopian.com/docs/user-guide/tools/alphalens>`__.
The following code cell loads the 1-day trailing returns for equities in
our universe, shifts them back, and formats the data for use in
Alphalens.

.. code:: ipython3

    from quantopian.pipeline.factors import Returns
    
    # Create and run a Pipeline to get day-over-day returns.
    returns_pipe = Pipeline(
        columns={
            '1D': Returns(window_length=2),
        },
        domain=US_EQUITIES,
    )
    returns_data = run_pipeline(returns_pipe, '2016-01-01', '2019-02-01')
    
    # Import alphalens and pandas.
    import alphalens as al
    import pandas as pd
    
    # Shift the returns so that we can compare our factor data to forward returns.
    shifted_returns = al.utils.backshift_returns_series(returns_data['1D'], 2)
    
    # Merge the factor and returns data.
    al_returns = pd.DataFrame(
        data=shifted_returns, 
        index=factor_data.index,
        columns=['1D'],
    )
    al_returns.index.levels[0].name = "date"
    al_returns.index.levels[1].name = "asset"
    
    # Format the factor and returns data so that we can run it through Alphalens.
    al_data = al.utils.get_clean_factor(
        factor_data['momentum_factor'],
        al_returns,
        quantiles=5,
        bins=None,
    )



.. parsed-literal::

    



.. raw:: html

    <b>Pipeline Execution Time:</b> 1.96 Seconds


.. parsed-literal::

    Dropped 0.1% entries from factor data: 0.1% in forward returns computation and 0.0% in binning phase (set max_loss=0 to see potentially suppressed Exceptions).
    max_loss is 35.0%, not exceeded: OK!


Then, we can create a factor tearsheet to analyze our momentum factor.

.. code:: ipython3

    from alphalens.tears import create_full_tear_sheet
    
    create_full_tear_sheet(al_data)


.. parsed-literal::

    Quantiles Statistics



.. raw:: html

    <div>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>min</th>
          <th>max</th>
          <th>mean</th>
          <th>std</th>
          <th>count</th>
          <th>count %</th>
        </tr>
        <tr>
          <th>factor_quantile</th>
          <th></th>
          <th></th>
          <th></th>
          <th></th>
          <th></th>
          <th></th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>1</th>
          <td>-0.074955</td>
          <td>0.421041</td>
          <td>0.210313</td>
          <td>0.087290</td>
          <td>75500</td>
          <td>20.047423</td>
        </tr>
        <tr>
          <th>2</th>
          <td>0.036037</td>
          <td>0.549411</td>
          <td>0.344777</td>
          <td>0.088091</td>
          <td>75320</td>
          <td>19.999628</td>
        </tr>
        <tr>
          <th>3</th>
          <td>0.176786</td>
          <td>0.749339</td>
          <td>0.492666</td>
          <td>0.094418</td>
          <td>74974</td>
          <td>19.907755</td>
        </tr>
        <tr>
          <th>4</th>
          <td>0.334028</td>
          <td>1.049384</td>
          <td>0.693494</td>
          <td>0.116546</td>
          <td>75320</td>
          <td>19.999628</td>
        </tr>
        <tr>
          <th>5</th>
          <td>0.550049</td>
          <td>8.979527</td>
          <td>1.236411</td>
          <td>0.522688</td>
          <td>75493</td>
          <td>20.045565</td>
        </tr>
      </tbody>
    </table>
    </div>


.. parsed-literal::

    Returns Analysis



.. raw:: html

    <div>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>1D</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>Ann. alpha</th>
          <td>-0.010</td>
        </tr>
        <tr>
          <th>beta</th>
          <td>0.113</td>
        </tr>
        <tr>
          <th>Mean Period Wise Return Top Quantile (bps)</th>
          <td>0.194</td>
        </tr>
        <tr>
          <th>Mean Period Wise Return Bottom Quantile (bps)</th>
          <td>-0.432</td>
        </tr>
        <tr>
          <th>Mean Period Wise Spread (bps)</th>
          <td>0.626</td>
        </tr>
      </tbody>
    </table>
    </div>


.. parsed-literal::

    /venvs/py35/lib/python3.5/site-packages/alphalens/tears.py:275: UserWarning: 'freq' not set in factor_data index: assuming business day
      UserWarning,



.. parsed-literal::

    <matplotlib.figure.Figure at 0x7fca4e78b358>



.. image:: notebook_files/notebook_9_6.png


.. parsed-literal::

    Information Analysis



.. raw:: html

    <div>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>1D</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>IC Mean</th>
          <td>0.005</td>
        </tr>
        <tr>
          <th>IC Std.</th>
          <td>0.135</td>
        </tr>
        <tr>
          <th>Risk-Adjusted IC</th>
          <td>0.038</td>
        </tr>
        <tr>
          <th>t-stat(IC)</th>
          <td>1.034</td>
        </tr>
        <tr>
          <th>p-value(IC)</th>
          <td>0.301</td>
        </tr>
        <tr>
          <th>IC Skew</th>
          <td>-0.288</td>
        </tr>
        <tr>
          <th>IC Kurtosis</th>
          <td>0.007</td>
        </tr>
      </tbody>
    </table>
    </div>


.. parsed-literal::

    /venvs/py35/lib/python3.5/site-packages/statsmodels/nonparametric/kdetools.py:20: VisibleDeprecationWarning: using a non-integer number instead of an integer will result in an error in the future
      y = X[:m/2+1] + np.r_[0,X[m/2+1:],0]*1j



.. image:: notebook_files/notebook_9_10.png


.. parsed-literal::

    /venvs/py35/lib/python3.5/site-packages/alphalens/utils.py:912: UserWarning: Skipping return periods that aren't exact multiples of days.
      + " of days."


.. parsed-literal::

    Turnover Analysis



.. raw:: html

    <div>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>1D</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>Quantile 1 Mean Turnover</th>
          <td>0.117</td>
        </tr>
        <tr>
          <th>Quantile 2 Mean Turnover</th>
          <td>0.111</td>
        </tr>
        <tr>
          <th>Quantile 3 Mean Turnover</th>
          <td>0.096</td>
        </tr>
        <tr>
          <th>Quantile 4 Mean Turnover</th>
          <td>0.070</td>
        </tr>
        <tr>
          <th>Quantile 5 Mean Turnover</th>
          <td>0.030</td>
        </tr>
      </tbody>
    </table>
    </div>



.. raw:: html

    <div>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>1D</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>Mean Factor Rank Autocorrelation</th>
          <td>0.996</td>
        </tr>
      </tbody>
    </table>
    </div>



.. image:: notebook_files/notebook_9_15.png


The Alphalens tearsheet offers insight into the predictive ability of a
factor.

To learn more about Alphalens, check out the
`documentation <https://www.quantopian.com/docs/user-guide/tools/alphalens>`__.

Step 4 - Discuss with the Quantopian Community
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When we have a factor that we like, we can share the result in the
`Quantopian community forum <https://www.quantopian.com/posts>`__ and
solicit feedback from community members. The ideas you come up with on
Quantopian belong to you, but sometimes sharing a result can spark a
discussion and create an opportunity to learn from others. In the
community forum, Research notebooks can be attached to posts. If you
want to share the result of your work **and** the code, you can share
your notebook as is. If you want to keep the code to yourself, you can
create a copy of your notebook, run your factor through Alphalens,
delete the code cells that have your Pipeline code, and just share the
Alphalens result in a community post.

If you want to share your work or your result in the community, make
sure to provide an explanation of some sort and ask questions to make it
more likely that others will respond!

Recap & Next Steps
~~~~~~~~~~~~~~~~~~

In this tutorial, we introduced Quantopian and walked through a factor
research workflow using Pipeline and Alphalens. Quantopian has a rich
set of `documentation <https://www.quantopian.com/docs/index>`__ which
you can use to learn more about the platform. We recommend starting with
the `User Guide <https://www.quantopian.com/docs/user-guide/overview>`__
section of the documentation if you would like to grow your
understanding of Quantopian or the `Data
Reference <https://www.quantopian.com/docs/data-reference/overview>`__
if you want to learn more about the data that’s available to you out of
the box.

If you would like to learn more about `Quantopian’s enterprise
offering <https://factset.quantopian.com/home>`__, please contact us at
enterprise@quantopian.com.
