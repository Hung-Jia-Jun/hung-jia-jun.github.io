---
{"tags":null,"dg-publish":true,"dg-draft":false,"dg-created_time":"2025-12-25","dg-updated_time":"2025-12-25","permalink":"/技術文件/Apache Beam/Apache Beam/","dgPassFrontmatter":true,"created":"2025-12-25","updated":"2025-12-25"}
---



#etl #apache-beam #pipeline

# Basics of the Beam model
![](https://i.imgur.com/AFAqk1j.png)
Apache Beam是一個統一的模型, 用於定義批處理和流式數據並行處理管道。要開始使用Beam, 您需要了解一組重要的核心概念：

-   [_Pipeline_](https://beam.apache.org/documentation/basics/#pipeline) 
    管道是用戶設定的轉換圖, 定義了所需的數據處理操作。
-   [_PCollection_](https://beam.apache.org/documentation/basics/#pcollection)
    PCollection - `PCollection` 是數據集或數據流。管道處理的數據是PCollection的一部分。
-   [_PTransform_](https://beam.apache.org/documentation/basics/#ptransform) 
    PTransform - `PTransform` （或transform）表示管道中的數據處理操作或步驟。變換應用於零個或多個 `PCollection` 對象, 並產生零個或多個 `PCollection` 對象。
-   [_Window_](https://beam.apache.org/documentation/basics/#window) 
    窗口 `PCollection` 可以根據各個元素的時間戳細分為窗口。通過將集合劃分為有限集合的窗口, 窗口支持對隨時間增長的集合進行分組操作。

![](https://i.imgur.com/gGnioKt.png)

**Bounded vs. unbounded**: Bounded vs. unbounded：

## PCollection
`PCollection` 可以是有界的或無界的。

-   A _bounded_ `PCollection` is a dataset of a known
    有界的 `PCollection` 是一個已知的、固定大小的數據集（或者, 一個不隨時間增長的數據集）。有界數據可以通過批處理管道進行處理。
-   An _unbounded_ `PCollection` is a dataset that grows over time
    無界的 `PCollection` 是一個隨時間增長的數據集, 數據在到達時被處理。無界數據必須通過流式管道處理。
    
這兩個類別直覺的來自於批處理和流處理, 但這兩者在Beam中是統一的
有界和無界PCollections可以在同一管道中共存。如果你的runner只能支持有界PCollection, 你必須拒絕包含無界PCollection的管道。如果你的runner只針對stream
那麼Beam的支持代碼中有Adapter可以將所有內容轉換為針對無界(unbound)數據的API。

## PTransform
`PTransform` （或transform）表示管道中的數據處理操作或步驟。變換通常應用於一個或多個輸入 `PCollection` 對象。

您以函數對象的形式提供轉換處理邏輯（通俗地稱為「用戶代碼」）, 並且您的用戶代碼應用於輸入PCollection（或多個PCollection）的每個元素。根據您選擇的管道運行器和後端, 集群中的許多不同工作者可能會並行執行用戶代碼的實例。在每個worker上運行的用戶代碼生成輸出元素, 這些元素被添加到零個或多個輸出 `PCollection` 對象。

-   Source transforms
    如 `TextIO.Read` 和 `Create` 。
-   Processing and conversion operations
    處理和轉換操作, 如 `ParDo` 、 `GroupByKey` 、 `CoGroupByKey` 、 `Combine` 和 `Count` 。
-   Outputting transforms
    輸出轉換, 如 `TextIO.Write` 。
-   User-defined, 
    用戶定義的、特定於應用程式的複合轉換。

## 自定義函數
某些Beam操作允許您運行用戶定義的代碼作為配置轉換的一種方式。例如, 當使用 `ParDo` 時, 用戶定義的代碼指定對每個元素應用什麼操作。對於 `Combine` , 它指定應該如何組合值。通過使用跨語言轉換, Beam管道可以包含用不同語言編寫的UDF, 甚至可以在同一管道中包含多種語言。

Beam有幾種UDF：
舉常用的為例
-   [_DoFn_](https://beam.apache.org/documentation/programming-guide/#pardo) 
    DoFn -每元素處理函數（用於 `ParDo` ）
## Runner

Beam運行器在特定平台上運行Beam管道。大多數runner都是大規模並行大數據處理系統的翻譯器或適配器, 例如Apache Flink, Apache Spark, Google Cloud Dataflow等。
例如, Flink runner將Beam管道轉換為Flink作業。Direct Runner在本地運行管道, 這樣您就可以測試、調試和驗證管道是否儘可能接近Apache Beam模型。

有關Runner的更多信息, 請參閱以下頁面：

-   [Choosing a Runner](https://beam.apache.org/documentation/#choosing-a-runner)
-   [Beam Capability Matrix](https://beam.apache.org/documentation/runners/capability-matrix/)

## Window 窗口
根據其各個元素的時間戳將 `PCollection` 細分為窗口。通過將集合劃分為有限集合的窗口, 窗口支持對無界集合的分組操作。

ex:
-   **Fixed time windows**
    固定時間窗口, 表示數據流中持續時間一致、不重疊的時間間隔。

#### 8.2.1.**Fixed time windows**(固定時間窗口)
最簡單的窗口形式是使用固定時間窗口：給定可能連續更新的帶時間戳的 `PCollection` , 每個窗口可以捕獲（例如）具有落入30秒間隔的時間戳的所有元素。

固定時間窗口表示數據流中的一致持續時間、非重疊時間間隔。考慮持續時間為30秒的窗口：在unbounded `PCollection` 中, 所有時間戳值從0：00：00到（但不包括）0：00：30的元素都屬於第一個窗口, 時間戳值從0：00：30到（但不包括）0：01：00的元素屬於第二個窗口, 依此類推。
![](https://i.imgur.com/2DwJ53K.png)
