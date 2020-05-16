IDEへの移行
----------------

ここまでのレッスンはResearch環境でパイプラインを作成して実行してきました。
いよいよIDE環境へ移行します。まず始まる前に、``Pipeline`` をインポートのインポートと
空のパイプライン出力を返す ``make_pipeline`` を作成したスケルトンアルゴリズムを作ります。

（訳注：このレッスンのコードはResearch環境では動作しません。IDE環境（アルゴリズム）環境で動作します）

.. code:: ipython2

    import quantopian.algorithm as algo
    from quantopian.pipeline import Pipeline

    def initialize(context):
        my_pipe = make_pipeline()
        algo.attach_pipeline(my_pipe, 'my_pipeline')

    def make_pipeline():
        return Pipeline()

パイプラインへのアタッチ
-------------------------

Research環境では ``make_pipline`` はパイプラインオブジェクトのインスタンス化を行い、
``run_pipeline`` は実行期間指定の指定とパイプライン実行を行ってきました。
アルゴリズムの中でこれを安全に実行することはできないため、どうにかしてアルゴリズムの
シミュレーション（訳注：IDE環境でアルゴリズムのバックテストを行うこと）でパイプラインを実行させる必要があります。
シミュレーションでパイプラインを実行させるためには、 ``attach_pipeline`` を使ってパイプラインをアタッチします。

``attatch_pipeline`` 関数は2つの引数を必要とします。 ``Pipeline`` オブジェクトへの参照、および文字列による任意のパイプライン名です。
``attatch_pipeline`` をインポートして、スケルトンアルゴリズムに作成した空のパイプラインをアタッチします。

.. code:: ipython2

    import quantopian.algorithm as algo
    from quantopian.pipeline import Pipeline

    def initialize(context):
        my_pipe = make_pipeline()
        algo.attach_pipeline(my_pipe, 'my_pipeline')

    def make_pipeline():
        return Pipeline()

パイプラインにアタッチできたので、パイプラインは毎日一度実行されるようになりました。もし2016年6月6日（月）から2016年6月10日（金）まで
アルゴリズムのバックテストやライブトレードを行った場合、パイプラインは毎日1回（合計5回）実行されます。
アタッチしたパイプラインは、実行日ごとに新しいDataFrameを出力します。しかしながら、現在のシミュレーション対象日はパイプライン計算の日付
として暗に示されるため出力されたDataFrameは日付によるインデクスを持ちません。

パイプライン出力
-------------------------

日々のパイプライン出力結果は、 ``before_trading_start`` の中の ``pipeline_output`` で取り出すことができます。
``pipeline_output`` は引数としてアタッチしたパイプライン名を必要とし、シミュレーション対象日におけるシミュレーション結果である
Dataframeを返します。``pipeline_output`` をインポートし、スケルトンアルゴリズムを修正して日々のパイプライン出力を ``context`` に
ストアできるようにします。

.. code:: ipython2

    import quantopian.algorithm as algo
    from quantopian.pipeline import Pipeline

    def initialize(context):
        my_pipe = make_pipeline()
        algo.attach_pipeline(my_pipe, 'my_pipeline')

    def make_pipeline():
        return Pipeline()

    def before_trading_start(context, data):
        # パイプライン出力のDataFrameをcontextにストアする
        context.output = algo.pipeline_output('my_pipeline')

これでスケルトンアルゴリズムは約8000行と0個の列を持つ空のDataframeを毎日出力するようになりました。
出力結果はこのようになります（Research環境とは異なり、 ``MultiIndex`` にならないことに注目してください）。


.. raw:: html

    <div style="max-height:1000px;max-width:1500px;overflow:auto;">
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>Equity(2 [AA])</th>
        </tr>
        <tr>
          <th>Equity(21 [AAME])</th>
        </tr>
        <tr>
          <th>Equity(24 [AAPL])</th>
        </tr>
        <tr>
          <th>Equity(25 [AA_PR])</th>
        </tr>
        <tr>
          <th>Equity(31 [ABAX])</th>
        </tr>
      </tbody>
    </table>
    </div>

Research環境で作成したパイプラインの使用法
------------------------------------------

`以前のレッスン <https://www.quantopian.com/tutorials/pipeline#lesson11>`__ で作成したパイプラインのアルゴリズムに移行するには、
Research環境で使った必要なimport文と、　``make_pipeline`` 関数を、アルゴリズムにコピーするだけです。
以下のパイプラインを実行すると150行と2つの列（ ``longs`` と ``shorts`` ）をもつDataframeをシミュレーション対象日ごとに
 ``context`` にストアします。

