.. code:: ipython2

    from quantopian.pipeline import Pipeline
    from quantopian.research import run_pipeline
    from quantopian.pipeline.data.builtin import USEquityPricing
    from quantopian.pipeline.factors import SimpleMovingAverage, AverageDollarVolume

カスタムファクター
--------------------

このレッスンでファクターについて最初に触れたとき、ビルトインファクターを見ていきました。
しかしながら多くの場合、実施したい計算処理はビルトインファクターとして準備されていません。
パイプラインAPIの最も強力な機能のひとつは、カスタムファクターを自身で作成できる機能です。
実施したい計算処理が組み込みに存在しなければ、自分でカスタムファクターを作れます。

概念的には、カスタムファクターはビルトインファクターと同じです。
カスタムファクターは、 ``inputs`` 、 ``window_length`` 、および ``mask`` をコンストラクタ引数として受け取ることができ、
``Factor`` オブジェクトを計算対象日ごとに返します。

ビルトインとして用意されていない複雑な計算例「標準偏差」を例にとってみます。
計算区間をずらしながら `標準偏差 <https://en.wikipedia.org/wiki/Standard_deviation>`__ を計算するには ``quantopian.pipeline.CustomFactor`` を
からサブクラスを作成してcomputeメソッドを実装します。このメソッドのシグネチャ（訳注：メソッドの入力と出力の定義）は：

::

   def compute(self, today, asset_ids, out, *inputs):
       ...

* ``*inputs`` はM行N列の `numpy arrays <http://docs.scipy.org/doc/numpy-1.10.1/reference/arrays.ndarray.html>`__ です。
  ここでMは ``window_length`` 、Nは銘柄数（ ``mask`` を使わない場合は通常8,000程度）です。 ``*inputs`` は計算区間のデータとなります。
  ファクターの入力リストに渡される ``BoundColumn`` ごとに１つのM行N列の配列が存在することに注意してください。各配列のデータ型は対応する
  ``BoundColumn`` の ``dtype`` となります
* ``out`` は長さNの空の配列です。 ``out`` は計算日ごとのカスタムファクターの計算結果となります。計算ジョブは出力をoutに書き出します。
* ``asset_ids`` は長さNの整数  `配列 <http://docs.scipy.org/doc/numpy-1.10.0/reference/generated/numpy.array.html>`__ となります。
  ``*inputs`` の列データに対応する銘柄コードが含まれます
* ``today`` は `pandas タイムスタンプ型 <http://pandas.pydata.org/pandas-docs/stable/timeseries.html#converting-to-timestamps>`__ となります。
  ``compute`` が呼び出されたときの日付を表します

当然 ``*inputs`` と ``out`` が最も一般的に使われます。

パイプラインに追加された ``CustomFactor`` のインスタンスはcomputeメソッドを毎日呼び出します。
たとえば過去5日間の終値による移動標準偏差を計算するカスタムファクターを定義してみます。
まずはじめに、 ``CustomFactor`` と ``numpy`` を import文に追加します。

.. code:: ipython2

    from quantopian.pipeline import CustomFactor
    import numpy

次に `numpy.nanstd <http://docs.scipy.org/doc/numpy-dev/reference/generated/numpy.nanstd.html>`__ を使い、計算期間に対して標準偏差を計算する
カスタムファクターを定義します。

.. code:: ipython2

    class StdDev(CustomFactor):
        def compute(self, today, asset_ids, out, values):
            # NaN を無視して、列ごとの標準偏差を計算する
            out[:] = numpy.nanstd(values, axis=0)

最後にカスタムファクターを ``make_pipeline()`` の中でインスタンス化します：

.. code:: ipython2

    def make_pipeline():
        std_dev = StdDev(inputs=[USEquityPricing.close], window_length=5)
    
        return Pipeline(
            columns={
                'std_dev': std_dev
            }
        )

パイプラインを実行すると、 以下のようなデータとともに ``StdDev.compute()`` が毎日呼び出されます：

* ``values``: 1つのM行N列の `numpy array <http://docs.scipy.org/doc/numpy-1.10.1/reference/arrays.ndarray.html>`__ です。
  ここでMは5(計算区間)で、Nは約8,000(計算対象日に存在する銘柄の数)です
