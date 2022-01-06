---
title: "Day 28 如何撰寫表徵測試"
date: 2021-10-12T17:24:59+08:00
metaAlignment: center
coverImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211620030.png' 
coverMeta: out
coverSize: partial
categories:
- "2021 鐵人賽"
tags:
- RSpec
---

表徵測試是什麼？怎麼寫？
<!--more-->
{{< toc >}}

# 什麼是表徵測試以及解決的問題是什麼？

假如我遇到一段想重構的代碼，但不幸的是：

1. 我甚至看不懂他在幹嘛
2. 他也沒有被測試給覆蓋

重構自己不理解的程式碼是有巨大的風險的，而且問題來了，如果我不知道他是做什麼的，那我要怎麼幫這段代碼寫測試呢？

有一個有趣的東西叫做：[表徵測試](https://zh.wikipedia.org/wiki/%E8%A1%A8%E5%BE%81%E6%B5%8B%E8%AF%95)，我是之前讀文章的時候看到的，他讓解決難題變得直接和簡單，等等會用一些簡單的範例做示範！

# 假設情境

假設這是一段舊專案的程式碼。

下面的兩個方法 `services_with_info` & `products_with_info` 有幾個問題。一個是他們具有很大的重複性，還有這兩個方法其實並沒有真正的幫助，這兩個方法應該可以放到其他地方去，讓 Appointment 專注當一個 Appointment 的 Model，不必把 `appointment_services` & `appointment_products` 序列化的過程放進來。

{{< tabbed-codeblock "Scenario">}}
<!-- tab ruby -->
class Appointment < ActiveRecord::Base
  def services_with_info
    services.collect { |item|
      item.serializable_hash.merge(
        "price": number_with_precision(item.price, precision: 2),
        "label": item.service.name,
        "item_id": item.service.id,
        "type": "service"
      )
    }
  end

  def products_with_info
    products.collect { |item|
      item.serializable_hash.merge(
        "price": number_with_precision(item.price, precision: 2),
        "label": item.product.name,
        "item_id": item.product.id,
        "type": "product"
      )
    }
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

# 重構 services_with_info 

這兩個方法中，我們先把目光放在 `services_with_info`，我的目標是抽離這個方法，創立一個抽象的概念，然後把主體移過去那邊。

重構時可能可以第一次就想到一個很棒的抽象概念，也可能不不行。我們可以先試試看一個抽象概念叫做 `Appointment::ServiceCollection` 的新類別。

下面是這個類別，目前做的就只是把 `services_with_info` 的內容複製貼上過來而已。


{{< tabbed-codeblock "Refactor services_with_info">}}
<!-- tab ruby -->
module Appointment
  class ServiceCollection
    def initialize(services)
      @services = services
    end

    def to_h
      @services.to_h { |item|
        item.serializable_hash.merge(
          "price": number_with_precision(item.price, precision: 2),
          "label": item.service.name,
          "item_id": item.service.id,
          "type": "service"
        )
      }
    end
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

# 第一目標：簡單的實體化物件

當開始為這個新類別寫測試時，盡可能投入較少的經歷，先問自己一個問題，這個 `ServiceCollection` 能夠實體化物件嗎？


{{< tabbed-codeblock "First Goal">}}
<!-- tab ruby -->
RSpec.describe Appointment::ServiceCollection do
  describe '#to_h' do
    it 'works' do
      appointment_service = create(:appointment_service)
      collection = Appointment::ServiceCollection.new([appointment_service])
    end
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

# 第二目標：被測試的方法能夠回傳值嗎？

剛剛的測試通過後，下一步將是期待一個看起來有點蠢的東西

{{< tabbed-codeblock "Second Goal">}}
<!-- tab ruby -->
RSpec.describe Appointment::ServiceCollection do
  describe '#to_h' do
    it 'works' do
      appointment_service = create(:appointment_service)
      collection = Appointment::ServiceCollection.new([appointment_service])
      expect(collection.to_h).to eq('asdf')
    end
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

我當然知道 `collection.to_h` 不會等於 `'asdf'`，但我不知道他等於什麼，所以任何的回傳值都會是錯誤。完全沒必要花心思去猜他會回傳什麼，反正測試結果會告訴我！

```ruby=
F

Failures:

  1) Appointment::ServiceCollection#to_h works
     Failure/Error: expect(collection.to_h).to eq('asdf')
     TypeError:
       wrong element type AppointmentService at 0 (expected array)
     # ./app/models/appointment/service_collection.rb:7:in `to_h'
     # ./app/models/appointment/service_collection.rb:7:in `to_h'
     # ./spec/models/appointment/service_collection_spec.rb:8:in `block (3 levels) in <top (required)>'

Finished in 0.50943 seconds (files took 3.44 seconds to load)
1 example, 1 failure
```

因為是重構一個不知道會回傳什麼的方法，所以錯誤很正常，我們可以嘗試把程式碼註解掉。

{{< tabbed-codeblock "Second Goal">}}
<!-- tab ruby -->
module Appointment
  class ServiceCollection
    def initialize(services)
      @services = services
    end

    def to_h
      @services.to_h { |item|
        #item.serializable_hash.merge(
          #"price": number_with_precision(item.price, precision: 2),
          #"label": item.service.name,
          #"item_id": item.service.id,
          #"type": "service"
        #)
      }
    end
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

結果還是錯誤，那我們就註解掉更多的程式碼看看

{{< tabbed-codeblock "Second Goal">}}
<!-- tab ruby -->
module Appointment
  class ServiceCollection
    def initialize(services)
      @services = services
    end

    def to_h
      #@services.to_h { |item|
        #item.serializable_hash.merge(
          #"price": number_with_precision(item.price, precision: 2),
          #"label": item.service.name,
          #"item_id": item.service.id,
          #"type": "service"
        #)
      #}
    end
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

沒有太意外的還是錯誤，但結果改變了～

```ruby=
F

Failures:

  1) Appointment::ServiceCollection#to_h works
     Failure/Error: expect(collection.to_h).to eq('asdf')
       
       expected: "asdf"
            got: nil
       
       (compared using ==)
     # ./spec/models/appointment/service_collection_spec.rb:8:in `block (3 levels) in <top (required)>'

