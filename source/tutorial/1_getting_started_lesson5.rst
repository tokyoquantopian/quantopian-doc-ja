前回のレッスンでは、Researchの機能を使って、ポートフォリオに組み入れる銘柄を選択し、その資産のアルファスコアを計算するデータパイプラインを作成しました。
残りのレッスンでは、 Quantopianの `Interactive Development Environment (IDE) <https://www.quantopian.com/algorithms>`__ を使って、取引アルゴリズムを作成します。IDEでも、データパイプラインを使い、アルファスコアを算出して、ポートフォリオを構築していきます。そして、過去データを用いてシミュレーションを行い、より現実的な条件下でのアルゴリズムのパフォーマンスを分析します。このようなヒストリカルシミュレーションは一般的に、"バックテスト"として知られています。



Algorithm API と 主な関数
-------------------------

次のステップでは、 QuantopianのAlgorithm APIを使用しながら、取引アルゴリズムの基本構造を構築しましょう。
Algorithm APIは、注文のスケジュールや実行を簡単に行う事ができる機能や、アルゴリズムのパラメータを初期化したり管理したりする機能を提供しています。
ここでは、アルゴリズムで使用する主要な関数をいくつか紹介します。


**initialize(context)**

アルゴリズムの実行を開始するときに 、 ``initialize`` という関数は真っ先に一度だけ呼ばれます。その時必ず引数として ``context`` を渡さなくてはいけません。IDEで用いるパラメータの初期化や、一回限りで行なう初期化や設定のロジックは、すべてこの ``initialize`` で行う必要があります。

``context`` は Pythonの `dictionary <https://docs.python.org/2/tutorial/datastructures.html#dictionaries>`__ を拡張したもので、シミュレーションの過程において、その状況を格納するために使われ、アルゴリズムのどの時点においても参照することができるようにしてあります。このため、IDEでの関数呼び出しで共有させたい変数は、グローバル変数を用いるのではなく ``context`` の中に属性として保存しておきましょう。 ``context`` に保存する属性はドット（ 例： ``context.some_attribute`` ）でアクセスして、その内容の設定や参照をすることができます。


**before_trading_start(context, data)**

``before_trading_start`` という関数は、シミュレーションの中で、毎日マーケットが開く前に1回だけ呼び出され、引数として ``context`` と ``data`` を必要とします。 ``context`` は ``initialize`` で使った辞書と同じものです。 ``data`` は複数のAPI関数を格納しているオブジェクトです。それらの関数を使って、任意の資産の現在または過去の価格や数量のデータを調べることができます。
``before_trading_start`` 関数は、 pipeline の出力結果を取得する場所でもあります。また、その結果として得られたデータをポートフォリオ構築に使用するための前処理も行います。これらについては次のレッスンで説明します。


**schedule_function(func, day_rule, time_rule)**

Quantopianのアルゴリズムは `ニューヨーク証券取引所の取引カレンダー <https://www.nyse.com/markets/hours-calendars>`__ に沿って、午前9時30分から午後4時の間株式の取引を行います。
``schedule_function`` を使うと、指定した日時にユーザーがカスタマイズした関数を実行することができます。
例えば、毎週最初の営業日のマーケットオープン時にポートフォリオのリバランスを行う関数は、以下のようにスケジュールすることができます。

.. code:: python3

    schedule_function(
        rebalance, ## ユーザーがカスタマイズした関数名
        date_rule=date_rules.week_start(),
        time_rule=time_rules.market_open()
    )

スケジューリング関数は ``initialize`` 関数の中で呼び出す必要があり、决められたスケジュールに従って呼び出される関数（通常ユーザーが作成するもの）と呼び出すタイミングを引数として取ります。この呼び出される関数は、必ず ``context`` と ``data`` を引数に取るように定義されている必要があります。

利用可能な ``date_rules`` と ``time_rules`` を使ってどのようにスケジューリングすることができるかは `documentation <https://www.quantopian.com/docs/api-reference/algorithm-api-reference#quantopian.algorithm.schedule_function>`__ を参照してください。

次に、取引アルゴリズムの基本構造を作ってみましょう。
下記のアルゴリズムは、シミュレーションで経過した日数を記録し、日付と時間に応じて異なるメッセージをログに出力しているだけです。
この後のレッスンで、 データパイプライン と、取引ロジックをこの基本構造に追加していきます。
"Clone" ボタンをクリックすると、IDEにコピーが作成され、アルゴリズムを実行することが出来ます。

.. important::

    Lesson１の冒頭で説明したとおり、原作側がTutorial１を大幅に改定した為、このスクリプトをクローンすることは出来ません。よって、 Quantopianにログイン後、`Research > Algorithm <https://www.quantopian.com/algorithms>`__ と進み、 ``New Algorithm`` ボタンをおしてIDEを開きスクリプトをコピー＆ペーストしてお使い下さい。

IDEに入ったら、"Build Algorithm" (左上） または "Run Full Backtest" (右上）をクリックしてバックテストを実行します。


.. code:: python3

    # Algorithm API をインポート
    import quantopian.algorithm as algo


    def initialize(context):
        # アルゴリズムパラメータを初期化
        context.day_count = 0
        context.daily_message = "Day {}."
        context.weekly_message = "Time to place some trades!"

        # rebalance 関数をスケジューリング
        algo.schedule_function(
            rebalance,
            date_rule=algo.date_rules.week_start(),
            time_rule=algo.time_rules.market_open()
        )


    def before_trading_start(context, data):
        # 毎日、トレード時間が始まる前に必ず実行する
        context.day_count += 1
        log.info(context.daily_message, context.day_count)


    def rebalance(context, data):
        # リバランスのロジックを実行する
        log.info(context.weekly_message)


取引アルゴリズムの基本的な構造ができたので、前回のレッスンで作成したデータパイプラインをアルゴリズムに追加してみましょう。
