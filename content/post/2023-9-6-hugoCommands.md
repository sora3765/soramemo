---
title:       "Hugoコマンドメモ"
subtitle:    "よく使うコマンド"
description: "よく使うコマンド"
date:        2023-09-06
author:      "sora"
image:       "img/home-bg-net.png"
tags:        ["tech", "memo","hugo"]
categories:  ["Tech" ]
---

# Hugoコマンドメモ

## 前振り

- 毎回使うたびにHugoのコマンドを調べているので、自分のためにもまとめました
  - HugoはGoで記述された静的サイトジェネレーターです

## サイトの作成

- `hugo new site サイト名`で、実行したディレクトリに作成されます
- コンテンツやテーマが含まれないテンプレートが作成されます
- テンプレートを使う事が多いのであまり使用しないかも？
  - テンプレートは[ここから](https://themes.gohugo.io/)
- <https://gohugo.io/commands/hugo_new_site/>

## 記事の作成

- `hugo new ファイル名.md`で記事を作成出来ます
- 記事はデフォルトでcontentフォルダに作成されます
  - content内でファイルを分けたい場合は`file/ファイル名.md`で作成出来ます
    - フォルダは自動作成されます

```bash
$ hugo new post/2023-9-6-hugoCommands.md
Content "/Users/.../soramemo/soramemo/content/post/2023-9-6-hugoCommands.md" created
```

- <https://gohugo.io/commands/hugo_new/>

## サイトのプレビュー

- `hugo server`でlocalhost上で確認を行いながら記事を書く事が出来ます
- サイトはデフォルトで<http://localhost:1313/>で確認が出来ます
- 文書を保存するたびに変更が反映されます
- オプションなどは公式ドキュメントで確認を
- <https://gohugo.io/commands/hugo_server/>

## 記事からHTMLの作成

- `hugo`コマンドで作成した記事からHTMLなどを生成し、公開できる状態にビルドします
- デフォルトでは未来の日付を生成しない事に注意です
  - `-F`オプションで出力する事が出来ます
- その他オプションは公式ドキュメントで確認を
- <https://gohugo.io/commands/hugo/>

## 公開のススメ

- コマンドとは関係ないですがついでなので
- gitHubに保存して、gitHubWorkflowにてビルド、公開するのがおすすめ
- 公開は<span style="color:deepskyblue;">gitHubAction</span>か<span style="color:deepskyblue;">netlify</span>が簡単