Finished in 0.49608 seconds (files took 3.41 seconds to load)
1 example, 1 failure
```

很明顯，問題出在 `#@appointment_services.to_h { |item|` 這行。問題大概是出在 to_h 身上，那我試著用 map 代替呢？

{{< tabbed-codeblock "Second Goal">}}
<!-- tab ruby -->
module Appointment
  class ServiceCollection
    def initialize(services)
      @services = services
    end

    def to_h
      @services.map { |item|
        #item.serializable_hash.merge(
          #"price": number_with_precision(item.price, precision: 2),
          #"label": item.service.name,
          #"item_id": item.service.id,
          #"type": "service"
        #)
      }
    end
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

成功了，這樣沒有得到像之前 `wrong element type` 的錯誤，而是不一樣的，更符合我們想看到的東西。

```ruby=
F

Failures:

  1) Appointment::ServiceCollection#to_h works
     Failure/Error: expect(collection.to_h).to eq('asdf')
       
       expected: "asdf"
            got: [nil]
       
       (compared using ==)
       
       Diff:
       @@ -1,2 +1,2 @@
       -"asdf"
       +[nil]
       
     # ./spec/models/appointment/service_collection_spec.rb:8:in `block (3 levels) in <top (required)>'

Finished in 0.48778 seconds (files took 3.5 seconds to load)
1 example, 1 failure
```

現在讓我把之前的註解都全部拔掉看看會怎麼樣～


{{< tabbed-codeblock "Second Goal">}}
<!-- tab ruby -->
module Appointment
  class ServiceCollection
    def initialize(services)
      @services = services
    end

    def to_h
      @services.map { |item|
        item.serializable_hash.merge(
          "price": number_with_precision(item.price, precision: 2),
          "label": item.service.name,
          "item_id": item.service.id,
          "type": "service"
        )
      }
    end
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

出現另一個不同的錯誤，說找不到 `number_with_precision` 這個方法

```ruby=
F

Failures:

  1) Appointment::ServiceCollection#to_h works
     Failure/Error: expect(collection.to_h).to eq('asdf')
     NoMethodError:
       undefined method `number_with_precision' for #<Appointment::ServiceCollection:0x007ff60274e640>
     # ./app/models/appointment/service_collection.rb:9:in `block in to_h'
     # ./app/models/appointment/service_collection.rb:7:in `map'
     # ./app/models/appointment/service_collection.rb:7:in `to_h'
     # ./spec/models/appointment/service_collection_spec.rb:8:in `block (3 levels) in <top (required)>'

Finished in 0.52793 seconds (files took 3.32 seconds to load)
1 example, 1 failure
```

剛好我們可以透過 Rails API 之後這個方法是放在哪裡，如果要在我們獨創的類別中使用，我們要 include `ActionView::Helpers::NumberHelper` 進來用。


{{< tabbed-codeblock "Second Goal">}}
<!-- tab ruby -->
module Appointment
  class ServiceCollection
  include ActionView::Helpers::NumberHelper
 
    def initialize(services)
      @services = services
    end

    def to_h
      @services.map { |item|
        item.serializable_hash.merge(
          "price": number_with_precision(item.price, precision: 2),
          "label": item.service.name,
          "item_id": item.service.id,
          "type": "service"
        )
      }
    end
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

# 回傳的 value

處理完這個問題後，我們終於可以看到回傳的結果是什麼了

```ruby=
F                                              

