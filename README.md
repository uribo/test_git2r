<!-- README.md is generated from README.Rmd. Please edit that file -->
RとGitおよびGitHubを結びつけるパッケージ
========================================

Rパッケージ
-----------

-   **`{git2r}`**
-   **`{github}`**
-   **`{gistr}`**

作業の流れ
----------

### gitリポジトリを作成

1.  **`{git2r}`**あるいはRStudioの新規プロジェクト作成機能で**ローカル**
2.  GitHub

``` r
library(git2r)
(getwd())
#> [1] "/Users/uri/git/test_git2r"
```

``` r
# 既存のディレクトリ内にリポジトリを新規作成
r <- init(path = getwd(), bare = FALSE)
```

``` r
library(github)
res <- github::create.repository(name = "test_git2r", 
                          description = "test R and Git, GitHub integration", 
                          private = FALSE)
# $ok
# [1] TRUE
# 
# $content
# $content$id
# [1] 49580376
# 
# $content$name
# [1] "test_git2r"
# 
# $content$full_name
# [1] "uribo/test_git2r"
```

``` r
rmarkdown::render("README.Rmd", output_format = "md_document")
```
