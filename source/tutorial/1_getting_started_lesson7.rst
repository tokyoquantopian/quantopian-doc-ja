ポートフォリオマネジメント
--------------------------

前回のレッスンでは、取引アルゴリズムにdata pipelineを組み込みました。
次は、pipeline によって生成されたアルファスコアを使用して、アルゴリズムがどのようにポートフォリオのリバランスを行うかを定義します。
私たちの目標は、アルファスコアに基づいてリターンを最大化するターゲットポートフォリオを見つけることです。アルファスコアは、いくつかのルールや制約に基づいて特定の構造を維持する為のスコアです。
これは通常、`ポートフォリオ最適化 <https://www.quantopian.com/docs/user-guide/tools/optimize>`__ と呼ばれている取り組みです。

QuantopianのOptimize APIを使うと、 pipelineの出力を、目的関数と制約条件に簡単に変換することができます。　
さらに、 ``order_optimal_portfolio`` を使って、現在のポートフォリオを私達の希望を満たすターゲットポートフォリオに移行させることができます。

最初のステップは、 目的関数を定義することです。ここでは ``MaximizeAlpha`` を使用し、アルファスコアにそって、資産に資本を配分することを試みます。


.. code:: python

    # Optimize API module インポート
    import quantopian.optimize as opt


    def rebalance(context, data):
      # pipeline 出力から alpha を取り出す
      alpha = context.pipeline_data.sentiment_score

      if not alpha.empty:
          # MaximizeAlpha objective を作成
          objective = opt.MaximizeAlpha(alpha)

次に、ターゲットポートフォリオが満たすべき制約のリストを指定します。
まず、いくつかの閾値を ``initialize`` で定義し、それを ``context`` 変数に格納してみましょう。



.. code:: python

    # 制約パラメータ
    context.max_leverage = 1.0
    context.max_pos_size = 0.015
    context.max_turnover = 0.95

上で定義したしきい値を使って``rebalance`` で制約を指定しましょう。


.. code:: python

    # Optimize API module インポート
    import quantopian.optimize as opt


    def rebalance(context, data):
      # pipeline 出力から alpha を取り出す
      alpha = context.pipeline_data.sentiment_score

      if not alpha.empty:
          # MaximizeAlpha objective を作成
          objective = opt.MaximizeAlpha(alpha)

          # ポジションサイズの制約
          constrain_pos_size = opt.PositionConcentration.with_equal_bounds(
              -context.max_pos_size,
              context.max_pos_size
          )

          # ポートフォリオレバレッジの制約
          max_leverage = opt.MaxGrossExposure(context.max_leverage)

          # ロング（買い持ち）とショート（売り持ち）のサイズをだいたい同じに合わせる
          dollar_neutral = opt.DollarNeutral()

          # ポートフォリオの出来高の制約
          max_turnover = opt.MaxTurnover(context.max_turnover)


最後に、 目的関数と制約のリストを ``order_optimal_portfolio`` に渡してターゲットポートフォリオを計算し、現在のポートフォリオを最適な状態にするために必要な注文を出します。


.. code:: python

    # Algorithm API インポート
    import quantopian.algorithm as algo

    # Optimize API インポート
    import quantopian.optimize as opt


    def rebalance(context, data):
      # pipeline 出力から alpha を取り出す
      alpha = context.pipeline_data.sentiment_score

      if not alpha.empty:
          # MaximizeAlpha objective を作成
          objective = opt.MaximizeAlpha(alpha)

          # ポジションサイズの制約
          constrain_pos_size = opt.PositionConcentration.with_equal_bounds(
              -context.max_pos_size,
              context.max_pos_size
          )

          # ポートフォリオレバレッジの制約
          max_leverage = opt.MaxGrossExposure(context.max_leverage)

          # ロング（買い持ち）とショート（売り持ち）のサイズをだいたい同じに合わせる
          dollar_neutral = opt.DollarNeutral()

          # ポートフォリオの出来高の制約
          max_turnover = opt.MaxTurnover(context.max_turnover)

          # 目的関数と制約リストを使ってポートフォリオをリバランスする
          algo.order_optimal_portfolio(
              objective=objective,
              constraints=[
                  constrain_pos_size,
                  max_leverage,
                  dollar_neutral,
                  max_turnover,
              ]
          )

リスクマネジメント
------------------

ターゲットポートフォリオの構造に制約を設定して、ポートフォリオを最適化すると同時に、ポートフォリオのパフォーマンスに影響を与える可能性のある一般的なリスク要因に触れることを制限したいと思います。
例えば、 ``stocktwits`` のセンチメントデータは一時的な性質のものであり、センチメントスコアが急上昇を利用して投資したいと思っているため、アルゴリズムは短期的な急降下のリスクにさらされる可能性があります。
Quantopianの `Risk Model <https://www.quantopian.com/risk-model>`__ を使用して、一般的なリスク要因に対するポートフォリオのエクスポージャーを管理します。
Risk Model は、16種類のリスク要因に対して資産がさらされているリスクを計算します。
11のセクターリスク要因と5つのスタイルリスク要因（短期的な反転を含む）です。


このデータは ``risk_loading_pipeline`` 関数を使えば、簡単にアクセス出来ます。
この関数は、Risk Modelに定義された各リスク要因の結果をコラムに持つ、 data pipeline を返します。


data pipelineと同様に、 risk data pipeline をアルゴリズムに追加し、それを識別するための名前を指定します。
そうすれば ``before_trading_start`` で出力を取得し、それを ``context`` に保存することができます。



