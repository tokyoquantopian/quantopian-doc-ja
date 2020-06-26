アルゴリズムにおけるデータ処理
---------------------------------

次のステップでは、Researchで構築したデータパイプラインをアルゴリズムに統合します。
Researchとの重要な違いは、バックテストの間、シミュレーションの進行に合わせてpipelineが必ず毎日実行されることです。
したがって、 ``start_date`` と ``end_date`` を記述する必要はありません。

アルゴリズムの中でデータパイプラインを使うためには、まずアルゴリズムの ``initialize`` 関数の中に データパイプライン への参照を追加します。
これは ``attach_pipeline`` メソッドを使って行います。これには2つの引数があり、一つめは、 ``Pipeline`` オブジェクトへの参照（これは、 ``make_pipeline`` を使って作成します。）、もう一つは、その参照に名前を付けるための任意の ``文字列`` です。


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

冒頭で述べたように、パイプラインは、バックテスト期間内の毎日、マーケットが開く前にデータを処理し、出力を生成します。この出力は、 ``before_trading_start`` 関数のなかで、 ``pipeline_output`` 関数を使って取得することができます。なお、その出力は、 pandasのDataFrame型のデータになっています。
それでは、毎日生成されるパイプラインの出力のうち、最初の10行だけ、 ``rebalance`` 関数のなかでログに書き出してみましょう。



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
このアルゴリズムでは、あらかじめ用意した取引対象銘柄に入っている銘柄のうち、センチメントスコアが付与されている全ての銘柄を（銘柄数の制限なしで）対象とします。
そのため、 ``sentiment_score`` の ``notnull`` メソッドを使って、センチメントスコアが付与されている銘柄を取り出し、それを ``&`` 演算子で取引対象銘柄と組み合わせて、取引すべき銘柄を絞り込みます。


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


ここまでのところで、バックテスト期間中の毎日、取引の対象となる銘柄候補を選び出し、それぞれについて、ポートフォリオ内の資産配分を決定するために使うアルファスコアを得られるようになりました。
次のレッスンでは、データパイプラインによって得られたアルファスコアに基づいて、最適なポートフォリオを構築する方法を学びます。