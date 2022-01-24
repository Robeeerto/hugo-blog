---
title: "Day 13 懶得想變數嗎？ RSpec 有提供你啦"
date: 2021-09-27T18:41:21+08:00
metaAlignment: center
thumbnailImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211620030.png'
coverImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211620030.png' 
coverMeta: out
coverSize: partial
categories:
- "2021 鐵人賽"
tags:
- RSpec
summary: 自動產生的變數，讓你使用～
keywords:
- RSpec
- subject
---

{{< toc >}}

還記得我們使用 `let` 方法來實作一個物件來讓我們可以快速使用！

但有沒有什麼好方法可以讓我不要想物件的名字？可能因為我有取名的障礙或是英文真的太爛了！

甚至我連 `let` 都懶得寫，我實在是太懶惰了...

有！ RSpec 有個有趣的方法叫做 `subject`，他會自動根據整個 RSpec 最上層的描述來決定 `subject` 該是什麼！

# Subject

剛剛的說法也都比不過一個實際的例子來得實際，看看以下範例！

{{< tabbed-codeblock "subject" >}}
<!-- tab ruby -->
RSpec.describe Hash do
  it "should be empty" do
    expect(subject.length).to eq(0)
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

這是 OutPut 的結果：

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109271843750.png)

現在一定是一頭霧水，想說這個 `subject` 到底是在哪裡？我們甚至沒有宣告他啊？

這就是 RSpec 提供你的小方法喔，如果你的最上層 `RSpec.describe` 後方接的是類別，那 `subject` 就會依照這個類別的預設值去做 `new` 的動作。

我們可以把 `subject` 給印出來看看！

{{< tabbed-codeblock "印出 subject" >}}
<!-- tab ruby -->
RSpec.describe Hash do
  it "should be empty" do
    p subject
    p subject.class
    expect(subject.length).to eq(0)
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

OutPut 的結果：

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109271844616.png)

所以我們可以想像，他在上方幫我們做了：

```ruby=
let(:subject) { Hash.new }
```

這是動態產生的，根據我們提供的類別來建立物件！是不是很方便呢？

加上他和 `let` 一樣，在每一個 example 中都是獨立的，不會有污染物件的情形，可以放心的服用！

# 加入初始值的 Subject

剛剛示範的是完全乾淨的一個物件，但如果我們想要建立一個有不一樣初始值的 subject 該怎麼做呢？

馬上來示範一下：

{{< tabbed-codeblock "加入初始值" >}}
<!-- tab ruby -->
RSpec.describe Hash do
  subject do 
    {a: 3, b: 4}
  end
  
  it "should be empty" do
    expect(subject.length).to eq(2)
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

其實很簡當，但如果我們不想要叫做 `subject` 呢？

只要在後面加入變數的命名就可以了喔！

{{< tabbed-codeblock "加入命名" >}}
<!-- tab ruby -->
RSpec.describe Hash do
  subject(:robert) do
    {a: 3, b: 4}
  end

  it "should be empty" do
    expect(subject.length).to eq(2)
    expect(robert.length).to eq(2)
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

至於為什麼要這樣用？我使用 `let` 還不是一樣，何必浪費時間寫三行呢？

你說的沒錯，用 `let` 的效果也是一樣，但 `subject` 有其他更魔法的方法可以使用，等等也會一併介紹，會讓寫出來的 Code 優雅到不行！

在介紹黑魔法的寫法之前，我先來介紹另一個也很有趣的語法。

# Described_class

剛剛介紹的是 `subejct` 的好用之處，這次介紹的 `described_class` 可能不一定會用到，但在改大量程式碼的時候很有效果！

我先用範例來解說一下：

{{< tabbed-codeblock "describe_class" >}}
<!-- tab ruby -->
class Burger
  attr_accessor :beef, :cheese
  
  def initialize(beef, cheese)
    @beef = beef
    @cheese = cheese
  end
end

RSpec.describe Burger do
  subject { Burger.new("Beef", "Cheedar") }
  let(:pork_burger) { Burger.new("Pork", "Cheedar") }
  
  it "...." do
    ....
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

這邊我們有個漢堡的類別，假設需求今天下來是要更改這個 Burger 類別的名稱，怎麼辦？

任務是把 Burger 改成 GoodBurger 這個名字，若是只有兩三行，那倒是沒什麼問題，但如果是 100 行呢？ 我們要怎麼做？

這時候 `described_class` 就可以派上用場，若是我們都用這樣的寫法：

{{< tabbed-codeblock "describe_class" >}}
<!-- tab ruby -->
class GoodBurger
  attr_accessor :beef, :cheese
  
  def initialize(beef, cheese)
    @beef = beef
    @cheese = cheese
  end
end

# 我們只需要更改這邊的類別名稱就可以了！
RSpec.describe GoodBurger do
  subject { described_class.new("Beef", "Cheedar") }
  let(:pork_burger) { described_class.new("Pork", "Cheedar") }
  
  it "...." do
    ....
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

`described_class` 雖然不是一個很常使用的東西，但他就是可以根據你上面放置的類別來動態產生物件，而且只需要改類別名稱就能夠一次跟上！

說不定在某些時候會幫助到你！ 

# 我只要寫一行！

前面有提到使用 `subject` 的小小好處，就是可以讓你只要寫一行！

直接先看程式碼：

{{< tabbed-codeblock "一行流" >}}
<!-- tab ruby -->
RSpec.describe 5 do
  it "should be 5" do
    expect(subject).to eq(5)
  end

  context "one line syntax" do
    it { is_expected.to eq(5) }
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

這是 OutPut 

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109271847800.png)

第三行是我們很常使用的寫法，所以沒有什麼問題。

但第七行就是 `subject` 的神奇之處， `is_expected` 這個方法本身的主詞就是 `subject`，所以可以這樣子就寫完範例。

不覺得看起來整個就超級 Ruby Style 的，該省的都要給我省下來，越短越好！

希望這可以讓你在寫某些簡單的 example 中可以派上用場！

> Don't Repeat Yourself 能不要重複撰寫就盡量節省！

# 小結

今天介紹了 `subject` & `described_class` 還有一行流的寫法！

不一定是常常會使用到，但都是 RSpec 很基礎的語法，至少先記在腦袋裡面，想起來再去查資料就可以了！

現在介紹的都是整理單個測試的程式碼，若是我們在不同的測試有著相同的 example 呢？