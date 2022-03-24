# 4-1 コードに実行結果と説明文をつけて文書化する (R Markdown)

## テンプレートファイルをHTML文書に出力してみる

### マウス操作で直感的にテキストの見た目を変えるVisual editorを有効化する

### Rmdファイルを新規作成する

### Rmdファイル中のコードをエディタ上で実行してみる

### HTMLファイルに変換してみる

``` r
# RmdファイルをHTMLファイルなどに変換する
rmarkdown::render("cars.Rmd")
```

# コラム：Rmdファイルの変換工程

# 4-2 本文を書く

## 段落と文字の修飾

## 見出し

## 箇条書き

## その他の書式

## キーボードショートカット

## Markdown記法

# 4-3 チャンクによるコードとその実行結果の挿入

### 必要なパッケージやデータの確実な読み込み

``` md
{r setup}
# setupチャンクを用いてパッケージやデータを読み込む
library(tidyverse)
mydata <- readr::read_csv("mydata.csv")
```

### チャンクオプション

``` md
{r, echo=FALSE, fig.width=3, fig.height=3}
# コードを隠し、グラフのサイズを3インチ四方に変更
plot(1, 1)
```

``` md
{r, include=FALSE}
fw <- fh <- 3
```

``` md
{r, results='hide', fig.width=fw, fig.height=fh}
# コードを隠し、グラフのサイズを3インチ四方に変更
plot(1, 1)
```

``` md
{r, include=FALSE}
# メッセージと警告を永続的に非表示にする
knitr::opts_chunk$set(message = FALSE, warning = FALSE)
```

``` md
{r, include=FALSE}
# メッセージの非表示
knitr::opts_chunk$set(message = FALSE)
```

``` md
{r setup}
# 図のデフォルトサイズ変更とデータの読み込み
knitr::opts_chunk$set(fig.width = 5, fig.height = 5)
foo_df <- readr::read_csv("foo.csv")
```

## 表組

### 手軽な表組み（`knitr::kable`関数）

``` md
{r, echo=FALSE}
# 表の出力
knitr::kable(cars[1:2, ])
```

``` r
# 鉱物の化学組成の表組み

# 元となるデータフレーム
minerals <- tibble::tribble(
  ~ 酸化物,    ~ 石英, ~ 紅柱石,
  "SiO~2~",       100,    37.08,    
  "Al~2~O~3~",      0,    62.92,
  "Total",        100,   100
)

# データフレームを表に出力
knitr::kable(
  minerals,
  format.args = list(
    nsmall = 2L # 数字を小数点2桁まで表示する
  )
)
```

``` r
# 「Al2O3」を「Al~2~O~3~」に置換する
stringr::str_replace_all(
  "Al2O3",   # 置換対象の文字列
  "([0-9])", # 数字1字を1番目のキャプチャグループに記録
  "~\\1~"    # マッチした数字を波括弧で挟む
)
```

``` r
# 鉱物の化学組成の表組み
# Markdown記法をminerals変数から分離する例
# （結果省略）

library(magrittr)

# 元となるデータフレーム
minerals <- tibble::tribble(
  ~ 酸化物, ~ 石英, ~ 紅柱石,
  "SiO2",      100,    37.08,    
  "Al2O3",       0,    62.92,
  "Total",     100,   100
)

# データフレームを表に出力
minerals %>%
  # 数字をMarkdown記法を用いて下付き文字での表現に置換する
  dplyr::mutate(
    `酸化物` = stringr::str_replace_all(`酸化物`, "([0-9])", "~\\1~")
  ) %>%
  knitr::kable(
    format.args = list(
      nsmall = 2L # 数字を小数点2桁まで表示する
    )
  )
```

### 凝った表組み（flextableパッケージとftExtraパッケージ）

``` r
# flextableによる表の作成
flextable::flextable(head(iris, 3)) %>%
  # テーマの設定
  ## 他にbooktabs（既定）、alafoli、zebraなど
  flextable::theme_box() %>%
  # キャプションの設定
  flextable::set_caption('アヤメのサイズ') %>%
  # 脚注の設定
  flextable::footnote(
    # 脚注を付けるセルの選択
    i = 1, j = 'Species',
    # 注釈文
    value = flextable::as_paragraph('ヒオウギアヤメ')
  ) %>%
  # フォントサイズを全体的に変更
  ## 脚注と同様にセル単位でも変更できる
  flextable::fontsize(size = 16, part = 'all') %>%
  # 幅の自動調整
  ## 手動で行うにはwidth関数を用いる
  flextable::autofit()
```

