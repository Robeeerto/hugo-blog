---
title: "Day 11 用 Context 來組織你的測試區塊"
date: 2021-09-25T19:54:22+08:00
metaAlignment: center
coverImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211620030.png' 
coverMeta: out
coverSize: partial
categories:
- "2021 鐵人賽"
tags:
- RSpec
---

使用 Context 來組織程式碼～
<!--more-->
{{< toc >}}
再開始介紹 `Context` 之前，想先推薦大家一個很棒的網站！

裡面有介紹 RSpec 撰寫的慣例，分別介紹了好的寫法和壞的寫法，當然這都是一個大多數人的共識，還是以自己工作的團隊一致認同的 Coding Style 為準啊～

[Better Specs](https://www.betterspecs.org/)，有空都可以看一下，通常會是 Good 的寫法都有一些道理存在，容易閱讀、語意清晰等等。

好啦～進入今天的正題：

# Context Method

首先，我們想要測試 `even?` 這個方法，可以看到我們在 `even?` 前方加上了一個 # 的符號，這是為了要區別類別方法以及實體方法的小記號，若是測試實體方法會使用 `.some_method method` 的寫法！

也是一個小小的共識！

可以看到我們測試的項目非常的簡單好懂，就是期待奇偶數帶來不同的結果，這樣寫有什麼不對的地方嗎？

沒有，但可以更好！

{{< tabbed-codeblock "測試方法" >}}
<!-- tab ruby -->
RSpec.describe "#even? method" do
  it 'should return true if number is even' do
    expect(...)
  end
  it 'should return false if number is odd' do
    expect(...)
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

為了讓語意更清晰，更容易閱讀，就有了箝套的概念，就像是你電腦的資料夾一樣，會整理相同邏輯的東西再一起，名字是圖片的資料夾裡面不會有 PPT 的報告，所以我們要來讓測試項目被分類，用有意義的敘述！

{{< tabbed-codeblock "加入 Context" >}}
<!-- tab ruby -->
RSpec.describe "#even? method" do

  context "with even number" do
    it "should return true" do
      expect(6.even?).to eq(true)
    end
  end
  
  context "with odd number" do
    it "should return false" do
      expect(9.even?).to eq(false)
    end
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

完成！這樣就可以把剛剛很長又不容易讀的東西進行分類，也可以在某個 `context` 之下做延伸，例如當數字是奇數會發生...的測試。

就不會所有的測試都混成一團，看起來就沒什麼脈絡，還有另外一個好處，就是在終端機的 Output 會是一個箝套狀，看起超讚的！

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109251958849.png)

而有些時候，你會看到有人用 `describe` 來替代 `context`，也是完全可以的，兩者在實際做用上沒有什麼差別。

但大多數的人喜歡使用 `context` 在 `describe` 內，因為看起來才有區別的感覺，不然會滿滿整篇都是 `describe`

我們也可以看看 Better Specs 中有提到的：

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109251959870.png)


整個 `context` 方法最重要的就是組織程式碼，使其易讀，且當你使用 `context` 時，請用 `when` `with` `without` 來做開頭！

解釋一個情境在測試的環境下也是很重要的，雖然對於非英語母語人士的我來說覺得好像沒什麼差別，但我自己是喜歡看到被組織起來的感覺。

# 小結

今天沒有帶到太多的東西，但提供了一個我自己覺得很棒的網站，可以透過閱讀別人的 Coding Style 來提升自己寫 Code 的想法！

當你今天學會了一種寫法，你下次就會用那種思考方式來寫，一直不斷地提升自己，但如果一直只寫會動的程式碼，就會停滯不前～

明天會先介紹 `before` & `after` 的差別，以及作用域的範圍，hooks 的先後順序等等。

其實開始學習程式後，理解生命週期也是超重要的一環！可以讓你理解整個流程的順序，這樣才知道是哪裡出錯、該從哪裡修理～
