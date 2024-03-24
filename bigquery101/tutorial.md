# はじめてのBigQueryハンズオン

## ハンズオンの概要
このラボでは、分析するデータ(CSVファイル)を Cloud Storageへアップロードして、BigQueryへ ImportしてからLooker Studioのダッシュボードに表示させます。

データは2つの種類を準備します。1日の店舗別の売上をまとめた店舗別売上情報と実際の店舗等で売上が発生した時に発生する売上の個別のデータです。

このラボの内容：
- CSV ファイルを Cloud Storage にアップロードします。
- CSV ファイルを BigQuery へインポートします。
- Looker Studio で売上ダッシュボードを作成します。

## ハンズオンの開始
<walkthrough-tutorial-duration duration=5></walkthrough-tutorial-duration>
ハンズオンに利用するファイルをダウンロードします。
既に実施済の手順はスキップすることができます。

### **1. チュートリアル資材をダウンロードする**
```bash
git clone https://github.com/tichiba/googlecloud.git
```

### **2. チュートリアル資材があるディレクトリに移動する**
```bash
cd ~/googlecloud/bigquery101
```

### **3. チュートリアルを開く**
```bash
teachme tutorial.md
```

## Google Cloud プロジェクトの設定
次に、ターミナルの環境変数にプロジェクトIDを設定します。
```bash
export PROJECT_ID=$(gcloud config list --format 'value(core.project)')
```

## GCS バケットの作成とファイルのアップロード

### **1.GCS バケットを作成する**
```bash
gcloud storage buckets create gs://${PROJECT_ID}_bigquery_handson --project=$PROJECT_ID --location=us-central1
```

### **2.チュートリアル資材のcsvをアップロードする**
```bash
gcloud storage cp daily_summary_data.csv stream_data.csv gs://${PROJECT_ID}_bigquery_handson
```

<walkthrough-footnote>ハンズオンに必要なCSVデータをGCSバケットにアップロードすることができました。次にBigQueryへのCSVデータのインポート方法を学びます。</walkthrough-footnote>

## BigQuery の Dataset 準備
<walkthrough-tutorial-duration duration=15></walkthrough-tutorial-duration>
ここからはより直感的に理解しやすいようCloud Console上で操作を行います。

まずは BigQuery の Dataset を作成します。

1. ナビゲーションメニューから [ **ビッグデータ** ] > [ **BigQuery** ]に移動します。
2. エクスプローラに表示される自分のProject IDの右側に表示されている縦の”・・・”をクリックし、”**データセットを作成**”を選択します。
3. [**データセット ID**] と[ **データのロケーション** ]に下記の情報を入力します。
    - [ データセット ID ] : sales_data
    - [ データのロケーション ] : us-central1
4. [ **データセットを作成** ]をクリックします。
5. エクスプローラの 自分の Project ID の頭にある矢印をクリックするとデータセットが作成されていることが確認できます。

## Daily summary data テーブルの作成
次に、作成した Dataset に新しいテーブルを作成します。

1. Data set (**sales_data**) の横にあるメニューをクリックし、”**テーブルを作成**”をクリックします。
2. [ **ソース** ]のテーブルの作成元から “**Google Cloud Storage**“を選択します。
3. [ **参照** ] をクリックして、Cloud Storageから “**daily_summary_data.csv**”ファイルを選択します。
4. [ **テーブル** ] 名として “**daily_summary_data**” を入力し、[ **スキーマ** ] > [ **テキストとして編集** ] をオンにして下記のように定義します。
```
[
    {
        "name": "store",
        "type": "STRING",
        "mode": "REQUIRED"
    },
    {
        "name": "address",
        "type": "STRING",
        "mode": "NULLABLE"
    },
    {
        "name": "category1",
        "type": "NUMERIC",
        "mode": "NULLABLE"
    },
    {
        "name": "category2",
        "type": "NUMERIC",
        "mode": "NULLABLE"
    },
    {
        "name": "category3",
        "type": "NUMERIC",
        "mode": "NULLABLE"
    },
    {
        "name": "category4",
        "type": "NUMERIC",
        "mode": "NULLABLE"
    },
    {
        "name": "lat_long",
        "type": "STRING",
        "mode": "NULLABLE"
    },
    {
        "name": "city",
        "type": "STRING",
        "mode": "NULLABLE"
    }
]
```
5. [ **テーブルを作成** ] をクリックします。

