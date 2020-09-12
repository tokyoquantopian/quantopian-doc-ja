中心傾向の尺度（Measures of Central Tendency）
=====================================================

By Evgenia “Jenny” Nitishinskaya, Maxwell Margenot, and Delaney
Mackenzie.

本書はQuantopian Lecture Seriesの一部となっています。

-  `www.quantopian.com/lectures <https://www.quantopian.com/lectures>`__
-  `github.com/quantopian/research_public <https://github.com/quantopian/research_public>`__

----------------------------------------------------------------------

このノートブックの目的は、単一の数値によってデータ集合を要約する方法について学習し、データ集合を要約する方法について学習することです。
データ分布に関する情報を把握できるようになることを目指しています。

算術平均（Arithmetic mean）
=========================================

算術平均は、数値データを要約する際に頻繁に使われています。
そして通常は「平均」といえば算術平均を意味しています。
観測したデータの合計値を、データの数で割ったものとして定義されます。

.. math:: \mu = \frac{\sum_{i=1}^N X_i}{N}

ここで :math:`X_1, X_2, \ldots , X_N` は観測値を表します。

.. code:: ipython3

    # 2つの便利な統計パッケージをインポートする
    import scipy.stats as stats
    import numpy as np
    
    # ここでは例として２つのデータサンプルを用意します
    x1 = [1, 2, 2, 3, 4, 5, 5, 7]
    x2 = x1 + [100]
    
    print 'x1 の平均：', sum(x1), '/', len(x1), '=', np.mean(x1)
    print 'x2 の平均：', sum(x2), '/', len(x2), '=', np.mean(x2)


.. parsed-literal::

    x1の平均： 29 / 8 = 3.625
    x2の平均： 129 / 9 = 14.3333333333

加重算術平均を定義することもできます。これは、各観測値に対する出現頻度を明示的に指定するのに便利です。
たとえば全保有銘柄のリストを作成するよりも、70%の株式がタイプXに属していると説明するほうがポートフォリオを理解する上で便利です。

加重算術平均は、以下の式によって定義されます。

.. math:: \sum_{i=1}^n w_i X_i 

ここで、 :math:`\sum_{i=1}^n w_i = 1` です。:math:`w_i = 1/n` for all :math:`i` のとき、加重算術平均は、通常の算術平均と一致します。

中央値（Median：メディアン）
============================

データを昇順または降順に並べたとき、リストの中央に位置する数字のことを中央値と呼びます。
もしデータ数 :math:`n` が奇数だった場合、単純にリストの :math:`(n+1)/2` 番目の値が中央値となります。
しかし、データ数が偶数だった場合には、データを２つに分割した中間に挟まるデータが存在しないので、中央値を
:math:`n/2` と :math:`(n+2)/2` 番目のデータの平均を取るものと定義します。

中央値は算術平均に比べ、極端な観測値の影響を受けにくいというという特徴があります。
また、中央値は観測データの中央に来る値を教えてくれますが、他の値との大小関係の情報を与えてくれるわけではありません。

.. code:: ipython3

    print 'x1の中央値：', np.median(x1)
    print 'x2の中央値：', np.median(x2)


.. parsed-literal::

    x1の中央値： 3.5
    x2の中央値： 4.0


最頻値（Mode：モード）
======================

最頻値とは、データ集合において最も出現する回数の多かった値のことです。
最頻値は、平均や中央値とは異なり、数値データではないデータ集合に対しても計算可能です。
データの観測値がそれぞれ独立しているような場合に、最頻値はとても役に立ちます。
例えば、歪みがあるサイコロを振った結果から、6の目がでたからといって5の目が出る可能性が高いと判断することはできません。
歪みがあるサイコロのデータでは、その平均値が4.5であるということよりも最頻値が6であるという情報のほうが役立ちます
（訳注：歪みのないサイコロの平均値は3.5。平均値が4.5であるということは、サイコロが大きい目が出やすいように歪んでいる）。

