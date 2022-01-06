---
title: "Day 27 RSpec 的 Mock & Stub"
date: 2021-10-11T16:49:26+08:00
metaAlignment: center
coverImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211620030.png' 
coverMeta: out
coverSize: partial
categories:
- "2021 鐵人賽"
tags:
- RSpec
---

差別是什麼呢？
<!--more-->
{{< toc >}}

在我一開始學習寫 Rails 測試時，會有很常見的問題，就是到底什麼是 mocks & stubs，然後我要怎麼使用它們？

如果你對 mocks & stubs 也覺得很困惑，其實真的很正常，所以我自己也看了蠻多不同文章對於對於 mocks & stubs 的見解，希望今天可以釐清這個問題，特別是在 Rails/RSpec 的背景下。

# 關於測試術語的問題

昨天的文章也有提到，測試術語可能有一百萬種，但對於含義卻沒有一個完全的共識。像是昨天提到的 end-to-end 以及 驗收測試是同一回事嗎？有些人是，有些人不是。而且也沒有一個中央權威機構說什麼是對的，什麼是錯的，這就是測試術語上會遇到的問題。

尤其在 mock & stub 上，有些人說 mock 就是 stub，反之。

這樣的結果就是，關於 mock & stub 的解釋都變得非常的混亂！

# Test Double

我在網路上看過 Gerard Meszaros 的 `xUnit Test Patterns: Refactoring Test Code` 這本書時提過，mocks & stubs 都是 double 的特殊類型時，我好像懂了些什麼。

這種理解對我來說就比較不容易被動搖，因為理解過 `double` 能夠讓我做模式比對，所以我知道這不太可能再以後被我看到的新知識給推翻，感覺就像是經驗值真正的提升了一點！

所以什麼是 double ？ 在之前我只提過她它在 RSpec 的用法，在書中，作者把 double 比喻成好萊塢的特技替身。

當電影想要拍攝一些對主角來說有風險的事情時，他們會雇用一個 `特技替身` 來代替演員在場景中的動作。特技替身是一個受過嚴格訓練的人，能夠滿足畫面的需求，他們可能不會演戲，但他們知道怎麼從高處墜落、撞車等等。特技替身通常要在身材和某些角度和演員很相似！

等等的文章會用第三方支付當作例子。當我們在測試第三方支付這件事情時，不會真的希望它每測試一次就收一次錢吧？我們希望用替身來代替真正的支付流程。

# Stub 的解釋