``` r
# データフレームを表に出力する
minerals %>%
  # 数字をMarkdown記法を用いて下付き文字での表現に置換する
  dplyr::mutate(
    `酸化物` = stringr::str_replace_all(`酸化物`, "([1-9])", "~\\1~")
  ) %>%
  # 表に出力する
  ftExtra::as_flextable() %>%
  # 数値を小数点2桁で揃える
  flextable::colformat_num(digits = 2) %>%
  # Markdown記法を書式に反映する
  ftExtra::colformat_md()
```

``` r
iris[1:3, ] %>%
  ftExtra::as_flextable() %>%
  ftExtra::separate_header(sep = '\\.')
```

## グラフの描写

### 複数のグラフの描写

``` md
{r}
# irisの数値型の列のヒストグラムを4つ描写
hist(iris$Sepal.Length)
hist(iris$Sepal.Width)
hist(iris$Petal.Length)
hist(iris$Petal.Width)
```

``` md
{r}
# for文でirisの数値型の列のヒストグラムを4つ描写
for(i in names(iris)[-5]) {
  hist(iris[[i]])
}
```

``` md
{r}
# for文とggplotパッケージでirisの数値型の列のヒストグラムを4つ描写
library(ggplot2)

# 散布図を描写
# print関数を省略すると描写されない
for(nm in names(iris)[-5]) {
  print(
    ggplot(iris) +
      aes(.data[[nm]]) +
      geom_histogram()
  )
}
```

### サイズ変更

``` md
{r, fig.width = 3, fig.height = 3}
plot(1)
```

``` r
cm2in <- function(x) x / 2.54
```

### キャプションの付与

``` md
{r, fig.cap='指数関数$y = \\exp(x)$のグラフ'}
# 指数関数のグラフ
plot(exp)
```

### フォントの自由な使用

``` md
{r, include=FALSE}
# グラフ描写時にフォントを自由に使えるようにする
knitr::opts_chunk$set(dev = "agg_png")
```

``` md
{r setup, include=FALSE}
# ggplot2のテーマのフォントを変更
ggplot2::theme_set(ggplot2theme_gray(base_family = "IPAex Mincho"))

# geom_text関数とgeom_label関数のフォントを変更
ggplot2::update_geom_defaults("text", list(family = "IPAex Gothic"))
ggplot2::update_geom_defaults("label", list(family = "IPAex Gothic"))
```

## 画像の挿入

``` md
# 特定のディレクトリ内にあるすべてのPNGファイルを挿入する
knitr::include_graphics(
  dir("path/to/directory", pattern = "\\.png$")
)
```

## 処理をキャッシュする

## 他のRmdファイルの挿入

``` md
{r, child="ichiro.Rmd"}
# childを持つチャンクの例
# このチャンク内のコードは無視されます
```

``` md
{r, child=c("ichiro.Rmd", "jiro.Rmd")}
# childに複数の子Rmdを指定した例
```

``` yaml
---
# YAMLフロントマターの例
title: 表題
author: 著者
date: 2021-02-23
output: html_document
---
```

## 出力形式の指定

``` yaml
output: html_document
```

``` yaml
output: revealjs::revealjs_presentation
```

``` yaml
output:
  html_document:
    toc: TRUE # 目次を表示
```

### 段落内の改行の半角スペース化を防ぐ

``` yaml
# 絵文字の利用と改行コードの無視
output:
  html_document:
    md_extensions: +hard_line_breaks
```

### 複数の出力形式を指定

``` yaml
# 出力形式にHTML文書とWord文書を指定する例
output:
  html_document: default
  word_document:
    md_extensions: "+ignore_line_breaks"
```

# 4-5 HTML文書を作成する

``` yaml
output: html_document
```

## 目次の追加や見た目の変更

``` yaml
# html_document関数に引数を指定した例
# 各引数の既定値はコメント末尾の括弧内に記した
output:
  html_document:
    # 目次と見出しに関するもの
    toc: TRUE             # 目次を追加 (FALSE)
    toc_depth: 3          # 見出しレベル3まで目次化 (3)
    toc_float: TRUE       # スクロールに合わせて目次を移動 (FALSE)
    number_sections: TRUE # 見出しと目次の各項に番号を付ける（FALSE）
    # 見た目に関するもの
    highlight: monochrome # シンタックスハイライトを利用する (default)
    theme: yeti           # テーマを設定する (default)
    # その他
    code_folding: hide    # コードを折り畳んで隠す (none)
    md_extensions: "+ignore_line_breaks" # 拡張機能を指定 (NULL)
```

## 操作可能なグラフや表の出力

### 検索やページ分けができる表を出力

``` r
DT::datatable(
  mtcars,
  filter = 'top', # 各列の上部に検索窓を配置
  options = list(
    pageLength = 3, # ページごとに表示する行数を指定
    scrollX = TRUE  # 列数に合わせて横スクロールを有効化
  )
)
```

