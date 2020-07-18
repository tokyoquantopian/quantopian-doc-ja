レッスン2から11は、research環境で実行します。researchを利用可能な状態にするためにはJupyter notebookを新規作成するか、"Get Notebook"をクリックすることでこのレッスンのnotebookバージョンを複製することができます。
もしあなたががまだresearch環境に不慣れであれば `Introduction to Research <https://www.quantopian.com/lectures/introduction-to-research>`__ レクチャーから始めるか、 `documentation <https://www.quantopian.com/docs/user-guide/environments/research>`__ に目を通しておくことをお薦めします。 

パイプラインを作成する
-------------------------

いくつかのimport文を追加するところから始めましょう。まず始めに、 ``Pipeline`` クラスをインポートします。

.. code:: ipython2

    from quantopian.pipeline import Pipeline

新しいセルの中で、パイプラインを作成するための関数を定義します。
関数の中にパイプラインの作成を内包することによって、後ほど紹介する、より複雑なパイプラインを構築するための土台が出来上がります。
ここで、以下の関数は単純に空のパイプラインを返します。

.. code:: ipython2

    def make_pipeline():
        return Pipeline()

新しいセルで ``make_pipeline()`` を実行することにより、パイプラインをインスタンス化しましょう:

.. code:: ipython2

    my_pipe = make_pipeline()

パイプラインを実行する
-------------------------

空のパイプラインを参照する ``my_pipe`` を作成しました。これを実行し、どのようになっているかを見てみましょう。
ただし、パイプラインを実行する前に ``run_pipeline`` をインポートする必要があります。
これは research 環境だけで使える関数で、指定した期間を通してパイプラインを実行することを可能にします。

.. code:: ipython2

    from quantopian.research import run_pipeline

では ``run_pipeline`` を使いパイプラインを1日（2015-05-05）だけ実行してその中身を表示してみましょう。 
なお、第2引数と第3引数はそれぞれシミュレーションの「開始日付」と「終了日付」です。

.. code:: ipython2

    result = run_pipeline(my_pipe, '2015-05-05', '2015-05-05')

``run_pipeline`` を実行すると、日付と証券でインデックスされた 
`pandas DataFrame <http://pandas.pydata.org/pandas-docs/stable/generated/pandas.DataFrame.html>`__ が返ってきます。
空のパイプラインがどのようになっているか見てみましょう。

.. code:: ipython2

    result.head()

.. raw:: html

    <div style="max-height:1000px;max-width:1500px;overflow:auto;">
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th></th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th rowspan="61" valign="top">2015-05-05 00:00:00+00:00</th>
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

空のパイプラインは列データを持たないDataFrameを出力しています。今回の場合、パイプラインは2015年5月5日に対して8000超(上の表では途中中断)
の証券からなるインデックスを持っていますが、列データを何も持っていません。

以降のレッスンでは、パイプラインの出力に対してどのように列を追加していくのか、どのようにフィルタをかけて証券を絞り込んでいくのかをみていきます。
