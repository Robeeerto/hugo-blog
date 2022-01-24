---
title: "Day 8 超多的範例？怎麼辦呢？"
date: 2021-09-22T20:32:53+08:00
metaAlignment: center
thumbnailImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211620030.png'
coverImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211620030.png' 
coverMeta: out
coverSize: partial
categories:
- "2021 鐵人賽"
tags:
- RSpec
summary: 好多的範例，怎麼辦才好？
keywords:
- RSpec
- Syntax
- Assert
---

{{< toc >}}

昨天我們做了一個關於漢堡種類的測試，但真正的測試怎麼可能這麼少呢！

所以我們今天要更進一步的來把它變得稍微複雜一點，並且讓他看起來更乾淨，更好一些！

# Describe 後面改用常數吧！

{{< tabbed-codeblock "漢堡的測試" >}}
<!-- tab ruby -->
RSpec.describe Burger do
  it "has a type" do
    burger = Burger.new('Big mac')
    expect(burger.type).to eq('Big mac')
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

還記得我們兩天才撰寫測試的時候，`describe` 後面加入的是 `"Burger"` 字串，但我們現在可以改用常數，也就是我們測試類別的名稱。

只是少了兩個括號有什麼差別嗎？當然有，他的好處可以讓我們少寫很多的程式碼，我們後面會提到直接使用 常數 到底有哪些優勢！(關鍵字：`subject` )


# 一個蘿蔔一個坑

我們現在的測試只有單單測試一個漢堡的種類，但我們現在要讓他更複雜一些。

我們想要讓漢堡可以有肉類和起司的選項，我們要把大麥克給拆開來看看！

以下為逐步圖，從原本的作法一步一步拆開來！

{{< tabbed-codeblock "原本的狀況" >}}
<!-- tab ruby -->
RSpec.describe Burger do
  it "has a type" do
    burger = Burger.new('Big mac')
    expect(burger.type).to eq('Big mac')
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

這邊我們希望漢堡可以放入兩種種類，分別是牛肉、以及切達起司！

我們一樣保持先把測試給完成再去改你的程式碼，漸漸地習慣這種流程，不要急著去改，這樣就喪失了 TDD 的意義了！

{{< tabbed-codeblock "第一步" >}}
<!-- tab ruby -->
RSpec.describe Burger do
  it "has a type" do
    burger = Burger.new('Beef', 'Cheddar')
    expect(burger.type).to eq('Big mac')
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

下面的寫法看起來非常合理，但卻有些瑕疵。

在 RSpec 裡面我們會希望測試可以越乾淨越好，一個 `example` 盡量只有一個 `expectation` 這樣會讓我們可以把功能切的更細、考慮到更多，尤其在 Output 也會完全不一樣！

而且這邊的敘述也不夠精確， `has a type` 但卻沒有說是什麼樣的類型，當測試的數量不斷地增加的時候，這就會造成你在看錯誤訊息時更大的負擔！

{{< tabbed-codeblock "第二步" >}}
<!-- tab ruby -->
RSpec.describe Burger do
  it "has a type" do
    burger = Burger.new('Beef', 'Cheddar')
    expect(burger.meat).to eq('Beef')
    expect(burger.cheese).to eq('Cheddar')
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

下面是我們把 `expectation` 拆成兩個 `example` 的樣子！

也把兩個 `example` 的敘述變得更仔細，讓人可以一目瞭然這個測試的目的是什麼！

有沒有覺得畫面變得更清楚啦？雖然我們的例子都很短，你可能覺得放一起也沒關係，但養成好習慣會讓以後事半功倍喔！

{{< tabbed-codeblock "第三步" >}}
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


好的，現在我們的測試已經根據新的需求有了改變，想當然我們輸入測試的指令一定不會通過的。

接著又來到 TDD 的第二個步驟了，我們要讓這個測試 PASS ！

# 閱讀錯誤再次通過測試！

輸入了 `rspec spec/burger_spec.rb` 後

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109222040721.png)

我們再來閱讀這個錯誤的問題在哪，錯誤的地方在於類別的 `initialize` 方法這邊。

錯誤的問題則是下方的 `ArgumentError` 可以看到他寫到只期待一個參數，但我們卻給了兩個參數。

好的沒問題，我們來到 `burger.rb` 存放類別的檔案，稍作修改一下！

{{< tabbed-codeblock "更改程式碼" >}}
<!-- tab ruby -->
class Burger
  attr_accessor :type

  def initialize(meat, cheese)
   @type = type 
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

我們給了他兩個參數了，接著我們在輸入 `rspec spec/burger.rb` 來看看錯誤的訊息是什麼？


![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109222042446.png)


喔！這次發生錯誤的地方在測試的檔案中，原因是我們的 `Burger` 沒有 `meat` 這個方法，昨天也有提到這樣的問題該如何解決！簡簡單單～

給他兩個方法讓他可以讀取就好啦～

{{< tabbed-codeblock "更改程式碼" >}}
<!-- tab ruby -->
class Burger
  attr_accessor :meat, :cheese

  def initialize(meat, cheese)
   @type = type 
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

肯定會通過的吧？

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109222043486.png)


啊...太大意了，我們應該要注意到實體變數的問題，目前這個物件裡面沒有存放肉和起司的變數，快加給他！

{{< tabbed-codeblock "更改程式碼" >}}
<!-- tab ruby -->
class Burger
  attr_accessor :meat, :cheese

  def initialize(meat, cheese)
    @meat = meat
    @cheese = cheese
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

恭喜啊！我們又通過了一次測試了！

記得我剛剛前面有提到拆分 `example` 的重要性嗎？

看看這邊的 Output 就很明顯像是作文一樣，告訴我們漢堡有肉有起司，但若是你全部都寫在一個 `example` 裡面，就會變得不清不楚了～

> 一個 `it` 代表的是一個 `example` 喔，這會關係到測試的可讀性！所以不是越省越好喔～

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109222043979.png)

# 小結

記得今天的開頭，我有提到先把 `describe` 後面的字串描述改成類別的常數吧？

仔細看看，今天我們的測試碼就有兩行是一模一樣的重複代碼，TDD 的第三步就是要重構程式碼以及測試代碼！

明天我將會介紹整理 RSpec 的方式，有許多的小幫手能夠讓我們的測試碼更簡潔，更易讀！





