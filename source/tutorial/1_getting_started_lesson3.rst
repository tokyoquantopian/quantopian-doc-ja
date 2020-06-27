Pipeline API
------------

.. warning:: 

   このTutorialで使用している PsychSignal Trader Mood データは２０２０年５月で更新終了しました。ご注意下さい。データに関する詳しい情報は `PsychSignal Trader Mood (DEPRECATED) <https://www.quantopian.com/docs/data-reference/psychsignal#psychsignal-data-reference>`__ を参照して下さい。

Pipeline APIは、横断的に資産データ分析を行うための強力なツールです。これにより、複数のデータに対して一連の演算を行い、一度に大量の資産を分析することができます。
一般的なPipeline API の用途として、以下のようなものがあります。

- フィルタリングルールに基づいた資産の選択
- スコアリング関数に基づく資産のランク付け
- ポートフォリオの配分の計算

まず、Pipelineクラスをインポートして、空のpipelineを返す関数を作成します。
関数のなかでpipelineを使う形にすれば、複雑な処理をすっきりと扱うことができます。そうしておけば、pipelineを使った仕組みをResearch環境からIDEに移す時にも、関数ごとまとめて扱えてるので便利です。


.. code:: ipython2

    # Pipeline class
    from quantopian.pipeline import Pipeline
    
    def make_pipeline():
        # 空の Pipeline を作成し返す。
        return Pipeline()

pipelineからデータ出力を取り出すためには、まず、データセットに収録されているデータ項目と、そのデータに対して行いたい演算を指定します。具体的には、日々の終値を取り出す場合、 ``USEquityPricing`` データセットにある close （終値） を用いて、下記のように記述します。



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

Pipeline APIには、計算機能が予め多数用意されており、移動平均など、データのなかで一定期間を切り出して処理をする演算機能なども利用できます。例えば、Psychsignalの ``stocktwits`` データセットで提供されている ``bull_minus_bear`` データについて、3日移動平均を計算して出力するコードは以下のように定義できます。


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

評価対象となる資産を選ぶ
~~~~~~~~~~~~~~~~~~~~~~~~

戦略を開発するうえで、どのような取引対象資産を選ぶかは重要です。
つまり、取引対象として考えられる銘柄からなる資産セットを予め用意し、そのなかから銘柄を選んで取引することを考えます。この資産セットのことを、トレーディング・ユニバース（trading universe）と呼びます。

トレーディング・ユニバースは、できるだけ多くの資産が含まれているのが望ましいですが、一方で、不必要な資産については排除しておく必要もあります。例えば、流動性の低い銘柄や、取引困難な銘柄などは外しておきたいところです。そこで便利なのが、予めそのようなことを考慮して用意されている ``QTradableStocksUS`` ユニバースです。
早速、pipelineのスクリーニングパラメータを使って、 ``QTradableStocksUS`` を私達のトレーディング・ユニバースとして設定しましょう。


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

これでpipelineの定義は完了しました。次に、 ``run_pipeline`` を使い、期間を指定してpipelineを実行してみましょう。結果はpandasのDataFrameで出力され、そのインデックスが日付と資産名、列は pipelineで定義したカラムとなります。

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



次のレッスンでは、アルゴリズムが取引銘柄を選ぶ戦略を構築し、ファクター分析ツールを使って、過去のデータに対する戦略の予測力の評価をします。