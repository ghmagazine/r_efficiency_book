# 6-1 スクレイピングによるデータ収集

## なぜRでスクレイピングをするのか

## スクレイピングのためのWeb知識

### HTML

``` html
<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="utf-8">
    <title>ブラウザのタブなどに表示されるページ名</title>
  </head>
  <body>
    Webページを開いた際に見えるコンテンツ
  </body>
</html>
```

``` html
<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="utf-8">
    <title>サンプルHTML</title>
  </head>
  <body>
    <h1>基本的なbodyの要素</h1>
    
    <h2>段落</h2>
    <p>サンプル文字列</p>
    
    <h2>画像</h2>
    <img src="./imgs/sample.jpg" alt="サンプル画像" width="400">
    
    <h2>リスト</h2>
    <ul>
      <li>リスト 1</li>
      <li>リスト 2</li>
    </ul>
    
    <h2>テーブル</h2>
    <table>
      <tr>
        <th>カラム 1</th>
        <th>カラム 2</th>
        <th>カラム 3</th>
      </tr>
      <tr>
        <td>内容 1-1</td>
        <td>内容 1-2</td>
        <td>内容 1-3</td>
      </tr>
      <tr>
        <td>内容 2-1</td>
        <td>内容 2-2</td>
        <td>内容 2-3</td>
      </tr>
    </table>
  </body>
</html>
```

### id属性/class属性

    セレクタ {
      プロパティ: 値;
    }

``` css
table {
  font-size: 14px; # fontサイズを指定
  width: 60%;      # 表のwidthを指定
}
```

### スクレイピングするWebサイトのHTMLの確認方法

#### ① DevToolsを開く

#### ② 目的の要素を探す

## rvestパッケージのインストールと読み込み

``` r
install.packages("rvest")
install.packages("purrr")

library(rvest)
library(purrr)
```

## 基本的な使い方

### HTMLを読み込む

``` r
html <- read_html(url)　# urlに実際のURLを指定する
```

### HTMLから目的の要素を抜き出す

``` r
# HTML要素を抜き出すためのサンプルコード
html <- minimal_html("<p class='sample-class'>サンプルテキスト1</p><p id='sample-id'>サンプルテキスト2</p>")

html %>% html_nodes(".sample-class")  # サンプルテキスト1を抜き出す
html %>% html_nodes("#sample-id")  # サンプルテキスト2を抜き出す
html %>% html_nodes("p") # サンプルテキスト1、サンプルテキスト2両方を抜き出す
```

``` r
html <- minimal_html("<a href='https://gihyo.jp/book'>技術評論社</a>")

html %>% html_nodes("a") %>% html_attr("href") # ここではhref要素のURLを抜き出す
```

### 文字列を抜き出す

``` r
html <- minimal_html("<p>サンプルテキスト</p>")

html %>% html_text() # 「サンプルテキスト」文字列を抜き出す
```

### テーブル要素を抜き出す

``` r
html <- minimal_html("<table><tr><th>カラム1</th><th>カラム2</th><th>カラム3</th></tr><tr><td>内容1-1</td><td>内容1-2</td><td>内容1-3</td></tr><tr><td>内容2-1</td><td>内容2-2</td><td>内容2-3</td></tr></table>")
html %>% html_table()
```

## スクレイピングの実践

### rvestパッケージでスクレイピングする

``` r
# 必要なパッケージの読み込み
library(rvest)
library(purrr)

# 新刊の数だけタイトルなどの情報を抜き出したいため、
# 個々の抜き出したデータを`extract_book_info`という関数に切り出しておく
extract_book_info <- function(node) {
  title <- node %>% html_node("h3") %>% html_node("a") %>% html_text() # タイトル
  author <- node %>% html_node(".author") %>% html_text()              # 著者名
  price <- node %>% html_node(".price") %>% html_text()                # 価格
  publish_at <- node %>% html_node(".sellingdate") %>% html_text()     # 発売日
  
  # タイトルや著者名など必要な情報をリストにして返却
  return(list(title=title, author=author, price=price, publish_at=publish_at))　
}

target_url <- "https://gihyo.jp/book/list"

books <- read_html(target_url) %>%
  html_nodes(".data") %>%
  map_dfr(extract_book_info)
```

