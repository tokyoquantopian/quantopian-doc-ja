.. code:: ipython2

    from quantopian.pipeline import Pipeline
    from quantopian.research import run_pipeline
    from quantopian.pipeline.data.builtin import USEquityPricing
    from quantopian.pipeline.factors import SimpleMovingAverage, AverageDollarVolume


フィルタの結合
------------------

ファクタと同じく、フィルタも結合できます。
フィルタの結合には ``&`` （and）や ``|`` （or）演算子を使います。
例えば直近の終値が20ドルよりも高く、かつ、平均売買代金が全銘柄の上位10パーセントという条件でスクリーニングを実施したいとしましょう。
まず売買代金フィルタを ``AverageDollarVolume`` ファクター と ``percentile_between`` を使って作成します：

.. code:: ipython2

    dollar_volume = AverageDollarVolume(window_length=30)
    high_dollar_volume = dollar_volume.percentile_between(90, 100)  

（備考） ``percentile_between`` は、 ``フィルタ`` を返す ``ファクター`` メソッドです。

次に ``latest_close`` ファクターを作成し、終値が20ドルよりも高い証券に絞り込むフィルタを定義します：

.. code:: ipython2

    latest_close = USEquityPricing.close.latest
    above_20 = latest_close > 20

そして ``high_dollar_volume`` フィルタと ``above_20`` フィルタを ``&`` 演算子を使って結合します：

.. code:: ipython2

    tradeable_filter = high_dollar_volume & above_20

このフィルタは ``high_dollar_volume`` と ``above_20`` の双方が ``True`` となる証券に対して ``True`` と判定します。
それら以外は ``False`` となります。 
類似の演算が ``|`` (or) 演算子を使って実行可能です。
このフィルタをパイプライン内でスクリーニングに利用したい場合、 ``screen`` 引数として ``tradeable_filter`` を指定するだけです。

.. code:: ipython2

    def make_pipeline():
    
        mean_close_10 = SimpleMovingAverage(inputs=[USEquityPricing.close], window_length=10)
        mean_close_30 = SimpleMovingAverage(inputs=[USEquityPricing.close], window_length=30)
    
        percent_difference = (mean_close_10 - mean_close_30) / mean_close_30
    
        dollar_volume = AverageDollarVolume(window_length=30)
        high_dollar_volume = dollar_volume.percentile_between(90, 100)
    
        latest_close = USEquityPricing.close.latest
        above_20 = latest_close > 20
    
        tradeable_filter = high_dollar_volume & above_20
    
        return Pipeline(
            columns={
                'percent_difference': percent_difference
            },
            screen=tradeable_filter
        )

実行すると、パイプラインは約700銘柄を出力します。

.. code:: ipython2

    result = run_pipeline(make_pipeline(), '2015-05-05', '2015-05-05')
    print 'Number of securities that passed the filter: %d' % len(result)
    result


.. todo::
  print文を python3用に書き換えるか検討


