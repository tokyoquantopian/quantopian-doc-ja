Data Processing in Algorithms
-----------------------------

次のステップでは、Researchで構築したdata pipelineをアルゴリズムに統合します。
Researchとの重要な違いは、バックテストの間、シミュレーションの進行に合わせてpipelineが必ず毎日実行されることです。
したがって、 ``start_date`` と ``end_date`` を記述する必要はありません。

アルゴリズムの中でdata pipelineを使うためには、まずアルゴリズムの ``initialize`` 関数の中に data pipeline への参照を追加します。
これは ``attach_pipeline`` メソッドを使って行います。これには2つの入力が必要です。
一つめは、``Pipeline`` オブジェクトへの参照（これは、 ``make_pipeline`` を使って作成します。）、　もう一つは、その参照名の ``文字列`` です。

.. code:: python

    # Algorithm API　をインポート
    import quantopian.algorithm as algo


    def initialize(context):
        # アルゴリズムを pipeline に 取り付ける
        algo.attach_pipeline(
            make_pipeline(),
            'data_pipe'
        )

        # rebalance 関数をスケジュールする
        algo.schedule_function(
            rebalance,
            date_rule=algo.date_rules.week_start(),
            time_rule=algo.time_rules.market_open()
        )


    def before_trading_start(context, data):
        pass


    def rebalance(context, data):
        pass


冒頭で述べたように、pipelineは、毎日実行されます。　
毎日、マーケットが開く前にデータを処理して出力を生成します。
この出力は、 ``pipeline_output`` 関数を使って ``before_trading_start`` で得ることが出来ます。
得られた出力は、 pandasのDataFrame型のデータです。
では、 rebalance関数で、毎日生成されるpipelineの最初の10行だけをログに出力してみましょう。


.. code:: python

    # Algorithm APIをインポート
    import quantopian.algorithm as algo


    def initialize(context):
        # algorithm に pipeline を取り付ける
        algo.attach_pipeline(
            make_pipeline(),
            'data_pipe'
        )

        # rebalance 関数をスケジュールする
        algo.schedule_function(
            rebalance,
            date_rule=algo.date_rules.week_start(),
            time_rule=algo.time_rules.market_open()
        )


    def before_trading_start(context, data):
        # pipeline の出力結果を取得。
        # それを、context.pipeline_data 変数へ格納する。
        context.pipeline_data = algo.pipeline_output(
            'data_pipe'
        )


    def rebalance(context, data):
        # pipeline の出力結果の最初の10行だけをログに出力
        log.info(context.pipeline_data.head(10))


では、Researchで作った ``make_pipeline`` 関数をアルゴリズムに追加してみましょう。
このアルゴリズムでは、トレーディング・ユニバースに入っている資産の中で、センチメントスコアを持っている全ての資産を数の制限をすることなしに、取引対象として考慮しなければなりません。
そのためには ``sentiment_score`` 出力が持つ ``notnull`` メソッドを使ってフィルタを作成し、 ``&`` 演算子を使って、トレーディング・ユニバースから取引すべきユニバース（取引対象銘柄群）を得ます。


.. code:: python

    # Algorithm API インポート
    import quantopian.algorithm as algo

    # Pipeline インポート
    from quantopian.pipeline import Pipeline
    from quantopian.pipeline.data.psychsignal import stocktwits
    from quantopian.pipeline.factors import SimpleMovingAverage
    from quantopian.pipeline.filters import QTradableStocksUS


    def initialize(context):
        # algorithm に pipeline を取り付ける
        algo.attach_pipeline(
            make_pipeline(),
            'data_pipe'
        )

        # rebalance 関数をスケジュールする
        algo.schedule_function(
            rebalance,
            date_rule=algo.date_rules.week_start(),
            time_rule=algo.time_rules.market_open()
        )


    def before_trading_start(context, data):
        # pipeline の出力結果を取得。
        # それを、context.pipeline_data 変数へ格納する。
        context.pipeline_data = algo.pipeline_output('data_pipe')


    def rebalance(context, data):
        # pipeline の出力結果の最初の10行だけをログに出力
        log.info(context.pipeline_data.head(10))


    # Pipeline definition
    def make_pipeline():

        base_universe = QTradableStocksUS()

        sentiment_score = SimpleMovingAverage(
            inputs=[stocktwits.bull_minus_bear],
            window_length=3,
        )

        return Pipeline(
            columns={
                'sentiment_score': sentiment_score,
            },
            screen=(
                base_universe
                & sentiment_score.notnull()
            )
        )


いままでコードでアルゴリズムは、毎日取引可能なユニバースを作り、ポートフォリオ内の資産配分を決定するために使うアルファスコアを生成するようになりました。
次のレッスンでは、data pipelineによって生成されたアルファスコアに基づいて最適なポートフォリオを構築する方法を学びます。