``` r
books
```

    ## # A tibble: 25 × 4
    ##    title                         author              price          publish_at  
    ##    <chr>                         <chr>               <chr>          <chr>       
    ##  1 ISOマネジメントシステムが一…  "一般財団法人 日本… 定価1,980円（… 2021年10月2…
    ##  2 大きな字でわかりやすいTwitte… "リンクアップ　著"  定価1,430円（… 2021年10月2…
    ##  3 AutoCAD　パーフェクトガイド…  "芳賀百合　著"      定価3,960円（… 2021年10月2…
    ##  4 PowerPoint［最強］時短仕事術… "井上香緒里　著"    定価1,760円（… 2021年10月2…
    ##  5 オオカミ特許革命　事業と技術… "田所照洋　著"      定価2,948円（… 2021年10月2…
    ##  6 「会社は無理ゲー」な人がノビ… "堀田孝治　著"      定価1,760円（… 2021年10月2…
    ##  7 アートで魅せる数学の世界      "岡本健太郎　著"    定価2,970円（… 2021年10月2…
    ##  8 働き方のデジタルシフト—— リ…  "村上智之，村上健…  定価1,980円（… 2021年10月2…
    ##  9 WEB+DB PRESS Vol.125          ""                  定価1,628円（… 2021年10月2…
    ## 10 一気にビギナー卒業！　動画で… "サンゼ（和田光司…  定価2,970円（… 2021年10月2…
    ## # … with 15 more rows

# 6-2 ブラウザの操作（RSeleniumパッケージ）

## なぜRでブラウザを操作するのか

## Seleniumとは

## インストールとパッケージの読み込み

``` r
install.packages("RSelenium")
library(RSelenium)
```

## Seleniumサーバ

### Seleniumサーバの起動

``` console
docker run -d -p 4444:4444 selenium/standalone-chrome
```

### Seleniumサーバの停止

``` console
docker container ls
```

    CONTAINER ID        IMAGE                        COMMAND                  CREATED             STATUS              PORTS                    NAMES
    e821f5f5872e        selenium/standalone-chrome   "/opt/bin/entry_poin…"   6 seconds ago       Up 5 seconds        0.0.0.0:4444->4444/tcp   modest_jennings

``` console
docker stop modest_jennings
```

### RからSeleniumサーバへの接続

``` r
driver <- remoteDriver(
  remoteServerAddr = "localhost",
  port = 4444,
  browserName = "chrome"
)
```

## コラム：VNCを使用して画面を確認しながら進める

``` docker
docker run -d -p 4444:4444 -p 5900:5900 selenium/standalone-chrome-debug
```

``` r
driver <- remoteDriver(
  remoteServerAddr = "localhost", 
  port = 4444,      
  browserName = "chrome"
)

driver$open()
driver$navigate("https://example.com")
```

## 基本的な使い方

### RSeleniumサーバへの接続

``` r
driver$open()
driver$getStatus()
```

### 画面の遷移

``` r
driver$navigate("https://example.com")
```

``` r
driver$goBack()
driver$goForward()
```

``` r
driver$getCurrentUrl()
```

### 要素の検索

``` r
driver$findElement(using = "id", value = "searchFormKeyword")
```

### テキストボックスでの検索

``` r
# id='search' の要素を見つける
element <- driver$findElement(using = "id", value = "search")
# RStudioという文字列でエンターを押し検索させる
element$sendKeysToElement(list("RStudio", key = "enter"))
```

### スクロール

``` r
driver$navigate("https://note.com")
element <- driver$findElement("tag name", "body")
element$sendKeysToElement(list(key = "end"))
```

### リンクをクリック

``` r
# 選択したい要素にマウスを移動させる
driver$mouseMoveToLocation(webElement = element[[1]]) 
driver$click()
```

``` r
driver$click(2)
```

### エラーの表示

``` r
driver$errorDetails()$message
```

## RSeleniumの実践

``` r
driver <- remoteDriver(
  remoteServerAddr = "localhost",
  port = 4444L,
  browserName = "chrome"
)
driver$open()

# gihyoページに遷移
driver$navigate("https://gihyo.jp/")

# 右上の検索窓で「RStudio」を検索
element <- driver$findElement(using = "id", value = "searchFormKeyword")
element$sendKeysToElement(list("RStudio", key = "enter"))
driver$getCurrentUrl()

# 本のタイトルを取得
title_elems <- driver$findElements(using = "css selector", value = "a.gs-title")
titles <- sapply(title_elems, function(elem){ elem$getElementText() }) %>% .[. != ""]

driver$close()
```

