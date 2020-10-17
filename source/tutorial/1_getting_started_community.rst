Welcome
-------

`原作ページ <https://www.quantopian.com/tutorials/getting-started>`__

Quantopianへようこそ。このチュートリアルではまずQuantopianについて紹介します。
Quantopianで解決できる問題や、解決するために助けとなるツールの紹介を行います。
レッスンを終えれば、Quantopianを使うのに十分な理解を得ることが出来るでしょう。

チュートリアルでは、Quantopianを使いこなせるようになることを目的としていますが、Quantopianのエキスパートになることを目的としたものではありません。Quantopianの基本的な使い方に慣れている方は、Quantopianのツールをより深く理解するためのリソースがありますのでこちらをお勧めします。

* `Documentation <https://www.quantopian.com/docs/index>`__ 
* `Pipeline Tutorial <https://www.quantopian.com/tutorials/pipeline>`__  (`翻訳 <https://quantopian-doc.readthedocs.io/ja/latest/tutorial/index.html#pipeline>`__  )
* `Alphalens Tutorial <https://www.quantopian.com/tutorials/alphalens>`__ (`未翻訳 <https://quantopian-doc.readthedocs.io/ja/latest/tutorial/index.html>`__ ) 

このチュートリアルを始めるためには、基本的な `Python <https://docs.python.org/3.5/>`__ のプログラミングスキルが必要です。

.. note:: 

   `原作ページ <https://www.quantopian.com/tutorials/getting-started>`__ のチュートリアルはQuantopian Research 環境がすく利用できる形で記載されています。Researchはクラウドホスト型のJupyter notebook です。Pythonコードをインタラクティブに実行することができます。
   
   Quantopianにログイン後、`原作ページ <https://www.quantopian.com/tutorials/getting-started>`__ のノートブックをクローンすることでチュートリアルのコードを実行することができます。クローンするには原作ページの `Get Notebook` ボタンを押して下さい。

   Research の詳細については、 `documentation <https://www.quantopian.com/docs/user-guide/environments/research>`__ を参照してください。



Quantopianとは？
-------------------

Quantopianは、Pythonを使って世界中の先進国および新興国の株式市場における定量的な金融要因を調査できるクラウドベースのソフトウェアプラットフォームです。Quantopiahでは、あらゆる種類の `金融データ <https://www.quantopian.com/docs/data-reference/overview>`__ に加え、高速で統一されたAPIを提供します。それによってあなたは自身の戦略アイデアのテストを簡単にかつ何度も繰り返し実行できます。また、Quantopianには、 `あなた独自の金融データセットをアップロード <https://www.quantopian.com/docs/user-guide/tools/self-serve>`__ したり、ファクターの有効性を分析したり、グローバルなクオンツコミュニティと結果を共有したりするためのツールも用意されています。

さまざまな業種の株式に対してファクターを調査する方法は、Quantopianでの手順はいかのようなステップになります：

1. 取引対象銘柄を定義する
2. 取引対象銘柄に対して、ファクターを定義する
3. ファクターが機能するかどうかテストする
4. 他のクオンツ（訳注：ここでは forum に参加している他のユーザーのことを言う）に結果をシェアしあなたの戦略アプローチについて議論する

ステップ１と２は、  `Pipeline
API <https://www.quantopian.com/docs/user-guide/tools/pipeline>`__ で実行し、
ステップ３は  `Alphalens <https://www.quantopian.com/docs/user-guide/tools/alphalens>`__ を使います。ステップ４は  `Quantopian community
forum <https://www.quantopian.com/posts>`__ で行います。

さてこの後のチュートリアルでは、Quantopianでのリサーチワークフローの簡単な説明を行っていきます。


Research 環境
~~~~~~~~~~~~~~~~~~~~

