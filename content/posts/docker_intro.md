---
title: "簡介 Docker"
date: 2022-02-16T16:55:11+08:00
metaAlignment: center
thumbnailImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202202161656501.png'
coverImage: "https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202202161701887.jpg" 
coverMeta: out
coverSize: partial
categories:
- Docker
tags:
- Introduce
summary: 簡介 Docker 這個東西！
keywords:
- Docker
- intro
---

{{< toc >}}

# Docker 如何改變現在的 IT 產業

Docker 作為一個開源的專案，從 2013 年由一間叫做 `dotCloud` 的公司開始運作，經過一年的發布，他已經成長得非常的巨大，並且關掉原本的公司，開了一間公司叫做 `Docker`。

經過了 8 年的時間，整個生態圈有了巨大的改變，不得不說現在 Docker 對於軟體產業的重要性已經有了質地上的轉變，`Container` 對於軟體的貢獻可以說是十年一遇，不僅限於網站開發，不論你是什麼樣子的工程師，幾乎都可以用上！

學習 Docker 對於你只有好處而沒有壞處，無論是在未來的升遷，或是現在的工作，都是多多益善！

回顧過去的科技業：

從 90 年代開始的 `Mainframe to PC`，也就是電影裡看到的超大伺服器的畫面。

接著從 00 年開始的 `Baremental to Virtual`，聽老闆說，當時都要到機房內操作這台虛擬機，接上螢幕和鍵盤，處理網站的問題。

而 07 / 08 年開始，Amazon 推出了 AWS 的解決方案，我們開始將東西上傳至雲端，我們可以在本地利用指令來開啟、關閉、快速的擴充網站等等。

直到現在，我們有了 `Container` 在遷移和改變上，有了更快速更好的體驗，相信如果從原本的 Heroku 要移轉到 AWS 或是 GCP 上的時候，是一件不容易且相當傷神的工程，但如果我們全部利用 Docker 的能力，我們可以更輕鬆的轉移網站，因為環境的相同，就可以解決掉麻煩的環境設定問題！

# 為什麼我們需要 Docker？好處是什麼？

最大的原因就是速度，軟體的部署速度，測試的速度，更新的速度，間接地影響到公司把任務完成的速度！

而為什麼軟體的部署速度會提高呢？

就是剛剛上面一直提到的 `Container` 也就是容器化的技術，讓我們可以脫離面對 `Matrix from Hell`。

**Matrix from Hell**

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202201210022071.png)

可以看到上面的圖片，根據不同的系統，我們要處理前端、後端、測試環境等等相容的問題！

但有了 `Docker` 我們可以不需要在 setup 的時候考慮那麼多種不同的可能，我們可以在各種不同的裝置上面跑我們的軟體、測試等等。

除了上述的好處，軟體開發人員在以前常常需要花費很長的時間在維護線上的軟體、修改 Bug，使用 Docker 也可以讓我們花費更多的心力在開發或是討論新功能，提升產品的水平！

回歸到這個章節一開始提到的，最終人們還是在追求更快的體驗，更方便的開發方式，但也讓整體的操作變得更複雜，分工變得更明確，我才想要寫一系列的文章來記錄自己學習 Docker 的歷程，也希望可以實際的應用在公司的專案上！