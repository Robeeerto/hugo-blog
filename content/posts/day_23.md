---
title: "Day 23 介紹 FactoryBot Rails 及設定"
date: 2021-10-07T18:38:21+08:00
metaAlignment: center
thumbnailImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211620030.png'
coverImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211620030.png' 
coverMeta: out
coverSize: partial
categories:
- "2021 鐵人賽"
tags:
- RSpec
summary: 超好用的 FactoryBot 介紹
keywords:
- RSpec
- FactoryBot
- FakeData
---

{{< toc >}}

首先，我們一樣先介紹基本的安裝以及設定，當一切就緒的時候，寫測試這件事情也就沒有那麼複雜和麻煩了～

# 安裝 factory_bot_rails

和 RSpec 一樣，我們把 `factory_bot_rails` 放在 Gemfile：

```ruby=
group :development, :test do
  gem 'factory_bot_rails', '~> 6.2'
end
```

記得，在實作專案的時候都要記得放上版本號喔！

`factory_bot_rails` 本身其實會根據以下的路徑自動載入：
```ruby=
factories.rb
test/factories.rb
spec/factories.rb
factories/*.rb
test/factories/*.rb
spec/factories/*.rb
```

但若是你有自己想要放置的檔案位置的話呢？

你可以在 `config/application.rb` 或是 `config/envrioments.rb` 中加入以下的指令：

```ruby=
config.factory_bot.definition_file_paths = ["你的資料夾名稱/factories"]
```

而 `factory_bot_rails` 在開發環境時會自動地幫你產生工廠來產生物件，若是你覺得很煩，你想要自己建立檔案的話呢？

你可以把 `factory_bot_rails` 移出 `:development` 的 group 或是在設定中加入：

```ruby=
config.generators do |g|
  g.factory_bot false
end
```

接著我們還有一些基本的設定要做，我們可以開一個資料夾，路徑是 `spec/support/factory_bot.rb` 然後在裡面放入這段程式碼，並確認他會被 `rails_helper.rb` 給 require

```ruby=
RSpec.configure do |config|
  config.include FactoryBot::Syntax::Methods
end
```

這段的目的應該不難理解，就是 include `FactoryBot` 的方法，讓我們可以在 `_spec.rb` 的檔案中寫 `FactoryBot` 的方法喔！

# 製造工廠

`FactoryBot` 最大的目的就是製造假的實體讓我們的測試可以通過，因為若是測試也需要把資料都存進資料庫的話，專案完成的過程中就會浪費很多的硬碟空間，這件事情是沒有必要發生的！

後面也會提到安裝 `Database_cleaner` 在每次測試前都清空資料庫確保每次的測試環境都是乾淨的。

這邊先示範一個最基本的製造工廠的程式碼：

{{< tabbed-codeblock "Basic" >}}
<!-- tab ruby -->
FactoryBot.define do
  factory :user do
    first_name { "John" }
    last_name  { "Doe" }
    admin { false }
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

我們從第二行開始看， `factory :user` 的意思是，`User` 類別的工廠，這個 Model 有三種屬性，分別是 `first_name` 以及 `last_name` 和 `admin` 。

這樣我們就可以在寫測試的時候使用：

```ruby=
build(:user) / create(:user)
# 這樣就可以建立 user 實體，也可以把它賦予在變數裡面
robert = build(:user)
# 這樣他本身就會有我們在工廠裡面設定好的屬性！
robert.first_name # "John"
```

我們也可以用不同的命名，但使用同樣的類別：


{{< tabbed-codeblock "Different naming" >}}
<!-- tab ruby -->
FactoryBot.define do
  factory :admin, class: 'User' do
    first_name { "John" }
    last_name  { "Doe" }
    admin { false }
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

這樣的話就可以這樣寫：

```ruby=
build(:admin)
```

# Sequence

除了剛剛提到基本屬性的製造外，我們常常會有不能重複的屬性，像是 `email` 這樣子的屬性！

這時候我們就可以使用 `sequence` 來製作獨特的屬性，使用方法是：


{{< tabbed-codeblock "sequence" >}}
<!-- tab ruby -->
FactoryBot.define do
  factory :user do
    first_name { "John" }
    last_name  { "Doe" }
    admin { false }
    email { generate(:email) }
    
    sequence(:email) { |n| "person#{n}@example.com" }
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

接著我們可以製造幾個實體並且印出來看看！

```ruby=
person_1 = build(:user)
person_2 = build(:user)

person_1.email # "person1@example.com"
person_2.email # "person2@example.com"
```

那個 `n` 會自動地幫我們遞增，也順便讓這個值成為 `unique`，當然 sequence 有很多的特性和縮寫，短短一篇文章也很難交代完整。

所以盡量以基本的使用方法來介紹～

# build & create 的不同

這兩種方式都可以製造出物件的實體，但最大的不同就在於有沒有被 `save`，那至於有沒有 `save` 有什麼關係嗎？

有的，如果沒有 `save` 的話，在 Feature test 就會找不到畫面的顯示，因為該物件根本沒有被儲存，所以不會出現在畫面上。

