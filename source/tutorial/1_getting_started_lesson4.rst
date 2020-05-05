戦略定義
---------

Quantopianのデータへのアクセスと操作方法を学んだところで、株式のロングショート戦略を行うためのデータをpipelineで構築してみましょう。
一般的に、株式のロングショート戦略は、各資産の価値を比較し、最も上昇すると思われる資産群（`ロング <https://www.investopedia.com/terms/l/long.asp>`__) と、下落すると思われる資産群（`ショート <https://www.investopedia.com/terms/s/short.asp>`__ ）に賭ける戦略です。

株式のロングショート戦略は、高資産価値と低資産価値の差（スプレッド）が利益になります。
この戦略は、株式のランキングモデルを完全に信頼して建てられた戦略です。
このチュートリアルでは、簡単なランキングスキーマを使ってみます。

**戦略**: センチメントスコアの3日移動平均を取得し、センチメントの高い銘柄と低い銘柄で、高資産価値と低資産価値を判断します。


戦略分析
-----------------

上記の戦略は、 ``SimpleMovingAverage`` 関数と、　``stocktwits``\ の ``bull_minus_bear`` データを使って定義出来ます。
先述したpipelineのレッスンと同じように書くことができます。


.. code:: ipython2

    # Pipeline インポート
    from quantopian.pipeline import Pipeline
    from quantopian.pipeline.data.psychsignal import stocktwits
    from quantopian.pipeline.factors import SimpleMovingAverage
    from quantopian.pipeline.filters import QTradableStocksUS
    
    
    # Pipeline 定義
    def  make_pipeline():
    
        base_universe = QTradableStocksUS()
    
        sentiment_score = SimpleMovingAverage(
            inputs=[stocktwits.bull_minus_bear],
            window_length=3,
        )
    
        return Pipeline(
            columns={
                'sentiment_score': sentiment_score,
            },
            screen=base_universe
        )

説明を簡単にするために、``sentiment_score`` を使って上位および下位350銘柄のみ分析します。
これは、pipelineフィルターを作成すればできます。
``sentiment_score`` の出力を、 ``top`` と ``bottom`` メソッドを使って上位と下位だけ取得するフィルターを作ります。
上位と下位のメソッドを ``|`` オペレータ でつないで和集合を作れば可能です。
次に、私達のトレーディングユニバースではない銘柄を排除するために、フィルターとユニバースを ``&`` オペレータでつなぎます。

.. code:: ipython2

    # Pipeline インポート
    from quantopian.pipeline import Pipeline
    from quantopian.pipeline.data.psychsignal import stocktwits
    from quantopian.pipeline.factors import SimpleMovingAverage
    from quantopian.pipeline.filters import QTradableStocksUS
    
    # Pipeline 定義
    def  make_pipeline():
    
        base_universe = QTradableStocksUS()
    
        sentiment_score = SimpleMovingAverage(
            inputs=[stocktwits.bull_minus_bear],
            window_length=3,
        )
    
        # センチメントスコアに基づいて上位下位350銘柄のみを取得するフィルターを作成
        top_bottom_scores = (
            sentiment_score.top(350) | sentiment_score.bottom(350)
        )
    
        return Pipeline(
            columns={
                'sentiment_score': sentiment_score,
            },
            # 定義したフィルターとトレーディングユニバースのどちらにも入っている銘柄のみにスクリーニングする
            screen=(
                base_universe
                & top_bottom_scores
            )
        )

では、分析に使用できる出力を得るために、3年間のpipelineを実行してみましょう。これには1分ほどかかります。


.. code:: ipython2

    # run_pipeline インポート
    from quantopian.research import run_pipeline
    
    # 評価する期間を指定
    period_start = '2013-01-01'
    period_end = '2016-01-01'
    
    # 指定期間で pipeline 実行
    pipeline_output = run_pipeline(
        make_pipeline(),
        start_date=period_start,
        end_date=period_end
    )

センチメントデータに加えて、この期間の資産価格も必要です。
pipelineが出力するDataFrameのindexは資産リストですので、そのリストを  ``prices`` に渡せば価格データを得ることが出来ます。


.. code:: ipython2

    # prices 関数をインポート
    from quantopian.research import prices
    
    # pipeline が出力した dataframe の index から資産リストを取得し、 unique 関数を使って、重複しないリストを取得します。
    asset_list = pipeline_output.index.levels[1].unique()
    
    # 資産リストに入っている銘柄全てに対して、指定期間の価格を取得します。
    asset_prices = prices(
        asset_list,
        start=period_start,
        end=period_end
    )


