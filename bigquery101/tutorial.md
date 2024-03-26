# はじめてのBigQueryハンズオン

## ハンズオンの概要
このラボでは、分析するデータ(CSV ファイル)を Cloud Storage へアップロードして、BigQuery へ Import してから Looker Studio のダッシュボードに表示させます。

データは2つの種類を準備します。1日の店舗別の売上をまとめた店舗別売上情報と実際の店舗等で売上が発生した時に発生する売上の個別のデータです。

このラボの内容：
* CSV ファイルを Cloud Storage にアップロードします。
* CSV ファイルを BigQuery へインポートします。
* Looker Studio で売上ダッシュボードを作成します。

## ハンズオンの開始
<walkthrough-tutorial-duration duration=5></walkthrough-tutorial-duration>
ハンズオンに利用するファイルをダウンロードします。既に実施済の手順はスキップすることができます。

Cloud Shell 
<walkthrough-cloud-shell-icon></walkthrough-cloud-shell-icon> を開き、次のコマンドを実行します。

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

**Tips**: コードボックスの横にあるボタンをクリックすることで、クリップボードへのコピーおよび Cloud Shell へのコピーが簡単に行えます。

## GCS バケットの作成とファイルのアップロード

1. ファイルのアップロード先となる GCS バケットを作成します。
```bash
gcloud storage buckets create gs://${PROJECT_ID}_bigquery_handson --project=$PROJECT_ID --location=us-central1
```

2. 作成したGCSバケットにチュートリアル資材のcsvをアップロードします。
```bash
gcloud storage cp daily_summary_data.csv stream_data.csv customer_voice_data.csv gs://${PROJECT_ID}_bigquery_handson
```

ハンズオンに必要なCSVデータをGCSバケットにアップロードすることができました。

次にBigQueryへのCSVデータのインポート方法を学びます。

## BigQuery の Dataset 準備
<walkthrough-tutorial-duration duration=15></walkthrough-tutorial-duration>
ここからはより直感的に理解しやすいよう Cloud Console 上で操作を行います。

まずは BigQuery の Dataset を作成します。

1. ナビゲーションメニュー <walkthrough-nav-menu-icon></walkthrough-nav-menu-icon> から [**BigQuery**] に移動します。
<walkthrough-menu-navigation sectionId="BIGQUERY_SECTION"></walkthrough-menu-navigation>
2. エクスプローラペインに表示される自身のプロジェクト ID の右側に表示されている
<walkthrough-spotlight-pointer cssSelector="[instrumentationid=bq-dataset-explorer-resource-list] button.node-context-menu" single="true">**︙**(三点リーダー)</walkthrough-spotlight-pointer> をクリックし、[**データセットを作成**] を選択します。
 
3. [**データセットを作成する**] ペインで下記の情報を入力します。

  フィールド  | 値
  ------- | --------
  データセット ID | `sales_data`
  ロケーションタイプ | リージョン
  データのロケーション | `us-central1`

4. [**データセットを作成**] をクリックします。
5. エクスプローラペインの自身のプロジェクト ID を選択し、データセット `sales_data` が作成されていることを確認します。

## Daily summary data テーブルの作成
次に、作成した Dataset に新しいテーブルを作成します。

1. エクスプローラーペインで `sales_data` の横にある **︙** をクリックし、続いて [**テーブルを作成**] をクリックします。
2. [**ソース**] の [**テーブルの作成元**] に [**Google Cloud Storage**] を選択します。
3. [**参照**] をクリックして、Cloud Storage から [**daily_summary_data.csv**] ファイルを選択します。
4. [**送信先**] の [**テーブル**] の名前に `daily_summary_data` を入力し、[**スキーマ**] > [**テキストとして編集**] をオンにして下記のように定義します。
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
5. [**テーブルを作成**] をクリックします。

## Stream data テーブルの作成
続けて、もう1つのテーブルを作成します。

1. エクスプローラーペインで `sales_data` の横にある **︙** をクリックし、続いて [**テーブルを作成**] をクリックします。
2. [**ソース**] の [**テーブルの作成元**] に [**Google Cloud Storage**] を選択します。
3. [**参照**] をクリックして、Cloud Storage から [**stream_data.csv**] ファイルを選択します。
4. [**送信先**] の [**テーブル**] の名前に `stream_data` を入力し、[**スキーマ**] > [**テキストとして編集**] をオンにして下記のように定義します。
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
5. [**テーブルを作成**] をクリックします。

