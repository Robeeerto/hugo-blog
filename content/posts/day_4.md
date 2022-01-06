---
title: "Day 4 初始化的 Rspec 檔案剖析"
date: 2021-09-21T17:01:05+08:00
metaAlignment: center
thumbnailImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211620030.png'
coverImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211620030.png' 
coverMeta: out
coverSize: partial
categories:
- "2021 鐵人賽"
tags:
- RSpec
---

來看看初始化後的樣子吧！
<!--more-->
{{< toc >}}

# 利用 Command line 來創建你的 RSpec 資料夾

一樣打開終端機並確認在你想要的路徑( 桌面等等... )
{{< tabbed-codeblock "Command Line 來建立 RSpec">}}
<!-- tab shell -->
$ mkdir learn-rspec
$ cd learn-rspec
$ rspec --init
  create .rspec
  create spec/spec_helper.rb
<!-- endtab -->
{{</ tabbed-codeblock>}}

這邊指令是先創建一個資料夾 ( learn-rspec ) 並且進入裡面，然後下達 `rspec --init` 的指令。

`rpsec --init` 這個指令的目的就在於幫你初始化你的這個資料夾，並且幫助我們建立起基本的執行檔以及設定檔

這時候我們能夠透過昨天下載好的 VSCode 來打開這個資料夾 ( VSCode 的使用非常直覺，所以就不介紹了 )

# .rspec 的作用

剛剛做完 `rspec --init` 後，原本空空的資料夾多了一些東西，如果有仔細看 Terminal 的話，應該是早就發現了！

我們先介紹 .rspec 這個執行檔的功能，點開檔案後會看到 `--require spec_helper`

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211705699.png)

這個指令是什麼意思呢？

> 每一次下達測試的指令之前，不論你是要測試一小個方法，或是一個完整的功能
> 他都會確保在所有動作之前去讀取 `spec_helper.rb` 這個檔案，確保你所有的設定檔都會被成功執行！

# spec_helper.rb

這個檔案的是整個 RSpec 的設定檔，裡面有許多的敘述以及設定，這邊就大致的講述，以及示範各種設定的效果！

這個頁面的上半部都是屬於註解的敘述，這個檔案會在我們剛剛提到的 `.rspec` 執行檔的設定下，保證被讀取。

以及確保這個檔案保持輕量，避免測試讀取的時間過久，效率變低！他希望我們能夠只讀取我們需要的功能

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211705082.png)

還記得我們前幾天提過的 RSpec 生態圈嗎？

這邊可以根據不同的 Gem 去做設定，我們也提過 RSpec 可以在 `rspec-expections` 以及 `rspec-mocks` 加入不同的函式庫做結合！

紅色框框處就是設定 `rspec-expections` 的地方，這時候一定會不知道如何設定，也不知道有什麼語法可以用～

推薦一個叫做 Rubydoc.info 的網站，裡面存放著各式各樣的 Gem 的語法，以及設定，需要什麼樣的 arguments 甚至是 Github 的連接都幫你設計好了，你只需要找你想要的資料即可！

整個搜尋的流程會是：

```shell=
rspec -> rspec-expections -> 點擊 configuration
```

映入眼簾的會是：

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211706342.png)

這邊就會是官方提供的 API 讓你去設定的文件，內容都描述的蠻詳細的！

不論你想要改顏色，設定語法等等，滾動到下方都有更範例或是連結，非常實用。

所以同理，綠色框框的則是 `rspec-mocks` 的設定檔，依循著同樣的邏輯，也可以找到屬於 mocks 的 API 喔！

從 50 行開始，就是一些和 Gem 不相關的設定，比較像是使用的體驗，或是一些特殊的小功能！

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211706629.png)

在紅色框框處，我們能看到他會對某一種設定進行說明，以及用法！

往後的每一個區塊都會是 RSpec 推薦給我們的設定，我們可以斟酌使用，這邊就示範一下加入紅色框框處的設定會長什麼樣子！

> 1. 首先，打開註解，找一個地方放，不要放到其他 Gem 的設定裡面喔！

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211706231.png)

> 2. 根據敘述隨意的做一個測試檔案，看看效果；我們可以知道敘述中說，這是一個可以過濾一些 example 的功能，只要加上 f 的前綴，快來試試看吧

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211707618.png)

> 3. 得到結論，加了 f 前綴的會被留下並測試，沒有加上 f 的會被過濾掉，雖然這個功能有點雞肋，目前想不到用在哪，因為 RSpec 可以單個 example 做測試，之後會教；但這邊主要想傳達閱讀文件，自己手動試試看的感覺！

其他的部分就靠大家自己玩玩看囉～

# Out Put 的效果

RSpec 的 Out Put 有分成預設以及稍微正式一點的模式

在上面的範例中，我有在 command line 後方加入 `--format doc` 的指令，就是為了讓格式看起來有被箝套的效果，這樣方便我能夠輕鬆的了解哪個 example 是在哪裡 context 之下 ( 這邊聽不懂 example context 等等，很正常，我們馬上就會學到 )

我放上圖片讓大家看一下預設以及箝套效果的不同：

預設版本 ( 只有綠燈 )：

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211707386.png)


箝套版本 ( 包含敘述的結構 )：

> test-configure 是最外層
> would be focus and test 是被測試的範例的敘述

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211707262.png)

現在教大家如何改變自己的 Out Put 效果，然後不使用 command line。

還記得 `spec_helper.rb` 是我們拿來設定整個 RSpec 的檔案，仔細看的話，會找到關於文字格式的敘述，也就是我們要的 Out Put 效果！

在 `spec_helper.rb` 中加入這一行 ( 他其實就在下面，可以複製上來用就好 )

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211707780.png)

接著你就算不加入 `--formate doc` 也可以直接印出易懂的 Out Put 效果囉！

# 小結

今天帶大家認識了基本的 RSpec 檔案結構，以及裡面的東西是什麼？該去哪裡找文件等等！

現在已經對於 RSpec 有最初初初初初步的認識了，明天我們將會談談什麼測試驅動開發，以及他對於 RSpec 的價值在哪裡？