其實只要想像到底會不會需要物件的 `id` 或是關聯需要 `id` 的？

如果需要，請用 `create` 來製造物件！

若是我想要一次製造很多的物件呢？

`FactoryBot` 也有提供了 `create_list / build_list` 的方法，可以這樣使用：

```ruby=
create_list(:user, 3) # save 3 個 user
build_list(:user, 3)
```

至於後面的數字就是你需要幾個的意思！

# Association

在寫 Rails 時，我們常常會需要很多的關聯，在測試時也不例外。

一開始我在學習使用 `FactoryBot` 的時候感到很複雜，也很困擾，但搞懂之後就覺得根本沒有那麼困難～

我們先假設一間商店有很多的漢堡，這時候我們的工廠該怎麼寫呢？


{{< tabbed-codeblock "association" >}}
<!-- tab ruby -->
FactoryBot.define do
  factory :burger do
    association :store, factory: :store
  end
end

# 也可以縮寫成這樣

FactoryBot.define do
  factory :burger do
    store
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

首先是 `association :store` 這件事，就是建立關聯，接著我們告訴 `FactoryBot` 去哪找這個關聯，`factory: :store` 這間工廠！

這樣寫的關聯就是 `Store has_many :burgers` 的意思，因為身上會有 `store_id` 的表格應該是 `Burger` 才對！

# After Create Hooks

今天的情境是，我們想要一間商店在 `create` 之後，就自動擁有 3 個漢堡。

一般來說在測試裡面，你可能會這樣寫：

```ruby=
let(:store) { create(:store) }

before do 
  create(:burger, store: store)
  create(:burger, store: store)
  create(:burger, store: store)
end
```

OK，這或許沒有什麼問題，也可以達到你的目的，但當測試檔案變大的時候，就會很難看懂，因為太多這樣的 `before do end` 的情形了

所以我們希望這個 `store` 在建立起來的時候就有漢堡了！

我們可以在工廠裡面先訂立好規則：

{{< tabbed-codeblock "after create hook" >}}
<!-- tab ruby -->
FactoryBot.define do
  factory :store do
    ...
    
    after :create do |store|
      create_list(:burger, 3,  store: store)
    end
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

這段 Code 的意思就是在 create 的瞬間幫我產生 3 個漢堡～

這樣我們就可以讓 `_spec.rb` 的檔案很乾淨，很清晰！

# Traits

這是我個人覺得超級好用的功能，可以讓你幫同一個物件貼上不同的標籤。

什麼意思呢？

假設我們的 `User` 有不同的角色，有管理員，有一般人，這時候原本的我會這樣寫：

```ruby=
robert = create(:user)
robert.role = "Admin"
....
```

利用後來覆寫的方式來改變狀態，但其實我們可以不用這樣做，有超級方便的方法讓我們來使用！

我們一樣在工廠裡面做好設定：

{{< tabbed-codeblock "trait" >}}
<!-- tab ruby -->
FactoryBot.define do
  factory :user do
    ...
    
    traits :admin do
      role { 'Admin' }
    end
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

我們可以想像在工廠裡面已經有一張貼紙叫做 `:admin`，接著我們可以這樣用：

```ruby=
robert = create(:user, :admin)
robert.role # 'Admin'
```

是不是超級好用的，我們就可以把各式各樣的 `traits` 先建立起來，用於應付各式各樣的測試環境喔！

# 安裝 DatabaseCleaner

剛剛有提過，我們希望在每次測試的開始前都有乾淨的環境可以測試，所以我們也要在環境中加入這個 Gem。

我們依照手冊來安裝：

```ruby=
# Gemfile
group :test do
  gem 'database_cleaner-active_record', '~> 2.0', '>= 2.0.1'
end
```

一樣在 `spec_helper.rb` 中加入：


{{< tabbed-codeblock "database cleaner" >}}
<!-- tab ruby -->
RSpec.configure do |config|

  config.before(:suite) do
    DatabaseCleaner.strategy = :transaction
    DatabaseCleaner.clean_with(:truncation)
  end

  config.around(:each) do |example|
    DatabaseCleaner.cleaning do
      example.run
    end
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

可以看到他在每一次的測時前都會執行這些指令唷！

關於 `transaction` 以及 `truncation` 在這個套件的差別：

> transaction: rollback 的方式
> truncation: 清除資料庫的所有資料

# 小結

`FactoryBot` 的使用方式實在是太多了，就像寫測試一樣，你問一百個人會有一百種寫法～

我真的沒辦法一一詳細介紹到每個方法，但這些方法對我來說就很夠用了，甚至你也可以使用 `traits` 搭配 `after_create hooks` 來製造標籤以達到很多種不一樣的效果。

如果真的很閒可以看 [官方文件](https://github.com/thoughtbot/factory_bot/blob/master/GETTING_STARTED.md)，但我都是想到實作某種方式才去查我需要的東西。

明天會介紹另一個也很有趣的套件 Capybara ～