# 6-3 文字列処理（stringrパッケージ）

``` r
library(stringr)
```

## 基本的な使い方

### 文字列の抽出

``` r
str <- "I have 2 dogs and 3 cats."
str %>% str_extract("[0-9]+")
```

    ## [1] "2"

``` r
str <- "I have 2 dogs and 3 cats."
str %>% str_extract_all("[0-9]+")
```

    ## [[1]]
    ## [1] "2" "3"

``` r
str <- "田中花子 著"
str %>% str_sub(start = 1, end = -3)
```

    ## [1] "田中花子"

### 文字列の結合

``` r
str_c("2021", "01", "01", sep = "-")
```

    ## [1] "2021-01-01"

``` r
date_list <- c("2021", "01", "01")
str_c(date_list, collapse = "-")
```

    ## [1] "2021-01-01"

### 不要な文字列を取り除く

``` r
str <- "2,200"
str %>% str_remove(",")
```

    ## [1] "2200"

## stringrの実践

``` r
# 必要なパッケージを読み込み
library(rvest)
library(purrr)
library(stringr)

# 書籍情報を抜き出す関数を定義
extract_book_info <- function(node) {
  # タイトル抽出
  title <- node %>%
    html_node("h3") %>%
    html_node("a") %>%
    html_text()
  #著者名抽出
  author <- node %>%
    html_node(".author") %>%
    html_text() %>%
    str_sub(start = 1, end = -3)
  # 価格抽出
  price <- node %>%
    html_node(".price") %>%
    html_text() %>%
    str_extract("[0-9,]+") %>%
    str_remove(",") %>%
    as.numeric()
  # 出版日抽出
  publish_at <- node %>%
    html_node(".sellingdate") %>%
    html_text() %>%
    str_extract_all("[0-9]+") %>%
    unlist() %>%
    str_c(collapse = "-") %>%
    as.Date()
  
  return(list(title = title, author = author, price = price, publish_at = publish_at))
}

target_url <- "https://gihyo.jp/book/list"

books <- read_html(target_url) %>%
  html_nodes(".data") %>%
  map_df(extract_book_info)
```

``` r
books
```

    ## # A tibble: 25 × 4
    ##    title                                  author                price publish_at
    ##    <chr>                                  <chr>                 <dbl> <date>    
    ##  1 ISOマネジメントシステムが一番わかる    "一般財団法人 日本品…  1980 2021-10-28
    ##  2 大きな字でわかりやすいTwitter ツイッ…  "リンクアップ"         1430 2021-10-28
    ##  3 AutoCAD　パーフェクトガイド［改訂2版］ "芳賀百合"             3960 2021-10-27
    ##  4 PowerPoint［最強］時短仕事術　もう迷…  "井上香緒里"           1760 2021-10-27
    ##  5 オオカミ特許革命　事業と技術を守る真…  "田所照洋"             2948 2021-10-27
    ##  6 「会社は無理ゲー」な人がノビノビ稼ぐ…  "堀田孝治"             1760 2021-10-25
    ##  7 アートで魅せる数学の世界               "岡本健太郎"           2970 2021-10-25
    ##  8 働き方のデジタルシフト—— リモートワー… "村上智之，村上健太…   1980 2021-10-25
    ##  9 WEB+DB PRESS Vol.125                   ""                     1628 2021-10-23
    ## 10 一気にビギナー卒業！　動画でわかるAft… "サンゼ（和田光司）"   2970 2021-10-23
    ## # … with 15 more rows

# 途中でエラーが起こったときのエラーハンドリング

``` r
read_html(url)
```

    Error in open.connection(x, "rb") : Could not resolve host: gihyo.jp

