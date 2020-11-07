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
    result.head()

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
      </tbody>
    </table>
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
    print('Number of securities that passed the filter: %d' % len(result))

.. parsed-literal::

    Number of securities that passed the filter: 2110


フィルタの反転
---------------------

``~`` 演算子はフィルタの反転に使われ、 ``True`` 値を ``False`` 値に置き換えます（逆もしかり）。
例えば、売買代金の少ない証券にフィルタをかける場合は以下のように書きます：

.. code:: ipython2

    low_dollar_volume = ~high_dollar_volume

この場合、過去30日間の平均売買代金が10,000,000ドル以下の銘柄に対して ``True`` が返ります。

次のレッスンでは、フィルタの結合について見ていきます。
