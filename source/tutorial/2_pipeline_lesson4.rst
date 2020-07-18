ファクターの結合
-------------------------

ファクターは組込み数学演算子（+、-、\*など）のいずれかを使い、別のファクターやスカラ値と結合できます。
これにより、複数のファクターを結合させた複雑な式を簡単にかけます。
例えば2つの異なるファクターの平均を取るファクターの作成はシンプルに：

::

   >>> f1 = SomeFactor(...)
   >>> f2 = SomeOtherFactor(...)
   >>> average = (f1 + f2) / 2.0

と書けます。
このレッスンでは、10日間平均ファクターと30日間平均ファクターを組み合わせた ``relative_difference`` ファクターを計算するパイプラインを作成します。
いつもと同じく、import文から始めます：

.. code:: ipython2

    from quantopian.pipeline import Pipeline
    from quantopian.research import run_pipeline
    from quantopian.pipeline.data.builtin import USEquityPricing
    from quantopian.pipeline.factors import SimpleMovingAverage

今回は、10日間移動平均と30日間移動平均の2つのファクターが必要です：

.. code:: ipython2

    mean_close_10 = SimpleMovingAverage(inputs=[USEquityPricing.close], window_length=10)
    mean_close_30 = SimpleMovingAverage(inputs=[USEquityPricing.close], window_length=30)

では、``mean_close_30`` ファクターと ``mean_close_10`` ファクターを使って相対変化（ ``percent_difference`` ）を計算するファクターを作成します。

.. code:: ipython2

    percent_difference = (mean_close_10 - mean_close_30) / mean_close_30

この例を見ると、 ``percent_difference`` はより単純なファクターの組み合わせによって構成されていますが、依然として ``Factor`` です。 
``percent_difference`` をパイプラインの列として追加できます。

それでは ``percent_difference`` を列に持つ（終値平均を計算するファクターは列に加えません）パイプラインを作成する ``make_pipeline`` を定義しましょう：

.. code:: ipython2

    def make_pipeline():
    
        mean_close_10 = SimpleMovingAverage(inputs=[USEquityPricing.close], window_length=10)
        mean_close_30 = SimpleMovingAverage(inputs=[USEquityPricing.close], window_length=30)
    
        percent_difference = (mean_close_10 - mean_close_30) / mean_close_30
    
        return Pipeline(
            columns={
                'percent_difference': percent_difference
            }
        )

新しい出力がどのようになっているか確認します：

.. code:: ipython2

    result = run_pipeline(make_pipeline(), '2015-05-05', '2015-05-05')
    result.head()

.. raw:: html

    <div style="max-height:1000px;max-width:1500px;overflow:auto;">
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th></th>
          <th>percent_difference</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th rowspan="61" valign="top">2015-05-05 00:00:00+00:00</th>
          <th>Equity(2 [AA])</th>
          <td>0.017975</td>
        </tr>
        <tr>
          <th>Equity(21 [AAME])</th>
          <td>-0.002325</td>
        </tr>
        <tr>
          <th>Equity(24 [AAPL])</th>
          <td>0.016905</td>
        </tr>
        <tr>
          <th>Equity(25 [AA_PR])</th>
          <td>0.021544</td>
        </tr>
        <tr>
          <th>Equity(31 [ABAX])</th>
          <td>-0.019639</td>
        </tr>
      </tbody>
    </table>
    </div>

次のレッスンでは、フィルタについて学習します。
