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


カスタムファクターはパイプライン内で独自の計算処理を定義することを可能にします。
`パートナーデータセット <https://www.quantopian.com/data>`__ や複数のデータ列を用いた計算を実行するうえでベストの方法になることが多いです。
カスタムファクターに関する完全なドキュメントは `ここ <https://www.quantopian.com/help#custom-factors>`__ を参照してください。

次のレッスンでは、これまでに学んだすべての内容を使い、アルゴリズムのためのパイプラインを作成します。
