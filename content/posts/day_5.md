---
title: "Day 5 TDD 測試驅動開發"
date: 2021-09-21T17:10:58+08:00
metaAlignment: center
thumbnailImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211620030.png'
coverImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211620030.png' 
coverMeta: out
coverSize: partial
categories:
- "2021 鐵人賽"
tags:
- RSpec
summary: 介紹 RSpec 的核心價值！
keywords:
- TDD
- RSpec
---

{{< toc >}}

# 什麼是 TDD (Test-Driven Development) ?

是一種有別於傳統的開發方式，這邊說的是 **開發方式** 而不是測試方式喔！

這時候一定會想說，奇怪？我們主軸不是寫測試嗎？怎麼突然提到開發方式，是因為 RSpec 本身就是遵循的 TDD 的理念和精神而開發的，也是 RSpec 的核心價值。

所以對於 TDD 我們是不得不提，要來歌頌一下他的好處！

最直白的說明這個開發方式的流程就是：

1. 先寫測試代碼
2. 試著讓它通過測試
3. 解決重複以及優化程式碼！

直接上圖片，這樣更容易理解整個脈絡：

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211717219.png)

這就是 TDD 測試的核心，經由這三個步驟形成一個無限迴圈，也確保程式碼的品質能夠在這個迴圈中精進，簡化！

# 先寫測試代碼

從第一個步驟開始細講，要如何先開始寫測試代碼呢？

我們在工作上，會遇到客戶提出的需求以及功能，這時候我們就可以依照客人想要的功能來撰寫測試代碼！

我們主要的思考是要如何把需要做的事情寫成測試代碼，意思就是，我們讓這些測試代碼通過，就等於完成客戶的需求 ( 廣義 )。

當然這測試碼必須要寫得非常的詳細，考慮周全，站在使用者的角度，想想什麼東西應該要有，什麼其實不必要也不在需求內。

所以才會聽到有人說，**寫測試是在寫規格** 的原因就是這樣。

因為完成一段測試代碼的時候，等於是把所有的因素考慮進去，也就是一個功能的規格，他會有固定的 Output 有限制的 Input 也有例外地捕捉等等...

以上都屬於第一步驟，這邊的測試全部都不會通過，只是憑空想像，發揮你的想像力，讓物件像真實的世界一樣動起來吧！

以下進入 TDD 的情境，客戶來了一個需求 ( 只是假設，不會有這麼好笑的需求 )

> 我想要我的網站可以開商店
> 商店要有名字
> 商店能夠賣商品

接著我們撰寫關於這些需求的測試碼：

{{< tabbed-codeblock "RSpec 測試碼" >}}
<!-- tab ruby -->
RSpec.describe Store do
  it 'have name' do
    store = Store.new('Happy Store')
    expect(store.name).to eq('Happy Store')
  end
  
  it 'sale product' do
    store = Store.new('Happy Store')
    expect(store.sale).to eq('Earn lost of money')
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

接著在終端機輸入 `rspec .`

測試結果：

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211718113.png)


過程中，我們寫了一段關於客戶需求的測試碼，想當然一定不可能會通過。

因為我們還沒有寫真正的程式碼，但關於這個測試碼，破爛英文如我，也都大概看得懂，雖然不清楚詳細的語法，但猜一下也不難，就在於我們沒有一個 `Store` 的類別。

回頭思考一下，我們建立了一個商店，然後測試他的名字，以及他賣東西的動作，根據需求來撰寫，這是之後會一直強調的地方！

# 試著讓它通過測試

剛剛看到一片滿江紅，應該也不覺得是什麼好事，現在我們需要開始撰寫真實的程式碼來通過這一個測試。

接著我們撰寫程式碼來讓它通過：

{{< tabbed-codeblock "實際程式碼" >}}
<!-- tab ruby -->
class Store
  def initialize(name)
    @name = name
  end
  
  def name
    @name
  end
  
  def sale
    'Earn lots of money'
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

測試結果：

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211719016.png)


成功！

我們撰寫一個類別，然後可以讀取他的名字，也有販賣的動作。

測試代碼也成功地通過了，那接下來我們就要進入到第三個階段，重構的部分。

# 解決重複以及優化代碼

接著我們回頭看一下程式碼，看看有哪裡需要修正的，或是重複的測試代碼也可以，試著讓我們交出去的程式碼是乾淨且易讀的。

我就想到了，剛剛 `Store` 類別中的 `name` 方法，記得有個像是 `attr_reader` 的方法可以使用，這樣就可以三行變一行，快來試試看！

{{< tabbed-codeblock "重構實際程式碼" >}}
<!-- tab ruby -->
class Store
  attr_reader :name
  
  def initialize(name)
    @name = name
  end
  
  def sale
    'Earn lots of money'
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

還有測試代碼中 `Store.new` 也重複的寫了兩次，我們來讓他變得更精簡吧！

{{< tabbed-codeblock "重構測試程式碼" >}}
<!-- tab ruby -->
Rspec.describe Store do
  let(:store) { Store.new('Happy Store') }
  
  it 'have name' do
    expect(store.name).to eq('Happy Store')
  end
  
  it 'sale product' do
    expect(store.sale).to eq('Earn lots of money')
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

我們把重複的地方利用方法整理了一下，然後也把 `name` 方法用 Ruby
內建的小魔法處理了一下！

**！！！記住！！！**

重構結束之後也記得要在測試一次喔！

這時候寫測試的好處就來了，當你在重構代碼的時候，只要輕輕一按，就能知道剛剛的修正有沒有什麼錯誤的產生，我們快來試試看吧！

測試結果：

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211719016.png)


依然通過測試，代表這一個迴圈就暫時告一個段落，而且也依照客戶的需求完成了功能。

這就是 TDD 測試的流程，雖然範例蠻爛的，但礙於這篇只是想介紹 TDD 的價值和精神，所以就隨便的舉例，希望能夠讓你們看懂！

# 小結

明天開始，我們就會進入 RSpec 的語法介紹篇！

我會從基本的方法介紹起來，也會搭配範例服用，那就明天見啦！





