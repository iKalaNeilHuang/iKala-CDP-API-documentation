iKala CDP API
===

API documentation: https://ikala-data-lake.github.io/iKala-CDP-API-documentation

## 流程簡介
user_profiles 與 tagging 上傳流程簡介：一開始需要先用 Create application token 來建立一個「CSV上傳任務」的概念，建立 ingestion 後會取得 put_url，將 CSV 檔案透過 put 的方式上傳到該 put_url，上傳完成後再呼叫  Notify that data is ready for process 通知伺服器將 CSV 檔案的內容傳入 CDP，最後可以透過 Get details of data ingestion 來了解 CSV 傳入 CDP 的進度，當 status 為 `succeeded` 時，代表整個上傳作業完成。

## API 列表

API url：https://api.ikala-c4m.io

|API|URL|Description|
|-|-|-|
| Create application token| POST `/v1/app/token`| 使用 client_id 與 client_secret 取得 token，夾帶在其他 API 的 header 中來通過認證|
| Create data ungestion| POST `/v1/app/user_data/ingestions` |建立一個上傳任務，接著您必須自行完成上傳 CSV 的步驟，將檔案上傳到 `pur_url`|
| Notify that data is ready for process| POST `/v1/app/user_data/ingestions/{id}/data_ready`| 當 CSV 上傳完成後，使用此API主動通知伺服器將 CSV 檔案內容傳入 CDP|
| Get details of data ingestion|GET `/v1/app/user_data/ingestions/{id}`| Ingestion 目前狀態與 meta data，status 值包含 `created`, `uploaded`, `validating`, `validated`, `processing`, `succeeded`, `failed`|

## 如何使用 put 的方式上傳 CSV

在 expired_at（即創建日期的一天後）之前上傳CSV，不然 put_url 會失效，可以使用任何 HTTP client 上傳您的 CSV 並指定 Content-Type。下面是一個 curl 範例：

`curl -X PUT -H 'Content-Type: text/csv' --data-binary @"file.csv" 'PUR_URL_HERE'`

如果上傳成功，您將會收到 HTTP 200 response，在 x-goog-stored-content-length header 能知道檔案實際的byte數。

### Tagging CSV field

[check tagging CSV template](./tagging_CSV_template.csv)

\* means required field

|Name|Description|Type|
|-|-|-|
|user_id*|primary key，這跟要跟GA的user_id一樣|UUID|
|tag_name*| 標籤名稱, ex: 3c | STRING (URF-8 encoding), ^[0-9a-z\-_]+$|

### User profiles CSV field

[check user profiles CSV template](./user_profiles_CSV_template.csv)

\* means required field

|Name|Description|Type|
|-|-|-|
|user_id*|primary key，這跟要跟GA的user_id一樣|UUID|
|email*|電子郵件|String|
|phone_number*|電話號碼 (0912-345-678 或 0912345678)|String|
|gender|- 保留 (rather_not_say)<br> - 男 (male)<br> - 女 (female) |String|
|status| 帳號狀態<br> - 未驗證 (pending_activation)<br> - 註冊 (active)<br> - 註銷 (deleted)<br> - 鎖定中 (disabled) |String|
|register_timestamp|註冊時間|Timestamp(RFC3339)|
|birthday|YYYY-mm-dd|Date|
|is_dm|是否接收型錄|Boolean|
|is_edm|是否接收電子型錄|Boolean|
|is_email|是否接收電子報|Boolean|
|is_sms|是否接收簡訊通知|Boolean|
|is_call_in|是否接收外撥|Boolean|
|post_code|居住的郵遞區號(111或11163)|String|
|country|居住的國家|String|
|region|居住的地區|String|
|city|居住的城市|String|
|first_purchase_timestamp|第一次溝買的時間戳記|Timestamp(RFC3339)|
|job_industry|產業別(techolongy etc)|String|
|job_title|職稱(teacher etc)|String|
|register_medium|註冊媒介，例如：(facebook, brick-and-mortar etc) |String|
