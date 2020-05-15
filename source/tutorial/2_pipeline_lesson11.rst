.. code:: ipython2

    from quantopian.pipeline import Pipeline
    from quantopian.research import run_pipeline
    from quantopian.pipeline.data.builtin import USEquityPricing
    from quantopian.pipeline.factors import SimpleMovingAverage, AverageDollarVolume

全部のせ 
-----------

ここまでパイプラインAPIの基本コンポーネントについて見てきたので、アルゴリズムで使いたくなるパイプラインの構築をしてみましょう。

まず始めに、パイプライン出力の銘柄数を絞り込むためのフィルタの作成を行います。
今回は、以下の条件を全て満たす銘柄のみを抽出するフィルタを作成します。

* プライマリ株式であること（訳注：`ここ <https://www.quantopian.com/docs/data-reference/morningstar_fundamentals#is-primary-share>`__ 
  によれば、会社がIPOを行い、現在も活発に取引が行われている最初の株式のことを表す）
* 普通株式（common stock）であること
* `預託証券 <http://www.investopedia.com/terms/d/depositaryreceipt.asp>`__ (ADR/GDR)ではないこと
* `相対取引 <http://www.investopedia.com/terms/o/otc.asp>`__ (OTC)されていないこと
* `発行日前取引 <http://www.investopedia.com/terms/w/wi.asp>`__ ではないこと（訳注：`発行日前取引 <http://www.jsda.or.jp/about/jishukisei/words/0180.html>`__ ）
* 会社名に `limited partnership <http://www.investopedia.com/terms/l/limitedpartnership.asp>`__ (LP)が含まれないこと
* 企業情報にLPであることが記載されていないこと
* `ETF <http://www.investopedia.com/terms/e/etf.asp>`__ ではないこと (モーニングスターの財務データを利用する)

基準の選択理由
^^^^^^^^^^^^^^^^^

プライマリ株式や普通株式を選択することで、1企業に対して1銘柄のみを選択できます。
通常、プライマリ株式は企業の代表的な証券であるため、パイプラインではこれらを選択することにします（訳注：株式には普通株のほかに優先株や劣後株といった複数の種類が存在するため、代表的な株式のみをパイプラインの出力対象としていると考えられます）。

ADRやGDRは異なる取引所で売買される株式に対する米国株式市場における預託証書の取引です。
これらには為替変動による固有のリスクが発生することがしばしば起こるのでパイプラインから除外することにします。

OTCやWI、そしてLP株式はほとんどのブローカで取り扱っていないためパイプラインから除外となります。

.. todo:: 以下の一文はTutorialには存在するが、Notebookには存在しない。掲載すべきか？
証券を比較したりランク付けする場合、ETFには財務データがないため、通常の株式とETFとの比較はほぼ無意味です。
ETFの価値は保有する多数の証券から生まれます。りんごとオレンジを比較することがないよう、ETFはパイプラインから除外します。


パイプライン作成
--------------------

それぞれの基準毎にフィルタを作成し組み合わせることで、 ``tradable_stocks`` フィルタを作成します。
まず、モーニングスターの ``DataSet`` と ``IsPrimaryShare`` ビルトインフィルタをインポートします。

.. code:: ipython2

    from quantopian.pipeline.data import Fundamentals
    from quantopian.pipeline.filters.fundamentals import IsPrimaryShare

次に、フィルタを定義します。

.. code:: ipython2

    # プライマリ株式に対するフィルタ。IsPrimaryShareはビルトインフィルタです。
    primary_share = IsPrimaryShare()
    
    # 普通株式に対するフィルタ（普通株と対照的な株式には「優先株」がある）
    # 'ST00000001' は普通株を意味する
    common_stock = Fundamentals.security_type.latest.eq('ST00000001')
    
    # ADR ではないものに対するフィルタ。~ 演算子は、フィルタを反転させる
    not_depositary = ~Fundamentals.is_depositary_receipt.latest
    
    # OTC ではないものに対するフィルタ。
    not_otc = ~Fundamentals.exchange_id.latest.startswith('OTC')
    
    # WI ではないものに対するフィルタ。
    not_wi = ~Fundamentals.symbol.latest.endswith('.WI')
    
    # 名前にLPを含まないものに対するフィルタ。.matches は正規表現を使って検索します。
    not_lp_name = ~Fundamentals.standard_name.latest.matches('.* L[. ]?P.?$')
    
    # モーニングスターの財務データ中の limited_partnarship フィールドがNullであるものに対するフィルタ。
    not_lp_balance_sheet = Fundamentals.limited_partnership.latest.isnull()
    
    # 直近のモーニングスターの時価総額フィールドがNullでない場合はETFではない。
    have_market_cap = Fundamentals.market_cap.latest.notnull()
    
    # 以上全てのフィルタに適合する銘柄に対するフィルタ
    tradeable_stocks = (
        primary_share
        & common_stock
        & not_depositary
        & not_otc
        & not_wi
        & not_lp_name
        & not_lp_balance_sheet
        & have_market_cap
    )

フィルタを定義する際、これまでのレッスンで取り扱っていない ``notnull`` 、 ``startswith`` 、 ``endswidh``　、 ``matches`` といった
``Classifier`` メソッドを使っていることに注意してください。
これらのメソッドに関するドキュメントは `ここ <https://www.quantopian.com/help#quantopian_pipeline_classifiers_Classifier>`__ を参照してください。

次に、20日間の平均売買代金の上位30%に対するフィルタを作成します。ここではこのファクターを ``base_universe`` と命名します。

.. code:: ipython2

    base_universe = AverageDollarVolume(window_length=20, mask=tradeable_stocks).percentile_between(70, 100)

