---
title: "Day 1 介紹測試框架 RSpec"
date: 2021-09-21T16:17:55+08:00
metaAlignment: center
thumbnailImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211620030.png'
coverImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211620030.png' 
coverMeta: out
coverSize: partial
categories:
- "2021 鐵人賽"
tags:
- RSpec
summary: '第一天，先認識 RSpec 吧！'
---


{{< toc >}}

# RSpec 是什麼？

1. 是一款在 2005 年釋出的開放原始碼的測試函式庫
2. 最熱門的 Ruby Gem 之一，超過 5 億的下載量

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211630135.png)

3. 最近的一次版本是 RSpec 3，在 2014 年釋出
4. RSpec 是一種 DSL
> 領域特定語言（英語：domain-specific language、DSL）指的是專注於某個應用程式領域的計算機語言
> RSpec 的語言是專注在測試方面，同時也是為了測試而存在
> DSL 舉例： HTML VimScript 等等

# 為什麼要測試程式碼？

- **增加新功能時避免產生故障**
  - 新增功能時，只要根據原先的測試規格下去跑，就能知道結果 
- **提供規格給整個應用程式當做文件閱讀**
  - 有些複雜的應用程式，程式碼被模組化四散各地，這時候閱讀測試文件能夠快速掌握重點以及引用的模組
- **單獨處理某些問題或是 Bug**
  - 測試的重點就是一次只測試一個方法或是簡單的功能 (Unit Test) 
- **提升程式碼品質，尤其在設計方面**
  - 後面會提到測試、開發、重構的循環，擁有測試的時候，你才有重構的勇氣
- **縮短開發時程**
  - 測試文件若在開發前期就有穩固的基礎，你只需要確認程式碼能通過測試，基本上就有一定的穩定性

# RSpec 的生態系

RSpec 本身包含了三種獨立的 Ruby Gem，雖然下載 RSpec 就會一起包給你了 :ghost: 

- **RSpec core**
  - 核心的函式庫，負責組織和執行測試的部分
- **RSpec expections**
  - 配對語法 ( matcher ) 的函式庫，負責實現我們的例子
- **RSpec mocks**
  - 一個用來假裝物件或類別行為的函式庫

RSpec 本身也可以和其他函式庫的 mocks & expections 整合

**Rspec-rails** 則是整合了 Ruby on Rails 和 RSpec 而衍生出來的一個 Gem 

# 專案中的資料架構

- **都會看到一個 `spec` 的資料夾中存放所有測試的檔**
- **在裡面會看到類似模仿專案架構的資料名稱**
- **RSpec 的檔案都會以 `_spec.rb` 當做結尾，讓測試時能夠找到他**
- **舉例：一個 `User` Class 就會有一個 `user_spec.rb` 的檔案在 `spec` 資料夾裡面**

# 小結

今天非常簡單的介紹了 RSpec 這個測試框架，也可以當作整個鐵人賽的小目錄。

基本上都會以上面提過的東西去做延伸，後面會加入一些範例的程式碼來做舉例，希望可以讓不熟悉 RSpec 的人能夠對於這個測試框架有所理解和幫助！

明天我將會介紹測試的種類，因為一個龐大的應用程式對於測試的分類也有所不同，這樣才能將執行的效率大幅提升！



