---
title: "Day 15 Matcher 基礎三兄弟！"
date: 2021-09-29T17:17:01+08:00
metaAlignment: center
thumbnailImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211620030.png'
coverImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211620030.png' 
coverMeta: out
coverSize: partial
categories:
- "2021 鐵人賽"
tags:
- RSpec
summary: 解釋一下，eq 間微小的差別！
keywords:
- RSpec
- matcher
---

{{< toc >}}

今天會開始介紹許多好用的 `Matcher` 也就是配對仔，從開始到現在我們都只使用過 `eq` 這種用法！

其實還有很多有趣的 `Matcher` 可以使用！

今天會介紹三種不同的 `Matcher` 但又很相似的，分別是 `eq` `eql` `be` ！

# eq Matcher

`eq` 是我們最常使用的 `Matcher`，其概念也非常簡單，就是單純的比較 value，而不比較 type ！

我們可以用一些示範來比喻一下！

{{< tabbed-codeblock "範例" >}}
<!-- tab ruby -->
RSpec.describe "eq matcher" do
  let(:a) { 5.0 }
  let(:b) { 5 }
  
  it 'only tests for value' do
    expect(a).to eq(5)
    expect(b).to eq(5.0)
    expect(a).to eq(b)
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

這會是一個全部通過的情況！我們也說過 `eq` 只看值，不看型別。

為什麼會這樣說呢？因為其實在 Ruby 的世界中 `5` and `5.0` 兩個人是屬於不同的類別！

若是在 IRB 中試試看：

{{< tabbed-codeblock "IRB mode" >}}
<!-- tab ruby -->
> 5.class
> Integer
> 5.0.class
> Float
<!-- endtab -->
{{</ tabbed-codeblock>}}

在 Ruby 的整數和浮點數是不同的類別，自然而然也不會是同一個 type！

但 `eq` 會略過型別的比對，專注在值是否吻合！

而一直強調型別，那會根據型別來判斷的 `Matcher` 是哪個呢？

# eql Matcher

雖然只差了一個 `l` 的字母，但在處理的嚴謹度上就完全不相同了！

如果一樣使用剛剛的範例，但改用 `eql` 來看看會發生什麼事情？

{{< tabbed-codeblock "eql 的差別" >}}
<!-- tab ruby -->
RSpec.describe "eql matcher" do
  let(:a) { 5.0 }
  let(:b) { 5 }

  it 'tests for value and type' do
    expect(a).to eql(5)
    expect(b).to eql(5.0)
    expect(a).to eql(b)
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

沒通過的結果：

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109291720026.png)

他會告訴你，`5` 和 `5.0` 是不相同的東西，測試不通過！！！

你會想說，這樣有什麼差別啦，現實世界中 5 就是 5.0 啦～

但情境如果是在處理精密的匯率或是稅率等等，使用 `eq` 來做 `Matcher` 是不是會有一點草率呢？

使用 `eql` 能夠幫你在撰寫方法的時候考慮得更多！你也可以想像成嚴格模式啦～

因為它必須是 **相同的 value 以及 相同的類別** 才能被認定為相同！

接下來我想介紹更嚴格的一個 `Matcher`，他不只要你是同一個型別，他還要你是同一個物件！

# equal Matcher & be Matcher

會放在一起講的原因是 `equal` 等同於 `be`，所以接下來我們會一起示範這要怎麼用！

但在放上範例之前，我想提一個概念：

`equality` 意思是在確認兩個物件在基本的設計上一樣的！

`identity` 意思是在確認兩個物件是同一個物件！

為什麼會講這樣的概念呢？因為 `equal` 就是 `identity` 的概念，而上面的 `eq` `eql` 都像是 `equality` 的概念！

我們換個類比的方法，假設我在一條大街上，和我隔壁的鄰居蓋了一間一模一樣的房子 ( 是真的一模一樣，包含窗框、草皮、房門等等 )

你心裡會說，這兩個房子長得一模一樣欸！但你不會說這是同一個房子，對吧？

就算他們在怎麼像，像到完全看不出來差別，你還是能用地址來做判斷，他們還是兩間房子！

等等示範的 `equal` 就是在幫你檢查是不是同一個地址，同一個房子的概念！

而 `eq` `eql` 都只是看看照片說，這兩個根本一模一樣嘛～

廢話不多說，我們直接上 Code 吧！

{{< tabbed-codeblock "eq & eql" >}}
<!-- tab ruby -->
RSpec.describe "equal matcher & be matcher" do
  let(:a) { [7, 7, 7]}
  let(:b) { [7, 7, 7]}
  let(:c) { a }
  
  it "test for object identity" do
    expect(a).to eq(b)
    expect(a).to eql(b)
    
    expect(a).to equal(c)
    expect(a).not_to equal(b)
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

上面是一個會全部通過的測試！

這邊的 `c` 變數，指向 `a` 變數，這代表著什麼？

我把我家的地址填得和你家地址一樣，我們兩個是一模一樣在同一個地方的物件！

而 `a` 和 `b` 在值以及型別上來看，都一模一樣，但他們在記憶體中的地址卻不相同，我們可以故意讓測試壞掉來看看 OutPut 是什麼？

我們讓 `a equal b` 來試試看！

錯誤的提示：

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109291721465.png)

可以看到兩個在 `Array` 的記憶體位置是不相同的 ( 一個是 `2800`， 一個是 `2820` )，這就像是房子的案例一樣，我們雖然一樣，但終究是兩個個體！

下面的提示也有告訴你，這是在比較物件的 `identity`！

> 剛剛範例上都使用 `equal`，但其實你也可以用 `be` 比較省力喔～

# 小結

重新複習一下三者的不同之處！

`eq`：單純比較值，不在意型別

`eql`：比較值以及型別

`equal 以及 be`：比較值，型別，記憶體位置(是否為同一個物件)

希望以後大家在使用的時候，可以根據要測試的嚴謹程度來決定要使用哪一個 `Matcher` 喔！

這三個都是非常基礎的 `Matcher` 但卻常常不知道錯在哪，希望這篇文章可以幫助到你～




