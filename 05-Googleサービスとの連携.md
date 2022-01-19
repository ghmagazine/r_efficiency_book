# 5-1 GoogleAPIの利用

# 5-2 Google BigQueryの操作（bigrqueryパッケージ）

## BigQueryの課金体系

## インストールと認証方法

``` r
install.packages("bigrquery")
library(bigrquery)
library(DBI)
```

### インタラクティブな認証方法

### インタラクティブではない認証方法

``` r
bq_auth(path = "/path/to/service_account.json")
```

## 基本操作

### テーブル一覧の表示

``` r
con <- dbConnect(
  bigquery(),
  project = "publicdata",
  dataset = "samples",
  billing = "プロジェクトID"
)

dbListTables(con)
```

    [1] "github_nested"   "github_timeline" "gsod"            "natality"        "shakespeare"     "trigrams"        "wikipedia" 

### テーブル情報の確認

``` r
dbListFields(con, "gsod")
```

     [1] "station_number"                     "wban_number"                        "year"                              
     [4] "month"                              "day"                                "mean_temp"                         
     [7] "num_mean_temp_samples"              "mean_dew_point"                     "num_mean_dew_point_samples"        
    [10] "mean_sealevel_pressure"             "num_mean_sealevel_pressure_samples" "mean_station_pressure"             
    [13] "num_mean_station_pressure_samples"  "mean_visibility"                    "num_mean_visibility_samples"       
    [16] "mean_wind_speed"                    "num_mean_wind_speed_samples"        "max_sustained_wind_speed"          
    [19] "max_gust_wind_speed"                "max_temperature"                    "max_temperature_explicit"          
    [22] "min_temperature"                    "min_temperature_explicit"           "total_precipitation"               
    [25] "snow_depth"                         "fog"                                "rain"                              
    [28] "snow"                               "hail"                               "thunder"                           
    [31] "tornado"       

### どれくらいのデータが読み込まれるか確認

``` r
sql <- "SELECT year, month, day, mean_temp FROM publicdata.samples.gsod limit 100"
billing <- "プロジェクトID"

bq_perform_query_dry_run(
  sql,
  billing
)
```

### クエリの発行

``` r
sql <- "SELECT year, month, day, mean_temp FROM gsod limit 100" # 実行したいSQL文
dbGetQuery(con, sql, n = 10)                                    # SQL文の実行
```

    Complete
    Billed: 3.66 GB
    Downloading 10 rows in 1 pages.
    # A tibble: 10 x 4                                                                                                                                      
        year month   day mean_temp
       <int> <int> <int>     <dbl>
     1  1929    12     6      45.8
     2  1929    12    23      47  
     3  1929    11    27      47.3
     4  1929    10    25      45.5
     5  1929    11    26      44.7
     6  1929     8    30      66.2
     7  1929     9     3      62.7
     8  1929    12     8      49.7
     9  1929    10     5      54.1
    10  1929    11     8      48.5

``` r
df <- head(iris)
sqlAppendTable(con, "iris", df)
```

# 5-3 Google ドライブの操作

## インストールと認証方法

``` r
install.packages("googledrive")
library(googledrive)
```

### インタラクティブな認証方法

``` r
drive_find(n_max = 5)
```

### インタラクティブではない認証方法

``` r
drive_auth(path = "/path/to/service_account.json")
```

``` r
drive_user()
```

    Logged in as:
    • displayName: rbook-googledrive@refficientbook.iam.gserviceaccount.com
    • emailAddress: rbook-googledrive@refficientbook.iam.gserviceaccount.com

## 基本操作

### 一覧表示

``` r
drive_find(n_max = 5)
```

    # A dribble: 5 x 3
       name           id                                           drive_resource   
       <chr>          <drv_id>                                     <list>           
     1 test3.csv      1U6JbIelO32UzqttF8ZwzosVR4erxS5_e            <named list [39]>
     2 test2.csv      1Ot2bnwfvJecSHNA_HKeQQkHDzXBShTok            <named list [39]>
     3 test1.csv      1UEOiwcNqq_iXn7nwWWPOC0OE43vZXB2g            <named list [39]>
     4 csv_files      1O5zMnWOxr86Qx8D4LVdLJC_ND91cjT3Q            <named list [33]>
     5 test_project   1352ytbwk1FfFnCMpBHKBQGaK1kmzGw1W            <named list [33]>

``` r
drive_find(pattern = '*.csv')
```

    # A dribble: 3 × 3           
      name      id                                drive_resource   
      <chr>     <drv_id>                          <list>           
    1 test3.csv 17drTVB4Ngebc74DNZrjr76yO4vh5-iMw <named list [39]>
    2 test2.csv 18f44DRAxu9Y_BqfNy08V4GWi-xoda5Jr <named list [39]>
    3 test1.csv 1u_MCjaLPO46lmg3NNpI_tOL1ncN4gl7Z <named list [39]>

