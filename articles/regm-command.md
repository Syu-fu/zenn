---
title: "正規表現をインタラクティブに確認するコマンドをdenoで作った"
emoji: "*️⃣"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["正規表現", "deno", "cli"]
published: true
published_at: 2024-05-02 11:30
---

## はじめに

ブラウザで検索すれば多くの正規表現チェッカーがありますがローカルから、かつCLI上から正規表現を簡単に確認したかったのでそれができるコマンド[regm](https://github.com/Syu-fu/regm)を作ってみました。  
JavaScriptから他の言語の正規表現をエミュレートできるライブラリを見つけたので以前から興味があったDenoで作成してみました。

@[card](https://github.com/Syu-fu/regm)

![作成したregmコマンドが動いている様子です。正規表現が入力欄に入力されると、それに合わせてその下の文字列に対して正規表現のマッチが行われています。文字列のマッチした箇所に対して色がつけられています。](/images/regm.gif)

## インストール方法

Homebrewを用いてインストール可能です。

```bash
# Homebrew
$ brew install Syu-fu/tap/regm
```

また、DenoのランタイムがあればDenolandからもインストール可能です。

```bash
# Deno
$ deno install --allow-env --allow-read --import-map https://deno.land/x/regm/import_map.json https://deno.land/x/regm/regm.ts --name regm --force
```

## 使い方

コマンドを実行する際に`-f`オプションで確認対象の文字列が書かれたファイルを指定します。  
また、Enterキーを押した際に入力されていた文字列が標準出力に出力されるのでパイプでつなげることが可能です。  
上記のgifでは`pbcopy`コマンドにつなげることでクリップボードにコピーしています。

### 正規表現オプション

Denoで作成していますがJavaScript以外で利用できる正規表現にも対応しています。  
オプションを指定しない場合はJavaScriptの正規表現を利用します。  
対応している正規表現は以下になります。  
括弧内に利用できる言語やプログラムを記載しています。

| オプション     | 説明                        |
| -------------- | --------------------------- |
| -e, --ecma     | ECMA (JavaScript, Javaなど) |
| -b, --basic    | BRE (grepなど)              |
| -x, --extended | ERE (egrepなど)             |
| -p, --pcre     | PCRE (Perl, PHPなど)        |
| -v, --vim      | Vim (Vim, Neovimなど)       |
| -r, --re2      | RE2 (Go, Pythonなど)        |

## 実装

### 正規表現エミュレート

JavaScript以外の正規表現のエミュレートには[regex-translator](https://github.com/Anadian/regex-translator/tree/main)というライブラリを使っています。  
一度仲介用の文字列に変えた後、JavaScript用の正規表現に戻してそれを`new RegExp()`の引数として渡しています。

```typescript
const mediaryString = RegexTranslator.getMediaryStringFromRegexString(
  pattern,
  flavor,
);
const regexString = RegexTranslator.getRegexStringFromMediaryString(
  mediaryString,
  "ecma",
);
return { regexp: new RegExp(regexString, "gm"), isInvalid: false };
```

### 問題点

本当はfzfのようにパイプで受け取った文字列に対してのマッチで実装しようと考えていたのですがDenoでパイプからの入力とキーボードでの入力を分けて扱うことがうまくいかなかったため現在はファイルからの入力のみをサポートしています。

## 終わりに

問題点は残っていますが最低限は使えるレベルになっていると感じているのでよければ使ってみてください。  
また、バグや機能追加の要望等ありましたら[Issue](https://github.com/Syu-fu/regm/issues)までお願いします。  
コントリビューションお待ちしております。
