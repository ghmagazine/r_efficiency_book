# 2-1 Excel作業を置き換える意義

# 2-2 Excelファイルを読み込む (readxlパッケージ)

``` r
install.packages("readxl")
```

## ファイルの準備

``` r
getwd()
```

## ファイルの読み込み

``` r
library(readxl)
```

``` r
file.copy(readxl_example("datasets.xlsx"), ".")
```

``` r
normalizePath(".")
```

``` r
read_excel("datasets.xlsx")
```

``` r
# 50行表示
print(read_excel("datasets.xlsx"), n = 50)
# 全ての行を表示
print(read_excel("datasets.xlsx"), n = Inf)
```

``` r
excel_sheets("datasets.xlsx")
```

``` r
read_excel("datasets.xlsx", sheet = "mtcars")
```

``` r
read_excel("datasets.xlsx", sheet = 2)
```

# 2-3 CSV・TSVファイルを読み込む (readrパッケージ)

``` r
library(readr)
```

## ファイルの読み込み

``` r
read_csv("sample.csv")
```

``` r
read_tsv("sample.tsv")
```

# 2-4 Word文書のテーブルを読み込む（docxtractrパッケージ）

``` r
install.packages("docxtractr")
```

``` r
library(docxtractr)
```

## ファイル内のテーブルの読み込み

``` r
file.copy(system.file("examples/data3.docx", package = "docxtractr"), ".")
```

``` r
doc <- read_docx("data3.docx")
```

``` r
docx_tbl_count(doc)
```

``` r
docx_extract_tbl(doc, 1)
```

# 2-5 Excelの代わりにRを使う

## 準備

``` r
install.packages("gapminder")
```

``` r
library(gapminder)
library(tidyverse)
```

## フィルター

``` r
gapminder
```

``` r
gapminder %>% View()
```

``` r
gapminder %>% filter(continent == "Asia")
```

``` r
gapminder %>% filter(country == "Japan")
```

``` r
gapminder %>% filter(country != "Japan")
```

``` r
gapminder %>% filter(country == "Japan", year >= 1982)
```

``` r
gapminder %>% filter(country == "Japan", year >= 1990, year < 2000)
```

``` r
gapminder %>% filter(country == "Japan", year == 1997 | year == 2007)
```

``` r
gapminder %>% filter(country == "Japan", year %in% c(1997, 2007))
```

## 並べ替え

``` r
gap2007 <- gapminder %>% filter(year == 2007)
gap2007
```

``` r
gap2007 %>% arrange(pop)
```

``` r
gap2007 %>% arrange(desc(pop))
```

``` r
gapminder %>% arrange(continent, country, desc(year))
```

## 集計

``` r
1:10 %>% sum()
```

``` r
asia2007 <- gapminder %>% filter(continent == "Asia", year == 2007)
asia2007
```

``` r
asia2007 %>% summarise(sum(pop))
```

``` r
asia2007 %>% summarise(total_pop = sum(pop))
```

``` r
asia2007 %>% summarise(max(pop), min(pop))
```

``` r
gap2007 %>% group_by(continent) %>%
  summarise(n = n(), total_pop = sum(pop))
```

``` r
gapminder %>% filter(year >= 1990) %>%
  group_by(continent, year) %>% summarise(total_pop = sum(pop))
```

## データの結合

``` r
orders <- tibble(item_id = c(3, 1, 2))
items <- tibble(item_id = 1:4, price = c(200, 300, 600, 400))
orders
```

``` r
items
```

``` r
orders %>% left_join(items, by = "item_id")
```

``` r
orders <- tibble(item_id = c(3, 1, 2))
items <- tibble(id = 1:4, price = c(200, 300, 600, 400))
```

``` r
orders %>% left_join(items, by = c("item_id" = "id"))
```

``` r
country_codes
```

``` r
gap2007 %>% left_join(country_codes, by = "country")
```

``` r
x <- tibble(key = c(1, 2, 3), value_x = c(10, 20, 30))
y <- tibble(key = c(1, 2, 4), value_y = c(100, 200, 400))

left_join(x, y)
```

``` r
right_join(x, y)
```

``` r
inner_join(x, y)
```

# 2-6 まとめ
