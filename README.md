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
init(path = getwd(), bare = FALSE)
r <- git2r::repository(getwd())
```

``` r
library(github)
# リモートリポジトリをGitHub上に作成
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
# リモートリポジトリに指定
remotes(r)
# character(0)

remote_add(repo = r, name = "origin", url = res$content$clone_url)
remotes(r)
# [1] "origin"
```

一旦RStudioを再起動してgitパネルを有効化する。

### コミットからプッシュまで

``` r
# .gitignore ファイルの編集
devtools::source_gist(id = "b9cf68217c596369721a")
add_ignore()
status(r)
# Untracked files:
#   Untracked:  .gitignore
#   Untracked:  README.Rmd
#   Untracked:  README.md
#   Untracked:  test_git2r.Rproj

remoji::emoji(alias = "hatching_chick")
add(r, list.files(all.files = TRUE, recursive = TRUE))
status(r)
# Staged changes:
#   New:        .gitignore
#   New:        README.Rmd
#   New:        README.md
#   New:        test_git2r.Rproj
git2r::commit(r, message = "Initial commit :hatching_chick:")
# [e1a31ce] 2016-01-14: Initial commit :hatching_chick:

# リモートリポジトリにpush
cred <- cred_user_pass("EMAIL", Sys.getenv("GITHUB_PAT"))
push(r, "origin", "refs/heads/master", credentials = cred)
status(r)
```

以降は、ファイルに変更や管理対象のファイルに操作を加えた場合にコミットをしていく。例えば、README.Rmdを編集した場合、ステータスが変化するのでコミットする。

``` r
status(r)
#> Unstaged changes:
#>  Modified:   README.Rmd
```

``` r
rmarkdown::render("README.Rmd", output_format = "md_document")
```