``` r
# reactableパッケージによる表の出力例
reactable::reactable(
  mtcars,
  filterable = TRUE,  # 列ごとに検索を有効化
  defaultPageSize = 3 # ページごとの行数の上限を設定
)
```

### 拡大縮小や移動が可能な地図を出力

``` r
# leafletパッケージで日本のへそを地図に表示
leaflet::leaflet() %>%    # leafletを開始。ggplot2::ggplot関数相当
  leaflet::addTiles() %>% # 地図を追加。ggplot2::geom関数群相当
  leaflet::setView(       # 表示箇所を指定。ggplot2::coord関数群に近い
  # 経度       緯度      倍率
    lng = 135, lat = 35, zoom = 16
  )
```

``` r
# 日本のへそに「日本のへそ」のふきだしを表示する
data.frame(name = '日本のへそ', lng = 135, lat = 35) %>%
  leaflet::leaflet() %>%
  leaflet::addTiles( # 国土地理院地図の表示
    "https://cyberjapandata.gsi.go.jp/xyz/std/{z}/{x}/{y}.png"
  ) %>%
  leaflet::addPopups( # 指定座標にふきだしを表示
    lng = ~ lng, lat = ~ lat, popup = ~ name
  ) %>%
  leaflet::setView( # 地図の中心座標と拡大率を指定
    lng = 135, lat = 35, zoom = 16
  )
```

# 4-6 HTMLスライドを作成する（revealjsパッケージ）

``` yaml
output: revealjs::revealjs_presentation
```

## 改ページ

### ページ数が多いスライドを二次元に並べて構造化

### ページ数が少ないスライドを一次元に並べる

## 見た目の調整

``` yaml
# revealjs::revealjs_presentation関数に引数を指定した例
# 各引数の既定値はコメント末尾の括弧内に記した
output:
  revealjs::revealjs_presentation:
    center: TRUE         # 内容を上下に中央揃えする (FALSE)
    slide_level: 2       # スライドを2次元的に配置する (2)
    theme: sky           # テーマを設定する (simple)
    highlight: default   # シンタックスハイライトを利用する (default)
```

## ページ番号を付与したPDF化

``` yaml
output:
  revealjs::revealjs_presentation:
    reveal_options:
      slideNumber: TRUE
```

# 4-7 Word文書の作成

``` md
# 引数を指定しない場合
output: word_document
```

## 目次の追加やテンプレートの変更

``` yaml
# word_document関数に引数を指定した例
# 各引数の既定値はコメント末尾の括弧内に記した
output:
  word_document:
    toc: TRUE            # 目次を追加 (FALSE)
    toc_depth: 3         # 見出しレベル3まで目次化 (3)
    highlight: tango     # シンタックスハイライトを利用する (default)
    reference_docx: NULL # テンプレートの指定 (NULL)
    md_extensions:  "+ignore_line_breaks" # 拡張機能を指定
```

## テンプレートの利用

``` yaml
# Word文書のテンプレートを指定する
output:
  word_document:
    reference_docx: reference.docx
```

## テンプレートの作成

``` yaml
output:
  word_document:
    reference_docx: NULL
```

## テンプレートの更新

### 見出しや段落などの文字列のスタイル変更

### ページサイズなどのレイアウト変更

#### ヘッダー・フッター・ページ番号の挿入

## 改ページ

``` md
---
output: word_document
---


次の段落から新しいページにする。

\newpage

ここから新しいページが始まる。
```

## Word文書作成の効率化

``` md
---
title: officedownパッケージを用いたWord文書の作成
output: officedown::rdocx_document
---

前書き

\pagebreak

<!---BLOCK_TOC--->
```

``` md
<!---BLOCK_LANDSCAPE_START--->

```md
{r, echo=FALSE, fig.width=10, fig.height=3}
# 横長のヒストグラム
hist(rnorm(1000))
```

``` md
<!---BLOCK_LANDSCAPE_STOP--->
```

``` md
{r, echo=FALSE}
officer::block_section(officer::prop_section(type = "nextPage"))
```

``` md
{r, echo=FALSE}
block_pour_docx("example.docx")
```

# 4-8 相互参照可能なHTML文書やWord文書の作成

``` yaml
# bookdownパッケージ由来のフォーマットを利用する
output: bookdown::html_document2
```

# 4-9 図表の相互参照

## ラベルをFigureから変更

``` yaml
language:
  label:
    fig: "図"
    tab: "表"
```

``` yaml
output:
  officedown::rdocx_document:
    plots:
      caption:
        pre: "図"
    tables:
      caption:
        pre: "表"
```

# 4-10 その他の形式の文書やスライドを作成する

# 4-11 まとめ

## 参考資料