チュートリアルのコードは、 Quantopian の **Research** 環境で実行できます。 Research は `Jupyter <https://jupyter-notebook-beginner-guide.readthedocs.io/en/latest/what_is_jupyter.html>`__ ノートブック環境で提供されており、Pythonをインタラクティブに実行することができます。 Researchでは、Quantopianが独自に作成した Pythonのライブラリと、いくつかのサードパーティライブラリがインストールされています。くわしくは、 `documentation <https://www.quantopian.com/docs/user-guide/environments/research>`__ を参照してください。

`原作ページ <https://www.quantopian.com/tutorials/getting-started>`__ にある  `Get Notebook ` ボタンをおすことで、このチュートリアルのコードを試してみることができます。
Pythonを実行するには、コードを記入したセル上で **Shift+Enter** 同時に押下してください。コードセルは、背面が少し灰色のセルです。

Step 1 - 取引対象銘柄を定義する
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

さまざまな業種の株式に対してファクターを調査する時、最初のステップは「ユニバース（取引対象銘柄）」を設定することです。ここでいう「ユニバース」とは、後で分析計算を行う際に考慮したい株式の集合を意味します。
Quantopianでは、ユニバースの定義は `Pipeline
API <https://www.quantopian.com/docs/user-guide/tools/pipeline>`__ を使って行います。同APIを使って、ファクター計算も行います。

Pipeline APIは、いくつかの組み込みデータセットやユーザーがアップロードする `カスタムデータセット <https://www.quantopian.com/custom-datasets>`__ への統一されたインターフェースを提供しています。Pipelineを使用すると、組み込みデータやカスタムデータを使用した計算や式を簡単に定義することができます。たとえば、次のコード例は、2 つの組み込みデータセット、  `FactSet
Fundamentals <https://www.quantopian.com/docs/data-reference/factset_fundamentals>`__  と  `FactSet Equity
Metadata <https://www.quantopian.com/docs/data-reference/equity_metadata>`__ をインポートし、それらを使用してユニバースを定義しています。


.. code:: ipython3

    from quantopian.pipeline import Pipeline
    from quantopian.pipeline.data.factset import Fundamentals, EquityMetadata
    
    is_share = EquityMetadata.security_type.latest.eq('SHARE')
    is_primary = EquityMetadata.is_primary.latest
    primary_shares = (is_share & is_primary)
    market_cap = Fundamentals.mkt_val.latest
    
    universe = market_cap.top(1000, mask=primary_shares)

上記の例では、時価総額でランク付けされた主要銘柄の普通株式の上位1000銘柄をユニバースとして定義しています。
ユニバースは、Quantopianで利用可能なデータを使用して定義することができます。
また、セルフサーブデータツールを使用して、インデックス構成銘柄やその他カスタマイズしたユニバースなど、独自のデータをプラットフォームにアップロードすることもできます。カスタムデータセットのアップロード方法については、 `Self-Serve Data
documentation <https://www.quantopian.com/docs/user-guide/tools/self-serve>`__ を参照してください。今回は、上記に定義したユニバースを使っていきます。


Step 2 - ファクターを定義
~~~~~~~~~~~~~~~~~~~~~~~~~

ユニバースを定義したら、次はファクターを定義しテストしてみましょう。
Quantopianにおいてファクターとは、ユニバース内のアセットに対して一定の頻度で数値を生成する計算のことです。ステップ1と同様に、  ` Pipeline
API <https://www.quantopian.com/docs/user-guide/tools/pipeline>`__ を使用してファクターを定義します。Pipelineは、予め整備されたデータセットに、高速かつ統一された方法でアクセスするAPIを提供します。また、多数の  `ビルドインクラス <https://www.quantopian.com/docs/api-reference/pipeline-api-reference#id8>`__ や  `メソッド <https://www.quantopian.com/docs/api-reference/pipeline-api-reference#factor-methods>`__ が提供されており、これらを使用して素早くファクターを定義することができます。

たとえば、次のコード例は、移動平均算出日数が少ない移動平均(fase) と多い移動平均(slow) を使ってモメンタムを計算しています。


