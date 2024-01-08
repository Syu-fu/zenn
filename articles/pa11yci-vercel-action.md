---
title: "Vercelのプレビューに対してしたpa11y-ciを実行してアクセシビリティについて自動テストする"
emoji: "📑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["アクセシビリティ", "a11y", "githubactions", "vercel", "pa11yci"]
published: true
published_at: 2024-01-08 15:00
---

## pa11y-ciとは

[pa11y](https://github.com/pa11y/pa11y)とは、アクセシビリティを自動的にテストできるツールです。  
こちらをCI上から実行しやすいようsitemapやURLのリストに対して実行できるようにしたものが[pa11y-ci](https://github.com/pa11y/pa11y-ci)になります。

## 事前準備

### Vercelの設定

Vercelのsettings → Deployment Protection → Vercel Authenticationをオフにする必要があります。
こちらを設定しないと作成されたVercelのプレビューを確認する際に認証が必要となるためpa11y-ciがデプロイしたアプリケーションをチェックできません。

### sitemap.xmlの作成

pa11y-ciの実行対象として参照するためsitemap.xmlを作成します。  
例えば、Next.js + AppRouterを利用する場合は以下のように実装します。

```typescript :src/app/sitemap.ts
import { MetadataRoute } from "next";

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const baseURL = "https://pa11yci-vercel-action.vercel.app";
  const lastModified = new Date();

  const staticPaths = [
    {
      url: baseURL,
      lastModified,
    },
  ];

  return [...staticPaths];
}
```

pa11y-ciの都合上[sitemap index](https://developers.google.com/search/docs/crawling-indexing/sitemaps/large-sitemaps?hl=ja)ファイルは利用できませんので注意してください。  
こちらを利用してしまうとsitemap indexファイルに記載されている他のsitemap.xmlをテストしてしまうためです。
例えば、[next-sitemap](https://github.com/iamvishnusankar/next-sitemap)ではデフォルトでsitemap indexファイルを作成します。

## 実装

### workflow

```yaml :pa11y.yaml
name: pa11y
on:
  issue_comment:
    types: [edited]

jobs:
  pa11y:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Get Preview URL
        id: capture_preview_url
        uses: aaimio/vercel-preview-url-action@v2.2.0
        with:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Get Redirect URL
        id: get_redirected_url
        run: |
          PREVIEW_URL="${{ steps.capture_preview_url.outputs.vercel_preview_url }}"
          REDIRECTED_URL=$(curl -Ls -o /dev/null -w %{url_effective} "$PREVIEW_URL" | sed 's/\/$//')
          echo "REDIRECTED_URL=$REDIRECTED_URL" >> "$GITHUB_OUTPUT"

      - uses: actions/setup-node@v4
        with:
          node-version: "18"
          cache: npm

      - name: Install dependencies
        run: npm install -g pa11y-ci

      - name: pa11y-ci
        env:
          REDIRECTED_URL: ${{ steps.get_redirected_url.outputs.REDIRECTED_URL }}
          REDIRECTED_SITEMAP: ${{ steps.get_redirected_url.outputs.REDIRECTED_URL }}${{'/sitemap.xml'}}
          REPLACE_BASE_URL: "https://pa11yci-vercel-action.vercel.app"
        run: |
          pa11y-ci --config .pa11yci.json --sitemap $REDIRECTED_SITEMAP --sitemap-find $REPLACE_BASE_URL --sitemap-replace $REDIRECTED_URL
```

全体の流れとしては[aaimio/vercel-preview-url-action](https://github.com/aaimio/vercel-preview-url-action)を利用してVercelに自動デプロイしたURLを取得します。
こちらのURLはクエリパラメータがついています。
そのため[VercelとGitHubActionsでフロントエンドのパフォーマンスを自動計測したい #Next.js - Qiita](https://qiita.com/arfes0e2b3c/items/e958aaac514b174d1832)を参考にリダイレクト先のURLを取得しています。
また、ここで取得できるURLは「/」で終わっているためsedを使って除去しています。
その後はpa11y-ciのインストール及び実行をしています。

#### pa11y-ciの実行について

```bash
pa11y-ci --config .pa11yci.json --sitemap $REDIRECTED_SITEMAP --sitemap-find $REPLACE_BASE_URL --sitemap-replace $REDIRECTED_URL
```

GitHub Actions上でpa11y-ciを上記のように実行しています。
各オプションについて簡単に説明をまとめます。  
`--config`によってコンフィグファイルのパスを指定してます。  
`--sitemap`によってプレビューのサイトマップの取得先を指定しています。
ただ、プレビューのサイトマップの内容としてプレビュー先ではなくプロダクション環境のURLが指定されています。
そのため`--sitemap-find`によってプロダクション環境のbaseURLを取得した後、`--sitemap-replace`オプションによって置き換えを行っています。

### pa11y-ciの設定

```json :.pa11yci.json
{
  "defaults": {
    "standard": "WCAG2AA",
    "timeout": 100000,
    "chromeLaunchConfig": {
      "args": ["--no-sandbox"],
      "executablePath": "/usr/bin/chromium"
    }
  }
}
```

pa11y-ciの設定ファイルの一例を載せます。  
細かいオプションについてはREADME.mdの[Configration](https://github.com/pa11y/pa11y-ci?tab=readme-ov-file#configuration)を参照してください。
自分が試した限りではchromeLaunchConfigを設定していない場合に実行が失敗していたため設定が必要なはずです。

## 参考

実際に動作させたリポジトリは以下になります。  
テストしているサイトはNext.jsを利用しています。  
@[card](https://github.com/Syu-fu/pa11yci-vercel-action)
