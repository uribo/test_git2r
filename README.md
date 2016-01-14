<!-- README.md is generated from README.Rmd. Please edit that file -->
RとGitおよびGitHubを結びつけるパッケージ
========================================

Rパッケージ
-----------

-   [**`{git2r}`**](https://github.com/ropensci/git2r)
-   [**`{github}`**](https://github.com/cscheid/rgithub)
-   [**`{gistr}`**](https://github.com/ropensci/gistr)
-   [**`{gh}`**](https://github.com/gaborcsardi/gh)

作業の流れ
----------

### gitリポジトリを作成

1.  **`{git2r}`**あるいはRStudioの新規プロジェクト作成機能で**ローカル**に作成
2.  GitHubからクローン

``` r
library(git2r)
(getwd())
#> [1] "/Users/uri/git/test_git2r"
cred <- cred_user_pass("EMAIL", Sys.getenv("GITHUB_PAT"))
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
push(r, "origin", "refs/heads/master", credentials = cred)
status(r)
```

以降は、ファイルに変更や管理対象のファイルに操作を加えた場合にコミットをしていく。例えば、README.Rmdを編集した場合、ステータスが変化するのでコミットする。

``` r
status(r)
# Unstaged changes:
#   Modified:   README.Rmd
#   Modified:   README.md
add(r, path = list.files(pattern = "^README"))
status(r)
# Staged changes:
#   Modified:   README.Rmd
#   Modified:   README.md
git2r::commit(r, message = "Upgrade commit example")
push(r, "origin", "refs/heads/master", credentials = cred)
```

### ブランチの作成

``` r
# 現在のブランチ一覧

# $master
# [ac0ece] (Local) (HEAD) master
# 
# $`origin/master`
# [ac0ece] (origin @ https://github.com/uribo/test_git2r.git) master

# test/create_branchというブランチへcheckout。create引数をTRUEにして作成も同時に行う
git2r::checkout(r, branch = "test/create_branch", create = TRUE)
#   HEADが先ほど作成したブランチにきていることを確認
git2r::branches(r, flags = "local")
# $master
# [ac0ece] (Local) master
# 
# $`test/create_branch`
# [ac0ece] (Local) (HEAD) test/create_branch

# ファイルをコミットする
git2r::add(r, list.files(pattern = "^README"))
git2r::commit(r, message = "Upgrade README\nCreate branch")
git2r::push(r, "origin", "refs/heads/test/create_branch", credentials = cred)
```

### プルリクエストの発行

``` r
# ref) https://developer.github.com/v3/pulls/#create-a-pull-request
gh::gh("POST /repos/uribo/test_git2r/pulls",
       title = "ブランチの作成について追加",
       head  = "uribo:test/create_branch",
       base = "master",
       body = "このプルリクエストは{gh}パッケージを使ってAPI経由で発行したもの")
```

### ブランチのマージ

``` r
# ref) https://developer.github.com/v3/pulls/#merge-a-pull-request-merge-button
gh::gh("PUT /repos/:owner/:repo/pulls/:number/merge",
       owner = "uribo",
       repo  = "test_git2r",
       number = 2,
       commit_message = "{gh}パッケージからmerge! :muscle:")
```

### リモートリポジトリの変更を反映させる

プルリクエストにより、リモートリポジトリに変更が加えられた場合、ローカルリポジトリとの間に差が生じる。そのため更新情報をリモートリポジトリに反映させる必要がある。

> git pull origin master

``` r
# うまくいかない?
# masterブランチに切り替え。変更が加えられていて、未コミットのファイルがあるとcheckoutできないので注意
git2r::checkout(r, "master")
git2r::fetch(r, name = "origin", credentials = cred)
# [new]     7a071fde0521bed4d6e7 refs/remotes/origin/master
pull(r, credentials = cred)
```

GitHub issuesの活用
-------------------

ref) <https://github.com/Cucurbitaceae/cucumber-flesh/issues/1>

**`{github}`**あるいは**`{gh}`**パッケージとGitHub APIを使って行う。**`{github}`**ではAPIの種類に応じて関数が用意されており、 **`{gh}`**では`gh()`関数内の*endpoint*引数でAPIを指定する。

``` r
library(github)
create.issue(owner = "uribo", repo = "cucumber_flesh",
             content = list(title = "Create issue from {github} package", 
                            body = ":package: {github}パッケージを使ってGithub issuesの作成\n改行はどうなるか :smile_cat:"))
```

``` r
library(gh)
# Sys.setenv("GITHUB_TOKEN") でtokenを与えていない場合、.token引数で指定する。
gh("POST /repos/Cucurbitaceae/cucumber-flesh/issues", 
   title = "Create issue to employ {gh} package",
   body = ":package: {github}パッケージからもissuesを作成可能。\n参考#1",
   labels = array("sandbox"),
   .token = "<Personal Access Token>")

gh("PATCH /repos/:owner/:repo/issues/:number",
   owner = "Cucurbitaceae",
   repo  = "cucumber-flesh",
   number = "3",
   state = "closed")

gh(endpoint = "GET /users/:username/repos", username = "uribo")
```

``` r
rmarkdown::render("README.Rmd", output_format = "md_document")
```