## BigQuery でテーブルのデータを確認
作成した2つのテーブルのデータをプレビューで確認します。

### **1. Daily summary data テーブルのデータを確認する**

1. エクスプローラーペインから **プロジェクト ID** > [**sales_data**] > [**daily_summary_data**] テーブルを選択します。
2. [**プレビュー**] をクリックします。

### **2. Stream data テーブルのデータを確認する**

3. エクスプローラーペインから **プロジェクト ID** > [**sales_data**] > [**stream_data**] テーブルを選択します。
4. [**プレビュー**] をクリックします。

BigQUery へ CSV データをインポートすることができました。

次に BigQuery のデータに対するクエリの実行方法を学びます。

## BigQueryで簡単なクエリを実行
<walkthrough-tutorial-duration duration=15></walkthrough-tutorial-duration>

2つのテーブルのデータを JOIN して集計するクエリを実行します。

1. テーブルのプレビュー画面から、 [**クエリ**] > [**新しいタブ**] を選択します。
2. 下記のSQLを入力し、[**実行**] をクリックして実行結果を確認します。
```sql
SELECT a.sub_classification as sub_category
    , count(item_name) as sales_number
 FROM `sales_data.stream_data` as a
INNER JOIN `sales_data.daily_summary_data` as b
   ON a.store = b.store
WHERE b.city = "Chiba"
GROUP BY a.sub_classification
```

3. 実行したクエリを保存して、チームへの共有や次回に再利用することができます。 [**保存**] をクリックし、続いて [**クエリを保存**] をクリックします。
4. [**名前**] に [**カテゴリ別販売数**] と入力し、[**保存**] をクリックします。
5. 保存されたクエリはエクスプローラペインの **プロジェクト ID** > [**クエリ**] の下で確認ができます。

## BigQueryでクエリの定期実行を設定
次に、定期的にクエリを実行する スケジュールの作成をします。

1. エクスプローラーペインから **プロジェクト ID** > [**クエリ**] > [**カテゴリ別販売数**] を選択します。
2. クエリを以下のように修正し、[**クエリを保存**] をクリックします。1行目が追加され、実行結果を別テーブルに保存するようにしています。
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
3. [**スケジュール**] をクリックします。APIの有効化を求められた場合は有効化を行います。
4. **新たにスケジュールされたクエリ** ペインで次のとおり入力します。

フィールド | 値
---------------- | ----------------
クエリの名前 | `日次カテゴリ別販売数`
繰り返しの頻度 | 日
時刻 | `01:00`
すぐに開始 | 選択する
ロケーションタイプ | リージョン
リージョン | `us-central1`

5. 他はデフォルトのまま [**保存**] をクリックし、スケジュールを保存します。
6. すぐに開始 を選択したため、エクスプローラーペインの **プロジェクト ID** > [**sales_data**] の下に新しいテーブル `daily_count_per_subcategory` が作成されていることが確認できます。
7. スケジュールされたクエリの実行結果を、ナビゲーションペインの **スケジュールされたクエリ**  から確認します。

BigQuery のデータに対するクエリの実行方法を学びました。

次に、Looker Studio のダッシュボードを用いて BigQuery のデータを可視化します。

## Looker StudioにBigQueryのデータを追加
<walkthrough-tutorial-duration duration=15></walkthrough-tutorial-duration>

まず、Looker Studio に可視化したい BigQuery のデータを追加します。