ビルトインユニバース
^^^^^^^^^^^^^^^^^^^^^^^

いまここで、売買代金にもとづいて`売買可能な`銘柄を選択する基本ユニバースを構築しましたが、Quantopianでは同様のことを実現するビルトインフィルタを
用意しています。その中でも最良で最新のフィルタが、 
`QTradableStocksUS <https://www.quantopian.com/help#quantopian_pipeline_filters_QTradableStocksUS>`__ です。

QTradableStocksUSは日々の銘柄ユニバースを選択するビルトインフィルタです。
このフィルタは3種類のフィルタを通して選択基準を維持する、サイズ制約のない可能な限り流動性の高いユニバースを構築します。
QTradableStocksUS は以前の `Q500US <https://www.quantopian.com/help#quantopian_pipeline_filters_Q500US>`__
や `Q1500US <https://www.quantopian.com/help#quantopian_pipeline_filters_Q1500US>`__ フィルタとは異なりサイズ上限がありません。

QTradableStocksUSに関するより詳細な銘柄選定基準は、`ここ <https://www.quantopian.com/posts/working-on-our-best-universe-yet-qtradablestocksus>`__ 
を参照してください。

パイプラインを簡略化するため、ここまでに書いた ``base_universe`` を ``QTradableStocksUS`` ビルトインファクターに置き換えます。
まず、import文が必要です。

.. code:: ipython2

    from quantopian.pipeline.filters import QTradableStocksUS

次にbase_universeに対して、 ``QTradableStocksUS`` をセットします。

.. code:: ipython2

    base_universe = QTradableStocksUS()

これで銘柄の絞り込みを行う ``base_universe`` が用意できました。次は対象銘柄に適用するファクターの構築に目を向けます。
今回は、平均回帰戦略（mean reversion strategy）のためのパイプラインを作成します。
この戦略は、10日間と30日感の移動平均値段（終値）を使います。
2つの移動平均の変動率が最も小さい75銘柄に対して同じ金額だけ株式を保有する一方、
変動率が最も大きい75銘柄に対して同じ金額を空売りするアルゴリズムとします。

これを実現するため、 ``base_universe`` フィルタをパイプラインのマスクとして適用し、2つの移動平均ファクタを作成します。
そして2つのファクターを組み合わせて、変動率を計算するファクターを作成します。

.. code:: ipython2

    # 10日間の終値移動平均
    mean_10 = SimpleMovingAverage(inputs=[USEquityPricing.close], window_length=10, mask=base_universe)
    
    # 30日間の終値移動平均
    mean_30 = SimpleMovingAverage(inputs=[USEquityPricing.close], window_length=30, mask=base_universe)
    
    percent_difference = (mean_10 - mean_30) / mean_30


次に ``percent_difference`` を使い、上位75銘柄と下位75銘柄を選択するフィルタをそれぞれ作成します。

.. code:: ipython2

    # 空売りする銘柄を選択するフィルタを作成
    shorts = percent_difference.top(75)
    
    # 購入する銘柄を選択するフィルタを作成
    longs = percent_difference.bottom(75)

``shorts`` と ``longs`` を結合して、パイプラインのスクリーニングに使うフィルタを作成します。

.. code:: ipython2

    securities_to_trade = (shorts | longs)


コードの前の方にあるフィルタはこの最終フィルタ（ ``securities_to_trade`` ）を構築するためのマスクとして使用してきたので、
``securities_to_trade`` をスクリーンとして用いるとパイプライン出力される銘柄はこのレッスンの冒頭で見てきた基準
（プライマリ株、非ETFなど）を満たします。同様に売買代金も高いものとなります。

（訳注：base_universeはの選択基準は途中でQTradableStockUSに置き換えているため、実際にはQTradableStockUSの選択基準を満たしています）

最後に、パイプラインをインスタンス化します。
同じ金額だけ購入あるいは空売りをするアルゴリズムとしているので、パイプライン出力する情報は取引を行う銘柄（パイプラインのインデックスとして出力されます）と、
それを購入するのか空売りするのかという情報だけです。``longs`` と ``shorts`` フィルタをパイプラインに追加し、screenとして ``securities_to_trade`` をセットします。

.. code:: ipython2

    def make_pipeline():
        
        # ベースとなるユニバース
        base_universe = QTradableStocksUS()
        
        # 10日間の終値移動平均
        mean_10 = SimpleMovingAverage(inputs=[USEquityPricing.close], window_length=10, mask=base_universe)
    
        # 30日間の終値移動平均
        mean_30 = SimpleMovingAverage(inputs=[USEquityPricing.close], window_length=30, mask=base_universe)
    
        # 変化率ファクタ
        percent_difference = (mean_10 - mean_30) / mean_30
        
        # 空売り銘柄を選択するフィルタ
        shorts = percent_difference.top(75)
    
        # 購入銘柄を選択するフィルタ
        longs = percent_difference.bottom(75)
        
        # 売買を行う銘柄を選択するフィルタ
        securities_to_trade = (shorts | longs)
        
        return Pipeline(
            columns={
                'longs': longs,
                'shorts': shorts
            },
            screen=securities_to_trade
        )

パイプラインの実行結果は、2つの列データを持つDataFrameが返ってきます。
これらの列には銘柄に対する購入するか空売りするかを示すboolean値が日付ごとに格納されます。

.. code:: ipython2

    result = run_pipeline(make_pipeline(), '2015-05-05', '2015-05-05')
    result.head()


.. raw:: html

    <div>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th></th>
          <th>longs</th>
          <th>shorts</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th rowspan="5" valign="top">2015-05-05 00:00:00+00:00</th>
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


次のレッスンでは、このパイプラインをアルゴリズムに追加します。