我在網路上有看到 [這篇文章](https://blog.pragmatists.com/test-doubles-fakes-mocks-and-stubs-1a7491dfa3da) 是一個叫做 Michal Lipski 的人寫的。

根據他的文章，我也有了自己的解釋方式。

Test stub 是一個假的對象，用來代替一個真正的對象，目的是讓程式碼可以按照我們需要的方式來進行測試。Stub 有很大一部分工作是在我們原本的程式碼被呼叫時直接調用。

你也可以看看 Gerard Meszaros 的 `xUnit Test Patterns: Refactoring Test Code` 關於 Test stub 的講法，可以獲得更詳細和精準的解釋，我可能沒有辦法講得很好，因為我的英文沒有好到可以翻譯的很準確。

> 我沒辦法告訴你怎麼樣可以去看到這本書，但如果是我可能會先上網 google 之後在買書。

至於什麼時候要用 Stub，等等就會看到了！

# Mock 的解釋

Mock 也是一個假的對象，用來代替一個真實的對象，目的是為了監聽這個 Mock 上面被呼叫的方法。Mock 對象的主要工作是確保正確的方法被調用到他身上。

這和 Stub 明顯有一點點不一樣，但如果想更理解的話，去看書吧！

等等也會使用到 Mock！

# Mock & Stub 的不同

根據我的理解，用一個廣泛的說法，Stub 幫忙輸入， Mock 幫忙輸出。

Stub 是你做一個假的東西，直接插在你的程式碼裡面，騙他這是真的。

Mock 是你做一個假的東西，放在旁邊，在你程式碼不能直接測試的地方監視並且替代。

希望這樣簡單的說法能夠讓人比較快理解，但我相信這不是準確的答案，還是看看例子比較實際！

# 實際的應用程式例子

下面是一個 Ruby 的例子，讓這個例子能夠滿足 Mock & Stub。

我知道這例子可能漏洞百出，但我是為了示範 Mock & Stub 而寫的～

以下會有三個類別：

1. Payment： 模擬 ActiveRecord
2. ECpayKey： 模擬第三方支付
3. Logger： 紀錄支付情況

{{< tabbed-codeblock "Whole Example" >}}
<!-- tab ruby -->
class Payment
  attr_accessor :total_amount

  def initialize(ecpay_key, logger)
    @ecpay_key = ecpay_key
    @logger = logger
  end

  def save
    response = @ecpay_key.charge(total_amount)
    @logger.record_payment(response[:payment_id])
  end
end

class ECpayKey
  def charge(total_amount)
    p "這會觸發真正的付款"

    { payment_id: rand(1000) }
  end
end

class Logger
  def record_payment(payment_id)
    p "Payment id: #{payment_id}"
  end
end

RSpec.describe Payment do
  it 'records the payment' do
    ecpay_key = ECpayKey.new
    logger = Logger.new

    payment = Payment.new(ecpay_key, logger)
    payment.total_amount = 1800
    payment.save
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

我們的最下方的測試有兩個問題，他改變了資料( 這邊沒有真的寫出來，你可以假裝 `這會觸發真正的付款` 那段是真的向某人收了錢 )

另一個問題是，這個測試沒有驗證付款記錄有沒有被記錄下來，就算我把 `@logger.record_payment(response[:payment_id])` 給註解掉，測試還是會通過。

```ruby=
這會觸發真正的付款
Payment id: 334
.

Finished in 0.00255 seconds (files took 0.09594 seconds to load)
1 example, 0 failures
```

首先我們先用 Stub 來取代 ECpayKey 這件事情。

# Stub 的範例

下面我把 `ecpay_key = ECpayKey.new` 改成 `ecpay_key = double()`。

而且我告訴他，他會期望收到一個 `charge` 的方法，當它收到時，回傳一個 `3333` 的 ID


{{< tabbed-codeblock "Stub Example">}}
<!-- tab ruby -->
RSpec.describe Payment do
  it 'records the payment' do
    ecpay_key = double()
    allow(ecpay_key).to receive(:charge).and_return(payment_id: 3333)

    logger = Logger.new

    payment = Payment.new(payment_gateway, logger)
    payment.total_cents = 1800
    payment.save
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

這時候我們在 Run 一次測試，就不會再看到 `這會觸發真正的付款` 這樣的訊息，這是因為我們的測試沒有去 call ECpayKey 的 `charge` 方法，而是 call 了 Stub 物件身上的方法。

這時候我們的 `ecpay_key` 也不再是 ECpayKey 的實體，而是 `RSpec::Mocks::Double` 的一個實體了！

```ruby=
Payment id: 3333
.

Finished in 0.00877 seconds (files took 0.09877 seconds to load)
1 example, 0 failures
```

# Mock 的範例

現在讓我補上 紀錄支付資訊 的測試，我們把 `logger = Logger.new` 換成 `logger = double()`。

RSpec 本身沒有區分 mocks 以及 stubs 的分別。他們都是 Test Doubles，所以我們要怎麼區分，RSpec 讓我們自己決定～

我們還告訴了 Mock 物件，他必須接受一個值為 `3333` 的 record_payment 的方法！

{{< tabbed-codeblock "Mock Example">}}
<!-- tab ruby -->
RSpec.describe Payment do
  it 'records the payment' do
    ecpay_key = double()
    allow(ecpay_key).to receive(:charge).and_return(payment_id: 3333)

    logger = double()
    expect(logger).to receive(:record_payment).with(3333)

    payment = Payment.new(payment_gateway, logger)
    payment.total_cents = 1800
    payment.save
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

為了驗證 Mock 物件有沒有在監視這段程式碼，我們把 `@logger.record_payment(response[:payment_id])` 給註解掉，就會看到以下的錯誤訊息！

```ruby=
Failures:

  1) Payment records the payment
     Failure/Error: expect(logger).to receive(:record_payment).with(3333)
     
       (Double (anonymous)).record_payment(3333)
           expected: 1 time with arguments: (3333)
           received: 0 times
```

事實證明沒有錯，他確實有去監視是不是真的有執行這行程式碼，和 Stub 直接插入不一樣的做法！

# 小結

但講了這麼多，我其實也真的沒有在測試中寫過幾次的 Mock & Stub。

主要原因是因為也不是常常遇到只需要 Mock & Stub 就能解決的問題。在 Rails 中我們並沒有做到真正的 Unit Test，因為我們沒有真正的把一個物件和其他物件完全隔離開來進行測試，至少我現在還沒遇過，取而代之的是，我們寫的 Model spec 會去觸發資料庫，並且能夠訪問到應用程式裡面的每一個物件，這樣做是好是壞我不清楚，但這樣的方式基本上消除了對於 Mock & Stub 的需求。

另一個原因是，我自己覺得 Mock & Stub 的測試基本上就是重述了被測試的物件。像剛剛的程式碼 `@logger.record_payment`，測試寫 `expect(logger).to receive(:record_payment)`。

看起來真的有種我到底寫了什麼的感覺，這很像是在測試期待，而不是測試結果，但可以的話，我寧願只測試結果，而且在 Rails 裡面，大部分都可以測試結果，而不是測試期待～ 





