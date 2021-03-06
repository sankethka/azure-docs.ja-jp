---
title: "Azure Machine Learning での特徴エンジニアリングと特徴選択 | Microsoft Docs"
description: "特徴選択と特徴エンジニアリングの目的について説明し、機械学習のデータ強化プロセスにおけるこれらの役割の例を示します。"
services: machine-learning
documentationcenter: 
author: bradsev
manager: jhubbard
editor: cgronlun
ms.assetid: 9ceb524d-842e-4f77-9eae-a18e599442d6
ms.service: machine-learning
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/18/2017
ms.author: zhangya;bradsev
ROBOTS: NOINDEX
redirect_url: machine-learning-data-science-create-features
redirect_document_id: TRUE
ms.translationtype: Human Translation
ms.sourcegitcommit: ba61d00f277af579c87a130336ead9879b82a6de
ms.openlocfilehash: c6b88355df430e78594fc1283c9df01ad6e27e20
ms.contentlocale: ja-jp
ms.lasthandoff: 12/13/2016


---
# <a name="feature-engineering-and-selection-in-azure-machine-learning"></a>Azure Machine Learning での特徴エンジニアリングと特徴選択
このトピックでは、機械学習のデータ強化プロセスにおける特徴エンジニアリングと特徴選択の目的について説明します。 これらのプロセスに含まれる内容については、Azure Machine Learning Studio から提供される例を使用して説明します。

[!INCLUDE [machine-learning-free-trial](../../includes/machine-learning-free-trial.md)]

機械学習で使用されるトレーニング データは、多くの場合、収集された生データから特徴を選択したり抽出したりすることで向上させることができます。 手書き文字の画像の分類方法の学習において、エンジニアリングされた特徴の例は、生のビット分布データで構成されているビット密度マップです。 このマップは、生の分布よりも効率的に文字のエッジを検索できます。

エンジニアリングされ、選択された特徴は、データに含まれるキー情報の抽出を試みるトレーニング プロセスの効率を高めます。 また、入力データを正確に分類して、関心のある結果をより確実に予測するために、これらのモデルのパワーを向上させます。 特徴エンジニアリングと特徴選択は、学習を計算的により扱いやすくするために組み合わせることもできます。 これは、強化した後、モデルの調整やトレーニングに必要な特徴の数を減らすことによって行われます。 数学的に言うと、モデルのトレーニングに選択される特徴は、データのパターンを説明し、正常に結果を予測する独立変数の最小セットです。

特徴のエンジニア リングと選択は、大きなプロセスの一部であり、通常は次の&4; つのステップから構成されます。

* データ収集
* データ強化
* モデル構築
* 後処理

エンジニアリングと選択は、機械学習のデータ強化手順を構成する要素です。 このプロセスの&3; つの要素を次の目的で区別する場合があります。

* **データ前処理**: このプロセスは、収集したデータがクリーンで一貫性があることを保証しようとします。 これには、複数のデータ セットの統合、欠落しているデータの処理、一貫性のないデータの処理、データ型の変換などのタスクが含まれます。
* **特徴エンジニアリング**: このプロセスは、データ内の既存の生の特徴から、関連する特徴を作成し、学習アルゴリズムの予測力を高めようとします。
* **特徴選択**: このプロセスは、トレーニング問題の次元を削減するために、元のデータの特徴の主要なサブセットを選択します。