.. code:: ipython3

    from quantopian.pipeline import Pipeline
    from quantopian.pipeline.data import EquityPricing
    from quantopian.pipeline.factors import SimpleMovingAverage
    
    # 1ヶ月 (21 営業日) moving average factor.
    fast_ma = SimpleMovingAverage(inputs=[EquityPricing.close], window_length=21)
    
    # 6ヶ月 (126 営業日) moving average factor.
    slow_ma = SimpleMovingAverage(inputs=[EquityPricing.close], window_length=126)
    
    # fast_ma を slow_ma で割ってモメンタムを算出し、その z-scoreを算出
    momentum = fast_ma / slow_ma
    momentum_factor = momentum.zscore()

ユニバースとファクターを定義したので、市場と期間を選択し、ファクターをシミュレーションすることができます。Pipeline APIの特徴の1つは、  `株価調整 <https://www.quantopian.com/docs/data-reference/overview#corporate-action-adjustments>`__ 、`point-in-time
データ <https://www.quantopian.com/docs/data-reference/overview#point-in-time-data>`__ 、 `銘柄シンボルマッピング <https://www.quantopian.com/docs/data-reference/overview#asset-identifiers>`__ 、上場廃止、データアラインメントなどの一般的なデータエンジニアリングの問題を気にすることなく、ユニバースとファクターを定義できることです。Pipeline はこれらの作業をすべて裏で行ってくれるので、ファクターの構築やテストに時間を割くことができます。

以下のコードは、Pipeline インスタンスを作成してファクターをコラムとして追加し、私達のユニバースに合致するように株式をスクリーニングしています。このPipelineを2016年から2019年の米国株式に対して実行します。

.. code:: ipython3

    from quantopian.pipeline import Pipeline
    from quantopian.pipeline.data import EquityPricing
    from quantopian.pipeline.data.factset import Fundamentals, EquityMetadata
    from quantopian.pipeline.domain import US_EQUITIES, ES_EQUITIES
    from quantopian.pipeline.factors import SimpleMovingAverage
    
    is_share = EquityMetadata.security_type.latest.eq('SHARE')
    is_primary = EquityMetadata.is_primary.latest
    primary_shares = (is_share & is_primary)
    market_cap = Fundamentals.mkt_val.latest
    
    universe = market_cap.top(1000, mask=primary_shares)
    
    # 1ヶ月移動平均ファクター
    fast_ma = SimpleMovingAverage(inputs=[EquityPricing.close], window_length=21)
    
    # 6ヶ月移動平均ファクター
    slow_ma = SimpleMovingAverage(inputs=[EquityPricing.close], window_length=126)
    
    # fast_ma を slow_ma で割ってモメンタムを算出し、その z-scoreを算出
    momentum = fast_ma / slow_ma
    momentum_factor = momentum.zscore()
    
    
    # 作成したモメンタムファクターをつかって米国株のパイプラインを作成。ファクタを実行して株式をスクリーニング
    pipe = Pipeline(
        columns={
            'momentum_factor': momentum_factor,
        },
        screen=momentum_factor.percentile_between(50, 100, mask=universe),
        domain=US_EQUITIES,
    )
    
    # 2016〜2019年で実行し、実行結果を最初の5行だけ出力
    from quantopian.research import run_pipeline
    factor_data = run_pipeline(pipe, '2016-01-01', '2019-01-01')
    print("Result contains {} rows of output.".format(len(factor_data)))
    factor_data.head()

このコードを実行すると ファクター値を持った pandas のデータフレームが生成され、最初の5行が表示されます。株式取引日ごとに、その日ユニバースに入った全銘柄にたいしてファクター値が算出されデータフレームに入ります。これで、ユニバースの各株式銘柄のモメンタム値と、2016年から2019年までの各日のモメンタム値がわかりました。