.. code:: ipython3

    # Scipy は組み込みでmode（最頻値）関数を持っていますが、
    # ２つの値に対して頻度が同じであったり、全ての値が１度しか現れなかった
    # 場合においても必ず１つの結果を返してしまいます。

    # そこでここでは、mode関数することにします
    def mode(l):
        # リスト内の要素に対し、出現回数をカウントする
        counts = {}
        for e in l:
            if e in counts:
                counts[e] += 1
            else:
                counts[e] = 1
                
        # 最も出現回数が高かった要素を返す
        maxcount = 0
        modes = {}
        for (key, value) in counts.items():
            if value > maxcount:
                maxcount = value
                modes = {key}
            elif value == maxcount:
                modes.add(key)
                
        if maxcount > 1 or len(l) == 1:
            return list(modes)
        return 'No mode'
        
    print 'x1の全最頻値：', mode(x1)


.. parsed-literal::

    x1の1最頻値： 2
    x1の全最頻値： [2, 5]

株価収益率のように、多くの異なる値を取りうるデータでは、
2回以上観測されるデータが現れない可能性があります。
そのようなデータを扱う場合には、階級（ビン）値を使って度数分布表（ヒストグラム）を作成を作成します。
それぞれのデータは階級に置き換えられるため、階級によってデータの最頻値を確認することができます。

.. code:: ipython3

    # 銘柄の株価収益率を取得し、データ内の最頻値を計算する
    start = '2014-01-01'
    end = '2015-01-01'
    pricing = get_pricing('SPY', fields='price', start_date=start, end_date=end)
    returns = pricing.pct_change()[1:]
    print '株価収益率の最頻値：', mode(returns)
    
    # 全ての株価収益率が異なる値を取っているため、最頻値の代わりに度数分布を利用する。
    # np.histogram は階級ごとの出現回数と、階級の終端を返す。
    hist, bins = np.histogram(returns, 20) # データを20個の階級に振り分け
    maxfreq = max(hist)
    # 出現頻度が最も大きかった階級に対応する区間を出力する
    print '階級の最頻値：', [(bins[i], bins[i+1]) for i, j in enumerate(hist) if j == maxfreq]


.. parsed-literal::

    株価収益率の最頻値： No mode
    階級の最頻値： [(-0.001330629195540084, 0.00097352774911502182)]


幾何平均（Geometric mean）
==========================

幾何平均が足し算によって計算されるのに対し、幾何平均は掛け算によって計算します：

.. math::  G = \sqrt[n]{X_1X_1\ldots X_n} 

ここで観測値は :math:`X_i \geq 0` です。
また、対数を用いると、算術平均のように書きなおすこともできます:

.. math::  \ln G = \frac{\sum_{i=1}^n \ln X_i}{n} 

（非負のデータを用いた）幾何平均の計算結果は、常に算術平均と等しいか、それ未満になります。
なお、算術平均と結果が同じになるのは、全ての観測値が同じ場合に限られます。

.. code:: ipython3

    # scipyの関数を使って、幾何平均を算出する
    print 'x1の幾何平均：', stats.gmean(x1)
    print 'x2の幾何平均：', stats.gmean(x2)


.. parsed-literal::

    x1の幾何平均： 3.09410402498
    x2の幾何平均： 4.55253458762

負の値を持つ観測値が含まれるときに幾何平均を計算したい場合はどうでしょうか？資産クラスの株価収益率の場合はこの問題を簡単に解決することができます。
資産クラスの株価収益率の最小値は :math:`-1` です。しかし、株価収益率 :math:`R_t` に対して
1を足すことで、:math:`1 + R_t` を計算することができます。この値は、価格間の変化率
:math:`R_t` とは対象的に、２つの期間の資産価格の比率を表します。この値は必ず非負の値となるため、
株価収益率の幾何平均を計算することができます。

.. math::  R_G = \sqrt[T]{(1 + R_1)\ldots (1 + R_T)} - 1

.. code:: ipython3

    # 株価収益率の配列の全要素に1を足してからR_Gを計算する
    ratios = returns + np.ones(len(returns))
    R_G = stats.gmean(ratios) - 1
    print '株価収益率の幾何平均：', R_G


