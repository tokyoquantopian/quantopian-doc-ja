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

チュートリアルのコードは、 Quantopian の **Research** 環境で実行できます。 Research は `Jupyter <https://jupyter-notebook-beginner-guide.readthedocs.io/en/latest/what_is_jupyter.html>`__ ノートブック環境で提供されており、Pythonをインタラクティブに実行することができます。 Researchでは、Quantopianが独自に作成した Pythonのライブラリと、いくつかのサードパーティライブラリがインストールされています。くわしくは、 `documentation <https://www.quantopian.com/docs/user-guide/environments/research>`__ を参照してください。

`原作ページ <https://www.quantopian.com/tutorials/getting-started>`__ にある  `Get Notebook ` ボタンをおすことで、このチュートリアルのコードを試してみることができます。
Pythonを実行するには、コードを記入したセル上で **Shift+Enter** 同時に押下してください。コードセルは、背面が少し灰色のセルです。

Step 1 - 取引対象銘柄を定義する
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

さまざまな業種の株式に対してファクターを調査する時、最初のステップは「ユニバース（取引対象銘柄）」を設定することです。ここでいう「ユニバース」とは、後で分析計算を行う際に考慮したい株式の集合を意味します。
Quantopianでは、ユニバースの定義は `Pipeline
API <https://www.quantopian.com/docs/user-guide/tools/pipeline>`__ を使って行います。同APIを使って、ファクター計算も行います。

Pipeline APIは、いくつかの組み込みデータセットやユーザーがアップロードする `カスタムデータセット <https://www.quantopian.com/custom-datasets>`__ への統一されたインターフェースを提供しています。Pipelineを使用すると、組み込みデータやカスタムデータを使用した計算や式を簡単に定義することができます。たとえば、次のコード例は、2 つの組み込みデータセット、  `FactSet
Fundamentals <https://www.quantopian.com/docs/data-reference/factset_fundamentals>`__  と  `FactSet Equity
Metadata <https://www.quantopian.com/docs/data-reference/equity_metadata>`__ をインポートし、それらを使用してユニバースを定義しています。


.. code:: ipython3

    from quantopian.pipeline import Pipeline
    from quantopian.pipeline.data.factset import Fundamentals, EquityMetadata
    
    is_share = EquityMetadata.security_type.latest.eq('SHARE')
    is_primary = EquityMetadata.is_primary.latest
    primary_shares = (is_share & is_primary)
    market_cap = Fundamentals.mkt_val.latest
    
    universe = market_cap.top(1000, mask=primary_shares)

上記の例では、時価総額でランク付けされた主要銘柄の普通株式の上位1000銘柄をユニバースとして定義しています。
ユニバースは、Quantopianで利用可能なデータを使用して定義することができます。
また、セルフサーブデータツールを使用して、インデックス構成銘柄やその他カスタマイズしたユニバースなど、独自のデータをプラットフォームにアップロードすることもできます。カスタムデータセットのアップロード方法については、 `Self-Serve Data
documentation <https://www.quantopian.com/docs/user-guide/tools/self-serve>`__ を参照してください。今回は、上記に定義したユニバースを使っていきます。


Step 2 - ファクターを定義
~~~~~~~~~~~~~~~~~~~~~~~~~

ユニバースを定義したら、次はファクターを定義しテストしてみましょう。
Quantopianにおいてファクターとは、ユニバース内のアセットに対して一定の頻度で数値を生成する計算のことです。ステップ1と同様に、  ` Pipeline
API <https://www.quantopian.com/docs/user-guide/tools/pipeline>`__ を使用してファクターを定義します。Pipelineは、予め整備されたデータセットに、高速かつ統一された方法でアクセスするAPIを提供します。また、多数の  `ビルドインクラス <https://www.quantopian.com/docs/api-reference/pipeline-api-reference#id8>`__ や  `メソッド <https://www.quantopian.com/docs/api-reference/pipeline-api-reference#factor-methods>`__ が提供されており、これらを使用して素早くファクターを定義することができます。

