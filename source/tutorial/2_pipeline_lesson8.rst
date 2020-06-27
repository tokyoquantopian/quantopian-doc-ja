.. code:: ipython2

    from quantopian.pipeline import Pipeline
    from quantopian.research import run_pipeline
    from quantopian.pipeline.factors import SimpleMovingAverage, AverageDollarVolume

クラシファイア
-----------------------------

クラシファイアとは、とある資産のある一時点から ``文字列`` や ``整数`` ラベルといった `分類型の値を出力 <https://en.wikipedia.org/wiki/Categorical_variable>`__ する関数のことです。

.. math::

   F(asset, timestamp) \rightarrow category

たとえばクラシファイアは当該証券の取引所IDを表す文字列を出力します。
このクラシファイアを作成するには、 ``Fundamentals.exchange_id`` をインポートして、インスタンスの `latest <https://www.quantopian.com/tutorials/pipeline#lesson3>`__ アトリビュートを参照します。

.. code:: ipython2

    from quantopian.pipeline.data import Fundamentals
    
    # Fudamentals.exchange_id はstring型なので、.latestはクラシファイアを返します。
    exchange = Fundamentals.exchange_id.latest

以前のレッスンでは ``latest`` アトリビュートは ``ファクター`` のインスタンスを返していましたが、今回のケースでは ``string`` を返しているため、
``latest`` アトリビュートは ``クラシファイア`` を作成していることがわかります。

同様に、ある証券に対して直近のモーニングスター社による業種コードを返すコードは ``クラシファイア`` と言えます。
このケースでは返り値は ``int`` となりますが、この整数の値は数値を表すものではなく区分を表すものなので、このコードもまたクラシファイアを作成しています。

直近の業種コードは、組込の ``Sector`` クラシファイアを使うことで取得できます。

.. code:: ipython2

    from quantopian.pipeline.classifiers.fundamentals import Sector  
    morningstar_sector = Sector()

``Sector`` を用いるのと、 ``Fundamentals.morningstar_sector_code.latest`` は同じ結果となります。

クラシファイアからフィルタを作る
----------------------------------

クラシファイアは ``isnull`` 、 ``eq`` 、 ``startwith`` などのメソッドを使ってフィルタを作成することにも使われます。
フィルタを作ることができる ``クラシファイア`` メソッドの一覧は `ここ <https://www.quantopian.com/help#quantopian_pipeline_classifiers_Classifier>`__ 
にあります。

たとえばニューヨーク証券取引所で取引されている証券を選択するフィルタが必要な場合、 ``exchange`` クラシファイアに ``eq`` メソッドを使うことで実現できます。

.. code:: ipython2

    nyse_filter = exchange.eq('NYS')

このフィルタは直近の ``exchange_id`` が ``NYS`` であるものに対して ``True`` を返します。

分位数
----------------

クラシファイアは、さまざまな ``ファクタ`` メソッドからも作り出すことができます。
その最たる例が ``quantiles（分位数）`` メソッドです。このメソッドは引数として分割数（bin）を受け取ります。
``quantile`` メソッドは、全ての非数値型でないファクター出力に対して、0から（bin - 1）までの数値でラベル付けを行うことで、 ``クラシファイア`` 
を返します。  非数値（ ``NaN`` ）には、-1がラベル付けされます。
`四分位数（quartiles） <https://www.quantopian.com/help/#quantopian_pipeline_factors_Factor_quartiles>`__ (``quantiles(4)`` と等価) や、 
`五分位数（quintiles） <https://www.quantopian.com/help/#quantopian_pipeline_factors_Factor_quintiles>`__ (``quantiles(5)`` と等価)、
`十分位数（deciles） <https://www.quantopian.com/help/#quantopian_pipeline_factors_Factor_deciles>`__ (``quantiles(10)`` と等価) といった別名メソッドを使うこともできます。

たとえばファクター出力に対して10分割（deciles）を行い、その第10分位に含まれる銘柄を選択するフィルタはこのようになります：

.. code:: ipython2

    dollar_volume_decile = AverageDollarVolume(window_length=10).deciles()
    top_decile = (dollar_volume_decile.eq(9))

パイプラインに作成したクラシファイアをセットしてその結果を確認すると以下のようになります：

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


クラシファイアはファクター出力に対する複雑なグループ化処理を表現することにも役立ちます。
`demean <https://www.quantopian.com/help#quantopian_pipeline_factors_Factor_demean>`__ や、 
`groupby <https://www.quantopian.com/help#quantopian_pipeline_factors_Factor_groupby>`__ といった集計処理は
このチュートリアルの範囲を超えます。より一歩進んだクラシファイアの使い方については、今後のチュートリアルで取り上げることになるでしょう。

次のレッスンでは、パイプラインの中で使うことができるほかのデータセットを見ていきます。