このトピックでは、データ強化プロセスの特徴エンジニアリングと特徴選択についてのみ説明します。 データ前処理の手順の詳細については、「[Pre-processing Data in Azure Machine Learning Studio (Azure Machine Learning Studio のデータ前処理)](https://azure.microsoft.com/documentation/videos/preprocessing-data-in-azure-ml-studio/)」のビデオをご覧ください。

## <a name="creating-features-from-your-data--feature-engineering"></a>データから特徴を作成する - 特徴エンジニアリング
トレーニング データは、例 (行に格納されているレコードや観測) からなるマトリックスで構成され、それぞれが特徴のセット (列に格納されている変数やフィールド) を持っています。 実験計画で指定された特徴は、データ内のパターンを特徴付けることが期待されます。 生のデータ フィールドの多くは、モデルのトレーニングに使用される、選択された特徴セットに直接含めることができますが、多くの場合、追加の (エンジニアリングされた) 特徴は、強化されたトレーニング データ セットを生成するために生データの特徴から構築される必要があります。

モデルをトレーニングする場合に、データ セットを強化するためには、どのような特徴を作成する必要があるでしょうか。 トレーニングを強化するようにエンジニアリングされた特徴は、データ内のパターンをより適切に識別する情報を提供します。 新しい特徴は、元の特徴セットや既存の特徴セットで明確に捕捉できたり、簡単に見てとることができたりしない追加情報を提供することが期待されます。しかし、その処理にはかなりの技量が必要になります。 正当な生産性の高い意思決定は、多くの場合、専門知識が必要です。

Azure Machine Learning から始めると、Machine Learning Studio で提供されているサンプルを使用して、最も簡単にこのプロセスを具体的に把握できます。 次に、2 つの例を示します。

* ターゲット値が既知の場合の教師あり実験における、[レンタル自転車の数の予測](http://gallery.cortanaintelligence.com/Experiment/Regression-Demand-estimation-4)の回帰の例
* [特徴ハッシュ][feature-hashing]を使用したテキスト マイニングの分類例

### <a name="example-1-adding-temporal-features-for-a-regression-model"></a>例 1: 回帰モデルに時間的な特徴を追加する
回帰タスクの特徴をエンジニアリングする方法を説明するために、Azure Machine Learning Studio で "自転車の需要予測" の実験を使用します。 この実験の目的は、自転車の需要、つまり特定の月、日、時間内でのレンタル自転車の数を予測することです。 データ セットの **Bike Rental UCI データ セット**は、生の入力データとして使用します。

このデータ セットは、米国のワシントン D.C. で自転車のレンタル ネットワークを管理している Capital Bikeshare 社の実際のデータに基づいています。 データ セットは、2011 年から 2012 年までの 1 日の特定の時間帯のレンタル自転車の数を表し、17,379 行と 17 列が含まれています。 生の特徴セットには、気象条件 (温度、湿度、風速) やその日の種類 (休日、平日) が含まれます。 予測するフィールドは **cnt** です。これは、特定の時間帯のレンタル自転車数を 1 ～ 977 の範囲内で表しています。

トレーニング データに効果的な特徴を構築するために、4 つの回帰モデルは同じアルゴリズムを使用して構築されましたが、トレーニング データ セットはそれぞれ異なります。 次の&4; つのデータ セットは同じ生の入力データを表しますが、特徴セットの数は増加しています。 これらの特徴は、次の&4; つのカテゴリに分類されます。

1. A = 予測日の天候 + 休日 + 平日 + 週末の各特徴
2. B = 過去 12 時間ごとにレンタルされた自転車の数
3. C = 過去 12 日ごとにレンタルされた自転車の数 (同じ時間帯)
4. D = 過去 12 週ごとにレンタルされた自転車の数 (同じ時間帯、同じ日)

元の生データに既に存在する特徴セット A のほかに、他の&3; つの特徴セットが特徴エンジニアリング プロセスによって作成されます。 特徴セット B では、直近の自転車の需要を収集します。 特徴セット C では、特定の時間帯の自転車の需要を収集します。 特徴セット D では、特定の曜日の特定の時間帯の自転車の需要を収集します。 この&4; つのトレーニング データ セットは、それぞれ A、A+B、A+B+C、A+B+C+D の特徴セットを含んでいます。

Azure Machine Learning の実験では、これら&4; つのトレーニング データ セットは、前処理された入力データ セットからの&4; つの分岐を使用して形成されます。 一番左の分岐を除いて、これらの各分岐には [R スクリプトの実行][execute-r-script]モジュールが含まれており、生成された一連の特徴 (特徴セット B、C、D) は個別に構築され、インポートされたデータ セットに追加されます。 次の図は、左の&2; 番目の分岐にある特徴セット B の作成に使用される R スクリプトを示します。

![機能セットの作成](./media/machine-learning-feature-selection-and-engineering/addFeature-Rscripts.png)

次の表は、4 つのモデルのパフォーマンス結果の比較をまとめたものです。 特徴 A+B+C よって最適な結果が表示されます。 トレーニング データに特徴セットを追加すると、エラー率が低下する点に注意してください。 これにより、特徴セット B および C は、関連する追加情報を回帰タスクに提供するという推測が検証されます。 D の特徴セットを追加しても、エラー率はそれ以上は低下しないようです。

![パフォーマンス結果の比較](./media/machine-learning-feature-selection-and-engineering/result1.png)

### <a name="example2"></a> 例 2: テキスト マイニングで特徴を作成する
特徴エンジニアリングは、ドキュメントの分類やセンチメント分析などのテキスト マイニングに関連するタスクに広く適用されています。 たとえば、ドキュメントをいくつかのカテゴリに分類する場合は、通常、1 つのドキュメントのカテゴリに含まれる単語や語句が別のドキュメントのカテゴリに存在する可能性が低いことを前提とします。 つまり、単語や語句の分布の頻度から、さまざまなドキュメント カテゴリを特徴付けることができます。 テキスト マイニング アプリケーションでは、通常、テキスト コンテンツの各部分が入力データとして機能するため、出現する単語や語句の頻度を含む特徴を作成するために特徴エンジニアリング プロセスが必要になります。

このタスクを実現するには、"*特徴ハッシュ*" と呼ばれる手法を適用して、任意のテキストの特徴を効率的にインデックスに変えます。 各テキストの特徴 (単語や語句) を特定のインデックスに関連付ける代わりに、ハッシュ関数を特徴に適用し、そのハッシュ値を直接インデックスとして使用することでこのメソッドが機能します。

Azure Machine Learning には、これらの単語や語句の特徴を作成する[特徴ハッシュ][feature-hashing] モジュールがあります。 このモジュールを使用する例を次に示します。 入力データ セットには、1 ～ 5 の書籍の評価と実際のレビュー内容の 2 つの列が含まれています。 この[特徴ハッシュ][feature-hashing] モジュールの目的は、特定の書籍レビュー内の対応する単語や語句の出現頻度を示す新しい特徴を取得することです。 このモジュールを使用するには、次の手順を完了する必要があります。

1. 入力テキストが含まれている列を選択します (この例では **Col2**)。
2. *Hashing bitsize* を 8 に設定します。これにより、2 ^8 = 256 の特徴が作成されます。 テキストの単語や語句は、その結果、256 のインデックスにハッシュされます。 *Hashing bitsize* パラメーターの範囲は 1 ～ 31 です。 パラメーターに大きな数値を設定すれば、単語や語句が同じインデックスにハッシュされる可能性が低くなります。
3. *N-grams* パラメーターを 2 に設定します。 これは、入力テキストからユニグラム (単語ごとの特徴) とバイグラム (隣接する&2; 単語ごとの特徴) の出現頻度を取得します。 *N-grams* パラメーターの範囲は 0 ～ 10 で、特徴に含まれる最大の連続単語数を示します。  

![特徴ハッシュ モジュール](./media/machine-learning-feature-selection-and-engineering/feature-Hashing1.png)

次の図は、これらの新しい特徴の外観を示しています。

![特徴ハッシュの例](./media/machine-learning-feature-selection-and-engineering/feature-Hashing2.png)

## <a name="filtering-features-from-your-data--feature-selection"></a>データから特徴をフィルタリングする - 特徴選択
"*特徴選択*" は、分類や回帰タスクなどの予測モデリング タスク用のトレーニング データ セットの構築に一般的に適用されるプロセスです。 目標は、最小限の特徴セットを使用してデータ内の最大量の分散を表すことで、元のデータ セットからその次元を削減する特徴のサブセットを選択することです。 この特徴のサブセットには、モデルのトレーニングに含まれる唯一の特徴が含まれます。 特徴選択は、次の&2; つの主な目的を果たします。

* 特徴選択は多くの場合、無関係な特徴、重複した特徴、関連性の高い特徴を排除することで、分類の精度を向上させます。
* 特徴の数を減らすことで、モデルのトレーニング プロセスの効率を高めます。 これは、サポート ベクター マシンなどのトレーニングに費用がかかる学習者にとって特に重要です。

特徴選択は、モデルのトレーニングに使用されるデータ セット内の特徴の数を減らそうと努めていますが、通常は "*次元削減*" とは呼ばれません。 特徴選択メソッドは、データ内の元の特徴のサブセットを変更せずに抽出します。  次元削減メソッドは、元の機能を変換し、さらに変更できるようにエンジニアリングされた特徴を使用します。 次元削減メソッドの例には、主成分分析、正準相関分析、特異値分解が含まれます。

教師ありコンテキストの特徴選択メソッドの&1; つとして広く適用されているカテゴリは、フィルターに基づく特徴選択です。 各特徴と対象の属性間の相関関係を評価することで、これらのメソッドは、統計的尺度を適用して各特徴にスコアを割り当てます。 特徴は、スコアによって順位付けされます。これは、しきい値を設定して特定の特徴を保持や削除する場合に使用できます。 これらのメソッドで使用される統計的尺度の例には、ピアソン相関関係、相互情報、カイ二乗検定が含まれます。

Azure Machine Learning Studio には、特徴選択のためのモジュールが用意されています。 次の図に示すように、これらのモジュールには、[フィルターに基づく特徴選択][filter-based-feature-selection]と [Fisher 線形判別分析][fisher-linear-discriminant-analysis]が含まれます。

![特徴選択の例](./media/machine-learning-feature-selection-and-engineering/feature-Selection.png)

たとえば、[フィルターに基づく特徴選択][filter-based-feature-selection]モジュールと、前に説明したテキスト マイニングの例を使用します。 [特徴ハッシュ][feature-hashing] モジュールから 256 の特徴セットが作成された後、回帰モデルを作成する場合を想定します。応答変数は **Col1** で、書籍レビューの評価を 1 ～ 5 で表すこととします。 **[Feature scoring method (特徴スコア付けの方法)]** を **[Pearson Correlation (ピアソン相関関係)]**、**[Target column (対象列)]** を **[Col1]**、**[Number of desired features (目的の特徴の数)]** を **50** に設定します。 [フィルターに基づく特徴選択][filter-based-feature-selection]モジュールによって、**Col1** という対象の属性を持つ 50 の特徴を含むデータセットが生成されます。 次の図に、この実験のフローと入力パラメーターを示します。

![特徴選択の例](./media/machine-learning-feature-selection-and-engineering/feature-Selection1.png)

次の図は、結果として得られるデータ セットを示します。 それぞれの特徴は、それ自体と対象の属性 **Col1** 間のピアソン相関関係に基づいて評価されます。 上位のスコアを持つ特徴は保持されます。

![フィルターに基づく特徴選択のデータ セット](./media/machine-learning-feature-selection-and-engineering/feature-Selection2.png)

次の図は、選択した特徴に対応するスコアを示しています。

![選択した特徴のスコア](./media/machine-learning-feature-selection-and-engineering/feature-Selection3.png)

この[フィルターに基づく特徴選択][filter-based-feature-selection]モジュールを適用すると、256 のうち 50 の特徴が選択されます。これらは、**ピアソンの相関関係**スコア付け メソッドに基づいたとき、対象の属性 **Col1** と最も関連性が高い特徴であるためです。

## <a name="conclusion"></a>まとめ
特徴エンジニアリングと特徴選択は、機械学習モデルを構築する際に、トレーニング データを準備するために一般的に実行される&2; つの手順です。 通常、特徴エンジニアリングは追加の特徴を生成するために最初に適用され、その後、無関係な特徴、重複した特徴、関連性の高い特徴を排除するために特徴選択の手順が実行されます。

必ずしも特徴エンジニアリングや特徴選択を実行する必要はありません。 必要があるかどうかは、持っているデータや収集するデータ、選択するアルゴリズム、実験の目的によって異なります。

<!-- Module References -->
[execute-r-script]: https://msdn.microsoft.com/library/azure/30806023-392b-42e0-94d6-6b775a6e0fd5/
[feature-hashing]: https://msdn.microsoft.com/library/azure/c9a82660-2d9c-411d-8122-4d9e0b3ce92a/
[filter-based-feature-selection]: https://msdn.microsoft.com/library/azure/918b356b-045c-412b-aa12-94a1d2dad90f/
[fisher-linear-discriminant-analysis]: https://msdn.microsoft.com/library/azure/dcaab0b2-59ca-4bec-bb66-79fd23540080/

