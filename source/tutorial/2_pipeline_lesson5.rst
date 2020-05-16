.. code:: ipython2

    from quantopian.pipeline import Pipeline
    from quantopian.research import run_pipeline
    from quantopian.pipeline.data.builtin import USEquityPricing
    from quantopian.pipeline.factors import SimpleMovingAverage

フィルタ
------------

フィルタとは、とある資産のある一時点からブール型の値を出力する関数です：

.. math::

   F(asset, timestamp) \rightarrow boolean

`フィルタ <https://www.quantopian.com/help#quantopian_pipeline_filters_Filter>`__ は
計算過程やパイプラインの最終出力に含まれる証券の絞り込みに使われます。
``フィルタ`` 作成には2つの一般的な方法があります。比較演算子と ``ファクター`` / ``クラシファイア`` メソッドを使う方法です。 

比較演算子
------------

``ファクター`` と ``クラシファイア`` に用いられる比較演算子から ``フィルタ`` が作られます。 
ここまで ``クラシファイア`` について触れていませんが、とにかく ``ファクター`` を使った例で進めていきます。
以下の例では、直近の終値が20ドルを超える場合に ``True`` を返すフィルタを作成します。 

.. code:: ipython2

    last_close_price = USEquityPricing.close.latest
    close_price_filter = last_close_price > 20

またこの例では、10日間平均が30日間平均を下回るる場合に ``True`` を返すフィルタを作成しています。

.. code:: ipython2

    mean_close_10 = SimpleMovingAverage(inputs=[USEquityPricing.close], window_length=10)
    mean_close_30 = SimpleMovingAverage(inputs=[USEquityPricing.close], window_length=30)
    mean_crossover_filter = mean_close_10 < mean_close_30

それぞれの証券は日付ごとに ``True`` または ``False`` の値を持っていることに留意してください。

``ファクター`` / ``クラシファイア`` メソッド
--------------------------------------------

``ファクター`` と ``クラシファイア`` クラスが ``フィルタ`` を作成する方法は多様です。
相変わらず ``クラシファイア`` について触れていませんが ``ファクター`` を使った例で進めていきます（このあと ``クラシファイア`` メソッドを見ていきます）。
``Factor.top(n)`` メソッドは、与えられた ``ファクター`` における上位 ``n`` 件の証券に ``True`` を返す ``フィルタ`` を作成します。
以下の例はすべての利用可能な証券のうち、日付ごとに終値が上位200位以内に含まれる銘柄に対し ``True`` を返すフィルタを作成します。

.. code:: ipython2

    last_close_price = USEquityPricing.close.latest
    top_close_price_filter = last_close_price.top(200)

``フィルタ`` を返す ``ファクター`` メソッドの一覧は、`ここ <https://www.quantopian.com/help#quantopian_pipeline_factors_Factor>`__ 
を参照してください。

``フィルタ`` を返す ``クラシファイア`` メソッドの一覧は、`ここ <https://www.quantopian.com/help#quantopian_pipeline_classifiers_Classifier>`__ 
を参照してください。


売買代金（Dollar Volume）フィルタ
---------------------------------

最初の例として、証券の30日間の平均売買代金が10,000,000ドルより大きい場合に ``True`` を返すフィルタを作成します。
まず、 30日間の平均売買代金を計算する ``AverageDollarVolume`` ファクターを作成します。
import文を使って組込みの ``AverageDollarVolume`` ファクターを利用可能にします。

.. code:: ipython2

    from quantopian.pipeline.factors import AverageDollarVolume

次に、AverageDollarVolumeファクターをインスタンス化します。

.. code:: ipython2

    dollar_volume = AverageDollarVolume(window_length=30)

``AverageDollarVolume`` はデフォルトで ``USEquityPricing.close`` と ``USEquityPricing.volume`` を利用しているため ``inputs`` 引数を指定する必要はありません。
さてこれで売買代金ファクターが用意できたので、booleanを使ったフィルタを作成できます。
以下の行で、``dollar_volume`` が10,000,000よりも大きい証券に対して ``True`` を返すフィルタを作成します：

.. code:: ipython2

    high_dollar_volume = (dollar_volume > 10000000)

フィルタの中がどのようになっているか確認するため、前のレッスンで作成したパイプラインにフィルタを追加します。

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

パイプラインを作成・実行すると、各証券に対してフィルタの結果を表すboolean値が入った ``high_dollar_volume`` 列が作られます。

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


スクリーニング
------------------

通常、パイプラインはQuantopianのデータベースに存在するすべての資産を対象に、日付ごとに計算結果を出力します。
しかしながら、特定の基準を満たした証券の部分集合だけが必要なケースが頻繁に起こります（例えば、日々の取引が活発で注文が即座に成立するような銘柄のみが必要となることがあります）。
パイプラインの中で ``screen`` キーワードを使うと、パイプラインの実行でフィルタが ``False`` を返した銘柄をふるい落とすことができます。

出力結果を30日間の平均売買代金が10,000,000ドルよりも大きい証券だけに絞り込むには、``screen`` の引数として ``high_dollar_volume`` をあてはめるだけです。
``make_pipeline`` はこのような感じになります：

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

実行すると、パイプラインの出力にはそれぞれの日付において ``high_dollar_volume`` フィルタを通過した証券のみが含まれています。
例えばこのパイプラインを2015年5月5日に対して実行した出力結果は、約2,100銘柄程度となります。

（翻訳者注：以下のソースコードはprint文がpython2の文法であるため、python3では通常エラーとなる）

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


フィルタの反転
---------------------

``~`` 演算子はフィルタの反転に使われ、 ``True`` 値を ``False`` 値に置き換えます（逆もしかり）。
例えば、売買代金の少ない証券にフィルタをかける場合は以下のように書きます：

.. code:: ipython2

    low_dollar_volume = ~high_dollar_volume

この場合、過去30日間の平均売買代金が10,000,000ドル以下の銘柄に対して ``True`` が返ります。

次のレッスンでは、フィルタの結合について見ていきます。