次に、Quantopianが作ったオープンソース分析ツールである、 `Alphalens <https://www.quantopian.com/lectures/factor-analysis-with-alphalens>`__ を使って、私達の戦略の品質を検証してみましょう。
まず、 ``get_clean_factor_and_forward_returns`` 関数をつかって、ファクターと価格データを組み合わせます。
この関数は、ファクターデータをクォンタイルに分類し、評価日から数日のあいだ銘柄を保有した場合、収益がいくらになるか計算します。
（複数の期間にたいして計算します。）
ここでは、ファクターデータを上位と下位半分ずつにわけ、1日、5日、10日後の収益結果をみます。


.. code:: ipython2

    # Alphalens インポート
    import alphalens as al
    
    # センチメントスコアに基づいて、quantileに指定された分位数にわける
    factor_data = al.utils.get_clean_factor_and_forward_returns(
        factor=pipeline_output['sentiment_score'],
        prices=asset_prices,
        quantiles=2,
        periods=(1,5,10),
    )
    
    # 上から5行を表示
    factor_data.head(5)



.. raw:: html

    <div>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th></th>
          <th>1D</th>
          <th>5D</th>
          <th>11D</th>
          <th>factor</th>
          <th>factor_quantile</th>
        </tr>
        <tr>
          <th>date</th>
          <th>asset</th>
          <th></th>
          <th></th>
          <th></th>
          <th></th>
          <th></th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th rowspan="5" valign="top">2013-01-02 00:00:00+00:00</th>
          <th>Equity(52 [ABM])</th>
          <td>0.004430</td>
          <td>0.004430</td>
          <td>0.004430</td>
          <td>2.560000</td>
          <td>2</td>
        </tr>
        <tr>
          <th>Equity(114 [ADBE])</th>
          <td>-0.015389</td>
          <td>0.008086</td>
          <td>-0.012259</td>
          <td>-1.896667</td>
          <td>1</td>
        </tr>
        <tr>
          <th>Equity(166 [AES])</th>
          <td>-0.006368</td>
          <td>-0.008104</td>
          <td>-0.005403</td>
          <td>-2.630000</td>
          <td>1</td>
        </tr>
        <tr>
          <th>Equity(209 [AM])</th>
          <td>0.001801</td>
          <td>-0.022995</td>
          <td>-0.038365</td>
          <td>2.370000</td>
          <td>2</td>
        </tr>
        <tr>
          <th>Equity(337 [AMAT])</th>
          <td>-0.002525</td>
          <td>-0.014339</td>
          <td>0.007575</td>
          <td>2.370000</td>
          <td>2</td>
        </tr>
      </tbody>
    </table>
    </div>




出力結果を、 Alphalensに渡すと、分析や描画のツールを使うことができるようになります。
ではまず、指定した全期間における、平均の収益をクォンタイルごとに見てみましょう。
私達の戦略はロングショート戦略なので、下位のクォンタイルの収益がネガティブ、上位のクォンタイルの収益がポジティブであれば良いですね。


.. code:: ipython2

    # ファクターのクォンタイル別に、平均を算出
    mean_return_by_q, std_err_by_q = al.performance.mean_return_by_quantile(factor_data)
    
    # クォンタイルと保有ごとに、平均を描画
    al.plotting.plot_quantile_returns_bar(
        mean_return_by_q.apply(
            al.utils.rate_of_return,
            axis=0,
            args=('1D',)
        )
    );



.. image:: notebook_files/notebook_14_0.png


次に、5日間保有した場合の累積収益を見てみましょう。ただし今回は、ロングとショートのポートフォリオにファクターでウェイトをかけます。

.. code:: ipython2

    import pandas as pd
    # ファクターでウェイト付けしたロングショートのポートフォリオを収益を算出
    ls_factor_returns = al.performance.factor_returns(factor_data)
    
    # 5日間保有した場合の累積収益を描画
    al.plotting.plot_cumulative_returns(ls_factor_returns['5D'], '5D', freq=pd.tseries.offsets.BDay());



.. image:: notebook_files/notebook_16_0.png


上のプロットは大きなドローダウン期間を示していますね。
この分析には、取引コストやマーケットインパクトをまだ考慮に入れていません。
よって、あまり有望な戦略とはいえないようです。
今後、より深い分析をAlphalensを使って行うべきですし、繰り返し戦略アイデアに取組む必要があるでしょう。
しかし今はチュートリアルですので、この戦略のままで行きたいと思います。

さて、戦略を定義し検証しました。
次に、私達の株式ロングショート戦略をバックテスト用にビルドし検証しましょう。
このあとのチュートリアルでは、 InteractiveDevelopment Environment (IDE)を使って、Algorithm APIを使っていきます。

