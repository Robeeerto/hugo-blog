---
title: "Day 22 rspec-rails 介紹！"
date: 2021-10-06T21:21:32+08:00
metaAlignment: center
coverImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211620030.png' 
coverMeta: out
coverSize: partial
categories:
- "2021 鐵人賽"
tags:
- RSpec
---

介紹 RSpec Rails 的設定
<!--more-->
{{< toc >}}

今天我們會把 RSpec 安裝進入 Rails 裡面，以及一些基本的設定。

主要會介紹：
1. `rspec-rails` 安裝的方式
2. 自動根據 `rails generate` 的指令來產生 `_spec.rb` 的檔案
3. 介紹一些 rails 特定的 matcher
4. rails 一定會寫的測試種類
5. 一定會用到的測試套件！

# rspec-rails 的安裝方式

在 rails 的專案目錄中找到 `Gem.file` 檔案。

並且同時在 `development` 以及 `test` 的 group 下加入最新版本的 `rspec-rails` 最新版本 ( 記得要加上版本號喔！ )

```ruby=
group :development, :test do
  gem 'rspec-rails', '~> 5.0', '>= 5.0.2'
end
```

接著我們在終端機輸入指令：

```ruby=
$ bundle install
```

這步主要是把這個 `gem` 安裝進入我們這專案的裡面，接著我們需要繼續輸入指令：

```ruby=
$ rails generate rspec:install
      create  .rspec
      create  spec
      create  spec/spec_helper.rb
      create  spec/rails_helper.rb
```

裡面的 `.spec` 以及 `spec` 資料夾和 `spec_helper.rb` 已經在前五天就有介紹過了！

但其實 `rails_helper.rb` 其實沒有什麼太多重要的東西需要設定，畢竟測試的核心還是在 `spec_helper.rb` 裡面。

`rails_helper.rb` 裡面比較多的是額外插件的設定，或是更換關於 Rails 的東西，例如不想要用 `ActiveRecord` 也可以關掉，路徑以及環境的設定會放在這邊。

之後介紹的 `Factory-bot` 以及 資料庫的清洗都會在這邊設定，到時候會再示範一下～

# 自動產生測試檔案

自動產生檔案這件事情，在我們前面安裝完後其實就有了這項功能。

我們可以使用指令 `rails g model --help` 來看看有什麼不同。

我這邊截圖的是全新 new 出來的專案，看看指令後的情況！

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202110062122561.png)

一開始的預設是 `test_unit`，也就是我們在根目錄中會看到一整包 `test` 的資料夾，在我們安裝 RSpec 後可以直接全部刪掉了！

接著我們看看安裝完 RSpec 的專案是什麼樣子吧！

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202110062123524.png)

這就是為什麼我們在 `rails g model / controller` 等等時，會自動產生測試檔案的原因～

# rspec-rails 特別的 matcher

眾所皆知 rails 是一個 Web 開發的框架，所以當然有些比較特別的 matcher 是為了網站所打造的！


![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202110062123867.png)

> 此圖片截圖自 `rspec-rails` 的 [github README](https://github.com/rspec/rspec-rails)

其實用看的就能理解他要測試的東西都和網際網路有相關，所以才需要特製，關於 `render` 頁面，`path` ＆ `http_status` 等等，都能夠在 `rspec-rails` 中找到唷！

# Rails 測試的優先順序

一般來說測試是寫的越多越好，甚至可以達到傳說中的覆蓋率 100 % 那當然是無話可說！

但現實並不常常是如此，如果不是規模特別大的公司，工程師常常是身兼多職，從前端寫到後端，甚至測試以及 CI/CD 等等，都需要一條龍掌握住！

但這個時候問題就來了，工時其實就是那樣，除非你無限爆肝，不然也不可能達到 100 % 的測試覆蓋率～

那我們就可以針對比較重要的項目來進行測試，以下是 Rails 的測試項目：

1. Model specs
2. System specs / feature specs*
3. Request specs / controller specs*
4. Helper specs
5. View specs
6. Routing specs
7. Mailer specs
8. Job specs

上面的順序其實也反應了我自己對於測試的重要度感受。

Model 的測試不用說了，肯定是整個 Rails 的根基，也為了我們後面的 Feature / System test 打下基礎！

而 Feature / System test 則是會讓我感覺到安心的一個測試模式，畢竟他是真的模仿使用者在瀏覽網站，但也因為他的測試規模較大，耗時較長，所以我們才需要 Model test 來幫助我們檢查小螺絲～

至於 Request 的測試就相對比較少，因為我自己覺得有點和 System test 重疊的部分，如果我的 Controller 壞了，那我的 System test 也不會通過才對！

所以單獨測試 Controller 會讓我覺得有點小題大作～

至於從 Helper 到 Job 的測試都是平常比較少寫的，但如果是 API 模式，少了 Feature / System test 的話，可能就要把測試覆蓋到各個小地方去才好！

# 最常使用的延伸套件

絕對非 Capybara 以及 FactoryBot 為主，這兩項在撰寫 RSpec 的過程中應該是不可能不使用到才對～

明天也會介紹 FactoryBot 的基本使用以及一些在撰寫不同測試需要注意的事情！

而後天則會介紹 Capybara，也是我們在撰寫 Feature / System test 時最為重要的夥伴！

# 小結

之後的文章也會盡量加入一些自己在撰寫專案中踩到的小雷，或是需要特別注意的地方！

因為其實寫 RSpec 常常會遇到一些奇奇怪怪的情況，或是不知道為什麼就是不行動，但其實有一些小撇步可以讓寫測試也變得開心喔！

希望讓大家在寫測試的時候，都能夠感覺像是在實作功能一樣輕鬆簡單快快樂樂！