.. code:: ipython2

    import quantopian.algorithm as algo
    from quantopian.pipeline import Pipeline
    from quantopian.pipeline.data.builtin import USEquityPricing
    from quantopian.pipeline.factors import SimpleMovingAverage
    from quantopian.pipeline.filters import QTradableStocksUS

    def initialize(context):
        my_pipe = make_pipeline()
        algo.attach_pipeline(my_pipe, 'my_pipeline')

    def make_pipeline():
        """
        パイプラインを作成する
        """

        # ベースとなるユニバースとして、QTradableStockUSをセット
        base_universe = QTradableStocksUS()

        # 10日間の終値移動平均
        mean_10 = SimpleMovingAverage(
            inputs=[USEquityPricing.close],
            window_length=10,
            mask=base_universe
        )

        # 30日間の終値移動平均
        mean_30 = SimpleMovingAverage(
            inputs=[USEquityPricing.close],
            window_length=30,
            mask=base_universe
        )

        percent_difference = (mean_10 - mean_30) / mean_30

        # 空売り銘柄を選択するフィルタ
        shorts = percent_difference.top(75)

        # 購入銘柄を選択するフィルタ
        longs = percent_difference.bottom(75)

        # 全取引銘柄を選択するフィルタ
        securities_to_trade = (shorts | longs)

        return Pipeline(
            columns={
                'longs': longs,
                'shorts': shorts
            },
            screen=(securities_to_trade),
        )

    def before_trading_start(context, data):
        # パイプライン出力のDataFrameをcontextにストアする
        context.output = algo.pipeline_output('my_pipeline')

シミュレーション対象日ごとのパイプライン出力は以下のような感じになります：

.. raw:: html

    <div>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>longs</th>
          <th>shorts</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>Equity(39 [DDC])</th>
          <td>False</td>
          <td>True</td>
        </tr>
        <tr>
          <th>Equity(351 [AMD])</th>
          <td>True</td>
          <td>False</td>
        </tr>
        <tr>
          <th>Equity(371 [TVTY])</th>
          <td>True</td>
          <td>False</td>
        </tr>
        <tr>
          <th>Equity(474 [APOG])</th>
          <td>False</td>
          <td>True</td>
        </tr>
        <tr>
          <th>Equity(523 [AAN])</th>
          <td>False</td>
          <td>True</td>
        </tr>
      </tbody>
    </table>
    </div>

購入または空売りを行う銘柄に対する取引量の計算と、発注を行うための関数をいくつか作成し、パイプライン出力で指定します。
`Getting Started Tutorial <https://www.quantopian.com/tutorials/getting-started>`__ で学んだ基本知識を使い、
取引量の計算と発注を行う関数を実装します。

 .. code:: ipython2

    def compute_target_weights(context, data):
        """
        取引量を計算します。
        """
        # 目標取引量を格納する空のディクショナリの初期化
        # 銘柄と目標取引量をマッピングします。
        weights = {}

        # longs または shorts のリストに銘柄が存在する場合、
        # 全ての銘柄が等しくなるよう取引量を決定します。
        if context.longs and context.shorts:
            long_weight = 0.5 / len(context.longs)
            short_weight = -0.5 / len(context.shorts)
        else:
            return weights

        # longs または　shorts のリストの中に保有銘柄が含まれていない場合、
        # ポジションを解消（訳注：保有量をゼロにする）します。
        for security in context.portfolio.positions:
            if security not in context.longs and security not in context.shorts and data.can_trade(security):
                weights[security] = 0

        for security in context.longs:
            weights[security] = long_weight

        for security in context.shorts:
            weights[security] = short_weight

        return weights

    def before_trading_start(context, data):
        """
        パイプライン出力を取得します。
        """

        # シミュレーション日ごとにパイプライン出力を取得します。
        pipe_results = algo.pipeline_output('my_pipeline')

        # `longs`  列の値がTrueの場合は購入対象となります。
        # 取引可能かどうかを併せてチェックします。
        context.longs = []
        for sec in pipe_results[pipe_results['longs']].index.tolist():
            if data.can_trade(sec):
                context.longs.append(sec)

        # `shorts`  列の値がTrueの場合は空売り対象となります。
        # 取引可能かどうかを併せてチェックします。
        context.shorts = []
        for sec in pipe_results[pipe_results['shorts']].index.tolist():
            if data.can_trade(sec):
                context.shorts.append(sec)

    def my_rebalance(context, data):
        """
        週1度リバランスを実行します。
        """

        # リバランスの際の目標取引量を計算します。
        target_weights = compute_target_weights(context, data)

        # 計算できたらポートフォリオのリバランスを実行します。
        if target_weights:
            algo.order_optimal_portfolio(
                objective=opt.TargetWeights(target_weights),
                constraints=[],
            )

