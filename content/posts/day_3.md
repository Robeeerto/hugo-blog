---
title: "Day 3 安裝 RSpec 以及環境設定"
date: 2021-09-21T16:48:05+08:00
metaAlignment: center
coverImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211620030.png' 
coverMeta: out
coverSize: partial
categories:
- "2021 鐵人賽"
tags:
- RSpec
---

開始之前，做一些設定和準備吧！
<!--more-->
{{< toc >}}

# 檢查電腦是否有 Ruby

> 鐵人賽文章主要以 Mac 作業系統為主，避免誤人子弟

Mac 作業系統可以打開你的終端機 ( Terminal )，然後輸入：

{{< tabbed-codeblock "檢查 Ruby 版本">}}
<!-- tab shell -->
$ ruby -v
<!-- endtab -->
{{</ tabbed-codeblock>}}

因為 Mac 作業系統基本都預設了 Ruby 2.0 版本，但如果沒有的話，這邊也提供使用 [Homebrew](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211655164.png) 這個套件管理工具來安裝 Ruby！

{{< tabbed-codeblock "安裝 Ruby">}}
<!-- tab shell -->
$ brew install ruby
<!-- endtab -->
{{</ tabbed-codeblock>}}
而在 Ruby 的世界中，有不同的版本，而這時候就需要一個管理 Ruby 的工具來幫助我們存放各種不同的版本。

# RVM ( Ruby Version Manager )

首先我們需要先安裝這個管理工具，[RVM官方網站](https://rvm.io/)

![](https://i.imgur.com/PwejIZf.png)

輸入指令後就會幫助你安裝好這個管理工具，接著我們能夠使用 `rvm list known` 指令來得到可以安裝的 Ruby 版本列表

{{< tabbed-codeblock "可以安裝的 Ruby 版本">}}
<!-- tab shell -->
$ rvm list known
# MRI Rubies
[ruby-]1.8.6[-p420]
[ruby-]1.8.7[-head] # security released on head
[ruby-]1.9.1[-p431]
[ruby-]1.9.2[-p330]
[ruby-]1.9.3[-p551]
[ruby-]2.0.0[-p648]
[ruby-]2.1[.10]
[ruby-]2.2[.10]
[ruby-]2.3[.8]
[ruby-]2.4[.10]
[ruby-]2.5[.9]
[ruby-]2.6[.7]
[ruby-]2.7[.3]
[ruby-]3[.0.1]
<!-- endtab -->
{{</ tabbed-codeblock>}}
這邊我們得到許多不同版本的 Ruby，我推薦安裝 `2.7.3` 版本 ( 算是現在比較多人使用的版本 )

下達指令：

{{< tabbed-codeblock "安裝指定版本">}}
<!-- tab shell -->
$ rvm install 2.7.3
<!-- endtab -->
{{</ tabbed-codeblock>}}
跑完之後，可以在使用 `rvm list` 來確認我們這台電腦安裝的 Ruby

{{< tabbed-codeblock "確認安裝情況">}}
<!-- tab shell -->
$ rvm list

   ruby-2.6.2 [ x86_64 ]
   ruby-2.7.2 [ x86_64 ]
=* ruby-2.7.3 [ x86_64 ]
   ruby-3.0.1 [ x86_64 ]

# => - current
# =* - current && default
#  * - default
<!-- endtab -->
{{</ tabbed-codeblock>}}

會得到在這台電腦上面的 Ruby 版本，下方的符號則代表著預設以及現在所使用的版本！

所以我們現在已經可以確保電腦中具有 Ruby 這款程式語言了，接下來就可以來安裝 RSpec 囉

# 安裝 RSpec 

接著我們在終端機繼續操作，來確認一下 `Gem` 的版本

> Gem 是 Ruby 世界的套件管理工具，幫助你安裝各式各樣的工具

{{< tabbed-codeblock "確認 Gem 版本">}}
<!-- tab shell -->
$ gem -v
3.1.6
<!-- endtab -->
{{</ tabbed-codeblock>}}

版本可能不一定相同，但確認一下有就好，基本上 Ruby 安裝成功 Gem 也會一起裝起來！

接著我們需要下達指令來安裝 RSpec 

> RSpec 是 Ruby 世界的一個工具，所以我們透過 Gem 來安裝它

{{< tabbed-codeblock "使用 Gem 安裝 RSpec">}}
<!-- tab shell -->
$ gem install rspec
...
....
..
5 gems installed
<!-- endtab -->
{{</ tabbed-codeblock>}}

看到 `gem installed` 之後，我們的電腦環境裡就有了 RSpec 這款工具囉！

# 文字編輯器 VSCode

環境，軟體都安裝到位之後，我們需要一個編輯器來編輯我們的檔案！

[VSCode 官方網站](https://code.visualstudio.com/) 

VSCode 基本上可以稱為現代編輯器之王了，越來越高的市占率，隨插即用的插件，整個使用者體驗並不需要高深的程式基礎就能夠一目瞭然，所以推薦給不知道怎麼編寫程式碼的人！

至於要用什麼插件，快捷鍵，操作心法等等，上網 Google 可以收穫滿滿喔！

# 小結

今天把所有的環境都設定完成了，該安裝的也安裝了！

明天開始我們就能夠風雨無阻的開始寫測試文件了，如果在安裝過程中有遇到任何的問題也很歡迎留言告訴我！