``` r
drive_find(type = "csv")
```

    # A dribble: 3 × 3           
      name      id                                drive_resource   
      <chr>     <drv_id>                          <list>           
    1 test3.csv 17drTVB4Ngebc74DNZrjr76yO4vh5-iMw <named list [39]>
    2 test2.csv 18f44DRAxu9Y_BqfNy08V4GWi-xoda5Jr <named list [39]>
    3 test1.csv 1u_MCjaLPO46lmg3NNpI_tOL1ncN4gl7Z <named list [39]>

``` r
drive_ls(path = "test_project")
```

    # A dribble: 2 × 3
      name      id                                drive_resource   
      <chr>     <drv_id>                          <list>           
    1 dest      1mHguXI02gi2Gq1v394bbipnjFXcY-z2E <named list [33]>
    2 csv_files 17ltumniJVru-phgfHONeqjBJecZg-u9u <named list [33]>

### ファイルの特定

``` r
drive_get(path = "test_project/csv_files/test1.csv")
```

    # A dribble: 1 × 4
      name      path                               id                                drive_resource   
      <chr>     <chr>                              <drv_id>                          <list>           
    1 test1.csv ~/test_project/csv_files/test1.csv 1u_MCjaLPO46lmg3NNpI_tOL1ncN4gl7Z <named list [39]>

### フォルダ作成

``` r
drive_mkdir("dest", path="test_project", overwrite = TRUE)
```

    Created Drive file:
    • dest <id: 1o97JjBghdNQzhP8T10aPUrhRIL8CJiy8>
    With MIME type:
    • application/vnd.google-apps.folder

### ファイルのコピー／移動

``` r
drive_cp(
  "test_project/csv_files/test1.csv", # コピー元ファイル
  path = "test_project/csv_files/",   # コピー先パス
  name = "test4.csv"
)
```

    Original file:
    • test1.csv <id: 1u_MCjaLPO46lmg3NNpI_tOL1ncN4gl7Z>
    Copied to file:
    • csv_files/test4.csv <id: 1EL0v1DHoUzbkMV82WI6bxRY0BnVjPWjp>

``` r
drive_mv("test_project/csv_files/test4.csv", name = "test4.csv", path = "test_project/dest/")
```

    Original file:
    • test1.csv <id: 1UEOiwcNqq_iXn7nwWWPOC0OE43vZXB2g>
    Has been moved:
    • dest/test1.csv <id: 1UEOiwcNqq_iXn7nwWWPOC0OE43vZXB2g>

### ファイルのアップロード

``` r
library(readr)
write_csv(CO2, "CO2.csv")
drive_mkdir("spreadsheets", path = "test_project")
drive_upload(
  "CO2.csv",
  path = "test_project/spreadsheets",
  name = "CO2",
  type = "spreadsheet"
)
```

    Local file:
    • CO2.csv
    Uploaded into Drive file:
    • CO2 <id: 1SiOYtMsZScMv3_su7LXD2LTvIaizakK8MOihWDMyQA8>
    With MIME type:
    • application/vnd.google-apps.spreadsheet

### ファイルのダウンロード

``` r
drive_download(
  "CO2",
  path = "output",
  overwrite = TRUE
)
```

    File downloaded:
    • CO2 <id: 1SiOYtMsZScMv3_su7LXD2LTvIaizakK8MOihWDMyQA8>
    Saved locally as:
    • output.xlsx

### ファイルのシェア

``` r
co2_csv <- drive_get(path = "test_project/csv_files/test1.csv")
test1_csv %>% drive_reveal("permissions")
```

    # A dribble: 1 × 6
      name      shared path                               id                                drive_resource    permissions_resource
      <chr>     <lgl>  <chr>                              <drv_id>                          <list>            <list>              
    1 test1.csv FALSE  ~/test_project/csv_files/test1.csv 1u_MCjaLPO46lmg3NNpI_tOL1ncN4gl7Z <named list [39]> <named list [2]>          

``` r
test1_csv <- test1_csv %>%
  drive_share(
    type = "anyone",  # 権限を付与する対象
    role = "reader",  # 付与する権限
  )
```

    Permissions updated:
    • role = reader
    • type = anyone
    For file:
    • test1.csv <id: 1u_MCjaLPO46lmg3NNpI_tOL1ncN4gl7Z>

``` r
test1_csv %>% drive_reveal("permissions")
```

    # A dribble: 1 × 5
      name      shared id                                drive_resource    permissions_resource
      <chr>     <lgl>  <drv_id>                          <list>            <list>              
    1 test1.csv TRUE   1u_MCjaLPO46lmg3NNpI_tOL1ncN4gl7Z <named list [39]> <named list [2]>

### ファイル削除

``` r
test2_csv <- drive_get("test_project/csv_files/test2.csv")

drive_rm(test2_csv)                                 # dribbleオブジェクトで指定
drive_rm(name = "test3.csv")                        # ファイル名で指定
drive_rm(path = "test_project/csv_files/test4.csv") # パスで指定
```

# 5-4 Google スプレッドシートの操作

## インストールと認証方法

