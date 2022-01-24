---
title: "Day 20 真真假假的 Instance doubles"
date: 2021-10-04T19:02:31+08:00
metaAlignment: center
thumbnailImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211620030.png'
coverImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211620030.png' 
coverMeta: out
coverSize: partial
categories:
- "2021 鐵人賽"
tags:
- RSpec
summary: 實用的 Instance doubles
keywords:
- RSpec
- instance_doubles
- mock
---

實用的 Instance doubles
<!--more-->
{{< toc >}}

昨天有提到會稍微介紹一下 `allow method`，其實在昨天的範例裡面就有使用到。

使用的時機在於允許 `double` 接受方法以及回傳值。

{{< tabbed-codeblock "allow method">}}
<!-- tab ruby -->
RSpec.describe 'allow method review' do
  it "can customize return value for methods on doubles" do
    calculator = double
    
    allow(calculator).to receive(:add).and_return(15)
    
    expect(calculator.add).to eq(15)
    expect(calculator.add(3)).to eq(15)
    expect(calculator.add(132353123)).to eq(15)
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

可以看到一個 `calculator` 被允許接受一個 `add` 方法，然後回傳值為 15。

但這當然不是一個好的示範，寫死你的回傳值，我猜不是什麼有趣的事情！

接著我們要來說說昨天提到的 `double method` 有什麼壞處。

首先，整個 `double` 物件都是寫死的，這可能會在開發的測試上造成一些不必要的壓力，因為我們必須時時刻刻注意這個 `double` 物件的方法有沒有更動。

若是我們不慎移除掉真實的 Code，他依舊會通過測試，因為我們的測試裡面還是保有這個方法，但他不是真實的，一切都是捏造的！

還記得一開始用 `double` 的想法就是希望我們可以拋開其他關聯的干擾，專注在我們要測試的方法上面，但這有可能害我們錯失找到 Bug 的關鍵！

所以我們需要更嚴謹一些，於是就有了 `Instance Doubles`。

# Instance Doubles

所以說，為什麼要嚴謹一些呢？

{{< tabbed-codeblock "example">}}
<!-- tab ruby -->
class Burger
end

RSpec.describe Burger do
  it "#yummy" do
    burger = double(yummy: "Yum!")
    expect(burger.yummy).to eq("Yum!")
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

上面這個會是成功通過的測試，當面臨重構的情況，不小心移除掉某些程式碼時，測試依舊會通過，一切看起來安然無恙，但真的會壞掉...

所以才有了 `instance doubles`  的出現，他會幫助你檢查你真實的程式碼，創造彼此的連結！

我們用 `instance doubles` 來試試看！


{{< tabbed-codeblock "instance doubles">}}
<!-- tab ruby -->
class Burger
end

RSpec.describe Burger do
  it "#yummy" do
    burger = instance_double(Burger, yummy: "Yum!")
    expect(burger.yummy).to eq("Yum!")
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

他噴錯了，意思就是這個類別沒辦法實施這個 `yummy` 實體方法，因為我們沒有～

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202110041905317.png)

補上我們類別內的實體方法：


{{< tabbed-codeblock "add instance method">}}
<!-- tab ruby -->
class Burger
  def yummy
    "Yum!"
  end
end

RSpec.describe Burger do
  it "#yummy" do
    burger = instance_double(Burger, yummy: "Yum!")
    expect(burger.yummy).to eq("Yum!")
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

通過測試啦～

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202110041906040.png)

整個用法就是呢：

```ruby=
something = instance_double("你要測試的類別", "他的實體方法",.....)
```

他就會去幫你檢查到底是不是真的有這個方法，不管你是寫多寫少，都會被抓出來，不要想要亂寫方法進去，他會知道的！

在上一些用 `allow method` 的範例來看看吧


{{< tabbed-codeblock "use allow method">}}
<!-- tab ruby -->
class Burger
  def yummy
    "Yum!"
  end
end

RSpec.describe Burger do
  it "#yummy" do
    burger = instance_double(Burger)
    allow(burger).to receive(:yummy).and_return("Yum!")
    expect(burger.yummy).to eq("Yum!")
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

其實概念就和 `double` 非常相似，但就是多加了一個類別上去，讓 RSpec 去幫你檢查，所以我在這邊可以說，如果要使用 `double` 請用 `instance double`，小心駛得萬年船～

等等？你問我說只有實體變數有 `double` 嗎？

那類別有嗎？那是廢話...當然有啊～

# Class Doubles

我們一直強調使用 `double` 的情境，就是在於當你今天測試的目標會遭受其他的干擾時，我們就會利用 `double` 來專注在我們要測的項目上！

所以我們來創造一個使用到 class method 的情境：


{{< tabbed-codeblock "situation">}}
<!-- tab ruby -->
class Burger
  def make
    "Make!"
  end
end

class BurgerStore
  attr_reader :burgers
  
  def sale
    @burgers = Burger.make
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

接著我們想要對於 `BurgerStore` 的 `sale` 方法來寫測試：


{{< tabbed-codeblock "class method">}}
<!-- tab ruby -->
RSpec.describe BurgerStore do
  it "can only implement class methods defined on a class" do
    burger_klass = class_double(Burger)
    expect(burger_klass).to receive(:make).and_return("Make!")
    expect(subject.sale).to eq("Make!")
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

用我們剛剛學過關於 `double` 的知識來測試看看吧，一切看起來都非常合理呢～

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202110041908016.png)

看看是哪裡出了問題？他説我們期待這個 class method 應該接受到一個 `make` 方法。

但我看看自己的原始碼，在 `BurgerStore` 確實有 `Burger.make` 這段程式碼發生啊？

哎呀，原來是因為我們還需要一個方法叫做 `as_stubbed_const` 接在這個 `double` 的後方，讓我們使用這個 double 物件去確實的取代程式碼內的 `Burger.make` 的這個 `Burger`。

這個 `as_stubbed_const` 的用意就很像綁定這個 `double` 物件去取代真實的程式碼，這時候測試就會通過了，因為他確實接收到 `make` 這個方法的呼叫～

我們可以在用一段示範來看看 `as_stubbed_const` 的意義到底在哪裡？


{{< tabbed-codeblock "as_stubbed_const">}}
<!-- tab ruby -->
class Burger
  def make
    "Make!"
  end
end

class BurgerStore
  attr_reader :burgers
  
  def sale
    @burgers = Burger.make
  end
end

RSpec.describe BurgerStore do
  it "can only implement class methods defined on a class" do
    burger_klass = class_double(Burger).as_stubbed_const
    expect(burger_klass).to receive(:make).and_return([1,2,3])
    expect(subject.sale).to eq([1,2,3])
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

看得出來發生什麼了嗎？我們原始碼的回傳值是 `"Make!"`，但現在接受的卻是 `[1,2,3]` 就是因為 `as_stubbed_const` 的綁定，但他其實和 `instance_double` 是一樣聰明的喔，對於你寫入的方法會有所檢查～所以不要亂寫！

# 小結

今天把 `double` 大概都介紹了一遍，明天將會介紹 `spies` 也是 `mock` 偽裝軍團的一員～