最後にここまでの成果をひとまとめにします。リバランスの回数は週1回とします。

（訳注：Quantopianの `オンラインレッスン上 <https://www.quantopian.com/tutorials/pipeline#lesson12>`__ では、Cloneボタンをクリックすることで以下のアルゴリズムを自分のIDE環境にコピーできます）

.. code:: ipython2

    from quantopian.algorithm import order_optimal_portfolio
    from quantopian.algorithm import attach_pipeline, pipeline_output
    from quantopian.pipeline import Pipeline
    from quantopian.pipeline.data.builtin import USEquityPricing
    from quantopian.pipeline.factors import SimpleMovingAverage
    from quantopian.pipeline.filters import QTradableStocksUS
    import quantopian.optimize as opt

    def initialize(context):
        # リバランス関数を週の始めの取引開始時に実行します。
        schedule_function(
            my_rebalance,
            date_rules.week_start(),
            time_rules.market_open()
        )

        # パイプラインを作成しアルゴリズムにアタッチします。
        my_pipe = make_pipeline()
        attach_pipeline(my_pipe, 'my_pipeline')

    def make_pipeline():
        """
        パイプラインを作成します。
        """

        # ベースとなるユニバースとしてQTradableStocksUSをセットします。
        base_universe = QTradableStocksUS()

        # 10日間の終値移動平均を計算します。
        mean_10 = SimpleMovingAverage(
            inputs=[USEquityPricing.close],
            window_length=10,
            mask=base_universe
        )

        # 30日間の終値移動平均を計算します。
        mean_30 = SimpleMovingAverage(
            inputs=[USEquityPricing.close],
            window_length=30,
            mask=base_universe
        )

        percent_difference = (mean_10 - mean_30) / mean_30

        # 空売り銘柄を選択するフィルタ
        shorts = percent_difference.top(75)

        # 購入銘柄を選択するフィルタ
        longs = percent_difference.bottom(75)

        # 全取引銘柄を選択するフィルタ
        securities_to_trade = (shorts | longs)

        return Pipeline(
            columns={
                'longs': longs,
                'shorts': shorts
            },
            screen=(securities_to_trade),
        )

    def compute_target_weights(context, data):
        """
        取引量を計算します。
        """

        # 目標取引量を格納する空のディクショナリの初期化
        # 銘柄と目標取引量をマッピングします。
        weights = {}

        # longs または shorts のリストに銘柄が存在する場合、
        # 全ての銘柄が等しくなるよう取引量を決定します。
        if context.longs and context.shorts:
            long_weight = 0.5 / len(context.longs)
            short_weight = -0.5 / len(context.shorts)
        else:
            return weights

        # longs または　shorts のリストの中に保有銘柄が含まれていない場合、
        # ポジションを解消（訳注：保有量をゼロにする）します。
        for security in context.portfolio.positions:
            if security not in context.longs and security not in context.shorts and data.can_trade(security):
                weights[security] = 0

        for security in context.longs:
            weights[security] = long_weight

        for security in context.shorts:
            weights[security] = short_weight

        return weights

    def before_trading_start(context, data):
        """
        パイプライン出力を取得します。
        """

        # シミュレーション日ごとにパイプライン出力を取得します。
        pipe_results = pipeline_output('my_pipeline')

        # `longs`  列の値がTrueの場合は購入対象となります。
        # 取引可能かどうかを併せてチェックします。
        context.longs = []
        for sec in pipe_results[pipe_results['longs']].index.tolist():
            if data.can_trade(sec):
                context.longs.append(sec)

        # `shorts`  列の値がTrueの場合は空売り対象となります。
        # 取引可能かどうかを併せてチェックします。
        context.shorts = []
        for sec in pipe_results[pipe_results['shorts']].index.tolist():
            if data.can_trade(sec):
                context.shorts.append(sec)

    def my_rebalance(context, data):
        """
        週1度リバランスを実行します。
        """

        # リバランスの際の目標取引量を計算します。
        target_weights = compute_target_weights(context, data)

        # 計算できたらポートフォリオのリバランスを実行します。
        if target_weights:
            order_optimal_portfolio(
                objective=opt.TargetWeights(target_weights),
                constraints=[],
            )

パイプラインをバックテストで実行する場合、全体の計算スピードを向上させるためにバッチ実行で行われる点に注意してください。
そのためパフォーマンスチャートは周期的に止まって見えます。

パイプラインチュートリアルは以上です。
ぜひResearch環境でパイプラインを自分自身でデザインし、アルゴリズムで実行してみてください。