* ``out`` : 長さNの空の配列です。この例では、 ``compute`` メソッドの実行で、5日間の終値による移動標準偏差を ``out`` に書き込みます

.. code:: ipython2

    result = run_pipeline(make_pipeline(), '2015-05-05', '2015-05-05')
    result.head()

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
      </tbody>
    </table>
    </div>


デフォルト入力
~~~~~~~~~~~~~~

カスタムファクターの作成時、デフォルトの ``input`` と ``window_length`` を指定できます。
たとえば計算期間に対して `numpy.nanmean <http://docs.scipy.org/doc/numpy-dev/reference/generated/numpy.nanmean.html>`__ 
を使って2つのデータ列の差の平均を計算する ``TenDayMeanDifference`` カスタムファクターを定義してみます。
デフォルト ``input`` として　``[USEquityPricing.close, USEquityPricing.open]``　window_lengthを、デフォルト ``window_length`` として
10をセットします：

.. code:: ipython2

    class TenDayMeanDifference(CustomFactor):
        # Default inputs.
        inputs = [USEquityPricing.close, USEquityPricing.open]
        window_length = 10
        def compute(self, today, asset_ids, out, close, open):
             # NaN を無視して、列ごとの差の平均を計算する
            out[:] = numpy.nanmean(close - open, axis=0)

この例では ``close`` と ``open`` はそれぞれ、10行8000列の二次元 `numpy arrays. <http://docs.scipy.org/doc/numpy-1.10.1/reference/arrays.ndarray.html>`__ です。
もし ``TenDayMeanDifference`` に対して何も引数を与えずに呼び出せば、ここでセットしたデフォルト入力が使われます。

.. code:: ipython2

    # 10日間の始値と終値の差の平均を計算する。
    close_open_diff = TenDayMeanDifference()

デフォルト入力はコンストラクタ呼び出しの際に引数を与えることにより上書きできます。

.. code:: ipython2

    # 10日間の高値と安値の差の平均を計算する。
    high_low_diff = TenDayMeanDifference(inputs=[USEquityPricing.high, USEquityPricing.low])

より複雑な例
~~~~~~~~~~~~~~~

`モメンタム <http://www.investopedia.com/terms/m/momentum.asp>`__ カスタムファクターを作成し、それをフィルタの作成に使ってみます。
フィルタをパイプラインの ``screen`` として用います。

まず始めに、直近の終値を ``n`` 日前の終値で割った ``Momentum`` ファクターを定義します。ここで ``n`` は ``window_length`` となります。

.. code:: ipython2

    class Momentum(CustomFactor):
        # デフォルト入力
        inputs = [USEquityPricing.close]
    
        # モメンタムの計算
        def compute(self, today, assets, out, close):
            out[:] = close[-1] / close[0]

ここで、 ``Momentum`` ファクターのインスタンスを2つ（10日間モメンタムと20日間モメンタム）作成します。
同時に 10日間、20日間ともに正のモメンタムを持つ銘柄に対して``True`` を返す ``positive_momentum`` フィルターを作成します。

.. code:: ipython2

    ten_day_momentum = Momentum(window_length=10)
    twenty_day_momentum = Momentum(window_length=20)
    
    positive_momentum = ((ten_day_momentum > 1) & (twenty_day_momentum > 1))

次に、2つのモメンタムファクターと ``positive_momentum`` フィルタを ``make_pipeline`` に追加します。
また、``positive_momentum`` をパイプラインの ``screen`` 引数に対して渡します。

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

このパイプライン出力は、2つのモメンタムがともに正である銘柄の、標準偏差、10日間モメンタム、20日間モメンタムを出力します。

.. code:: ipython2

    result = run_pipeline(make_pipeline(), '2015-05-05', '2015-05-05')
    result.head()

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
      </tbody>
    </table>
    </div>


カスタムファクターはパイプライン内で独自の計算処理を定義することを可能にします。
`パートナーデータセット <https://www.quantopian.com/data>`__ や複数のデータ列を用いた計算を実行するうえでベストの方法になることが多いです。
カスタムファクターに関する完全なドキュメントは `ここ <https://www.quantopian.com/help#custom-factors>`__ を参照してください。

次のレッスンでは、これまでに学んだすべての内容を使い、アルゴリズムのためのパイプラインを作成します。