Failures:                                      

  1) Appointment::ServiceCollection#to_h works   
     Failure/Error: expect(collection.to_h).to eq('asdf')                                     
                                               
       expected: "asdf"                        
            got: [{"id"=>1, "appointment_id"=>1, "service_id"=>1, "created_at"=>Sun, 24 Feb 2019 16:45:16 EST -05:00, "updated_at"=>Sun, 24 Feb 2019 16:45:16 EST -05:00, "length"=>nil, "stylist_id"=>2, "price"=>"0.00", "label"=>"ve0xttqqfl", "item_id"=>1, "type"=>"service"}]        
                                               
       (compared using ==)                     
                                               
       Diff:                                   
       @@ -1,2 +1,12 @@                        
       -"asdf"                                 
       +[{"id"=>1,                             
       +  "appointment_id"=>1,                 
       +  "service_id"=>1,                     
       +  "created_at"=>Sun, 24 Feb 2019 16:45:16 EST -05:00,                                 
       +  "updated_at"=>Sun, 24 Feb 2019 16:45:16 EST -05:00,                                 
       +  "length"=>nil,                       
       +  "stylist_id"=>2,                     
       +  "price"=>"0.00",                     
       +  "label"=>"ve0xttqqfl",               
       +  "item_id"=>1,                        
       +  "type"=>"service"}]                  
                                               
     # ./spec/models/appointment/service_collection_spec.rb:8:in `block (3 levels) in <top (required)>'                                                                                      

Finished in 0.72959 seconds (files took 3.24 seconds to load)                                 
1 example, 1 failure
```

看到回傳的結果真的很有趣，因為這是在重構一個舊專案，所以看到回傳值的時候就覺得好像要解開秘密了～

# 使用更直接的 Expectation

我們要用更現實的數值來進行測試。我知道測試會失敗，因為數值是隨意捏造的。


{{< tabbed-codeblock "More Detail Expectation">}}
<!-- tab ruby -->
RSpec.describe Appointment::ServiceCollection do
  describe '#to_h' do
    it 'works' do
      appointment_service = create(:appointment_service)
      collection = Appointment::ServiceCollection.new([appointment_service])
      
      item = collection.to_h.first
      expect(item['price']).to eq('30.00')
      expect(item['label']).to eq('Robert Chang')
      expect(item['item_id']).to eq(appointment_service.item.id)
      expect(item['type']).to eq('service')
    end
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

這在執行測試是為了讓我的測試設置更符合我們期待的值！

{{< tabbed-codeblock "More Detail Expectation">}}
<!-- tab ruby -->
RSpec.describe Appointment::ServiceCollection do
  describe '#to_h' do
    it 'works' do
      appointment_service = create(
        :appointment_service, 
        price: 30,
        service: create(:service, name: 'Robert Chang'))
        
      collection = Appointment::ServiceCollection.new([appointment_service])
      
      item = collection.to_h.first
      expect(item['price']).to eq('30.00')
      expect(item['label']).to eq('Robert Chang')
      expect(item['item_id']).to eq(appointment_service.item.id)
      expect(item['type']).to eq('service')
    end
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

# 測試通過後，我要做什麼？

現在測試通過了，到目前為止，我們盡可能減少接觸原本程式碼的機會，因為我想最大限度的保留原本功能。現在我有了測試，我的測試可以保有原本的功能，而且我還可以自由地重構這段程式碼！


{{< tabbed-codeblock "What Is Next?">}}
<!-- tab ruby -->
module Appointment
  class ServiceCollection
    include ActionView::Helpers::NumberHelper
    ITEM_TYPE = 'service'

    def initialize(services)
      @services = services
    end

    def to_h
      @services.map do |service|
        service.serializable_hash.merge(extra_attributes(service))
      end
    end

    private

    def extra_attributes(service)
      {
        'price'   => number_with_precision(service.price, precision: 2),
        'label'   => service.service.name,
        'item_id' => service.service.id,
        'type'    => ITEM_TYPE
      }
    end
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

順便來整理一下測試的代碼：

{{< tabbed-codeblock "Refactor Test Code">}}
<!-- tab ruby -->
RSpec.describe Appointment::ServiceCollection do
  describe '#to_h' do
    let(:appointment_service) do
      create(
        :appointment_service,
        price: 30,
        service: create(:service, name: 'Robert Chang')
      )
    end

    let(:item) { Appointment::ServiceCollection.new([appointment_service]).to_h.first }

    it 'adds the right attributes' do
      expect(item['price']).to eq('30.00')
      expect(item['label']).to eq('Robert Chang')
      expect(item['item_id']).to eq(appointment_service.service.id)
      expect(item['type']).to eq('service')
    end
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

最後，我們就可以用我們新做的類別來取代原先的程式碼：

{{< tabbed-codeblock "Refactor Done">}}
<!-- tab ruby -->
# 原本的

def services_with_info
    services.collect { |item|
      item.serializable_hash.merge(
        "price": number_with_precision(item.price, precision: 2),
        "label": item.service.name,
        "item_id": item.service.id,
        "type": "service"
      )
    }
end

# 現在的

def services_with_info
  Appointment::ServiceCollection.new(services).to_h
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

# 小結

這在重構的路上可以說是杯水車薪，但現在我們已經減少了很多的程式碼，因為其實 Rails 的 MVC 也沒有那麼夠用，並不是真的讓 Model 肥就是好，像這種取得資訊的方法，我們說不定可以把它全部拉出來整理！

但至少我們今天也知道了，如果一段程式碼他沒有測試，而且你還不知道他會回傳什麼的時候，嘗試看看這個表徵測試的方法，一步一步的找到能夠整理的邏輯，並且同時補上測試！




