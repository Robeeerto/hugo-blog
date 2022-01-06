---
title: "Day 19 魁儡的 double object"
date: 2021-10-03T14:39:42+08:00
metaAlignment: center
coverImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211620030.png' 
coverMeta: out
coverSize: partial
categories:
- "2021 鐵人賽"
tags:
- RSpec
---

Mock 篇開場～
<!--more-->
{{< toc >}}

昨天結束了 `Matcher` 的介紹，今天開始進入 `mock` 的篇章。

還記得一開始提到的 unit test，我們希望著重在小的功能上進行測試，但一個 App 常常牽扯到各式各樣不同的關聯。

這時候的 unit test 就會變得很難寫，因為你需要顧慮到許多不同的方法所回傳的值，因為他們都會影響到你測試的結果！

這時候 `mock` 系列的功能就可以很有效地幫助到你，因為他就是為了假裝、為了模仿而生！

# Double method

從概念上來說，`double` 是一個製造假物件的方法，而這個假物件進而能夠接受方法，設定好回傳值。

我們先看看範例，再來解釋：

{{< tabbed-codeblock "Double method">}}
<!-- tab ruby -->
RSpec.describe "double method" do
  it "can defined method to be invoked" do
    basketball_player = double("Lebron James", dunk: "Ah!!!!", shoot: "Goal!!!!")
    expect(basketball_player.dunk).to eq("Ah!!!!")
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

我們 `double` 出了一個籃球員，可以灌籃然後發出 `Ah!!!` 的聲音～

上面這段測試是成功通過的！那到底是在做什麼呢？

`double` 可以讓我們預期的方法和回傳的值，變成`key-value` 的組合，然後存放在這個 `double` 物件中。

接著他一樣可以使用他身上的方法，然後得到預設好的回傳值～

我們也可以用 `allow`  的寫法，看起來會更直觀一些，雖然比較長。

{{< tabbed-codeblock "allow type">}}
<!-- tab ruby -->
RSpec.describe "double method" do
  it "can defined method to be invoked" do
    basketball_player = double("Lebron James")
    allow(basketball_player).to receive(:dunk).and_return("Ah!!!!")
    expect(basketball_player.dunk).to eq("Ah!!!!")
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

> 我們允許 `basketball_player` 物件接收一個 `dunk `方法，然後回傳 `Ah!!!!!`

所以這個測試也是理所當然的會成功通過！

但這樣一次寫一個會不會太麻煩了？而且我又不想要寫在初始化裡面很亂...

還有另一個叫做 `receive_messages` 的方法喔！

{{< tabbed-codeblock "receive message">}}
<!-- tab ruby -->
RSpec.describe "double method" do
  it "can defined method to be invoked" do
    basketball_player = double("Lebron James")
    allow(basketball_player).to receive_messages(dunk: "Ah!!!!", shoot: "Goal!!!!")
    expect(basketball_player.dunk).to eq("Ah!!!!")
    expect(basketball_player.shoot).to eq("Goal!!!!")
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

其實效果等同你在初始化的時候，直接寫在後面是一樣的道理，但我自己比較喜歡看 `allow` 的方法，會讓人比較理解的感覺～

但其實上面的三種方式都可以，都只是建立 `double` 物件的手段～

而或許你還是覺得，所以這能幹嘛？反正怎麼測都可以過，有意義嗎？

當然不是叫你針對你要測試的物件做 `double`，這樣錢也太好賺了吧 😂

而是我們要被不同的類別所牽扯，導致我們需要 `double` 的幫忙。

接下來就會用範例來看看，什麼叫做被別的類別給影響，導致你需要使用 `double` 的情形～

{{< tabbed-codeblock "should use double timing">}}
<!-- tab ruby -->
class Cowboy
  def initialize(name)
    @name = name
  end
  
  def fighting?
    true
  end
  
  def draw_the_gun
    "Bang!!!"
  end
  
  def be_shot
    "Help me..."
  end
  
  def continue?
    false
  end
end


class Bar
  attr_reader :cow_boy
  
  def initialize(cow_boy)
    @cow_boy = cow_boy
  end
  
  def start_fighting
    if cow_boy.fighting?
      cow_boy.draw_the_gun
      cow_boy.be_shot
      cow_boy.continue?
    end
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

從上面這個簡單的西部牛仔劇情片編織的類別，可以看到酒吧以及牛仔之間的關係！

但當我們今天要寫 `Bar` 這個類別的測試時怎麼辦呢？

