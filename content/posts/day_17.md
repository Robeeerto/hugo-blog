---
title: "Day 17 Matcher 介紹 (中)"
date: 2021-10-01T18:40:04+08:00
metaAlignment: center
thumbnailImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211620030.png'
coverImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211620030.png' 
coverMeta: out
coverSize: partial
categories:
- "2021 鐵人賽"
tags:
- RSpec
summary: 繼續介紹不一樣的 Matcher！
keywords:
- RSpec
- matcher
---

繼續介紹不一樣的 Matcher！
<!--more-->
{{< toc >}}
# all matcher

在一開始寫 Ruby 的時候，你可能會想用跑迴圈的方式來測試所有的值！

{{< tabbed-codeblock "all matcher">}}
<!-- tab ruby -->
RSpec.describe "loop test" do
  it "use each to test" do
    [1, 3, 5].each do |num|
      expect(num).to be_odd
    end
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

這樣也沒有什麼問題，但當 RSpec 有提供給你更好的方法的時候，我們就沒有必要自己來跑迴圈。

這樣很容易讓測試的閱讀性變得很困難，所以接下來的這個 `all matcher` 可以一次的測試所有的值，而且讓語法變得非常易讀！


{{< tabbed-codeblock "all matcher">}}
<!-- tab ruby -->
RSpec.describe "all matcher" do
  it "check every value" do
    expect([1, 3, 5]).to all(be_odd)
    expect([1, 3, 6, 5, 8]).to all(be_an(Integer))
    expect([1, 3, 4, 7, 9]).to all(be < 10)
    expect([], [], []).to all(be_empty)
    expect([2, 4, 6]).to all(be_even)
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

我們可以看到 `all` 後面可以接收所有 RSpec 的 `matcher`。

接著我們可以再用一行流的方式來練習看看～

{{< tabbed-codeblock "all matcher one line version">}}
<!-- tab ruby -->
RSpec.describe "all matcher" do
  it [1, 3, 5] do
    it { is_expected_to all(be_odd) }
    it { is_expected_to all(be < 10) }
    it { is_expected_to all(be_an(Integer)) }
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

# be matcher ( nil, falsy, truthy)

這個 `be matcher` 和之前提到的 `be` 不太一樣，這個主要是拿來測試 `true` & `false` 的值！

而在 Ruby 的世界中，只有兩個東西不是 `true` 的，就是 `false` & `nil`。

意思就是所有的物件都是 `true` 的，都是正向的。

我們可以用一些簡單的示範來看看：

{{< tabbed-codeblock "condition flow">}}
<!-- tab ruby -->
> if "-1"
> p 'true'
> else
> p 'false'
> end
> # 'true'
<!-- endtab -->
{{</ tabbed-codeblock>}}

這邊的 `-1` 雖然看起來像是負面的數值，但他在 Ruby 的世界中依然是一個 true 的物件，所以這邊的判斷式會走向印出 `"true"`。

接著就可以來使用這個 `be matcher`

{{< tabbed-codeblock "be matcher">}}
<!-- tab ruby -->
RSpec.describe "be matcher" do
  it "can test true" do
    expect(true).to be_truthy
    expect("-100").to be_truthy
    expect(:test).to be_truthy
    expect([-50, -100]).to be_truthy
    expect({}).to be_truthy
    expect(0).to be_truthy
  end
  
  it "can test nil false" do
    expect(nil).to be_falsy
    expect(false).to be_falsy
    # 整個 Ruby 世界中唯二的 falsy 物件
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

相信上面這個範例就可以很清楚的理解到這個 `matcher` 的使用方法啦～

# change matcher

這個 matcher 蠻有趣的，但實用度比較低，我就使用範例稍微介紹一下！

{{< tabbed-codeblock "change matcher">}}
<!-- tab ruby -->
RSpec.describe "change matcher" do
  subject {[1, 2, 3]}
  
  it "check object change state" do
    expect { subject.push(4) }.to change { subject.length }.from(3).to(4)
    expect { subject.push(4) }.to change { subject.length }.by(1)
    expect { subject.pop }.to change { subject.length }.by(-1)
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

先提一下為什麼我們要用 `{ }` 而不是像平常一樣使用 `( )`，是因為如果我們想要對物件執行 Ruby 方法的時候，都像這樣 block 的方式！

而他也可以接受不止一行的參數，你可以在裡面進行計算，執行原生的 Ruby 方法等等...

而整個 matcher 的重點就在後面很有趣的寫法！我們塞了一個數字進去 array 後，`subject` 的長度從 `3 到 4` 的過程！就是我們要測試的！

後面補上的 `by(1)` 寫法，就是改變的數值，因為每次都要寫 `from...to...` 實在是太麻煩了，於是 RSpec 也給我們這樣的黑魔法來使用！

# contain_exactly matcher

這個 `Matcher` 的工作在於確認 `確實涵蓋的值且不在乎順序`。

我們用範例來說明！

{{< tabbed-codeblock "contain_exactly matcher">}}
<!-- tab ruby -->
RSpec.describe "contain_exactly matcher" do
  it "check exactly contain" do
    expect([1,2,3]).to contain_exactly(1,3,2)
    expect([1,2,3]).to contain_exactly(2,1,2)
    expect([1,2,3]).to contain_exactly(3,2,1)
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

其實非常的簡單，他關注的點是裡面的內容物是不是都在，但他不在意順序！

這個其實使用的地方蠻少的，因為自己在寫的時候還是希望測試的格式是固定的，也有可能是我自己想不到使用的他的時機～

# start_with & end_with matcher

看到這個英文的開頭，應該能夠大概猜到他的使用方式！就是要測試這個物件的開頭或是他的結尾～

且主要的用法是在 `String` `Array` 身上！

我們來看看範例吧！

{{< tabbed-codeblock "start_with & end_with matcher">}}
<!-- tab ruby -->
RSpec.describe "start_with & end_with" do
   descirbe "SnoppyDog" do
     it "check the substring from begin to end" do
       expect(subject).to start_with("Snop")
       expect(subject).to start_with("S")
       expect(subject).to start_with("SnoppyDo")
       expect(subject).to end_with("pyDog")
       expect(subject).to end_with("og")
     end
     
     it { is_expected_to start_with("Sn") }
     it { is_expected_to end_with("Dog") }
   end
   
   descirbe [1, 3, 7] do
     it { is_expected_to start_with(1, 3) }
     it { is_expected_to end_with(3, 7) }
   end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

但他就不像剛剛的 `contain_exactly` 一樣不在意順序了！

`start_with` 本身就帶有一種方向的感覺，所以我們的 matcher 要給予照著順序的參數～

# 小結

`Matcher` 的數量真的是有夠多，當然要怎麼使用端看自己要測試的重點是什麼～

說不定有些冷門的 `Matcher` 剛剛好很契合你這個測試的目的也說不定！

重點就是多看一些有趣的 `Matcher` 然後收藏在淺意識中，想到的時候再去查資料就可以了～





