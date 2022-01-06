---
title: "Day 9 整理重複煩人的程式碼！"
date: 2021-09-23T19:00:06+08:00
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

重複的程式碼該怎麼做？
<!--more-->
{{< toc >}}

昨天我們實作了把 `example` 給拆開，並且讓整個測試更具備邏輯。

今天我們要讓程式碼更乾淨，解決一些不斷重複的程式碼，在我剛開始學習時，覺得多打一兩行也還好吧。

到現在要我打移動距離去改三個字都讓我覺得痛苦，如果今天放大成 100 倍呢？我相信如果一個小小的改動要連續做一百次，應該是不會有人願意吧哈哈！

# Before hooks

下面是我們昨天實作的測試碼，接著我們要來重構它。

重構的要點在於減少不必要的程式碼，加速效能，讓他看起來更舒服！

{{< tabbed-codeblock "昨日的測試碼">}}
<!-- tab ruby -->
RSpec.describe Burger do
  it "has cheese" do
    burger = Burger.new('Beef', 'Cheddar')
    expect(burger.cheese).to eq('Cheddar')
  end
  
  it "has meat" do
    burger = Burger.new('Beef', 'Cheddar')
    expect(burger.meat).to eq('Beef')
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

這段程式碼，最需要改動的地方就在於第 3, 8 行，因為是完全一模一樣的程式碼，所以可以使用我們要介紹的 `Before hooks` 來解決這件事情！

先上完成的程式碼：

{{< tabbed-codeblock "完成的程式碼">}}
<!-- tab ruby -->
RSpec.describe Burger do
  before do
    @burger = Burger.new('Beef', 'Cheddar')
  end
  
  it "has cheese" do
    expect(@burger.cheese).to eq('Cheddar')
  end
  
  it "has meat" do
    expect(@burger.meat).to eq('Beef')
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

上面的程式碼會得到 PASS！ 

接著我們來解釋這到底是什麼作用，首先這個 `before` 的原型應該是這樣：

{{< tabbed-codeblock "before 的原型">}}
<!-- tab ruby -->
before(:example) do
  ...
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

意思就是指，在每一個 `example` 前，做這件事情！應該是一個很好懂的方法，也非常實用，當然裡面的 `:example` 可以做改變，這是更進階的寫法，之後會再提到！

所以我們完全不寫參數給他的話，他就會自動的認定要在每一個 `example` 前執行 block 裡面的內容！

這邊示範為什麼要提到每一個 `example` 呢？

我們在 `before` 的 block 裡面加上一段會印出來的文字。

{{< tabbed-codeblock "before 的原理">}}
<!-- tab ruby -->
RSpec.describe Burger do
  before do
    p 'Hello, this is before hooks'
  end
  
  it "has cheese" do
    burger = Burger.new('Beef', 'Cheddar')
    expect(burger.cheese).to eq('Cheddar')
  end
  
  it "has meat" do
    burger = Burger.new('Beef', 'Cheddar')
    expect(burger.meat).to eq('Beef')
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

接著我們來看 Out put !

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109231905207.png)

很明顯地，他在每一個 `example` 前執行！

所以先前的測試代碼，在進入每一個 `example` 前，都會有一個實體變數 `@burger` 產生並一同加入 `example` 裡面！

那至於為什麼要使用 `@burger` 而不是 `burger` 呢？ 

這就牽扯到 Ruby 變數的作用域問題，我們不深入的討論這件事，但大致上來說，我們可以把一個 block 想像成一個新的領地，若你不加上 `@` 這個符號成為實體變數，你是沒有穿梭 block 的能力！

# 用原始的 Ruby Method

上面的方法是 RSpec 提供給我們的方法，但 RSpec 是可以直接寫 Ruby Code 的，所以我們動動小腦筋，要怎麼做到和 `before hook` 一樣的效果呢？

上代碼：

{{< tabbed-codeblock "Ruby Method">}}
<!-- tab ruby -->
RSpec.describe Burger do
  def burger
    Burger.new('Beef', 'Cheddar')
  end
  
  it "has cheese" do
    expect(burger.cheese).to eq('Cheddar')
  end
  
  it "has meat" do
    expect(burger.meat).to eq('Beef')
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

不要懷疑喔！這樣得程式碼也會過關喔！

但不熟悉 Ruby 的人一定想說你在寫三小... 但這真的會動，雖然不會有人這樣寫，因為後續會造成特殊的問題，但這也是一個方法啦，如果你確定你的數值不會有任何的改變，還是用 `before hooks` 吧～ 

他在讀取到第 7 行的時候呼叫了 `burger` 這個我們定義的方法，而在 Ruby 中最後一行可以不用寫 `return`， 所以他會自動的 return 一個 burger 物件。

所以我們可以這樣看第 7 行

```ruby=
expect(Burger.new('Beef', 'Cheddar').cheese).to eq('Cheddar')
```

這是他會動得原因，但不要這樣寫，只是稍微提一下！因為這樣真的會有問題～

# 小結

今天介紹了減少程式碼的好方法 `before`，這將會在之後的 RSpec 生涯中偶爾地出現！

什麼？你說這才偶爾，那幹嘛教呢？

哎唷，總是什麼方法都要看過才好啊！你才知道什麼是好，什麼是不好，像今天示範的用原始 Ruby 寫的方法，也會動啊！

但會遇到變動性的問題，所以明天將會介紹在 RSpec 裡面，遇到變動性問題該怎麼解決？以及最強大的方法 `let` 將會在明天一併介紹起來～




