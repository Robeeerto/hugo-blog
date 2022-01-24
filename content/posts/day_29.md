---
title: "Day 29 RSpec 裡的 Factories 和 Fixtures"
date: 2021-10-13T20:24:18+08:00
metaAlignment: center
thumbnailImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211620030.png'
coverImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211620030.png' 
coverMeta: out
coverSize: partial
categories:
- "2021 鐵人賽"
tags:
- RSpec
summary: 常常聽到的 Fixtures 是什麼？
keywords:
- RSpec
- factories
- fixtures
---

{{< toc >}}

測試有一個必要的條件就是要有資料來做測試，有三種方法可以 Rails 中產生測試資料。

1. 手動
2. Fixtures
3. Factory-Bot

這三種方法都是可行的解決方案，我們可以一個一個來探討一下。

# 手動建立資料

如果你測試的 Model 只有幾個屬性，而且沒有關聯的 Model，手動建立測試資料可以說是非常方便。

例如，假設我有一個 PaymentType Model，只有一個屬性，名字。而且也可以很輕易的做出合法以及不合法的狀態，很簡單！

{{< tabbed-codeblock "Manual">}}
<!-- tab ruby -->
valid_payment_type = PaymentType.new(name: 'Visa')
invalid_payment_type = PaymentType.new(name: '')
<!-- endtab -->
{{</ tabbed-codeblock>}}

接著，我們有一筆訂單，它是由多的 LineItems 和 Payments 組成，每個都有一個 PaymentType。

{{< tabbed-codeblock "Manual">}}
<!-- tab ruby -->
order = Order.create!(
  line_items: [
    LineItem.create!(name: 'PS5', price: 15000)
  ],
  payments: [
    Payment.create!(
      amount: 15000,
      payment_method: PaymentType.create!(name: 'Visa')
    )
  ]
)
<!-- endtab -->
{{</ tabbed-codeblock>}}

這樣寫起來其實超級煩的，我們不得不浪費一堆腦力，想一些不一定相關的細節。

如果我們想要測試的是，`amount` 等於 `line_items` 的總額時，`order.balance`  回傳 0，該怎麼做？

接這就是 Factory 派上用場的時候了！

# Factories

Factory 其實不是一個針對測試的概念，`Factory Method` 是一種設計模式，可以在 `Gang of Four` 這本書中找到，而這個模式恰好對測試的目的很有用。

Factory 的概念基本上是，你有一個方法或是函數，可以為你產生新的類別。

在 `Gang of Four` 這本書中，他用了一個例子，一個叫做 `Creator` 的工廠會根據不同的條件回傳 `MyProduct` 或是 `YourProduct` 的實體。可以直接說 `Creator.create(相關資料)`，就會自然回傳適當的實體！

你也許會想這樣的 Factory 有什麼用處，在測試的情況下，我們想要的 Factory 類型可能會有一點不同。我們關心的不是抽象的類別實體化。我們希望抽象出沒意義的細節和繁瑣的 Model Association

下面是一個例子，說明如果我們用一個 Factory，特別是 Factory-Bot，剛剛 Order 的設置會是怎麼樣呢？( 關於 Factory-Bot 在之前的文章有做過分享基本的用法 )

{{< tabbed-codeblock "Factory">}}
<!-- tab ruby -->
order = create(
  :order,
  line_items: [create(:line_item, price: 15000)],
  payments: [create(:payment, amount: 15000)]
)
<!-- endtab -->
{{</ tabbed-codeblock>}}

在這樣的情況下，我們只需要專注在我們要測試的項目，不用像剛剛手動的那樣關心一些奇怪的名目和支付方式，做好我們關心的 `price` 和 `amount`，就是我們最關心的一切。

那我們要怎麼用 fixtures 來實現同樣的事情呢？

# Fixtures

首先我自己對於 `fixtures` 的經驗和 `Factory-Bot` 相比，比較不熟。但可以看看這篇關於 [fixtures](https://chriskottom.com/blog/2014/11/fixing-fixtures/) 的文章！

說到這裡，如果我們想用做出和上面 Factory-Bot 一樣的測試資料，也可以使用 `fixtures` 來做，通常情況下，我看別人都是用 YAML 檔的形式來表達！

{{< tabbed-codeblock "Fixtures">}}
<!-- tab yaml -->
# orders.yml
payments_equal_line_item_total:
  # no attributes needed

# line_items.yml
ps5:
  order: payments_equal_line_item_total
  name: 'PS5'
  price: 15000

# payment_methods.yml
visa:
  name: 'Visa'

# payments.yml
first:
  order: payments_equal_line_item_total
  payment_method: visa
  amount: 15000
<!-- endtab -->
{{</ tabbed-codeblock>}}

一但確定好 fixtures 的資料後，用它就像在用 Hash 的 key 一樣簡單～


{{< tabbed-codeblock "Fixtures">}}
<!-- tab ruby -->
order = orders(:payments_equal_line_item_total)
<!-- endtab -->
{{</ tabbed-codeblock>}}

所以，你覺得哪一種生產測試資料的方式好用呢？

# 小結

我想前面已經證明，手動產生測試資料這件事情真的很容易就很麻煩，但是，也不是真的不好用，他其實在自己的 side project 裡面就很好用，手動生產就不需要開檔案，分那麼多的資料夾，而且可讀性又提高，東西都在一個檔案裡面～

至於 fixtures 和 Factory-Bot，我自己是用 Factory 居多，主要原因是 Gem 的強大，還有 YAML 檔，雖然很好讀，但我自己還是不太習慣，可能這就是工具用久的依賴性吧～

至於之前看到有人提過關於 fixtures 的見解，他認為，一個專案會創造一個 `資料世界`，然後用這個很大又很複雜的世界作為測試的基礎，發文的人說他更傾向於在可行的情況下，從一個乾淨的 Model 開始每一個測試，並為每個測試產生該測試所需要的最低限度資料，他發現，這讓測試變得容易理解，而不是用 fixtures 製造出一個固定的資料！

當然也不是說只能挑一個做使用，但如果今天你想要有一個固定的回傳資料或是什麼的可以( 可能是金流，那種從來不會改變的東西 )，你也可以做一個 fixtures 然後傳到 Factory，在整個測試基礎上產生動態資料！

我自己還是大推 Factory-Bot 啦，但可能我自己對於 fixtures 的理解也不夠深，說不定一年過後就會推翻這些自己覺得是對的東西，那也沒關係啦，繼續變強才是重點嘛～