.. parsed-literal::

    株価収益率の幾何平均： 0.000540898532267

幾何平均は、株価収益率を計測したすべての期間において :math:`R_G` で一定の場合、
証券の最終価格が株価収益率 :math:`R_1, \ldots, R_T` の場合と同じになるように定義されます。

.. code:: ipython3

    T = len(returns)
    init_price = pricing[0]
    final_price = pricing[T]
    print '期初の値段：', init_price
    print '期末の値段：', final_price
    print 'R_Gから計算した期末の値段：', init_price*(1 + R_G)**T


.. parsed-literal::

    期初の値段： 179.444
    期末の値段： 205.53
    R_Gから計算した期末の値段： 205.53


調和平均（Harmonic mean）
===========================

調和平均はほかの平均と比べてあまり使われません。定義は以下のとおりとなっています。

.. math::  H = \frac{n}{\sum_{i=1}^n \frac{1}{X_i}} 

幾何平均と同じく、調和平均を算術平均のように書き換えることができます。
調和平均の逆数は、観測値の逆数の算術平均と一致します：

.. math::  \frac{1}{H} = \frac{\sum_{i=1}^n \frac{1}{X_i}}{n} 

非負の観測値 :math:`X_i` による調和平均は常に最大でも幾何平均と同じ値になります。また、幾何平均と算術平均の関係より、
全ての観測値が等しい場合には、調和平均と幾何平均と算術平均が等しくなります。

.. code:: ipython3

    print 'x1の調和平均：', stats.hmean(x1)
    print 'x2の調和平均：', stats.hmean(x2)


.. parsed-literal::

    x1の調和平均： 2.55902513328
    x2の調和平均： 2.86972365624

調和平均は、データを比率で表現できる場合に使用できます。
例えばドルコスト平均法（一定の金額で定期的に購入を続ける方法のこと。
値段が高いときは購入数を減らし、値段が低いときは購入数を増やすことで、購入金額を常に一定とする。）
による平均取得単価は調和平均で計算できます。

点推定は間違っている可能性がある
====================================

平均を取るということは、観測値のデータ分布を破壊して単一の値に圧縮してしまうため、データに含まれる多くの情報を隠してしまいます。
その結果、点推定や１つの数値を利用する指標は、データ内の大きな全体像を誤って評価してしまうことが起こりえます。
データを要約する際には、たとえば、データの散らばりぐあいを考慮しないで平均を利用してしまうなどによって、
重要な情報が失われないように注意を払わなければなりません。

根底にある分布は間違っている可能性がある
-----------------------------------------

平均と散らばり度合いに関する正しい指標を使っていたとしても、根底にある
分布があなたが考えているものと違っていた場合には意味がありません。
例えば、ある事象の発生頻度を標準偏差を用いて計測する場合、通常、正規性を
仮定しています。仮定する必要がある場合を覗いて、分布を想定しないようにし
てください。そして分布を仮定する必要がある場合には、データが、あなたの
仮定する分布に適合するか厳密にチェックしなければなりません。

References
----------

-  “Quantitative Investment Analysis”, by DeFusco, McLeavey, Pinto, and
   Runkle

免責（原文）
*This presentation is for informational purposes only and does not
constitute an offer to sell, a solicitation to buy, or a recommendation
for any security; nor does it constitute an offer to provide investment
advisory or other services by Quantopian, Inc. (“Quantopian”). Nothing
contained herein constitutes investment advice or offers any opinion
with respect to the suitability of any security, and any views expressed
herein should not be taken as advice to buy, sell, or hold any security
or as an endorsement of any security or company. In preparing the
information contained herein, Quantopian, Inc. has not taken into
account the investment needs, objectives, and financial circumstances of
any particular investor. Any views expressed and data illustrated herein
were prepared based upon information, believed to be reliable, available
to Quantopian, Inc. at the time of publication. Quantopian makes no
guarantees as to their accuracy or completeness. All information is
subject to change and may quickly become unreliable for various reasons,
including changes in market conditions or economic circumstances.*
