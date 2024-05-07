# はじめてのBigQueryハンズオン

## ハンズオンの概要
このラボでは、分析するデータ (CSV ファイル) を Cloud Storage へアップロードして、BigQuery へ Import してから Looker Studio のダッシュボードに表示させます。

データは3つの種類を準備しています。店舗の住所などをまとめた店舗情報データ、店舗で実際に発生した売上の個別データ、そして店舗に寄せられたお客様からの声のデータです。

このラボの内容：
* CSV ファイルを Cloud Storage にアップロードします。
* CSV ファイルを BigQuery へインポートします。
* Looker Studio で売上ダッシュボードを作成します。
* (Optional) BigQuery から生成AIモデル Gemini Pro を呼び出してお客様の声を分析します。

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
<walkthrough-info-message>**Tips:** コードボックスの横にあるボタンをクリックすることで、クリップボードへのコピーおよび Cloud Shell へのコピーが簡単に行えます。</walkthrough-info-message>

## GCS バケットの作成とファイルのアップロード

1. ファイルのアップロード先となる GCS バケットを作成します。
```bash
gcloud storage buckets create gs://${PROJECT_ID}_bigquery_handson --project=$PROJECT_ID --location=us-central1
```

2. 作成した GCS バケットにチュートリアル資材の CSV をアップロードします。
```bash
gcloud storage cp store_data.csv sales_data.csv customer_voice_data.csv gs://${PROJECT_ID}_bigquery_handson
```

ハンズオンに必要な CSV データを GCS バケットにアップロードすることができました。

次に BigQuery への CSV データのインポート方法を学びます。

## BigQuery の Dataset 準備
<walkthrough-tutorial-duration duration=15></walkthrough-tutorial-duration>
ここからはより直感的に理解しやすいよう Cloud Console 上で操作を行います。

まずは BigQuery の Dataset を作成します。

1. ナビゲーションメニュー <walkthrough-nav-menu-icon></walkthrough-nav-menu-icon> から [**BigQuery**] に移動します。
<walkthrough-menu-navigation sectionId="BIGQUERY_SECTION"></walkthrough-menu-navigation>
2. エクスプローラペインの **自身の プロジェクト ID** の右側に表示されている
<walkthrough-spotlight-pointer cssSelector="[instrumentationid=bq-dataset-explorer-resource-list] button.node-context-menu" single="true">**︙ (三点リーダー)**</walkthrough-spotlight-pointer> をクリックし、[**データセットを作成**] を選択します。
3. [**データセットを作成する**] ペインで下記の情報を入力します。

  フィールド  | 値
  ------- | --------
  データセット ID | `bq_handson`
  ロケーションタイプ | リージョン
  データのロケーション | `us-central1`

4. [**データセットを作成**] をクリックします。
5. エクスプローラペインの自身のプロジェクト ID の下に、データセット `bq_handson` が作成されていることを確認します。

## Store data テーブルの作成
次に、作成した Dataset に新しいテーブルを作成します。

