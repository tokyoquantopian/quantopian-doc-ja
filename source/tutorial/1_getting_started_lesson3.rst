Pipeline API
------------

Pipeline APIは、以下のような横断的に資産データ分析を行うための強力なツールです。複数のデータをにたいして一連の演算を行い、一度に大量の資産を分析します。いくつかの一般的なPipeline API の用途としては、以下のようなものがあります。

- フィルタリングルールに基づいた資産の選択
- スコアリング関数に基づく資産のランク付け
- ポートフォリオの配分を計算する

まず、Pipelineクラスをインポートして、空の pipeline を返す関数を作成します。
関数のなかで pipeline を定義しておくと、 複雑な処理を行う場合でも、整った方法で定義をおこなうことが出来ます。
これは、特に Research から pipeline の定義を IDE に移行する時に便利です。

.. code:: ipython2

    # Pipeline class
    from quantopian.pipeline import Pipeline
    
    def make_pipeline():
        # 空の Pipeline を作成し返す。
        return Pipeline()

To add an output to our pipeline we need to include a reference to a
dataset, and specify the computations we want to carry out on that data.
For example, we will add a reference to the ``close`` column from the
``USEquityPricing`` dataset. Then, we can define our output to be the
latest value from this column as follows:

pipeline にデータ出力を追加するには、データセットに参照を追加し、そのデータに対して行いたい演算を定義します。
例えば、``USEquityPricing`` データセットから ``close`` 列への参照を追加し、日々の最新の値を出力するように定義するにはこのように記述します。


.. code:: ipython2

    # Pipeline class と USEquityPricing dataset　を import
    from quantopian.pipeline import Pipeline
    from quantopian.pipeline.data import USEquityPricing
    
    def make_pipeline():
        # 日々の最終価格を取得
        close_price = USEquityPricing.close.latest
    
        # 上記のデータを Pipeline に入れて返す　
        return Pipeline(
            columns={
                'close_price': close_price,
            }
        )

The Pipeline API also provides a number of built-in calculations, some
of which are computed over trailing windows of data. For example, the
following code imports Psychsignal’s ``stocktwits`` dataset and defines
an output as the 3 day moving average of its ``bull_minus_bear`` column:

Pipeline API では、組み込み計算機能が多数提供されています。一定期間を移動しながら計算する演算機能なども機能として提供されています。たとえば、Psychsignalの ``stocktwits`` データセットで提供されている、  ``bull_minus_bear`` データを使って3日移動平均を出力するコードは以下のように定義できます。

.. code:: ipython2

    # Pipeline と データセットをインポート
    from quantopian.pipeline import Pipeline
    from quantopian.pipeline.data import USEquityPricing
    from quantopian.pipeline.data.psychsignal import stocktwits
    
    # 移動平均を計算する関数をインポート
    from quantopian.pipeline.factors import SimpleMovingAverage
    
    
    def make_pipeline():
        # 日々の最終価格を取得
        close_price = USEquityPricing.close.latest
    
        # bull_minus_bearスコアの3日移動平均を演算
        sentiment_score = SimpleMovingAverage(
            inputs=[stocktwits.bull_minus_bear],
            window_length=3,
        )
    
        # pipelineに、最終価格と、センチメントスコアを入れて、返す
        return Pipeline(
            columns={
                'close_price': close_price,
                'sentiment_score': sentiment_score,
            }
        )

Universe Selection
~~~~~~~~~~~~~~~~~~

戦略開発の重要な部分は、資産のセットを定義することです。つまり、複数の銘柄からなるポートフォリオをつくって、取引することを考えることです。この資産セットのことを、トレーディング・ユニバース（trading universe）と呼びます。

トレーディング・ユニバースは可能な限り大きくなければなりません。しかし同時、不必要な資産を排除する必要もあります。
例えば、流動性の低い銘柄や、取引困難な銘柄などは取り除きたいでしょう。
Quantopianの ``QTradableStocksUS`` ユニバースは、このような特色をすでに持つユニバースです。よって、 pipeline のスクリーニングパラメータを使って、 ``QTradableStocksUS`` を私達のトレーディング・ユニバースに設定してみましょう。


.. code:: ipython2

    # Pipeline と　データセットをインポート
    from quantopian.pipeline import Pipeline
    from quantopian.pipeline.data import USEquityPricing
    from quantopian.pipeline.data.psychsignal import stocktwits
    
    # 移動平均を計算する関数をインポート
    from quantopian.pipeline.factors import SimpleMovingAverage
    
    # 組み込みトレーディング・ユニバースをインポート
    from quantopian.pipeline.filters import QTradableStocksUS
    
    
    def make_pipeline():
        # トレーディング・ユニバースへの参照を作成
        base_universe = QTradableStocksUS()
    
        # 日々の最終価格を取得
        close_price = USEquityPricing.close.latest
    
        # bull_minus_bearスコアの3日移動平均を演算
        sentiment_score = SimpleMovingAverage(
            inputs=[stocktwits.bull_minus_bear],
            window_length=3,
        )
    
        # pipelineに、最終価格と、センチメントスコア、スクリーニングとして、トレーディング・ユニバースを入れて返す
        return Pipeline(
            columns={
                'close_price': close_price,
                'sentiment_score': sentiment_score,
            },
            screen=base_universe
        )

これで pipeline の定義は完了しました。次に、``run_pipeline`` をつかって指定した期間で pipeline を実行してみましょう。出力結果は、 pandas の DataFrame で、日付と資産名が index に持ちます。列は、 pipeline で定義したカラムです。

.. code:: ipython2

    # run_pipelineをインポート
    from quantopian.research import run_pipeline
    
    # start_date と end_dateを指定してｍmake_pipeline関数を実行して pipeline を実行。
    pipeline_output = run_pipeline(
        make_pipeline(),
        start_date='2013-01-01',
        end_date='2013-12-31'
    )
    
    # 最初の10行を表示
    pipeline_output.tail(10)




.. raw:: html

    <div>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th></th>
          <th>close_price</th>
          <th>sentiment_score</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th rowspan="10" valign="top">2013-12-31 00:00:00+00:00</th>
          <th>Equity(43721 [SCTY])</th>
          <td>57.32</td>
          <td>-0.176667</td>
        </tr>
        <tr>
          <th>Equity(43919 [LMCA])</th>
          <td>146.22</td>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(43981 [NCLH])</th>
          <td>35.25</td>
          <td>-0.700000</td>
        </tr>
        <tr>
          <th>Equity(44053 [TPH])</th>
          <td>19.33</td>
          <td>0.333333</td>
        </tr>
        <tr>
          <th>Equity(44060 [ZTS])</th>
          <td>32.68</td>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(44089 [BCC])</th>
          <td>29.66</td>
          <td>1.000000</td>
        </tr>
        <tr>
          <th>Equity(44102 [XONE])</th>
          <td>60.50</td>
          <td>0.396667</td>
        </tr>
        <tr>
          <th>Equity(44158 [XOOM])</th>
          <td>27.31</td>
          <td>-0.160000</td>
        </tr>
        <tr>
          <th>Equity(44249 [APAM])</th>
          <td>64.53</td>
          <td>0.000000</td>
        </tr>
        <tr>
          <th>Equity(44270 [SSNI])</th>
          <td>21.05</td>
          <td>0.423333</td>
        </tr>
      </tbody>
    </table>
    </div>



次のレッスンでは、取引する資産を選択するためにアルゴリズムが使用する戦略を整えます。その後、過去のデータに対する戦略の予測力を評価するために、ファクター分析ツールを使ってみます。
