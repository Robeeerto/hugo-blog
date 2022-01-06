---
title: "Day 18 Matcher 介紹 (下)"
date: 2021-10-02T16:05:36+08:00
metaAlignment: center
coverImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211620030.png' 
coverMeta: out
coverSize: partial
categories:
- "2021 鐵人賽"
tags:
- RSpec
---

Matcher 最終章！！！
<!--more-->
{{< toc >}}

今天將一口氣把剩下比較常用的 `Matcher` 一併介紹起來！

# have_attributes matcher

這個 `Matcher` 很清楚明瞭的告訴你是來測驗物件的屬性的！

我們直接用範例來介紹一下～

{{< tabbed-codeblock "have_attributes">}}
<!-- tab ruby -->
class Burger
  attr_reader :meat, :cheese
  
  def initialize(meat, cheese)
    @meat = meat
    @cheese = cheese
  end
end

RSpec.describe "have_attributes matcher" do
  
  describe Burger.new("Beef", "Cheddar")
    it "checks attribute and value" do
      expect(subject).to have_attributes(meat: "Beef")
      expect(subject).to have_attributes(cheese: "Cheddar")
    end
    
    it { is_expected.to have_attributes(meat: "Beef") }
    it { is_expected.to have_attributes(cheese: "Cheddar") }
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

我們針對漢堡這個類別來進行測試，分別檢驗他的 `meat` & `cheese` 屬性所給予的值！

進而判斷這個屬性是否存在，其實這個真的很簡單，也沒有真的比較特殊的寫法，所以就這樣簡單的用範例來介紹一下～

# include matcher

接下來這個對於有學過 Ruby 的人來說，應該是超級好用的一個語法！

而在 RSpec 中也是異曲同工之妙，就是檢查 `字串` `陣列` `Hash` 中，所包含的字串、數字或是 `key & value` 這樣的用法！

我們就用範例來更清楚的了解一下吧！

{{< tabbed-codeblock "include matcher">}}
<!-- tab ruby -->
RSpec.describe "include matcher" do

  describe "yummy burger" do
    it "checks substring" do
       expect(subject).to include("yummy")
       expect(subject).to include("burger")
       expect(subject).to include("bur")
    end
  end
  
  describe [1, 5, 7] do
    it "checks inclusion value in the array" do
      expect(subject).to include(1) 
      expect(subject).to include(5) 
      expect(subject).to include(7) 
      expect(subject).to include(7, 5) 
    end
  end
  
  describe { t: 5, y: 7} do
    it "checks key or key value pair" do
      expect(subject).to include(:t) 
      expect(subject).to include(:t, :y)
      expect(subject).to include(t: 5)
    end
    
    it { is_expected.to include(:y)}
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

這個的用法也很簡單，有個重點就是他不看順序的，主要是看內容物來覺得測試通過與否！

# raise_error matcher

這個 `Matcher` 就是拿來測試噴錯的結果的，沒錯！工程師當然要能夠預期錯誤的結果 ( 雖然工作上大部分都沒有預期到... )

如果是寫在 Rails 的專案中，確實還蠻常使用到的，在某些特殊的情況需要加入 `begin...rescue` 時，就會寫到了～

那我們還是來簡單的示範一下狀況！

{{< tabbed-codeblock "raise error">}}
<!-- tab ruby -->
RSpec.describe "raise error matcher" do
  
  def wrong_method 
    gggg
  end

  it "check error being raised" do
    expect { wrong_method }.to raise_error
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

首先講寫法上面，記得我們之前提過，若是要執行方法等等，要記得使用 `{ }` 來做執行的動作～

再來是你能預期這個方法會噴什麼樣的錯誤嗎？

我們先看看測試的結果：

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202110021610541.png)


通過了測試，但是官方希望你可以更詳細的解釋這是一個什麼樣的 Error，所以我們只要在後方加上：

```ruby=
expect { wrong_method }.to raise_error(NameError)
```

這樣就可以看到正常的 Output 了，當然你也可以在設定檔裡面關閉這個提醒，但還是寫的仔細好一些～

甚至我們還可以自己建造錯誤類別來繼承原本的錯誤類別，執行自己的錯誤哈哈

{{< tabbed-codeblock "raise error">}}
<!-- tab ruby -->
class CustomError < StandardError; end

it "rails himself" do
  expect { raise CustomError }.to raise_error(CustomError)
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

# respond_to matcher

這個 `Matcher` 主要在測試一個 API 的接口所回應的方法！

也可以說成類別啦，一個類別會有的方法，以及所需的參數等等～

我們看看範例就能了解囉～我們先建造一些類別來使用！

{{< tabbed-codeblock "respond_to matcher">}}
<!-- tab ruby -->
class Burger
  def eat
   p "yummy!!!!" 
  end
  
  def discard
   p "So bad..." 
  end
  
  def buy(money)
   p "i will spend #{money} to buy this good shit!!" 
  end
end


RSpec.describe "respond_to matcher" do
  
  describe Burger do
    it "checks an object respond method" do
      expect(subject).to respond_to(:eat) 
      expect(subject).to respond_to(:eat, :discard)
      expect(subject).to respond_to(:eat, :discard, :buy)
    end
    
    it "checks method and arguments" do
        expect(subject).to respond_to(:buy).with(1).arguments 
    end  
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

寫起來有點多，但其實就是測試這個類別回應的公開方法而已，注意！！私有方法是沒辦法被 `respond_to` 的喔～

畢竟都已經是 `private` 還可以公開的回應，那好像也怪怪的！

# satisfy matcher

這個 `Matcher` 超級好用，而且彈性超大～

不像前幾個 `Matcher` 基本上都是 Ruby 的語法轉型變成 RSpec 的語法，這個是獨有的！！！

我們先看看示範：


{{< tabbed-codeblock "satisfy matcher">}}
<!-- tab ruby -->
RSpec.describe "satisfy matcher" do
  subject { 'level' }
  
  it "is a palindrome" do
    expect(subject).to satisfy { |value| value == value.reverse } 
  end
  
  it "can do something in block" do
    expect(10).to satisfy { |value| value / 2 == 5 } 
  end
  
  it "can add custom message" do
    expect(100).to satisfy("bigger than 50") do |value|
      value > 50
    end
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

他就是一個只要能夠在 block 裡面滿足條件，就可以通過測試的一個好東西！

彈性很高，可以在裡面寫入很多 Ruby 的語法，在測試的時候也比較不會綁手綁腳的。

客製化訊息也會在 Output 上一併出現，這個 `Matcher` 先推推！

# compound expectation

這個就是一個 `expectation` 而且蠻無聊的，想說就也提一下好了！

簡單來說就是合成你的 RSpec 語法，要 `and &&` 或是 `or ||` 對比上你的複數 `Matcher` 才能夠通過～

{{< tabbed-codeblock "compound expectation">}}
<!-- tab ruby -->
RSpec.describe 30 do
  it "test for multiple matchers" do
    expect(subject).to be_even.and be > 25
  end
  
  describe [1, 5, 7] do
    it "can use or" do
      expect(subject.sample).to eq(1).or eq(5).or eq(7)  
    end
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

就如範例所示，可以串接你的 `Matcher` 然後使用 `and` 或是 `or` 方法來串聯！

# 小結

終於結束了滿滿的 `Matcher` 海，接下來就進入到一開始我最難理解的 `double` 以及 `stub` 的章節了！

也是我覺得超級好用的地方，真的能夠辦到很多有趣的事情！





