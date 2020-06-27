ファクター
-------------------------

ファクターは、ある資産とある一時点から数値型を返す関数です。

.. math::

   F(asset, timestamp) \rightarrow float

パイプラインでは `ファクター <https://www.quantopian.com/help#quantopian_pipeline_factors_Factor>`__ は最も一般的な用語であり、数値として返される計算結果を表します。
ファクターには入力値として列データと計算区間が必要です。

パイプラインにおける最も単純なファクターは、`ビルトイン・ファクター <https://www.quantopian.com/help#built-in-factors>`__ と呼ばれるものです。
ビルトイン・ファクターは、一般的によく使われる計算を実行するために予め用意されています。
最初の例として、資産ごとに10日間の区間を動かしながら終値の平均値段を計算するファクターを作成しましょう。
指定した計算区間（10日間）を対象に入力データ（終値）の平均値を計算する ``SipleMovingAverate`` というビルトイン・ファクターを使います。
これを使うためには ``SimpleMovingAverage`` ビルトイン・ファクターと、 `USEquityPricing dataset <https://www.quantopian.com/help#importing-datasets>`__ をインポートします。

.. code:: ipython2

    from quantopian.pipeline import Pipeline
    from quantopian.research import run_pipeline
    
    # 前回のレッスンに加えて、USEquityPricing datasetをインポート
    from quantopian.pipeline.data.builtin import USEquityPricing
    
    # 前回のレッスンに加えて、SimpleMovingAverageビルトイン・ファクターをインポート
    from quantopian.pipeline.factors import SimpleMovingAverage

ファクターの作成
-------------------------

前のレッスンで作成した ``make_pipeline`` 関数に戻って、 ``SimpleMovingAverage`` ファクターをインスタンス化します。
``SimpleMovingAverage`` の作成には、``SimpleMovingAverage`` にinput（ ``BoundColumn`` 型のリスト）と、
window_length（移動平均の計算が受け取るデータ日数を表す整数値）という2つの引数を渡してコンストラクタを呼び出します。
（ ``BoundColumn`` については後ほど詳細に触れます。今の段階では、 ``BoundColumn`` がファクターを作成する上で必要な必要なものであるということだけ知っていれば十分です。）

以下の一行で、証券の10日間移動平均を計算する ``ファクター`` が出来上がります。

.. code:: ipython2

    mean_close_10 = SimpleMovingAverage(inputs=[USEquityPricing.close], window_length=10)

ここで重要なのは、ファクターの作成段階では、実際には計算を実行していないという点です。ファクターの作成は、関数を定義するのに似ています。
計算を実行するためには、ファクターをパイプラインに追加して、実行する必要があります。

ファクターをパイプラインに追加する 
-----------------------------------

元となる空のパイプラインを、移動平均を計算するファクターにアップデートさせましょう。
まず始めに、ファクターのインスタンス化を ``make_pipeline`` 内に移動させます。
次に、``columns`` （列名に対してファクター、フィルタ、あるいはクラシファイアを紐づける辞書型）引数を通じて、パイプラインにファクターの計算を指示します。
アップデートされた ``make_pipline`` 関数は、このような感じになるはずです。

.. code:: ipython2

    def make_pipeline():
        
        mean_close_10 = SimpleMovingAverage(inputs=[USEquityPricing.close], window_length=10)
        
        return Pipeline(
            columns={
                '10_day_mean_close': mean_close_10
            }
        )

中身がどのようになっているかを確認するため、パイプラインを実行して結果を表示させます。

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


これでパイプラインの出力に、8000超の全銘柄（画面上は途中まで）に対して計算された10日間終値移動平均の列が追加されました。
各行は、該当する証券と該当する日付における計算結果に対応しています。
この ``DataFrame`` は、 `マルチインデックス <http://pandas.pydata.org/pandas-docs/version/0.16.2/advanced.html>`__ 
（第1レベルは計算を行った日付を表す日時、第2レベルは証券に対応する `Equity <http://localhost:3000/help#api-sidinfo>`__ オブジェクト）
を持っています。
例えば1行目(``2015-05-05 00:00:00+00:00``, ``Equity(2 [AA])``)には、2015年5月5日のAA
（訳者注：AAはアルコア社（アルミニウム、アルミニウム製品およびアルミナの世界的なメーカー）を表す証券コード）
の ``mean_close_10`` ファクターの計算結果が格納されます。

もし1日よりも長い期間パイプラインを実行すれば、その結果はこのようになります。

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


備考：``Pipeline.add`` メソッドを用いることでも同様に ``Pipeline`` インスタンスに対してファクターを追加できます。 
``add`` を使う場合はこのような感じになります：>>> my_pipe = Pipeline() >>> f1 = SomeFactor(…) >>> my_pipe.add(f1, ‘f1’)

Latest
-------------------------

最もよく使われるビルトイン ``Factor`` は、 ``Latest`` です。 
``Latest`` ファクターは、与えられたデータ列中で最も直近の値を取得します。
このファクターは非常によく使われるので、他のファクターとは異なる方法でインスタンス化されます。
データ列から直近の値を取得するには、 ``.latest``  アトリビュートから取得するのが最良の方法です。
例として ``make_pipeline`` をアップデートして直近終値を取得するファクターを作成してパイプラインに追加してみましょう。

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

ここで再びパイプラインを作成し実行すると、出力されたdataframeには2つの列ができます。
一方は10日間終値移動平均の列で、もう一方は直近の終値の列になっています。

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

``.latest`` は ``ファクター`` 以外のものを返すことがあります（訳者注：latest_close_priceに数値ではないNaNを含む）。
これ以外の起こり得る返り値の型については後ほどみていきます。

デフォルト入力
------------------

いくつかのファクターは、変更すべきでないデフォルト入力があります。たとえば `VWAP ビルトイン・ファクター <https://www.quantopian.com/help#built-in-factors>`__
は常に ``USEquityPricing.close`` と ``USEquityPricing.volume`` から計算されます。ファクターが常に同じ `BoundColumns` から計算される場合、 ``input`` 引数を明示せずにコンストラクタを呼び出せます。

.. code:: ipython2

    from quantopian.pipeline.factors import VWAP
    vwap = VWAP(window_length=10)

次のレッスンでは、ファクターの結合を見ていきます。