.. attention:: ライセンスの制限により、Quantopianの一部のデータセットでは、直近の1～2年分のデータが外されている（ `holdouts <https://www.quantopian.com/docs/data-reference/overview#holdout-periods>`__ ）場合があります。各データセットには、最近のデータで外されているデータの長さが記録されています。例えば、FactSet Fundamentalsでは、直近1年分のデータがはずされています。 `Quantopian Enterprise <https://factset.quantopian.com>`__ ではデータは外されることはありません。

次の画像は、この結果のデータフレームを示しています。

.. image:: notebook_files/getting_started_screenshot1.png



Step 3 - ファクターをテスト
~~~~~~~~~~~~~~~~~~~~~~~~~

次にステップ2で定義したファクターの予測性をテストします。予測性の判断材料として、
ファクタを算出した銘柄にたいして、算出日から将来の数日間の収益を計算し、ファクターとともに `Alphalens <https://www.quantopian.com/docs/user-guide/tools/alphalens>`__ に渡します。次のコードは、各銘柄の一日先の収益を計算し、結果をその日のデータ（訳者注：つまり将来の結果を持ったデータ）としてファクターとともに Alphalensに渡すプログラムです。

.. code:: ipython3

  from quantopian.research import get_forward_returns
  import alphalens as al

  # 次の日の結果を取得
  returns_df = get_forward_returns(
      factor_data['momentum_factor'],
      [1],
      US_EQUITIES
  )

  # ファクターとリターンをアルファレンズに渡す
  al_data = al.utils.get_clean_factor(
      factor_data['momentum_factor'],
      returns_df,
      quantiles=5,
      bins=None,
  )

次に、私達のモメンタムファクターを分析するためのティアシートを作成します。


.. code:: ipython3

  from alphalens.tears import create_full_tear_sheet
  create_full_tear_sheet(al_data)

.. image:: notebook_files/getting_started_screenshot2.png
.. image:: notebook_files/getting_started_screenshot3.png
.. image:: notebook_files/getting_started_screenshot4.png

Alphalensのティアシートは、ファクターがどのくらい予見することができるかについて洞察する指標を提供します。詳しくは Alphalensの `documentation <https://www.quantopian.com/docs/user-guide/tools/alphalens>`__ を参照して下さい。


Step 4 - Quantopian Community でアイデアをシェアする
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

興味深いファクターが見つかったら、 `Quantopian コミュニティフォーラム <https://www.quantopian.com/posts>`__ に参加している他のユーザーへシェアしてフィードバックを募りましょう。Quantopianで思いついたアイデアは自分のものですが、結果を共有することで、議論のきっかけになったり、他の人から学ぶ機会が生まれたりすることもあります。コミュニティフォーラムでは、Research notebookを投稿に添付することができます。アイデアの結果とコードを両方共有したい場合は、ノートブックをそのまま共有しましょう。コードを秘密にしたい場合は、ノートブックのコピーを作成し、Alphalensでファクター計算を実行し、Pipelineのコードがあるコードセルを削除し、Alphalensの結果だけをコミュニティの投稿で共有することができます。自分のコードや結果をコミュニティで共有したい場合は、説明を加えたり、質問をしたりすることで、他のユーザーとの生産的な会話が生まれる可能性が高くなります。


まとめと次のステップ
~~~~~~~~~~~~~~~~~~

このチュートリアルでは、Quantopianの概要を説明し、PipelineとAlphalensを使ってファクターを調べるワークフローを説明しました。Quantopianには豊富な `documentation <https://www.quantopian.com/docs/index>`__ が用意されており、プラットフォームについて詳しく知ることができます。Quantopianについて理解を深めたい場合は、ドキュメントの `User Guide <https://www.quantopian.com/docs/user-guide/overview>`__ を、また、すぐに利用できるデータについて詳しく知りたい場合は `Data
Reference <https://www.quantopian.com/docs/data-reference/overview>`__ をご覧下さい。

もし、 `Quantopian enterprise
offering <https://factset.quantopian.com/home>`__ にご興味がある場合は、こちらへご連絡下さい。: enterprise@quantopian.com