たとえば、次のコード例は、移動平均算出日数が少ない移動平均(fase) と多い移動平均(slow) を使ってモメンタムを計算しています。


.. code:: ipython3

    from quantopian.pipeline import Pipeline
    from quantopian.pipeline.data import EquityPricing
    from quantopian.pipeline.factors import SimpleMovingAverage
    
    # 1ヶ月 (21 営業日) moving average factor.
    fast_ma = SimpleMovingAverage(inputs=[EquityPricing.close], window_length=21)
    
    # 6ヶ月 (126 営業日) moving average factor.
    slow_ma = SimpleMovingAverage(inputs=[EquityPricing.close], window_length=126)
    
    # fast_ma を slow_ma で割ってモメンタムを算出し、その z-scoreを算出
    momentum = fast_ma / slow_ma
    momentum_factor = momentum.zscore()

ユニバースとファクターを定義したので、市場と期間を選択し、ファクターをシミュレーションすることができます。Pipeline APIの特徴の1つは、  `株価調整 <https://www.quantopian.com/docs/data-reference/overview#corporate-action-adjustments>`__ 、`point-in-time
データ <https://www.quantopian.com/docs/data-reference/overview#point-in-time-data>`__ 、 `銘柄シンボルマッピング <https://www.quantopian.com/docs/data-reference/overview#asset-identifiers>`__ 、上場廃止、データアラインメントなどの一般的なデータエンジニアリングの問題を気にすることなく、ユニバースとファクターを定義できることです。Pipeline はこれらの作業をすべて裏で行ってくれるので、ファクターの構築やテストに時間を割くことができます。

以下のコードは、Pipeline インスタンスを作成してファクターをコラムとして追加し、私達のユニバースに合致するように株式をスクリーニングしています。このPipelineを2016年から2019年の米国株式に対して実行します。

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
    
    # 1ヶ月移動平均ファクター
    fast_ma = SimpleMovingAverage(inputs=[EquityPricing.close], window_length=21)
    
    # 6ヶ月移動平均ファクター
    slow_ma = SimpleMovingAverage(inputs=[EquityPricing.close], window_length=126)
    
    # fast_ma を slow_ma で割ってモメンタムを算出し、その z-scoreを算出
    momentum = fast_ma / slow_ma
    momentum_factor = momentum.zscore()
    
    
    # 作成したモメンタムファクターをつかって米国株のパイプラインを作成。ファクタを実行して株式をスクリーニング
    pipe = Pipeline(
        columns={
            'momentum_factor': momentum_factor,
        },
        screen=momentum_factor.percentile_between(50, 100, mask=universe),
        domain=US_EQUITIES,
    )
    
    # 2016〜2019年で実行し、実行結果を最初の5行だけ出力
    from quantopian.research import run_pipeline
    factor_data = run_pipeline(pipe, '2016-01-01', '2019-01-01')
    print("Result contains {} rows of output.".format(len(factor_data)))
    factor_data.head()

このコードを実行すると ファクター値を持った pandas のデータフレームが生成され、最初の5行が表示されます。株式取引日ごとに、その日ユニバースに入った全銘柄にたいしてファクター値が算出されデータフレームに入ります。これで、ユニバースの各株式銘柄のモメンタム値と、2016年から2019年までの各日のモメンタム値がわかりました。

.. attention:: ライセンスの制限により、Quantopianの一部のデータセットでは、直近の1～2年分のデータが外されている（ `holdouts <https://www.quantopian.com/docs/data-reference/overview#holdout-periods>`__ ）場合があります。各データセットには、最近のデータで外されているデータの長さが記録されています。例えば、FactSet Fundamentalsでは、直近1年分のデータがはずされています。 `Quantopian Enterprise <https://factset.quantopian.com>`__ ではデータは外されることはありません。

次の画像は、この結果のデータフレームを示しています。

.. image:: notebook_files/getting_started_screenshot1.png



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