``` r
# 必要なパッケージを読み込み
library(rvest)
library(purrr)
library(stringr)

# HTMLを取得する関数を定義
get_html <- function(url) {
  book_list <- try(read_html(url))

  if (inherits(book_list, "try-error")) {
    error <- paste("an error occured, ", url, sep = "")
    print(error)

    return(minimal_html("<p>error</p>"))
  } else {
    return(book_list)
  }
}

# 書籍情報を抜き出す関数を定義
extract_book_info <- function(node) {
  # タイトル抽出
  title <- node %>%
    html_node("h3") %>%
    html_node("a") %>%
    html_text()
  # 著者名
  author <- node %>%
    html_node(".author") %>%
    html_text() %>%
    str_sub(start = 1, end = -3)
  # 価格抽出
  price <- node %>%
    html_node(".price") %>%
    html_text() %>%
    str_extract("[0-9,]+") %>%
    str_remove(",") %>%
    as.numeric()
  # 出版日抽出
  publish_at <- node %>%
    html_node(".sellingdate") %>%
    html_text()

  return(list(title = title, author = author, price = price, publish_at = publish_at))
}

target_url <- "https://gihyo.jp/book/list"

books <- get_html(target_url) %>%
  html_nodes(".data") %>%
  map_df(extract_book_info)
```

    Error in open.connection(x, "rb") : Could not resolve host: gihyo.jp
    [1] "an error occured, https://gihyo.jp/book/list"

``` r
books
```

    # A tibble: 0 × 0

# 6-4 スクレイピングの作法

## 1. 著作権やWebサイトの利用規約を確認

## 2. クローリングしてもよいページかを確認

``` txt
user-agent: example-crawler
Disallow: /path_a
Allow: /path_b
```

``` txt
user-agent: *
Disallow: /
```

``` html
<meta name="robots" content="noindex" />
```

``` html
<meta name="example-crawler" content="noindex" />
```

## 3. User-Agentの適切な設定

## 4. Webサイト側の負荷の考慮

# 6−5 Rで実践する紳士的なスクレイピング方法（politeパッケージ）

``` r
install.packages("polite")
library(polite)
library(rvest)
```

## 丁寧なスクレイピング

``` r
session <- bow("https://gihyo.jp")
session
```

    <polite session> https://gihyo.jp/
        User-agent: polite R package - https://github.com/dmi3kno/polite
        robots.txt: 8 rules are defined for 3 bots
       Crawl delay: 5 sec
      The path is scrapable for this user-agent

``` r
session <- bow("https://gihyo.jp", user_agent = "custom user-agent & contact details", delay = 3)
```

    <polite session> https://gihyo.jp
        User-agent: custom user-agent & contact details
        robots.txt: 8 rules are defined for 3 bots
       Crawl delay: 3 sec
      The path is scrapable for this user-agent

``` r
session <- nod(session, "https://gihyo.jp/tagList")
session
```

    <polite session> https://gihyo.jp/tagList
        User-agent: polite R package - https://github.com/dmi3kno/polite
        robots.txt: 8 rules are defined for 3 bots
       Crawl delay: 5 sec
      The path is not scrapable for this user-agent

``` r
session <- nod(session, "https://gihyo.jp/")
scrape(session)
```

    {html_document}
    <html xmlns="http://www.w3.org/1999/xhtml" xmlns:og="http://opengraphprotocol.org/schema/" xmlns:fb="http://www.facebook.com/2008/fbml" xml:lang="ja" lang="ja">
    [1] <head>...
    [2] <body itemscope itemtype="http://schema.org/WebPage">...

``` r
session <- nod(session, "https://gihyo.jp/tagList")
scrape(session)
```

    NULL
    Warning message:
    No scraping allowed here! 

``` r
library(polite)
library(rvest)
library(purrr)

extract_book_info <- function(node) {
  title <- node %>% html_node("h3") %>% html_node("a") %>% html_text() # タイトル
  author <- node %>% html_node(".author") %>% html_text()              # 著者名
  price <- node %>% html_node(".price") %>% html_text()                # 価格
  publish_at <- node %>% html_node(".sellingdate") %>% html_text()     # 発売日
  return(list(title=title, author=author, price=price, publish_at=publish_at))
}

target_url <- "https://gihyo.jp/book/list"

# books <- read_html(target_url) %>% html_nodes(".data") %>% map_df(extract_book_info) 元のコード
books <- bow(target_url) %>% scrape() %>% html_nodes(".data") %>% map_df(extract_book_info)
```

# 6-6 まとめ

## 参考文献
