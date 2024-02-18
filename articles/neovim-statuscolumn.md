---
title: "Neovimでstatuscolumnの表示を美しく"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["neovim"]
published: true
published_at: 2024-02-19 07:30
---

こちらの記事の設定を適用すると画像のように列ごとに表示内容が決まったstatuscolumnを表示できます。

![Neovimでソースコードを開いた画像です。同一行に複数の情報がある場合もstatuscolumnの一列目はLSPのDiagnosticsが二列目はGitSignsが固定されて表示されています。](/images/beautiful-statuscolumn.png)

## statuscolumnの設定について

Neovimを利用する際、statuscolumnにGitSignであったりLSPやLinterからのDiagnosticsを表示している方を多く見ます。  
ただ、`vim.opt.signcolumn = 'yes'`を設定してもGitSignとDiagnosticsが被ってしまい片方の情報が見られないなどということが起こります。  
しかし、表示行を増やすため`vim.opt.signcolumn = 'yes:2'`を設定しても同じ行に２つDiagnosticsのマーカーが表示されたりして見栄えがよくありません。
以下の画像の一行目はGitにて更新がある行なのですがDiagnosticsが優先して表示されてしまいGitで更新があった行であるかが読み取れません。

![Neovimでソースコードを開いた画像です。DiagnosticsとGitSignsが両方表示されるはずの一行目にはDiagnosticsのみが表示されています。](/images/no-beautiful-statuscolumn.png)

そのため今回は[statuscol.nvim](https://github.com/luukvbaal/statuscol.nvim)を利用して表示を整えてみます。

### statuscol.nvimの設定

Diagnosticsを表示するためには各種LSPの設定が、Gitsignsを表示するためには[gitsigns.nvim](https://github.com/lewis6991/gitsigns.nvim)の設定が必要ですが今回は詳細を省きます。  
以下は最初に示した画像のように一列目にはDiagnostics、二列目にはGitSignsを表示するための設定です。  
また、更にその右には絶対行数を表示しソースコードとの境を`│`で区切っています。  
`bt_ignore`にはstatuscolumnを表示しないバッファタイプを列挙します。

```lua
local builtin = require('statuscol.builtin')
require('statuscol').setup({
  bt_ignore = { 'terminal', 'nofile', 'ddu-ff', 'ddu-ff-filter' },

  relculright = true,
  segments = {
    {
      sign = {
        name = { 'Diagnostic.*' },
        maxwidth = 1,
      },
    },
    {
      sign = {
        namespace = { 'gitsigns' },
        maxwidth = 1,
        colwidth = 1,
        wrap = true,
      },
    },
    {
      text = { builtin.lnumfunc },
    },
    { text = { '│' } },
  },
})
```

## この設定をした際の注意点

複数行を表示する場合でもsigncolumnは`2`以上を設定せずに`1`を設定します。
上記の[statuscol.nvim](https://github.com/luukvbaal/statuscol.nvim)の設定1つが一列扱いとなるためです。  
Luaの例を以下に示します。このあたりの設定をVim scriptで行っている方は読み替えてください。

```lua
vim.opt.signcolumn = 'yes'
```

## おわりに

他の設定に関しては[Syu-fu/dotfiles](https://github.com/Syu-fu/dotfiles)においてあるので参考にしてみてください。
Neovimの設定についての記事は数多くありますがstatuscolumnについて書いてあるものが少なかったため書いてみました。
statuslineを凝っている人は多いと感じますが、statuscolumnまで細かく設定している人は少ないのではないでしょうか。
statuscolumnも画面の多くを占め、またコーディングを効率化するための情報を表示できますのでみなさんも設定してみてください。