1. エクスプローラーペインで `bq_handson` の横にある **︙** をクリックし、続いて [**テーブルを作成**] をクリックします。
2. [**ソース**] の [**テーブルの作成元**] に [**Google Cloud Storage**] を選択します。
3. [**参照**] をクリックして、Cloud Storage から [**store_data.csv**] ファイルを選択します。
4. [**送信先**] の [**テーブル**] の名前に `store_data` を入力します。
5. [**スキーマ**] > [**テキストとして編集**] をオンにして下記のように定義します。
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
        "name": "lat_long",
        "type": "STRING",
        "mode": "NULLABLE"
    },
    {
        "name": "prefecture",
        "type": "STRING",
        "mode": "NULLABLE"
    }
]
```
6. [**テーブルを作成**] をクリックします。

## Sales data テーブルの作成
続けて、もう1つのテーブルを作成します。

1. エクスプローラーペインで `bq_handson` の横にある **︙** をクリックし、続いて [**テーブルを作成**] をクリックします。
2. [**ソース**] の [**テーブルの作成元**] に [**Google Cloud Storage**] を選択します。
3. [**参照**] をクリックして、Cloud Storage から [**sales_data.csv**] ファイルを選択します。
4. [**送信先**] の [**テーブル**] の名前に `sales_data` を入力します。
5. [**スキーマ**] > [**テキストとして編集**] をオンにして下記のように定義します。
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
6. [**テーブルを作成**] をクリックします。

## BigQuery でテーブルのデータを確認
作成した2つのテーブルのデータをプレビューで確認します。

### **1. Store data テーブルのデータを確認する**

1. エクスプローラーペインから **プロジェクト ID** > `bq_handson` > `store_data` テーブルを選択します。
2. <walkthrough-spotlight-pointer cssSelector="[instrumentationid=bq-table-preview-tab]" single="true">[**プレビュー**]</walkthrough-spotlight-pointer> をクリックします。

### **2. Sales data テーブルのデータを確認する**

3. エクスプローラーペインから **プロジェクト ID** > `bq_handson` > `sales_data` テーブルを選択します。
4. [**プレビュー**] をクリックします。

BigQUery へ CSV データをインポートすることができました。

次に BigQuery のデータに対するクエリの実行方法を学びます。

## BigQueryで簡単なクエリを実行
<walkthrough-tutorial-duration duration=15></walkthrough-tutorial-duration>

2つのテーブルのデータを JOIN して、商品カテゴリーごとの売上を集計するクエリを実行します。

1. <walkthrough-spotlight-pointer cssSelector="[instrumentationid=bq-sql-code-editor] button[name=addTabButton]" single="true">[**SQL クエリを作成**] アイコン</walkthrough-spotlight-pointer> をクリックして、新しいタブを開きます。
2. 下記の SQL を入力し、[**実行**] をクリックして実行結果を確認します。
```sql
SELECT
    a.store,
    a.classification as category,
    count(item_name) as sales_number,
    sum(price) as sales_amount
 FROM `bq_handson.sales_data` as a
INNER JOIN `bq_handson.store_data` as b
   ON a.store = b.store
GROUP BY a.store, a.classification
ORDER BY 1,2
```

3. 実行したクエリを保存して、チームへの共有や次回に再利用することができます。 [**保存**] をクリックし、続いて [**クエリを保存**] をクリックします。
4. [**名前**] に `販売サマリー` と入力し、[**保存**] をクリックします。
5. 保存されたクエリはエクスプローラペインの **プロジェクト ID** > [**クエリ**] の下で確認ができます。

## BigQueryでクエリの定期実行を設定
次に、定期的にクエリを実行する スケジュールの作成をします。

1. エクスプローラーペインから **プロジェクト ID** > [**クエリ**] > `販売サマリー` を選択します。
2. クエリを以下のように修正し、[**クエリを保存**] をクリックします。1行目が追加され、実行結果を別テーブルに保存するようにしています。
```sql
CREATE OR REPLACE TABLE `bq_handson.daily_sales_summary` AS
SELECT
    a.store,
    a.classification as category,
    count(item_name) as sales_number,
    sum(price) as sales_amount
 FROM `bq_handson.sales_data` as a
INNER JOIN `bq_handson.store_data` as b
   ON a.store = b.store
GROUP BY a.store, a.classification
ORDER BY 1,2
```
3. [**スケジュール**] をクリックします。
4. **新たにスケジュールされたクエリ** ペインで次のとおり入力します。

フィールド | 値
---------------- | ----------------
クエリの名前 | `日次販売サマリー`
繰り返しの頻度 | 日
時刻 | `01:00`
すぐに開始 | 選択する
ロケーションタイプ | リージョン
リージョン | `us-central1`

5. 他はデフォルトのまま [**保存**] をクリックし、スケジュールを保存します。認証を求められた場合は、ハンズオン用のユーザーを選んで認証します。
6. すぐに開始 を選択したため、エクスプローラーペインの **プロジェクト ID** > `bq_handson` の下に新しいテーブル `daily_items_count_per_subcategory` が作成されていることが確認できます。
7. スケジュールされたクエリの実行結果を、ナビゲーションペインの **スケジュールされたクエリ**  から確認します。

BigQuery のデータに対するクエリの実行方法を学びました。

## (Optional) Gemini in BigQuery の有効化
ここでは、Gemini のアシスタント機能を使用してデータ探索を行う方法を学びます。

まず、Gemini in BigQuery を有効化します。

1. クエリエディタの横にある <walkthrough-spotlight-pointer cssSelector="[instrumentationid=bq-sql-code-editor] button#_1rif_Gemini" single="true">[**Gemini**] アイコン</walkthrough-spotlight-pointer> にマウスカーソルを合わせ、表示されたツールチップの [**続行**] をクリックします。

2. [**Geminiを有効にする**]ペインで、Trusted Tester プログラムに関する利用規約への同意にチェックを入れ、[**次へ**] をクリックします。

3. [**Cloud AI Companion API**] が無効になっている場合は [**有効にする**] をクリックし、[**閉じる**] をクリックします。

## (Optional) Gemini in BigQuery を用いてデータを探索

次に、Gemini を用いて SQL クエリを生成します。

1. <walkthrough-spotlight-pointer cssSelector="[instrumentationid=bq-sql-code-editor] button[name=addTabButton]" single="true">[**SQL クエリを作成**] アイコン</walkthrough-spotlight-pointer> をクリックして、新しいタブを開きます。

2. <walkthrough-spotlight-pointer cssSelector="sqe-duet-trigger-overlay" single="true">[**コーディングをサポート**]アイコン</walkthrough-spotlight-pointer> をクリックするか、Ctrl + Shift + P を入力して、[**コーディングをサポート**]ツールを開きます。

3. 次のプロンプトを入力します。
```
bq_handsonデータセットから、販売金額トップ10の商品とそのカテゴリ、サブカテゴリを調べるクエリを書いて
```

4. [**生成**] をクリックします。
  Gemini は、次のような SQL クエリを生成します。
```terminal
-- bq_handsonデータセットから、販売金額トップ10の商品とそのカテゴリ、サブカテゴリを調べるクエリを書いて
SELECT
    t1.item_name,
    t1.classification,
    t1.sub_classification,
    sum(t1.price) AS total_sales
  FROM
    bq_handson.sales_data AS t1
  GROUP BY 1, 2, 3
