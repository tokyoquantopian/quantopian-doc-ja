パイプラインを作成する際、計算に必要な入力データを特定する方法が必要となります。
パイプライン計算に必要な入力データは、 ``Dataset`` と ``BoundColumns`` を使うことで特定できます。

データセット（Dataset）と連結列（BoundColumns）
------------------------------------------------

``DataSets`` はパイプラインAPIに対して計算に必要な入力データを、どこからどのように取り出すのかを知らせる単純なオブジェクトのコレクションです。
ここまでに ``Dataset`` の一例としてすでに ``USEquityPricing`` を紹介しました。

``BoundColumn`` とは、``DataSet`` と密接に連結したデータ列を表します。``BoundColumn`` のインスタンスは、``DataSets`` のアトリビュートにアクスすることで
動的に作られます。パイプライン計算に対する入力データは、 ``BoundColumn`` 型でなければなりません。 
``BoundColumn`` の一例としてすでに ``USEquityPrincing.close`` を紹介しました。

``DataSets`` と ``BoundColumn`` は、実際のデータを保持していない、ということを理解することが大切です。計算処理を記述してパイプラインに追加した場合、
実際の計算処理はパイプラインが実行されるまでは実際には計算を実行していないことを思い出してください。

``DataSets`` と ``BoundColumn`` は同じ方法で解釈することができます。これらは単純に計算の入力データを定義するために使われているのです。
そしてデータはパイプラインが実行された後に配置されます。

dtypes
--------

パイプライン計算を定義する場合、使用できる関数や操作を知るために、入力データのデータ型を知っている必要があります。
``BoundColumn`` の ``dtype`` は、パイプラインが実行されたときのデータ型を教えてくれます。
たとえば、 ``USEquityPricing`` は ``float`` 型の ``dtype`` を持っているのでファクターは ``USequityPricing.close`` を使って算術計算を実行できます。

``BoundColumn`` の ``dtype`` は、計算処理のタイプも決定できます。 
たとえば ``latest`` 計算では、 ``dtype`` の型によって計算処理がファクター（ ``float`` ）なのか、フィルタ（ ``bool`` ）なのか、はたまたクラシファイア（ ``string`` または ``int``）
なのかが決まります。

価格データ（Pricing Data）
-----------------------------

米国株の株価は、 ``USEquityPricing`` データセットに保存されています。 ``USEquityPricing`` は5つの列を持っています。

* USEquityPricing.open（始値）
* USEquityPriging.high（高値）
* USEquityPricing.low（安値）
* USEquityPricing.close（終値）
* USEquityPricing.volume（出来高）

それぞれの列は ``float`` の ``dtype`` を持っています。

財務データ（Fundamental Data）
-----------------------------

Quantopianでは `モーニングスター <https://www.morningstar.com/>`__ が提供する多くの財務データを利用できます。財務データは
``Fundamental`` データセットにおいて ``BoundColumn`` として存在しており、900種類以上が利用可能です。詳しくは 
`Quantopian Fundamental Reference <https://www.quantopian.com/docs/data-reference/morningstar_fundamentals>`__ を参照してください。

パートナーデータ（Partner Data）
---------------------------------

Quantopianでは`USEquityPricing`やモーニングスター財務データのほかにも多くのデータセットを利用できます。
コンセンサス予想データ、ニュースセンチメントなどがこれらに含まれます。ほとんどのデータセットは ``quantopian.pipline.data`` 
以下に、プロバイダ名で名前空間が定義されています。

``USEquityPricing`` 同様、そのほかのデータセットもパイプライン計算に利用される列（ ``BoundColumn`` ）を持ちます。
詳しい情報は、データを使う際のパイプラインの例とともに、 `Data Reference <https://www.quantopian.com/docs/data-reference/overview>`__ に記載されています。
``dtype`` は多岐にわたります。

``BoundColumns`` は次のレッスンで見ていくカスタムファクターで一般的に使われます。