## Stream data テーブルの作成
続けて、もう1つのテーブルを作成します。

1. Data set (**sales_data**) の横にあるメニューをクリックし、”**テーブルを作成**”をクリックします。
2. [ **ソース** ]のテーブルの作成元から “**Google Cloud Storage**“を選択します。
3. [ **参照** ] をクリックして、Cloud Storageから “**stream_data.csv**”ファイルを選択します。
4. [ **テーブル** ] 名として “**stream_data**” を入力し、[ **スキーマ** ] > [ **テキストとして編集** ] をオンにして下記のように定義します。
```
[
   {
       "name": "classification",
       "type": "STRING",
       "mode": "REQUIRED"
   },
   {
       "name": "sub_classification",
       "type": "STRING",
       "mode": "REQUIRED"
   },
   {
       "name": "item_name",
       "type": "STRING",
       "mode": "REQUIRED"
   },
   {
       "name": "price",
       "type": "NUMERIC",
       "mode": "NULLABLE"
   },
   {
       "name": "store",
       "type": "STRING",
       "mode": "NULLABLE"
   }
]
```
5. [ **テーブルを作成** ] をクリックします。

## BigQuery でテーブルのデータを確認
作成した2つのテーブルのデータを確認します。

### **1. Daily summary data テーブルのデータを確認する**

1. ナビゲーション・メニューから [ **BigQuery** ] に移動します。
2. 画面左から “ **Project名 -> sales_data -> daily_summary_data** ” テーブルを選択します。
3. [ **プレビュー** ] をクリックします。

### **2. Stream data テーブルのデータを確認する**

4. 画面左から “ **Project名 -> sales_data -> stream_data** ” テーブルを選択します。
5. [ **プレビュー** ] をクリックします。

<walkthrough-footnote>BigQUeryへCSVデータをインポートすることができました。次にBigQueryのデータに対するクエリの実行方法を学びます。</walkthrough-footnote>

## BigQueryで簡単なクエリを実行
<walkthrough-tutorial-duration duration=15></walkthrough-tutorial-duration>

2つのテーブルのデータをJOINして集計するクエリを実行します。

1. テーブルの画面から “クエリ” -> “新しいタブ” を選択します。
2. 下記のSQLを実行して結果を確認します。
```sql
SELECT a.sub_classification as sub_category
    , count(item_name) as sales_number
 FROM `sales_data.stream_data` as a
INNER JOIN `sales_data.daily_summary_data` as b
   ON a.store = b.store
WHERE b.city = "Chiba"
GROUP BY a.sub_classification
```

3. 実行したクエリを保存して、チームへの共有や次回に再利用することができます。”実行”ボタンのとなりにある ”保存”をクリック　→　”クエリを保存”をクリックします。
4. ”名前”に"**カテゴリ別販売数**"と入力してから”公開設定”を”プロジェクト”を選択してから”保存”をクリックします。
5. 保存されたクエリはエクスプローラのProjectの下で確認ができます。

## BigQueryでクエリの定期実行を設定
次に、定期的にクエリを実行する スケジュールの作成をします。

1. 画面左から “ **Project名 -> 保存したクエリ -> カテゴリ別販売数** ” を選択します。
2. クエリを以下のように修正します。1行目が追加され、実行結果を別テーブルに保存するようにしています。
```sql
CREATE OR REPLACE TABLE `sales_data.daily_items_count_per_sub_category` AS
SELECT a.sub_classification as sub_category
    , count(item_name) as sales_number
 FROM `sales_data.stream_data` as a
INNER JOIN `sales_data.daily_summary_data` as b
   ON a.store = b.store
WHERE b.city = "Chiba"
GROUP BY a.sub_classification
```
3. ”スケジュール”　→　”スケジュールされたクエリを新規作成”をクリックします。
4. 適切な名前を入れて、時刻を “10:00”（毎日10時実行）に設定します。他はデフォルトのままで保存をクリックするとクエリが実行されます（”すぐに開始”を選択したため）
5. dataSetの sales_dataのメニューから”コンテンツを更新”をクリックすると先ほど保存したクエリが実行されて、テーブルが作成されていることが確認できます。