我們要測試 `start_fighting` 這個方法時，可以看到被 `cow_boy` 給影響到了！

但其實我們根本可以不用關心 `cow_boy` 是怎麼作業的？他的方法回傳什麼那都不重要，我們要專注在現在這個方法應該要怎麼通過才對。

他的流程是不是正確的？這才是我們在乎的地方～

所以這就是前面提到，被影響到的時候，我們應該要用 `double` 來取代這邊的 `cow_boy` 然後使我們的測試可以通過！

當然這是很搞笑的介紹，但事實上的專案中，就是會有許多不同的關係的存在，也都會影響到整個方法的進行。

接著我們來用 `double` 測試這個 `Bar` 類別的實體方法吧！

{{< tabbed-codeblock "use double">}}
<!-- tab ruby -->
RSpec.describe Bar do
  let(:cow_boy) { double("Gene Autry", fighting?: true, draw_the_gun: "Bang!!!", be_shot: "Help me...", continue?: true) }
  subject { described_class.new(cow_boy) }
  
  describe "#start_fighting method" do
    it "expect cow_boy start_fight" do
      expect(cow_boy).to receive(:fighting?)
      expect(cow_boy).to receive(:draw_the_gun)
      expect(cow_boy).to receive(:be_shot)
      expect(cow_boy).to receive(:continue?)
      subject.start_fighting
    end
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

這是會成功通過的測試，因為我比較懶一點直接寫在初始化裡面 請見諒～

然後我們在測試中期待這個物件會接收到這些方法～

但如果我們不給予這些方法的話，這個方法是會沒辦法成功運作的喔！

因為我們把 `double` 物件放進去 `Bar` 類別生成實體，進而執行他的 `start_fighting` 這個方法，其中有使用到的方法都會先去參考 `double` 物件有沒有才繼續向下執行！

可以看看如果我們只寫了 `double` 物件的話：

{{< tabbed-codeblock "use double">}}
<!-- tab ruby -->
RSpec.describe Bar do
  let(:cow_boy) { double("Gene Autry") }
  subject { described_class.new(cow_boy) }
  
  describe "#start_fighting method" do
    it "expect cow_boy start_fight" do
      subject.start_fighting
    end
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

再看看執行的 Output !

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202110031445485.png)

程式的執行在 `if cow_boy.fighting?` 中斷了，內容也有提到我們的 `double` 物件沒有接受到這個方法～

所以我們可以用最上面介紹的三種寫法來填入方法，使我們可以專注在測試 `Bar` 的方法，而不受到 `cow_boy` 干擾！

而如果想要指定接收次數的話，也可以加入計數的方法喔！

畢竟某些時候你會想要限制有些方法只執行一次，或是最多兩次等等的條件～

{{< tabbed-codeblock "limit times">}}
<!-- tab ruby -->
describe "#start_fighting method" do
  it "expect cow_boy start_fight" do
    expect(cow_boy).to receive(:fighting?)
    expect(cow_boy).to receive(:draw_the_gun)
    expect(cow_boy).to receive(:be_shot).twice
   #expect(cow_boy).to receive(:be_shot).exactly(1).times
   #expect(cow_boy).to receive(:be_shot).at_most(1).times
    expect(cow_boy).to receive(:continue?)
    subject.start_fighting
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

上面有註解的部分，都是可以用這樣的寫法！

現在我們加上了 `twice` 就是希望這個方法能被執行兩次，那我們先來看看 Output

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202110031445566.png)

這邊的意思就是，我們期待會收到兩次的執行，但其實程式碼只有一次！

那我們先改寫要測的的方法，讓牛仔中槍兩次吧

{{< tabbed-codeblock "change test">}}
<!-- tab ruby -->
def start_fighting
  if cow_boy.fighting?
    cow_boy.draw_the_gun
    cow_boy.be_shot
    cow_boy.be_shot
    cow_boy.continue?
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

這樣就能成功通過測試啦～但其實你不寫任何次數的話也是會通過啦！

但既然是寫測試，方法會執行幾次，或是接收什麼樣的參數，都是需要考量和注意的！

# 小結

今天介紹了使用 `double` 的情境，以及為什麼要使用它～

就是希望可以讓我們的工作更順利，不要在一些奇怪的事情上煩惱，有時候被其他的物件干擾導致測試寫不出來，真的是超級痛苦的！

明天會稍微介紹一下今天有提到的 `allow method` 然後進入 `instance double` 的世界！

這是一個更真實更好用的東西！