ORDER BY
  total_sales DESC
LIMIT 10
```

<walkthrough-info-message>**注:** Gemini は、同じプロンプトに対して異なる SQL クエリを提案する場合があります。</walkthrough-info-message>

3. 生成された SQL クエリを受け入れるには、[**挿入**] をクリックして、クエリエディタにステートメントを挿入します。[**実行**] をクリックして、提案された SQL クエリを実行します。

Gemini in BigQuery のアシスタント機能を学びました。これ以外のプロンプトも自由に試してみてください。

## Looker Studio に BigQuery のデータを追加
<walkthrough-tutorial-duration duration=15></walkthrough-tutorial-duration>

まず、Looker Studio に可視化したい BigQuery のデータを追加します。

1. [Looker Studio](https://lookerstudio.google.com/) にアクセスします。

2. ナビゲーションペインの [**作成**] をクリックし、続いて [**レポート**] をクリックします。

2. アカウントの設定画面が表示された場合は、画面に従って入力してください。

3. [**データに接続**] セクションで [**BigQuery**] をクリックします。
アクセス権を求められた場合は承認してください。
4. マイプロジェクトからデータソースを次のとおり選択します。

フィールド | 値
---------------- | ----------------
プロジェクト | プロジェクトID
データセット | `bq_handson`
表 | `daily_sales_summary`

5. [**追加**] をクリックします。続いて [**レポートに追加**] をクリックします。

## Looker Studioにグラフを追加

続いて、追加したデータを元に Looker Studio にグラフを追加します。

1. メニューバーの[**グラフを追加**] > [**棒**] > [**縦棒グラフ**] をクリックします。
2. 追加されたグラフを選択した状態で、[**グラフ**] ペインでデータの設定をします。

フィールド | 値
---------------- | ----------------
ディメンション | `store`
内部ディメンション | `category`
指標 | (SUM)`sales_amount`

3. 追加されたグラフをドラッグして任意の位置に移動します。

おめでとうございます！これで Looker Studio のダッシュボードを用いた BigQuery のデータの可視化は完了です。他のテーブルやグラフも自由に追加して試してみてください。

## (Optional) BigQueryからVertex AIへの接続を作成
<walkthrough-tutorial-duration duration=15></walkthrough-tutorial-duration>
この項目はスキップしても構いません。BigQuery から Vertex AI への接続方法を学びます。

まず、BigQuery から Vertex AI への接続を作成します。

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

## (Optional) Vertex AIへの接続で用いるサービスアカウントにアクセス権限を付与

次に、作成した接続で用いるサービスアカウントにアクセス権限 (IAM) を付与します。

1. ナビゲーションメニュー <walkthrough-nav-menu-icon></walkthrough-nav-menu-icon> から [**IAMと管理**] > [**IAM**] に移動します。

2. [**アクセス権を付与**] をクリックします。

3. [**新しいプリンシパル**] に、前の手順でコピーしたアカウント ID を入力します。

4. [**ロールを選択**] をクリックし、[**Vertex AI**] > [**Vertex AI ユーザー**] を選択します。

5. [**保存**] をクリックし、アクセス権を付与します。

## (Optional) BigQueryから生成AIモデルGeminiへ接続
続いて、作成した接続を用いて BigQuery から生成 AI モデル Gemini Pro へ接続します。

1. ナビゲーションメニュー <walkthrough-nav-menu-icon></walkthrough-nav-menu-icon> から [**BigQuery**] に移動します。

1. Vertex AI を有効化するため、次のコマンドを実行します。

```bash
gcloud services enable aiplatform.googleapis.com
```

1. [**SQLクエリを作成**] をクリックして新しいタブを開き、以下の SQL を実行します。
```sql
CREATE OR REPLACE MODEL bq_handson.gemini_pro
  REMOTE WITH CONNECTION `us-central1.gemini-connect`
  OPTIONS(ENDPOINT = 'gemini-pro')