<walkthrough-footnote>おめでとうございます！BigQueryのデータに対するクエリの実行方法はこれで完了です。</walkthrough-footnote>

## (Optional) BigQueryからVertex AIへの接続を作成
<walkthrough-tutorial-duration duration=15></walkthrough-tutorial-duration>
この項目はスキップしても構いません。BigQueryからVertex AIへの接続方法を学びます。

1. 接続を作成するには、エクスプローラペインの [**+データを追加**] をクリックし、続いて [**外部データソースへの接続**] をクリックします。
2. [**接続タイプ**] リストで、[**Vertex AI リモートモデル、リモート関数、BigLake（Cloud リソース）**] を選択します。
3. [**接続 ID**] フィールドに**gemini-connect**を入力します。
4. [**ロケーションタイプ**]で[**リージョン**]を選択し、[**リージョン**]リストで[**us-central1**]を選択します。
4. [**接続を作成**] をクリックします。
5. [**接続へ移動**] をクリックします。
6. [**接続情報**] ペインで、次の手順で使用する**サービス アカウント ID** をコピーします。
7. 作成した接続で用いるサービスアカウントにVertex AIへのアクセス権限を付与するため、次のコマンドを実行します。
```bash
gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member=serviceAccount:**前の手順でコピーしたサービスアカウントIDに書き換えてください** --role=roles/aiplatform.user --condition=None
```

## (Optional) BigQueryから生成AIモデルGeminiへ接続
作成したVertex AIへの接続を用いてBigQueryから生成AIモデルGeminiへ接続します。

1. [**SQLクエリを作成**]をクリックして新しいタブを開き、以下のSQLを実行します。
```sql
CREATE OR REPLACE MODEL sales_data.gemini_pro
  REMOTE WITH CONNECTION `us-central1.gemini-connect`
  OPTIONS(ENDPOINT = 'gemini-pro')
```

2. 同じタブで以下のSQLを実行し、Gemini Proからのレスポンスを確認します。
```sql
SELECT ml_generate_text_result as response
FROM ML.GENERATE_TEXT(
    MODEL sales_data.gemini_pro,
    (SELECT 'Google Cloud Nextについて教えてください' AS prompt),
    STRUCT(1000 as max_output_tokens, 0.2 as temperature)
  )
```

Gemini Proからのレスポンスがjson型であることを確認します。
BigQueryではjson型のデータを次のようなクエリで展開することが可能です。
```sql
SELECT ml_generate_text_result['candidates'][0]['content']['parts'][0]['text'] as response
FROM ML.GENERATE_TEXT(
    MODEL sales_data.gemini_pro,
    (SELECT 'Google Cloud Nextについて教えてください' AS prompt),
    STRUCT(1000 as max_output_tokens, 0.2 as temperature)
  )
```

## (Optional) 顧客の声データを生成AIを用いて分析
BigQuery上で生成AIモデルに接続することで、大量のデータの分析を行います。
```sql
CREATE or REPLACE TABLE sales_data.customer_voice_category_data AS
SELECT 
  ml_generate_text_result['candidates'][0]['content']['parts'][0]['text'] as category,
  customer_voice,
  store
FROM ML.GENERATE_TEXT(
    MODEL sales_data.gemini_pro,
    (SELECT 
      CONCAT(
        '次の顧客の声を分類してください。分類は次のいずれかを選んでください。¥n¥n',
        '商品・品揃え、価格、スタッフ、店舗環境、その他。¥n¥n',
        '顧客の声:',
        customer_voice
     ) AS prompt,
     customer_voice,
     store
     FROM `sales_data.customer_voice_data`),
    STRUCT(1000 as max_output_tokens, 0.2 as temperature)
  )
```