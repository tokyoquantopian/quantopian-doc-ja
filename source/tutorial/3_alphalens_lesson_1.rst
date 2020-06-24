Alphalens
===========

原作： https://www.quantopian.com/tutorials/alphalens

Alphalensチュートリアルへようこそ。このチュートリアルは、すでにあなたがすでに `Getting Started <https://www.quantopian.com/tutorials/getting-started>`__ と `Pipeline API <https://www.quantopian.com/tutorials/pipeline>`__ を完了していることを前提にしています。


謝辞: Quantopianは、コミュニティメンバーの `Luca <https://www.quantopian.com/users/54460194d718f327fd000380>`__ さんに Alphalensへの貢献とチュートリアル作成に対して感謝の意を表します。


Alphalensとは？
---------------------

Alphalensは将来の収益を予測するために使うアルファファクターの効率性を分析するためのツールです。
また、アルファファクターとは、ある一連の情報が将来の収益を予想する関係にあるかを表現する要因のことです。


いつAlphalensを使うべきか？
----------------------------

Alphalensは、`quant workflow <https://blog.quantopian.com/a-professional-quant-equity-workflow/>`__ という数量分析のワークフローを実現するために使います。

取引対象銘柄を定義し、パイプラインでファクターを構築したら、バックテストの前にAlphalensを使って解析するのがベストです。クオンツワークフローはこのようなフローです。

1. Pipeline APIを使用して取引対象銘柄を定義し、アルファファクターを構築
2. **Alphalensで自分のアルファファクターの予測可能性を分析**
3. `Optimize API <https://www.quantopian.com/docs/user-guide/tools/optimize>`__ を使いながら `QuantopianのIDE <https://www.quantopian.com/algorithms>`__ 上でアルファファクターをつかって取引ストラテジーを作成
4. 取引戦略をバックテストし、Pyfolioで結果を分析。

Pipeline APIのチュートリアルで、取引対象銘柄の定義とアルファファクターの構築方法を学びました。
Alphalensを使用すると、ファクターを検査して予測性を確認することができます。
ステップ1と2は `research notebook <https://www.quantopian.com/notebooks>`__ で行い、ステップ3と4は `algorithms IDE <https://www.quantopian.com/algorithms>`__ で行います。

なぜAlphalensを使うべきか？
-----------------------------

1. Alphalensは高速です。Alphalensを使ってファクターを分析すれば、QuantopianのIDEでfull backtestを実行するよりもはるかに高速に分析できます。
2. Alphalensは視覚的です。テーブルの数字だけでファクターの意味を見出すのは難しいので、Alphalensはチャートを自動作成しデータを視覚化します。
3. Alphalensは簡単です。Pipelineでアルファファクターを作成すれば、ほんの数ステップでファクターを分析できます。

次のレッスンでは、research notebookをつかって、Alphalensが出力したPipelineの結果を分析する方法を学びます。