```

2. 同じタブで以下の SQL を実行し、Gemini Pro からのレスポンスを確認します。
```sql
SELECT ml_generate_text_result as response
FROM ML.GENERATE_TEXT(
    MODEL bq_handson.gemini_pro,
    (SELECT 'Google Cloud Nextについて教えてください' AS prompt),
    STRUCT(1000 as max_output_tokens, 0.2 as temperature)
  )
```

Gemini Pro からのレスポンスが json 型であることを確認します。
BigQuery では json 型のデータを次のようなクエリで展開することが可能です。
```sql
SELECT JSON_VALUE(ml_generate_text_result.candidates[0].content.parts[0].text) as response
FROM ML.GENERATE_TEXT(
    MODEL bq_handson.gemini_pro,
    (SELECT 'Google Cloud Nextについて教えてください' AS prompt),
    STRUCT(1000 as max_output_tokens, 0.2 as temperature)
  )
```

## (Optional) 顧客の声データを生成AIを用いて分析
BigQuery 上で生成 AI モデルに接続することで、顧客の声データの分析を行います。

### **1.顧客の声データのテーブル作成**
1. エクスプローラーペインで `bq_handson` の横にある **︙** をクリックし、続いて [**テーブルを作成**] をクリックします。
2. [**ソース**] の [**テーブルの作成元**] に [**Google Cloud Storage**] を選択します。
3. [**参照**] をクリックして、Cloud Storage から [**customer_voice_data.csv**] ファイルを選択します。
4. [**送信先**] の [**テーブル**] の名前に `customer_voice_data` を入力します。
5. [**スキーマ**] > [**テキストとして編集**] をオンにして下記のように定義します。
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
6. [**テーブルを作成**] をクリックします。
7. エクスプローラーペインで `bq_handson` > `customer_voice_data` を選択し、[**プレビュー**] をクリックします。

### **2.顧客の声データの分析**
7. [**クエリ**] > [**新しいタブ**] をクリックし、次の SQL を入力して [**実行**] をクリックします。 
```sql
CREATE or REPLACE TABLE bq_handson.customer_voice_category_data AS
SELECT 
  customer_voice,
  store,  
  JSON_VALUE(ml_generate_text_result.candidates[0].content.parts[0].text) as category
FROM ML.GENERATE_TEXT(
    MODEL bq_handson.gemini_pro,
    (SELECT 
      CONCAT(
        '次の[顧客の声]を分類して、[出力形式]に従って出力してください。¥n¥n',
        '[出力形式]¥n商品・品揃え、価格、スタッフ、店舗環境、その他、のいずれか1つをプレーンテキストで出力。余計な情報は付加しないこと。¥n¥n',
        '[顧客の声]¥n',
        customer_voice
     ) AS prompt,
     customer_voice,
     store
     FROM `bq_handson.customer_voice_data`),
    STRUCT(1000 as max_output_tokens, 0.0 as temperature)
  )
```
8. エクスプローラーペインで `bq_handson` > `customer_voice_category_data` を選択し、[**プレビュー**] をクリックします。

生成AIモデル Gemini Pro を用いた顧客の声データの分析ができました。作成したテーブルを Looker Studio のダッシュボードに追加することも自由に試してみてください。

## Congratulations!
<walkthrough-conclusion-trophy></walkthrough-conclusion-trophy>

おめでとうございます！ハンズオンはこれで完了です。ご参加ありがとうございました。