---
title: "Day 21 Spies 間諜來襲！"
date: 2021-10-05T18:16:24+08:00
metaAlignment: center
coverImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211620030.png' 
coverMeta: out
coverSize: partial
categories:
- "2021 鐵人賽"
tags:
- RSpec
---

Mock 軍團的最後一個成員！
<!--more-->
{{< toc >}}

今天我們要介紹 Mock 軍團的最後一員，也就是 `Spies` 這個用法！

除了加上範例之外，兩者間的差別，也會好好地來解釋一番～

# 和 double 的差別

首先就是 `Spies` 本身更狡詐了一些，還記得我們在寫 `double` 或是 `instance double` 的時候，常常要在先去期望這個變數獲得什麼樣的方法，並寫在他的後面～

像是這樣：

{{< tabbed-codeblock "Spies" >}}
<!-- tab ruby -->
RSpec.describe 'double' do
  let(:store) { double('store', sell: 'earn the money!')}
  
  it 'should invoke method first' do
    expect(store.sell).to eq('earn the money!')
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

但 `Spies` 本身是可以先寫方法出來的，就像正常的測試一樣，只是 `expectation` 的部分需要做一些改造！

像是這樣：

{{< tabbed-codeblock "Spied write method first" >}}
<!-- tab ruby -->
RSpec.describe 'spies' do
  let(:burger) { spy('burger') }
  
  it 'confirm a message has been received' do
    burger.eat
    expect(burger).to have_received(:eat)
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

其實有趣的地方在於，這邊的 `expectation` 是 `have_received`，有種在玩英文文法的感覺，這個漢堡已經接收到 `eat` 方法的概念！

而 `Spies` 也有他的檢查機制存在，若是我們沒有收到的方法，但卻有的話會發生什麼事呢？

{{< tabbed-codeblock "What's wrong?" >}}
<!-- tab ruby -->
RSpec.describe 'spies' do
  let(:burger) { spy('burger') }
  
  it 'confirm a message has been received' do
    burger.eat
    expect(burger).to have_received(:eat)
    expect(burger).to have_received(:throw)
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202110051819719.png)


這個變數期待著一次的 `throw` 方法，但卻沒有，所以被噴錯了。

那至於這個和 `instance double` 之間要如何做選擇呢？這邊先賣個關子，後面會有有趣的實驗來看看差別在哪裡～

# 使用情境

其實使用的情境和 `instance double` 基本上是很像的，都是為了獨立某段程式碼的干擾，讓測試的獨立性更好，更專注在我們要測試的部分。

而不是為了生出別的實體變數而搞得滿目瘡痍～

這邊再用一段簡單的程式碼來敘述一下情境！


{{< tabbed-codeblock "situation" >}}
<!-- tab ruby -->
class Cheese
  def initialize(type)
    @type = type
  end
end

class Burger
  attr_reader :cheese
  
  def initialize
    @cheese = []
  end
  
  def add(type)
    @cheese << Cheese.new(type)
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

這邊很清晰的是我們要在漢堡裡面加入 `cheese` ，就像前幾天談到的一樣，我們要專注在 `Burger` 這個類別，所以我們希望可以把 第 15 行的 `Cheese.new(type)` 這段給獨立出來，用 mock 的方式替代！

接著我們來寫會通過的測試！

{{< tabbed-codeblock "pass the test" >}}
<!-- tab ruby -->
RSpec.describe Burger do
  let(:cheese) { spy(Cheese) }
  
  before do
    allow(Cheese).to receive(:new).and_return(cheese)
  end
  
  describe '#add' do
    it 'add cheese to burger' do
      subject.add('parmesan')
      expect(Cheese).to have_received(:new).with('parmesan')
      expect(subject.cheese.length).to eq 1
      expect(subject.cheese.first).to eq(cheese)
    end
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

我們逐行來解釋一下，首先在進入測試之前( 第五行 )，我們先做了允許 `Cheese` 這個類別接收 `new` 方法，並且 return 我們的 `spy(Cheese)`

接著進入測試的 block 中，`subject` 執行了 `add` 方法，並且傳入參數 parmesan。

接著我們又說這個 `Cheese` 已經接受了 `new` 方法，也就是我們在 `before` block 中所做的事情，在這個時候，我們已經徹底的替換掉正式的程式碼了。

接著的 `expectation` 就是基本的操作囉～

但你會發現，這個 `spy` 替換成 `instance_double` 也能夠正常的使用。

奇怪了... 那我們印出來看看吧！我們先印印看 `spy` 版本的


{{< tabbed-codeblock "spy version" >}}
<!-- tab ruby -->
RSpec.describe Burger do
  let(:cheese) { spy(Cheese) }
  
  before do
    allow(Cheese).to receive(:new).and_return(cheese)
  end
  
  describe '#add' do
    it 'add cheese to burger' do
      subject.add('parmesan')
      expect(Cheese).to have_received(:new).with('parmesan')
      p cheese # 印出來看看吧
      expect(subject.cheese.length).to eq 1
      expect(subject.cheese.first).to eq(cheese)
    end
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202110051820282.png)

有沒有搞錯？這個 `spies` 講得這麼厲害，但結果是 `double` 的語法糖衣嗎？

我們快印看看 `instance double` 是什麼？


{{< tabbed-codeblock "instance version" >}}
<!-- tab ruby -->
RSpec.describe Burger do
  let(:cheese) { instance_double(Cheese) }
  
  before do
    allow(Cheese).to receive(:new).and_return(cheese)
  end
  
  describe '#add' do
    it 'add cheese to burger' do
      subject.add('parmesan')
      expect(Cheese).to have_received(:new).with('parmesan')
      p cheese # 印出來看看吧
      expect(subject.cheese.length).to eq 1
      expect(subject.cheese.first).to eq(cheese)
    end
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202110051821216.png)

嗚嗚，騙我感情，最後還是 `instance double` 最好，以後都用他好了～

但最終要如何使用，還是取決於功能的考量，和團隊的取向，只要能夠好好的測試到需要測試的功能就好了！

所以你要說 `spies` 和 `double` 有真的一模一樣嗎？也沒有啦其實，`spies` 再做得事情更像是潛入敵營，偽裝成某一段程式碼的感覺，而且 `have_received` 也真的需要接受的方法，只是需要回傳值的話還是得使用 `allow`。

還是有很大的不同在於撰寫的模式，閱讀起來的舒適度！但開心就好～

# 小結

明天開始為期大概 10 天會正式進入 `rspec-rails` 的章節！

也是 RSpec 真正的戰場，剛好加上最近上班也都在寫的緣故，希望可以把一些實際的案例放入文章中，讓需要的人也能夠看到！

基本的 Model 測試 Feature 測試 以及 Stub API 的方式都會盡量地寫進去。

共勉之！