.. code:: python

    # Algorithm API インポート
    import quantopian.algorithm as algo

    # Risk API method インポート
    from quantopian.pipeline.experimental import risk_loading_pipeline

    def initialize(context):
        # 制約パラメータ
        context.max_leverage = 1.0
        context.max_pos_size = 0.015
        context.max_turnover = 0.95

        # data pipelines を取り付ける
        algo.attach_pipeline(
            make_pipeline(),
            'data_pipe'
        )
        algo.attach_pipeline(
            risk_loading_pipeline(),
            'risk_pipe'
        )

        # rebalance 関数をスケジュール
        algo.schedule_function(
            rebalance,
            algo.date_rules.week_start(),
            algo.time_rules.market_open(),
        )


    def before_trading_start(context, data):
        # pipeline出力を取得し、contextに格納する。
        context.pipeline_data = algo.pipeline_output(
          'data_pipe'
        )

        context.risk_factor_betas = algo.pipeline_output(
          'risk_pipe'
        )

次に、ポートフォリオ最適化ロジックに ``RiskModelExposure`` 制約を追加します。
この制約はリスクモデルによって生成されたデータを受け取り、リスクモデルに含まれる1つ1つの要因に対して、ターゲットポートフォリオの全体的なエクスポージャーの制限を設定します。


.. code:: python

    # ターゲットポートフォリオのリスクエクスポージャーを制限する。
    # デフォルト値は、セクターエクスポージャーの最大値は0.2、スタイルエクスポージャーの最大値は0.4
    factor_risk_constraints = opt.experimental.RiskModelExposure(
        context.risk_factor_betas,
        version=opt.Newest
    )


下記のコードが、私たちの戦略とポートフォリオ構築ロジックを記述したアルゴリズムです。このコードでバックテストすることが出来ます。
アルゴリズムをclone した後、IDEの右上にある「Run Full Backtest」をクリックして、完全なバックテストを実行してみましょう。

.. note::

    Quantopianにログイン後、本翻訳の原作ページ `https://www.quantopian.com/tutorials/getting-started#lesson7 <https://www.quantopian.com/tutorials/getting-started#lesson7>`__ で、 ``Clone`` ボタンを押してコードをクローンして下さい。



.. code:: python

    # Algorithm API インポート
    import quantopian.algorithm as algo

    # Optimize API インポート
    import quantopian.optimize as opt

    # Pipeline  インポート
    from quantopian.pipeline import Pipeline
    from quantopian.pipeline.data.psychsignal import stocktwits
    from quantopian.pipeline.factors import SimpleMovingAverage

    # built-in universe と Risk API method インポート
    from quantopian.pipeline.filters import QTradableStocksUS
    from quantopian.pipeline.experimental import risk_loading_pipeline


    def initialize(context):
        # 制約パラメータ
        context.max_leverage = 1.0
        context.max_pos_size = 0.015
        context.max_turnover = 0.95

        # data pipelines を取り付ける
        algo.attach_pipeline(
            make_pipeline(),
            'data_pipe'
        )
        algo.attach_pipeline(
            risk_loading_pipeline(),
            'risk_pipe'
        )

        # rebalance 関数をスケジュール
        algo.schedule_function(
            rebalance,
            algo.date_rules.week_start(),
            algo.time_rules.market_open(),
        )


    def before_trading_start(context, data):
        # pipeline出力を取得し、contextに格納する。
        context.pipeline_data = algo.pipeline_output('data_pipe')

        context.risk_factor_betas = algo.pipeline_output('risk_pipe')


    # Pipeline definition
    def make_pipeline():

        sentiment_score = SimpleMovingAverage(
            inputs=[stocktwits.bull_minus_bear],
            window_length=3,
            mask=QTradableStocksUS()
        )

        return Pipeline(
            columns={
                'sentiment_score': sentiment_score,
            },
            screen=sentiment_score.notnull()
        )


    def rebalance(context, data):
        # pipeline 出力から alpha を取り出す
        alpha = context.pipeline_data.sentiment_score

        if not alpha.empty:
            # MaximizeAlpha objective 作成
            objective = opt.MaximizeAlpha(alpha)

            # ポジションサイズ制約
            constrain_pos_size = opt.PositionConcentration.with_equal_bounds(
                -context.max_pos_size,
                context.max_pos_size
            )

            # ターゲットポートフォリオレバレッジ制約
            max_leverage = opt.MaxGrossExposure(context.max_leverage)

            # ロング（買い持ち）とショート（売り持ち）のサイズをだいたい同じに合わせる
            dollar_neutral = opt.DollarNeutral()

            # ポートフォリオの出来高の制約
            max_turnover = opt.MaxTurnover(context.max_turnover)

            # ターゲットポートフォリオのリスクエクスポージャーを制限する。
            # デフォルト値は、セクターエクスポージャーの最大値は0.2
            # スタイルエクスポージャーの最大値は0.4
            factor_risk_constraints = opt.experimental.RiskModelExposure(
                context.risk_factor_betas,
                version=opt.Newest
            )

            # 目的関数と制約リストを使ってポートフォリオをリバランスする
            algo.order_optimal_portfolio(
                objective=objective,
                constraints=[
                    constrain_pos_size,
                    max_leverage,
                    dollar_neutral,
                    max_turnover,
                    factor_risk_constraints,
                ]
            )

次のレッスンでは、バックテストの結果をより詳しく分析する方法を学びます。