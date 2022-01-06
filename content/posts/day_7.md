---
title: "Day 7 從閱讀錯誤到通過測試！"
date: 2021-09-21T22:36:26+08:00
metaAlignment: center
coverImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211620030.png' 
coverMeta: out
coverSize: partial
categories:
- "2021 鐵人賽"
tags:
- RSpec
---

來修改昨日的錯誤吧！
<!--more-->
{{< toc >}}
昨天我們的測試竟然沒有通過...

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109212238767.png)


到底發生了什麼？該怎麼讓它通過？那串紅紅的字看起來就很不順眼，我們該怎麼做呢？放心！今天就會讓你感受的綠燈的療癒～

# 閱讀錯誤到通過測試

我們來看看他說我們錯誤的地方在哪，首先是這串：

{{< tabbed-codeblock "錯誤訊息" >}}
<!-- tab ruby -->
Failure/Error: burger = Burger.new('Big mac')
<!-- endtab -->
{{</ tabbed-codeblock>}}

但好像沒有告訴我們錯在哪裡耶？喔喔！我們再往下看

{{< tabbed-codeblock "錯誤訊息" >}}
<!-- tab ruby -->
NameError:
  uninitialized constant Burger
<!-- endtab -->
{{</ tabbed-codeblock>}}

他的意思是我們並沒有初始化一個常數 `Burger`

這就是 Ruby 或是 寫 Rails 時超級常見的錯誤，請筆記！ 

我們需要擁有一個類別，然後呼叫 `initialize` 這個方法，這樣我們才可以生出 `burger` 這個物件！

我們要寫在哪裡呢？其實可以直接寫在測試的上面，但這並不是一個好方法，我們在寫測試的時候會希望把每隻檔案分開的放置，這樣才是看起來專業的方法！

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109212240269.png)

現在這是我們的檔案結構，這樣也很明顯地可以理解到，`burger_spec.rb` 就是拿來測試 `burger.rb` 這支檔案的！

接著我們在 `burger.rb` 中寫入：
{{< tabbed-codeblock "撰寫程式碼" >}}
<!-- tab ruby -->
class Burger

end
<!-- endtab -->
{{</ tabbed-codeblock>}}

再輸入一次 `rspec spec/burger_spec.rb`，看看錯誤的訊息是什麼？

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109212240984.png)

雖然還是錯誤的還是同一行，但錯誤的內容已經不一樣了，我們繼續來解決這個問題，他說我的給予的參數是錯誤的，代表說我們需要在類別初始化的時候加入參數，來讓這個錯誤訊息通過，實作吧！

{{< tabbed-codeblock "撰寫程式碼" >}}
<!-- tab ruby -->
class Burger 
  def initialize(type)
    @type = type
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

好的，我們一樣在執行一次 `rspec spec/burger_spec.rb`

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109212241107.png)

通常看到錯誤訊息有所改變的時候，都會覺得鬆一口氣的感覺，至少事情產生變化了～ 

因為我們又更近了一步了！代表每一次的改動都有正中紅心，也有根據錯誤訊息來做正確的改動！

這次的錯誤也是很常見的 `NoMethodError` 意指我們沒有 `type`  這個方法，如果是別的語言來的人可能會覺得很奇怪，取得屬性還要什麼方法？

事實上在 Ruby 的世界裡都是物件，而取值也會是物件的其中一個方法，包括設置值也是喔！

{{< tabbed-codeblock "撰寫程式碼" >}}
<!-- tab ruby -->
class Burger 

  def initialize(type)
    @type = type
  end
  
  def type # 這個是取值的方法
    @type
  end
  
  def type=(type) # 這個是賦值的方法
    @type = type
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

來看看我們把取值的方法加上去之後會發生什麼事情吧？

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109212242334.png)

登愣登愣！！成功啦！我們通過這次的測試了！

但別高興得太早，還記得 TDD 測試中通過綠色後要進行的環節嗎？

沒錯，就是重構的部分，這也是非常重要的！

我們這時候回頭看一下我們“測試碼”以及“真實程式碼”有沒有什麼需要改進的地方。

測試的部分因為只有兩行，好像沒什麼可以縮減的部分了，我們就看看“真實程式碼”吧！

關於取值、賦值兩個方法，Ruby 其實有提供我們更便捷的方式來用，不需要這樣子寫到六行，讓我們來重構看看這段程式碼吧！

{{< tabbed-codeblock "重構程式碼" >}}
<!-- tab ruby -->
class Burger 
  attr_accessor :type
  
  def initialize(type)
    @type = type
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

這個的效果，和我們原先寫的方式是一樣的喔！但礙於這個主要是在介紹 RSpec，關於 Ruby 的基礎我們就不提到太多！主要還是以通過測試的邏輯為主。

# 小小測驗區

從第一天到現在也說了蠻多的東西了，有一些小測驗希望可以讓剛學的人回想一下，自己想的是錯誤還是正確！

1. 哪一個方法可以在 RSpec 中建立一個 `example` ?
2. 哪一個方法可以在 RSpec 中建立一個 `example group` ?
3. 哪一個方法可以在 RSpec 中建立一個 `assertion` ? ( 類似主張、斷言的感覺)
4. `expect.eq(1 + 1, 2)` 這是正確的寫法嗎？如果是錯的，為什麼？
5. 我想要建立一個 `example`，只要打開一個 `_spec.rb` 的檔案直接寫就可以了嗎？
6. `describe` 語法一定要有參數才可以運作嗎？
7. 測試的順序是什麼？( `E2E Test, Integration Test, Unit Test` )
8. 要初始化一個 Rspec 測試的終端機指令是什麼？
9. 你應該寫測試在你寫 Code 之前嗎？為什麼？

**這些答案在之前的文章都有提過喔！**

# 小結

明天會在一個添加許多的 `example` 在一個測試之中，順便也訓練我們思考 TDD 的思緒，然後繼續的面對錯誤，修正，重構，這樣自然而然就會越來越有感覺～






