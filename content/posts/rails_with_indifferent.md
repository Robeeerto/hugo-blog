---
title: "Rails 的 HashWithIndifferentAccess"
date: 2021-08-23T16:26:16+08:00
metaAlignment: center
thumbnailImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202203140233343.png'
coverImage: "https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202108222257343.jpg" 
coverMeta: out
coverSize: partial
categories:
- Rails
tags:
- SourceCode
summary: 深掘程式碼，到底為什麼呢？
keywords:
- Rails
- HashWithIndifferentAccess
- core_code
---

{{< toc >}}

在學習 Rails 的期間，一定有想過，為什麼在 Rails 裡面的 params，從印出來的那剎那就是用字串當作 key，但是你用符號也依然能夠取得 value 呢？

{{< tabbed-codeblock Hash範例>}}
<!-- tab ruby -->
params = {"ingredient": starch, "price": 200}
params[:ingredient] # "starch"
# 用符號拿值是我們一開始最先認識的 Ruby
params["ingredient"] # "starch"
# 但在 Rails 裡面，這也合法
<!--endtab -->
{{</ tabbed-codeblock>}}

這肯定是 Rails 又犯了老毛病，又做了黑魔法想來造福社會大眾了！

可惡，我一定要找到為什麼！

---

# 第一站
遇到不知道的東西，就是先把東西丟上 Google 去 Rails 的 API 查找


![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202108231634900.png)


果然很快就被我們找到他的說明，但說明不重要，我也知道你幫我轉型，但我要知道怎麼轉的呀！

順著我圈起來的藍色框框，我們只能前往 Rails 最核心的基地，探訪原始碼了！！！

---

# 第二站

我知道一開始肯定會站在 Rails Github 總部的大門前呆滯，你根本不知道要到哪一個房間去，也不知道要從哪裡找起

沒關係，我們站在大門口，點擊 `Go to file` 的按鈕


![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202108231636889.png)


緊接著 Github 有非常貼心且快速的搜尋服務，把我們剛剛找到的藍色框框打進去，那就是寶藏的目的地，也是整個 Rails 專案的絕對位置！！！


![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202108231636979.png)


果然被我們找到了！快進去看看！

---

# 第三站

映入眼簾的是滿滿的註解和原始碼，我們可以看到這是一個 `HashWithIndifferentAccess` 的類別繼承至原始的 Hash 類別！

裡面有許多的實體方法，想必看得一頭霧水吧？

但我們只要找到我們最需要的那些方法就夠了，其他的如果真的有時間也可以一一的來解析

首先就是在我們取值的時候到底動了什麼手腳？

我們可以知道以 Ruby 的習性，取值肯定也是一種方法，所以我們找到了`[]` 這個方法


![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202108231638655.png)


上面也有非常詳細的解釋，但這遠遠不夠我理解啊！！

我們看到 `super(convert_key(key))`，我們一段一段來解釋

首先是 `super()` 帶著括號的 super 就是把：

> **括號內的值帶到上一層的同樣方法中運算**

所以我們看字面可以理解成，他把轉型過後的 key 帶到 Hash 的 `[]` 取值

接著我們來看看 `convert_key` 做了什麼？


![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202108231639141.png)


我們可以理解成，他把 key 通通都轉成字串了，換句話說，這些變形的 Hash 在一開始的預設 key 就都是字串了！

那到底是在哪裡改變的呢？ 我們回到開頭


![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202108231639673.png)


在這個類別的初始化中，他做了一系列的改造，有以下幾個步驟

* 確認這個傳進來的物件有沒有`to_hash` 這個方法
* 有的話，就執行 `super()` 這邊就是執行一次 Hash 的 `initialize` 方法，然後進行 `update` 


![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202108231640072.png)


* 在清理了重複的程式碼之後，他又對這個 arg 進行一次判斷，如果你是純種的 Hash 就沒事，但若不是就要執行這個類別的 `to_hash` 方法！


![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202108231641623.png)


* 上方也很明顯地提到，要把正規的 hash 轉成 string 的 keys，仔細看會發現這個方法非常的關鍵，他分別進行了 `set_defaults` 以及 `convert_value` 方法


![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202108231642851.png)


* 而這個 `set_defaults` 方法，又呼叫了 `default` 方法(這邊不提 `default_proc` 是因為牽扯到 block 所以又更複雜了，我希望可以更直線性地先看完到底發生了什麼事情！)


![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202108231642347.png)


* 可以看到在這個方法中， key 都被轉成字串了
* 接著回到 `to_hash` 方法，還沒有結束呢！ key 都變成字串後，要來 `convert_value` 了 


![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202108231643782.png)


* 根據帶進來的參數做不同的操作，這邊從外部來的 hash 一定都會經過這個類別的 `to_hash` 方法的，等於是步驟 3 ~ 8 會不斷的重演，再加上一些例外處理，造就了這個看起來很痛苦的程式碼。

---

# 最終站

這是我第一次認真的閱讀完一個事件的程式碼生命週期，也意識到要完成一個框架實在是一件超級無敵誇張的事情。

下次遇到 params 傳進來有字串的情況不需要在覺得奇怪了！ 

Rails 在 middleware 的時候就幫你丟進去這個類別中做了一套終極轉換！

但其實就是把所有東西都轉成字串，當你需要使用這個功能的時候，只需要呼叫這個類別，然後 `new` 他，他就可以隨意的轉型取值囉！
