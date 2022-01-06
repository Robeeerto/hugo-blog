---
title: "Day 10 實用的 let 方法以及客製化錯誤訊息"
date: 2021-09-24T18:50:55+08:00
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

介紹實用的好方法，順便客製化你的錯誤訊息！
<!--more-->
{{< toc >}}
# 改變數值的時候

昨天提到變動性的問題是什麼呢？

我們到現在的測試都是很單調的，也就是測一個物件本身的屬性，那如果我們實作的功能會改變屬性呢？我們還是可以通過測試嗎？

我們用昨天實作的 Ruby 方法來試試看，遇到變動會發生什麼事吧！

{{< tabbed-codeblock "測試變動">}}
<!-- tab ruby -->
RSpec.describe Burger do
  def burger
    Burger.new('Beef', 'Cheddar')
  end
  
  it "has cheese" do
    expect(burger.cheese).to eq('Cheddar')
    burger.cheese = 'Ricotta'
    expect(burger.cheese).to eq('Ricotta')
  end
  
  it "has meat" do
    expect(burger.meat).to eq('Beef')
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

上述測試代碼的第 8.9 行是新增上去的，根據語意可以知道我們想要改變起司的種類，在測試他的種類是不是我們新增上去的那一個！

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109241853881.png)

咦！竟然沒有通過，可以看到錯誤訊息中，我們期待得到 Ricotta 但卻拿到 Cheddar

原因就出在於，我們呼叫的是一個方法，每呼叫一次都是一個新的物件誕生，他並不會存活在 block 裡面

所以我說用 Ruby 的方法寫沒有不行，但遇到這種會改變數值的狀況就會很麻煩～

那我們既不想要用 `before hooks` 這麼大包的東西還要轉換成實體變數，又希望物件可以存活在 example 裡面！

有這麼好康的東西嗎？當然！

# let helper method

提到 `let` 方法之前，需要提一下 Memorization 的概念，記憶的過程嗎？詳細怎麼翻譯我不太清楚，但我想講述一下概念！

今天老師上課給了你一個很難的數學題目，你花了一個小時做完，也告訴老師答案了，過了 10 分鐘，老師又給了你一模一樣的題目，你還會花一個小時做完嗎？

不，你會直接給他答案！這就是 Memorization 的概念，而 `let` 方法就有一點點這樣的概念存在喔！

我們先上修改好的程式碼：

{{< tabbed-codeblock "使用 let">}}
<!-- tab ruby -->
RSpec.describe Burger do
  let(:burger) { Burger.new('Beef', 'Cheddar') }
  
  it "has cheese" do
    expect(burger.cheese).to eq('Cheddar')
    burger.cheese = 'Ricotta'
    expect(burger.cheese).to eq('Ricotta')
  end
  
  it "has meat" do
    expect(burger.meat).to eq('Beef')
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

完成了！而且可以正確地通過測試，讓我娓娓道來！

`let` 的後方括號內要放的是你想要命名的變數，這邊我們放的是 :burger 記得要使用符號的形式，而後方的 block 中則是你要執行的程式碼！

```ruby=
let(:burger) { Burger.new('Beef', 'Cheddar')}
```

他會把程式碼執行完的結果，放到括號內的符號裡，而你在使用它的時候就像是區域變數一樣，超級無敵方便的啦！

還有更棒的地方就是，他會根據每一個 example 初始化，所以不需要擔心物件被污染的問題。

所以我們第一次呼叫 `let` 是在第 5 行的地方，第二次則是在第 11 行的地方，根據剛剛提到過的 Memorization，在同一個 example 中，他都會是同一個物件，也不會再次的呼叫 `let` 方法！

可以來測試看看，我們把 burger 給印出來

{{< tabbed-codeblock "同一個物件">}}
<!-- tab ruby -->
RSpec.describe Burger do
  let(:burger) { Burger.new('Beef', 'Cheddar') }
  
  it "has cheese" do
    expect(burger.cheese).to eq('Cheddar')
    p burger
    burger.cheese = 'Ricotta'
    expect(burger.cheese).to eq('Ricotta')
  end
  
  it "has meat" do
    expect(burger.meat).to eq('Beef')
    p burger
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

可以看到結果儲存的記憶體位置是不相同的！

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109241855701.png)


還記得 `before hooks` 其實和這個很像，但他是在每一個 example 前都會先執行，也就是說即使你的 example 內沒有使用到，他也會執行！

但 `let` 有個很厲害的功能 Lazy-loading，也就是說當我們呼叫這個變數的時候，他才去執行這段程式碼。

我們也來測試看看 `before hooks` 和 `let` 到底差別在哪？

{{< tabbed-codeblock "before & let 的差別">}}
<!-- tab ruby -->
RSpec.describe Burger do
  let(:burger) { p 'let method' }
  before do
    p 'before hooks'
  end
  
  it "has cheese" do
    burger
  end
  
  it "has meat" do
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

我們只有在一個 example 內呼叫了 burger 這個變數，而 burger 這個變數的內容是印出 let method 這段文字！而 `before hooks` 則是印出 before hooks 這段文字，讓我們來看看結果吧。

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109241856349.png)

`before hooks` 在每一個 example 前都自動執行了，不管你要或不要，但 `let method` 卻只有在我們呼叫的 example 執行，另外一個則不執行！

如果我們確認這個東西是所有的 example 都需要使用的話，也可以用 `let! method`，但如果只有零星幾個，用 `let method` 可以增加效能，而且更短更好看，也不用使用實體變數～

至於 `let! method`，加了一個驚嘆號就代表說他會在所有的 example 前執行，就像 `before hooks` 一樣，所以我說 `let` 真的非常強大也非常好用，基本上用這個就可以應付大多數的測試了。

# 客製化你的錯誤訊息

我們常見的錯誤訊息：
 
![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109241856492.png)

但我們可以客製化我們想要的消息在上面，直接上程式碼再來講解！

{{< tabbed-codeblock "客製化錯誤訊息">}}
<!-- tab ruby -->
RSpec.describe Burger do
  let(:burger) { Burger.new('Pork', 'Cheddar')}

  it "has cheese" do
    expect(burger.cheese).to eq('Cheddar')
    burger.cheese = 'Ricotta'
    expect(burger.cheese).to eq('Ricotta')
  end
  
  # 修改肉變成 Pork 來故意出錯，顯示客製的訊息～
  it "has meat" do
    comparison = "Beef"
    expect(burger.meat).to eq(comparison), "我想要 #{comparison} 而不是 #{burger.meat}"
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

測試出錯結果：

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109241858257.png)

還記得我們有提過 `to` 是一個方法，而他除了一定要接受 `expection` 之外，也可以增加一些額外的選項，例如：客製化錯誤。

可以想像原本是這樣：

```ruby=
expect(burger.meat).to(eq(comparison), "我想要 #{comparison} 而不是 #{burger.meat}")
```

只是因為可以省略小括號，讓人看起來有點不直覺～但這就是 Ruby 的特色喔！

至於 `#{}` 這個寫法是很常見的在字串中安插變數的方法，也是 Ruby 很基本的使用方法。

# 小結

呼～今天講了很重要的 let 方法，個人覺得超級好用！ 

明天我們要開始介紹 `context` 這個方法以及 箝套的 `describe`

> 箝套就是 Nested，像洋蔥一樣一層一層包下去的概念