1. [Looker Studio](https://lookerstudio.google.com/) にアクセスします。
2. ナビゲーションペインの [**作成**] をクリックし、続いて [**レポート**] をクリックします。
3. [**データに接続**] セクションで [**BigQuery**] をクリックします。
    (**データに接続** のペインが表示されていない場合、メニューバーの [**データを追加**] をクリックしてください)
4. マイプロジェクトからデータソースを次のとおり選択します。

フィールド | 値
---------------- | ----------------
プロジェクト | プロジェクトID
データセット | `sales_data`
表 | `daily_summary_data`

5. [**追加**] をクリックします。続いて [**レポートに追加**] をクリックします。

## Looker Studioにグラフを追加

続いて、追加したデータを元に Looker Studio にグラフを追加します。

1. メニューバーの[**グラフを追加**] > [**棒**] > [**縦棒グラフ**] をクリックします。
2. 追加されたグラフを選択した状態で、[**グラフ**] ペインでデータの設定をします。

フィールド | 値
---------------- | ----------------
ディメンション | `store`
指標 | `category1`, `category2`, `category3`, `category4`
並べ替え | `store`

3. 追加されたグラフをドラッグして任意の位置に移動します。

おめでとうございます！これでLooker Studioのダッシュボードを用いたBigQueryのデータの可視化は完了です。他のテーブルやグラフも自由に追加して試してみてください。

## (Optional) BigQueryからVertex AIへの接続を作成
<walkthrough-tutorial-duration duration=15></walkthrough-tutorial-duration>
この項目はスキップしても構いません。BigQueryからVertex AIへの接続方法を学びます。

1. ナビゲーションメニュー <walkthrough-nav-menu-icon></walkthrough-nav-menu-icon> から [**BigQuery**] に移動します。
1. 接続を作成するには、エクスプローラペインの [**+データを追加**] をクリックし、続いて [**外部データソースへの接続**] をクリックします。
2. **外部データソース** ペインで、次の情報を入力します。

フィールド | 値
--- | ---
接続タイプ | **Vertex AI リモートモデル、リモート関数、BigLake（Cloud リソース）**
接続 ID | `gemini-connect`
ロケーションタイプ | **リージョン**
リージョン | `us-central1`

4. [**接続を作成**] をクリックします。
5. [**接続へ移動**] をクリックします。
6. [**接続情報**] ペインで、次の手順で使用する **サービス アカウント ID** をコピーします。
7. 作成した接続で用いるサービスアカウントにVertex AIへのアクセス権限を付与するため、次のコマンドを実行します。
```bash
gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member=serviceAccount:**前の手順でコピーしたサービスアカウントIDに書き換えてください** --role=roles/aiplatform.user --condition=None
```

## (Optional) BigQueryから生成AIモデルGeminiへ接続
作成したVertex AIへの接続を用いてBigQueryから生成AIモデルGeminiへ接続します。

1. [**SQLクエリを作成**] をクリックして新しいタブを開き、以下のSQLを実行します。
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
BigQuery上で生成AIモデルに接続することで、顧客の声データの分析を行います。

### **1.顧客の声データのテーブル作成**
1. エクスプローラーペインで [**sales_data**] の横にある **︙** をクリックし、続いて [**テーブルを作成**] をクリックします。
2. [**ソース**] の [**テーブルの作成元**] に [**Google Cloud Storage**] を選択します。
3. [**参照**] をクリックして、Cloud Storageから [**customer_voice_data.csv**] ファイルを選択します。
4. [**送信先**] の [**テーブル**] の名前に `customer_voice_data` を入力し、[**スキーマ**] > [**テキストとして編集**] をオンにして下記のように定義します。
```
[
   {
       "name": "store",
       "type": "STRING",
       "mode": "NULLABLE"
   },
   {
       "name": "customer_voice",
       "type": "STRING",
       "mode": "NULLABLE"
   }
]
```
5. [**テーブルを作成**] をクリックします。
6. エクスプローラーペインで [**sales_data**] > [**customer_voice_data**] を選択し、[**プレビュー**] をクリックします。

### **2.顧客の声データの分析**
7. [**クエリ**] > [**新しいタブ**] をクリックし、次のSQLを入力して [**実行**] をクリックします。 
```sql
CREATE or REPLACE TABLE sales_data.customer_voice_category_data AS
SELECT 
  customer_voice,
  store,  
  ml_generate_text_result['candidates'][0]['content']['parts'][0]['text'] as category
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
8. エクスプローラーペインで [**sales_data**] > [**customer_voice_category_data**] を選択し、[**プレビュー**] をクリックします。

生成AIモデルGemini Proを用いた顧客の声データの分析ができました。作成したテーブルをLooker Studioのダッシュボードに追加することも自由に試してみてください。

## Congratulations!
<walkthrough-conclusion-trophy></walkthrough-conclusion-trophy>

<walkthrough-footnote>おめでとうございます！ハンズオンはこれで完了です。ご参加ありがとうございました。</walkthrough-footnote>