``` r
install.packages("googledrive")
install.packages("googlesheets4")
library(googledrive)
library(googlesheets4)
```

### インタラクティブな認証方法

``` r
drive_auth()
gs4_auth(token = drive_token())
```

### インタラクティブではない認証方法

``` r
key_path <- "/path/to/service_account.json"
drive_auth(path = key_path)
gs4_auth(path = key_path)
```

``` r
gs4_user()
```

    ℹ Logged in to googlesheets4 as rbook-spreadsheet@refficientbook.iam.gserviceaccount.com.

## 基本操作

### スプレッドシートの一覧表示

``` r
gs4_find(n_max = 30)
```

``` r
gs4_find(pattern = 'CO2')
```

    # A dribble: 1 × 3           
      name  id                                           drive_resource   
      <chr> <drv_id>                                     <list>           
    1 CO2   1SiOYtMsZScMv3_su7LXD2LTvIaizakK8MOihWDMyQA8 <named list [37]>

### スプレッドシートの情報を取得

``` r
ss_list <- gs4_find(pattern = 'CO2')
gs4_get(ss_list[1,])
```

    Spreadsheet name: CO2
                  ID: 1SiOYtMsZScMv3_su7LXD2LTvIaizakK8MOihWDMyQA8
              Locale: ja_JP
           Time zone: America/Los_Angeles
         # of sheets: 1

    (Sheet name): (Nominal extent in rows x columns)
         CO2.csv: 1000 x 26

``` r
gs4_get("1SiOYtMsZScMv3_su7LXD2LTvIaizakK8MOihWDMyQA8")
gs4_get("https://docs.google.com/spreadsheets/d/1SiOYtMsZScMv3_su7LXD2LTvIaizakK8MOihWDMyQA8")
```

### スプレッドシートの内容を取得

``` r
ss <- gs4_get("1SiOYtMsZScMv3_su7LXD2LTvIaizakK8MOihWDMyQA8")
read_sheet(ss, range = "A1:C7", sheet = 1)
```

``` r
read_sheet(ss, skip = 5, n_max = 10, col_types = "ccc??")
```

    # A tibble: 10 × 5
       Qn1   Quebec nonchilled `500` `35.3`
       <chr> <chr>  <chr>      <dbl>  <dbl>
     1 Qn1   Quebec nonchilled   675   39.2
     2 Qn1   Quebec nonchilled  1000   39.7
     3 Qn2   Quebec nonchilled    95   13.6
     4 Qn2   Quebec nonchilled   175   27.3
     5 Qn2   Quebec nonchilled   250   37.1
     6 Qn2   Quebec nonchilled   350   41.8
     7 Qn2   Quebec nonchilled   500   40.6
     8 Qn2   Quebec nonchilled   675   41.4
     9 Qn2   Quebec nonchilled  1000   44.3
    10 Qn3   Quebec nonchilled    95   16.2

``` r
ws <- read_sheet(ss, n_max = 5, col_types = "ccc??")
ws %>% dplyr::mutate(Treatment = factor(Treatment))
```

    # A tibble: 5 × 5
      Plant Type   Treatment   conc uptake
      <chr> <chr>  <fct>      <dbl>  <dbl>
    1 Qn1   Quebec nonchilled    95   16  
    2 Qn1   Quebec nonchilled   175   30.4
    3 Qn1   Quebec nonchilled   250   34.8
    4 Qn1   Quebec nonchilled   350   37.2
    5 Qn1   Quebec nonchilled   500   35.3

### 新しいスプレッドシートの作成

``` r
ss <- gs4_create('new_sheet_name', sheets = c("worksheet1", "worksheet2"))
```

### ワークシートの追加/削除

``` r
sheet_add(ss, sheet = 'worksheet3', .after = 'worksheet2')
```

``` r
sheet_delete(ss, sheet = 'worksheet3')
```

### シートへの書き込み

``` r
write_sheet(
  iris,                # 書き込みたいデータフレーム
  ss = ss,             # 書き込むスプレッドシート
  sheet = 'worksheet1' # 書き込むワークシート名
)
```

``` r
range_write(
  ss,                          # 書き込むスプレッドシート
  iris[1:5,],                  # 書き込みたいデータフレーム
  sheet = 'worksheet2',        # 書き込むワークシート
  range = 'worksheet2!A5:E10', # セルの範囲
  col_names = TRUE             # データフレームのカラム名を送信するか
)
```

## googlesheets4パッケージの利用例

### iftttでスプレッドシートにtweetを書き込む

### Rでスプレッドシートからデータを取得

``` r
library(ggplot2)

# データを保存しているシートを読み込む
tweets_ss <- gs4_find(pattern = "tweets")
tweets <- read_sheet(tweets_ss, sheet = 1)

# 棒グラフで日毎のtweet数をプロットする
ggplot(tweets, aes(x = date)) +
  geom_bar() +
  theme_gray(base_family = "HiraKakuPro-W3") +
  theme(axis.text.x = element_text(angle = 90, hjust = 1))
```

# 5-5 まとめ

## 参考文献
