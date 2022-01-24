---
title: "Day 26 什麼樣的測試應該寫，什麼樣的不用？"
date: 2021-10-10T23:11:20+08:00
metaAlignment: center
thumbnailImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211620030.png'
coverImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211620030.png' 
coverMeta: out
coverSize: partial
categories:
- "2021 鐵人賽"
tags:
- RSpec
summary: 什麼都要寫測試嗎？
keywords:
- RSpec
- model_spec
- feature_spec
---

{{< toc >}}

# 決定要寫什麼樣的測試

工程師可能會寫很多不同種類的測試。在 RSpec 裡面，有 Model spec，Feature spec，Controller spec，Route spec 等等，你需要掌握所有不同類型的測試嗎？當然不，那應該要跳過哪些測試呢？

你可能會想，測試不就是應該要覆蓋率 100 % 嗎？越多越好，但事實是這樣嗎？會不會其實不需要真的這麼做，只需要掌握幾種重要的測試就好了！

# 我一定會寫的兩種測試

通常有兩種測試是我一定會寫的，就是 Model 以及 Feature 的測試，我會詳細解釋一下為什麼？

Model spec 是相對孤立的，一個 Model spec 將測試我的應用程式中特定的行為或是方法，例如有一個：Custom Model，Model spec 讓我知道我的功能是可以被切成很細微的粒子，同時給予我信心，因為這些方法都會常常被的使用到，不論是在 Controller 或是 View。

例如，我有一個方法是回傳客戶的付款總額，那我就會替這個方法寫一個測試。

確保應用程式的行為，在微觀的情況下，每一個方法都能經得起測試，並且正確的執行！

而 Feature spec 呢？剛剛說過，Model spec 是很孤立的，但 Feature spec 則相反；他是一個集合體，驗收整個應用程式的測試，Feature spec 讓我相信所有的東西都能一起工作，例如：如果我的 Custom resource 有 CRUD 的功能，我就為了新增、修改、刪除寫 Feature spec！

# 測試術語

在測試的領域中，術語並沒有達到共識，例如：有些人可能覺得 end-to-end 的測試、Feature spec 、驗收測試是指同一件事，有些人可能覺得這三種是完全不同的測試類型。

這對於寫測試意味著什麼？假設你看到 end-to-end 測試的引用和你對於 end-to-end 的理解不一致的話，這並不代表你的理解是錯誤的，因為並沒有正確的答案，在測試的領域中很多東西都很模糊，並不是每個人都用相同且確切的術語在討論相同的概念，但我認為這種溝通上的挑戰是有幫助的，否則人們可能會因為明顯的差異而受到干擾！

# RSpec 對應的測試術語

Model spec 和所謂的驗收測試是如何在 RSpec 中對應呢？ Model spec 是活在 `spec/models` 目錄中的測試。為了寫 Model spec 除了原生的 RSpec 其他什麼都不需要。外界所提到的驗收測試 ( 你可以說是 end-to-end 測試 ) 在 RSpec 中都被稱為 Feature spec。由於 Feature spec 是通過瀏覽器來行使功能的，因此之前提過的 Capybara 就會是必要的！

順便說一下，為什麼 RSpec 測試被稱為 `規格(spec)` 而不是 `測試 (test)`？這個想法是，每個 RSpec 文件是一個 `可執行的規範`。在 RSpec 術語中，每個測試案例都被稱作是一個 example 例子，一個關於該功能應該如何表現的例子。一堆的例子成就了一個規範。但其實這是在英文上的差別啦，在台灣大家都還是說測試測試，不太會有人硬要說這是一個規格，除非在寫文章強調正確性才會，或是要寫成英文的話，就會用 `spec` 而不是 `test` 。 

# 我覺得意義不大的測試

我自己覺得 View spec、Route spec、Request spec、Controller spec 的意義就不太大，讓我們分開來討論一下。

**Request spec、Controller spec**

首先是術語：Request spec、Controller spec 是兩件非常不同的事情，儘管為了我們的目的，我們可以把這兩個看作是同一個事情，而且 RSpec 官方也在推廣 Request spec 這件事，所以事實是， Request spec 是新的用法、也是一個新的術語。

不太寫這個的原因是，我們都希望 Controller 可以越瘦越好，所以其實 Controller 本身並沒有什麼商業邏輯，通常是都是調用 Model 的實體在做事情，所以不論是 Request spec 或是 Controller spec 都是可以被 Feature spec 給覆蓋的，所以我更傾向讓 Feature spec 去做這件事！

**View spec、Route spec**

說實在我不知道 View spec 的價值是什麼，我也沒有看過哪些教學會真的希望你去寫 View spec，原因是這些東西都會被 Feature spec 給覆蓋，且它的功能 Feature spec 都有提供。

而 Route spec 也是一樣的道理，也有許多的層面都是被 Feature spec 給覆蓋。

# 小結

這次是把自己看了蠻多測試書後的感想寫出來，因為真的有蠻多的東西在 Wiki 上的解釋和 RSpec 裡面是有灰色地帶，真的會因為術語太多而搞不清楚情況。

而對於有些 RSpec 提供的測試，自己也沒有很常使用，因為 Model spec 以及 Feature spec 就能有效的覆蓋大部分的情形。

如果有什麼想法也可以留言告訴我喔！可以一起討論～



