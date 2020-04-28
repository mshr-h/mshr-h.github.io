---
title: "Hugo+Github Pagesで新しい個人ウェブサイトを作った"
slug: "new-website-with-hugo"
subtitle:    ""
description: ""
date:        "2020-05-02"
author:      "Masahiro Hiramori"
image:       ""
tags:        ["Hugo"]
categories:  ["Tech" ]
draft:       false
---

HugoとGithub Pagesで個人ブログを作ったので作業メモ。
macOS Catalina 10.15.4で作業した。

# 手順

- GitHub repositoryを作成
- Hugoでサイト作成
- GitHub Pagesで公開
- 独自ドメイン設定
- 記事を追加

## GitHub repositoryを作成

GitHubで新しくリポジトリ(例えば`blog`という名前)を作成する。このリポジトリにはHugoのプロジェクトを配置する。プライベートリポジトリでも可。
次に、`username.github.io`リポジトリを作成する。ここにはHugoで生成したサイトのファイルを配置するため、パブリックリポジトリとして作成する必要がある。

- [mshr-h/mshr-h.github.io](https://github.com/mshr-h/mshr-h.github.io)

```
$ hugo new site blog
$ cd blog
$ git submodule add https://github.com/zhaohuabing/hugo-theme-cleanwhite themes/hugo-theme-cleanwhite
$ cp -r themes/beautifulhugo/exampleSite/* .
$ hugo server -D
```


## Hugoでサイト作成

Hugoをインストールする。

```bash
$ brew isntall hugo
```

作業ディレクトリに移動し、`hugo`コマンドで新規プロジェクトを作成する。

```bash
$ cd ~/workspace
$ hugo new site blog
$ cd blog
```

テーマをインストールする。今回は[Clean White](https://themes.gohugo.io/hugo-theme-cleanwhite)を選んだ。テーマはsubmoduleとして追加する。

```bash
$ git init
$ git submodule add https://github.com/zhaohuabing/hugo-theme-cleanwhite.git themes/hugo-theme-cleanwhite
```

テーマが用意してくれているサンプルページをベースに構築する。サンプルページをコピーする。

```bash
$ cd ~/workspace/blog
$ cp -r themes/hugo-theme-cleanwhite/exampleSite/* .
```

設定ファルを編集する。

```bash
$ vim config.toml
```

`baseurl`は独自ドメインを使わない場合`username.github.io`、使う場合はそのドメインを指定する。`title`はブログのタイトルを、`theme`はテーマ名(`hugo-theme-cleanwhite`)を指定する。

```toml
baseurl = "https://keepcodingkeepclimbing.com/"
title = "Keep Coding, Keep Climbing"
theme = "hugo-theme-cleanwhite"
languageCode = "ja-jp"
googleAnalytics = ""
paginate = 8 #frontpage pagination
hasCJKLanguage = true

[outputs]
home = ["HTML", "RSS", "Algolia"]

[params]
header_image = "img/home-bg-jeep.jpg"
SEOTitle = "Keep Coding, Keep Climbing"
description = "programming and rock climbing"
keyword = "programming, software, machine learning, climbing, bouldering, blog"
slogan = "programming and rock climbing"
image_404 = "img/404-bg.jpg"
title_404 = "404 Not Found"
omit_categories = false
# algolia site search
algolia_search = true
algolia_appId = ""
algolia_indexName = ""
algolia_apiKey = ""
# Sidebar settings
sidebar_about_description = "Edge deep learning research engineer, open source enthusiast and rock climber"
sidebar_avatar = "img/avatar.jpg" # use absolute URL, seeing it's used in both `/` and `/about/`
featured_tags = true
featured_condition_size = 1
about_me = true
custom_css = ["css/custom-font.css"]

[params.social]
rss = true
twitter = "https://twitter.com/mshrh3"
linkedin = "https://www.linkedin.com/in/masahiro-hiramori-63b992167/"
github = "https://github.com/mshr-h"
googlescholar = "https://scholar.google.com/citations?user=NSCMi88AAAAJ"
facebook = "https://www.facebook.com/masahiro.hiramori"

[[params.addtional_menus]]
title = "CV"
href = "/top/cv/"

[[params.addtional_menus]]
title = "ABOUT"
href = "/top/about/"

[outputFormats.Algolia]
baseName = "algolia"
isPlainText = true
mediaType = "application/json"
notAlternative = true

[params.algolia]
vars = ["title", "summary", "date", "publishdate", "expirydate", "permalink"]
params = ["categories", "tags"]

[markup.tableOfContents]
endLevel = 2
startLevel = 1

[markup.highlight]
style = "dracula"

[markup.goldmark.renderer]
unsafe = true
```

以下のコマンドでローカルにて確認できる。

```bash
$ hugo server -D
```

[http://localhost:1313/](http://localhost:1313/)をブラウザで開くとサンプルページが表示される。

## GitHub Pagesで公開

hugoでビルドしたページは`public/`ディレクトリに生成される。このディレクトリを`username.github.io`にプッシュすることで、GitHub Pagesが更新される。
hugoで作成した`blog`ディレクトリをリモートの`blog/`リポジトリに紐付け、`public/`ディレクトリを`username.github.io`リポジトリに紐付ける。

```bash
$ cd ~/workspace/blog
$ git remote add origin https://github.com/mshr-h/blog
$ git submodule add -b master https://github.com/mshr-h/mshr-h.github.io public
```

この時点で各リポジトリとディレクトリの対応関係は以下の通り。

```
~/workspace/blog/ <-> https://github.com/mshr-h/blog
            +-public <-> github.com/mshr-h/mshr-h.github.io
            +-themes/hugo-theme-cleanwhite/ <-> https://github.com/zhaohuabing/hugo-theme-cleanwhite.git
```

Clean Whiteテーマにはデプロイスクリプト`deploy.sh`が用意されているので、これ使ってデプロイする。

```bash
$ cd ~/workspace/blog
$ ./deploy.sh
```

## 独自ドメイン設定

独自ドメインを使う場合、`username.github.io`リポジトリとDNSレコードの設定が必要。

`username.github.io`リポジトリの設定タブからGitHub PagesのCustom domainに独自ドメインを指定する。

DNSのAレコードに以下を追加する。

```txt
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
```

詳細は[ここ](https://help.github.com/en/github/working-with-github-pages/configuring-a-custom-domain-for-your-github-pages-site)を参照する。

## 記事を追加

新しい記事を作成し、デプロイする。

```bash
$ hugo new post/2020-04-01-article-title.md
$ vim content/post/2020-04-01-article-title.md
$ ./deploy.sh
```
# Clean Whiteテーマ特有の設定

Clearn Whiteテーマはデフォルトで中華フォントのため、日本語フォントを使うためにはCustom CSSでフォントを指定する必要がある。

`static/css/custom-font.css`に以下の内容を保存する。

```css
body, h1, h2, h3, h4, h5, h6, .navbar-custom { 
    font-family: Helvetica,"Sawarabi Gothic",Meiryo,"メイリオ","Hiragino Kaku Gothic ProN", "ヒラギノ角ゴ ProN",YuGothic,"游ゴシック",Arial,sans-serif; 
}
```

`config.toml`にすでに`custom_css`に関して記載があるため、以下に書き換える。

```toml
custom_css = ["css/custom-font.css"]
```

これでブログの設定完了。