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

    result

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
        <tr>
          <th>Equity(39 [DDC])</th>
        </tr>
        <tr>
          <th>Equity(41 [ARCB])</th>
        </tr>
        <tr>
          <th>Equity(52 [ABM])</th>
        </tr>
        <tr>
          <th>Equity(53 [ABMD])</th>
        </tr>
        <tr>
          <th>Equity(62 [ABT])</th>
        </tr>
        <tr>
          <th>Equity(64 [ABX])</th>
        </tr>
        <tr>
          <th>Equity(66 [AB])</th>
        </tr>
        <tr>
          <th>Equity(67 [ADSK])</th>
        </tr>
        <tr>
          <th>Equity(69 [ACAT])</th>
        </tr>
        <tr>
          <th>Equity(70 [VBF])</th>
        </tr>
        <tr>
          <th>Equity(76 [TAP])</th>
        </tr>
        <tr>
          <th>Equity(84 [ACET])</th>
        </tr>
        <tr>
          <th>Equity(86 [ACG])</th>
        </tr>
        <tr>
          <th>Equity(88 [ACI])</th>
        </tr>
        <tr>
          <th>Equity(100 [IEP])</th>
        </tr>
        <tr>
          <th>Equity(106 [ACU])</th>
        </tr>
        <tr>
          <th>Equity(110 [ACXM])</th>
        </tr>
        <tr>
          <th>Equity(112 [ACY])</th>
        </tr>
        <tr>
          <th>Equity(114 [ADBE])</th>
        </tr>
        <tr>
          <th>Equity(117 [AEY])</th>
        </tr>
        <tr>
          <th>Equity(122 [ADI])</th>
        </tr>
        <tr>
          <th>Equity(128 [ADM])</th>
        </tr>
        <tr>
          <th>Equity(134 [SXCL])</th>
        </tr>
        <tr>
          <th>Equity(149 [ADX])</th>
        </tr>
        <tr>
          <th>Equity(153 [AE])</th>
        </tr>
        <tr>
          <th>...</th>
        </tr>
        <tr>
          <th>Equity(48961 [NYMT_O])</th>
        </tr>
        <tr>
          <th>Equity(48962 [CSAL])</th>
        </tr>
        <tr>
          <th>Equity(48963 [PAK])</th>
        </tr>
        <tr>
          <th>Equity(48969 [NSA])</th>
        </tr>
        <tr>
          <th>Equity(48971 [BSM])</th>
        </tr>
        <tr>
          <th>Equity(48972 [EVA])</th>
        </tr>
        <tr>
          <th>Equity(48981 [APIC])</th>
        </tr>
        <tr>
          <th>Equity(48989 [UK])</th>
        </tr>
        <tr>
          <th>Equity(48990 [ACWF])</th>
        </tr>
        <tr>
          <th>Equity(48991 [ISCF])</th>
        </tr>
        <tr>
          <th>Equity(48992 [INTF])</th>
        </tr>
        <tr>
          <th>Equity(48993 [JETS])</th>
        </tr>
        <tr>
          <th>Equity(48994 [ACTX])</th>
        </tr>
        <tr>
          <th>Equity(48995 [LRGF])</th>
        </tr>
        <tr>
          <th>Equity(48996 [SMLF])</th>
        </tr>
        <tr>
          <th>Equity(48997 [VKTX])</th>
        </tr>
        <tr>
          <th>Equity(48998 [OPGN])</th>
        </tr>
        <tr>
          <th>Equity(48999 [AAPC])</th>
        </tr>
        <tr>
          <th>Equity(49000 [BPMC])</th>
        </tr>
        <tr>
          <th>Equity(49001 [CLCD])</th>
        </tr>
        <tr>
          <th>Equity(49004 [TNP_PRD])</th>
        </tr>
        <tr>
          <th>Equity(49005 [ARWA_U])</th>
        </tr>
        <tr>
          <th>Equity(49006 [BVXV])</th>
        </tr>
        <tr>
          <th>Equity(49007 [BVXV_W])</th>
        </tr>
        <tr>
          <th>Equity(49008 [OPGN_W])</th>
        </tr>
        <tr>
          <th>Equity(49009 [PRKU])</th>
        </tr>
        <tr>
          <th>Equity(49010 [TBRA])</th>
        </tr>
        <tr>
          <th>Equity(49131 [OESX])</th>
        </tr>
        <tr>
          <th>Equity(49259 [ITUS])</th>
        </tr>
        <tr>
          <th>Equity(49523 [TLGT])</th>
        </tr>
      </tbody>
    </table>
    <p>8236 rows × 0 columns</p>
    </div>

空のパイプラインは列データを持たないDataFrameを出力しています。今回の場合、パイプラインは2015年5月5日に対して8000超(上の表では途中中断)
の証券からなるインデックスを持っていますが、列データを何も持っていません。

以降のレッスンでは、パイプラインの出力に対してどのように列を追加していくのか、どのようにフィルタをかけて証券を絞り込んでいくのかをみていきます。