.. parsed-literal::

    Number of securities that passed the filter: 737



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
          <th>Equity(24 [AAPL])</th>
          <td>0.016905</td>
        </tr>
        <tr>
          <th>Equity(62 [ABT])</th>
          <td>0.014385</td>
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
          <th>Equity(168 [AET])</th>
          <td>0.005853</td>
        </tr>
        <tr>
          <th>Equity(185 [AFL])</th>
          <td>-0.002239</td>
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
          <th>Equity(270 [AKRX])</th>
          <td>-0.024963</td>
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
          <th>Equity(357 [TWX])</th>
          <td>0.000365</td>
        </tr>
        <tr>
          <th>Equity(368 [AMGN])</th>
          <td>0.008860</td>
        </tr>
        <tr>
          <th>Equity(438 [AON])</th>
          <td>0.002327</td>
        </tr>
        <tr>
          <th>Equity(448 [APA])</th>
          <td>0.035926</td>
        </tr>
        <tr>
          <th>Equity(455 [APC])</th>
          <td>0.049153</td>
        </tr>
        <tr>
          <th>Equity(460 [APD])</th>
          <td>-0.006999</td>
        </tr>
        <tr>
          <th>Equity(624 [ATW])</th>
          <td>0.014957</td>
        </tr>
        <tr>
          <th>Equity(630 [ADP])</th>
          <td>-0.002134</td>
        </tr>
        <tr>
          <th>Equity(679 [AXP])</th>
          <td>-0.011809</td>
        </tr>
        <tr>
          <th>Equity(693 [AZO])</th>
          <td>0.002395</td>
        </tr>
        <tr>
          <th>Equity(698 [BA])</th>
          <td>-0.016685</td>
        </tr>
        <tr>
          <th>Equity(734 [BAX])</th>
          <td>0.009414</td>
        </tr>
        <tr>
          <th>Equity(739 [BBBY])</th>
          <td>-0.027796</td>
        </tr>
        <tr>
          <th>...</th>
          <td>...</td>
        </tr>
        <tr>
          <th>Equity(45269 [EVHC])</th>
          <td>-0.004877</td>
        </tr>
        <tr>
          <th>Equity(45451 [FEYE])</th>
          <td>0.042108</td>
        </tr>
        <tr>
          <th>Equity(45558 [BURL])</th>
          <td>-0.053654</td>
        </tr>
        <tr>
          <th>Equity(45570 [JNUG])</th>
          <td>0.053977</td>
        </tr>
        <tr>
          <th>Equity(45618 [AR])</th>
          <td>0.091085</td>
        </tr>
        <tr>
          <th>Equity(45769 [WUBA])</th>
          <td>0.234141</td>
        </tr>
        <tr>
          <th>Equity(45804 [ASHR])</th>
          <td>0.082573</td>
        </tr>
        <tr>
          <th>Equity(45815 [TWTR])</th>
          <td>-0.077268</td>
        </tr>
        <tr>
          <th>Equity(45971 [AAL])</th>
          <td>0.008087</td>
        </tr>
        <tr>
          <th>Equity(45978 [ATHM])</th>
          <td>0.063568</td>
        </tr>
        <tr>
          <th>Equity(45993 [HLT])</th>
          <td>-0.000895</td>
        </tr>
        <tr>
          <th>Equity(46015 [ALLY])</th>
          <td>0.009605</td>
        </tr>
        <tr>
          <th>Equity(46308 [ASPX])</th>
          <td>0.054145</td>
        </tr>
        <tr>
          <th>Equity(46631 [GOOG])</th>
          <td>0.004730</td>
        </tr>
        <tr>
          <th>Equity(46693 [GRUB])</th>
          <td>-0.016904</td>
        </tr>
        <tr>
          <th>Equity(46979 [JD])</th>
          <td>0.058362</td>
        </tr>
        <tr>
          <th>Equity(47169 [KITE])</th>
          <td>-0.049366</td>
        </tr>
        <tr>
          <th>Equity(47208 [GPRO])</th>
          <td>0.061078</td>
        </tr>
        <tr>
          <th>Equity(47230 [NSAM])</th>
          <td>-0.037879</td>
        </tr>
        <tr>
          <th>Equity(47430 [MBLY])</th>
          <td>0.050288</td>
        </tr>
        <tr>
          <th>Equity(47740 [BABA])</th>
          <td>-0.008354</td>
        </tr>
        <tr>
          <th>Equity(47777 [CFG])</th>
          <td>0.025703</td>
        </tr>
        <tr>
          <th>Equity(47779 [CYBR])</th>
          <td>0.101844</td>
        </tr>
        <tr>
          <th>Equity(48065 [AXTA])</th>
          <td>0.034600</td>
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
          <th>Equity(48892 [IGT])</th>
          <td>0.005591</td>
        </tr>
        <tr>
          <th>Equity(48934 [ETSY])</th>
          <td>-0.030142</td>
        </tr>
        <tr>
          <th>Equity(48962 [CSAL])</th>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(48972 [EVA])</th>
          <td>0.000000</td>
        </tr>
      </tbody>
    </table>
    <p>737 rows × 1 columns</p>
    </div>


次のレッスンでは、ファクターとフィルタのマスキングについて見ていきます。
