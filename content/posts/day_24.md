---
title: "Day 24 介紹 Capybara 及設定"
date: 2021-10-08T18:32:02+08:00
metaAlignment: center
thumbnailImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211620030.png'
coverImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211620030.png' 
coverMeta: out
coverSize: partial
categories:
- "2021 鐵人賽"
tags:
- RSpec
summary: 超常用的 Capybara！
keywords:
- RSpec
- Capybara
- setup
---

{{< toc >}}

在我們 new 出一個全新的 Rails 專案時，會在 Gemfile 裡面看到預設的測試套件，分別是：`capybara` & `selenium-webdriver` & `webdrivers`。

後面兩個從名字上就可以很明顯的感受的，像是要操縱網頁的概念，而事實也是如此喔！

Capybara 本身就是模仿一個真實使用者來測試你的應用程式，所以我們會需要有 `selenium` 這樣的工具來幫助我們自動化的啟動瀏覽器。

有點像我們撰寫程式語言，來操控一個虛擬的人物和瀏覽器互動的概念，從等等的程式碼就能感受得到了！

基於 Rails 專案本身就有安裝 capybara 了，我們就會直接用 RSpec 的視角來講一下這個東西該怎麼使用。

# Capybara 的優點

為什麼 Rails 會選擇 Capybara 做為預設的套件呢？

其實我也不清楚，但我自己有幾個覺得還蠻不錯的地方可以分享一下！

**1. 不需要特別的 set up**

這件事情對於我來說就很好，因為像是這種要脫離 Rails 專案設定一些關於網際網路的參數，或是一些看起來很麻煩的事情。

其實都會打斷開發的時程，所以有一個好的套件，可以讓你即插即用是很讚的一件事！

**2. 直觀的 API 像人類的語言**

等等示範的程式碼裡面，就會看到很有趣的 capybara API，都是一些超級簡單的英文，非常直觀。

例如：訪問一個網站

```ruby=
visit ( 訪問的 url )
```

這樣就會模擬一個用戶造訪這個網站，是不是看起來真的非常直白呢？

**3. 可以無痛切換無頭模式以及實際瀏覽器**

一般來說 capybara 會在無頭模式下進行測試網站，而無頭模式的瀏覽器是什麼意思？

無頭瀏覽器就是沒有我們現在使用的這些介面，通常透過 command line 來進行溝通。

而切換瀏覽器模式就是，我們可以在測試的時候讓他彈出瀏覽器的畫面，並且在你面前輸入帳號密碼等等，以便測試！

而 Capybara 可以讓你在這兩者間作切換，但不需要更動你原先的程式碼。

上面是三個我自己覺得很厲害的地方，可能因為 capybara 實在是太實用了吧，就被 Rails 官方給綁定了～

# 實際使用

在 RSpec 中，前天有提到的 Feature test，重要度非常高，就是使用 capybara 來完整整個流程。

那我們要如何在 RSpec 中使用它呢？

我們只要給他正確的 tag 就可以了。

像這樣：

```ruby=
RSpec.feature 'test something', type: :feature do
 ....
end
```

使用 `RSpec.feature`  並加入了 `type: :feature / type: :system` 就可以使用了喔！

有沒有真的很符合我提到的優點呢？ 基本上真的不太需要什麼特別的設定就可以簡單地使用了喔！

# 使用情境

接著會用一些簡單的程式碼讓人可以快速理解 capybara 在做些什麼，以及 feature test 到底在做些什麼呢？

假設我們要針對 `登入` 這項功能來做 feature test，我們可以先試想一下整個情境以及流程。

```ruby=
# 進入登入頁面 > 填寫帳號以及密碼 > 登入成功
```

這邊先不考慮失敗的情境，因為其實就是反過來而已！

我們先上程式碼再慢慢解說：

{{< tabbed-codeblock "Feature Test" >}}
<!-- tab ruby -->
RSpec.feature 'Sign In', type: :feature do
  
  before do
    create(:user)
    # 先讓資料庫有這筆資料
  end

  scenario 'sign in process' do
    visit '/sessions/new'
    within("#session") do
      fill_in 'Email', with: 'user@example.com'
      fill_in 'Password', with: 'password'
    end
    click_button 'Sign in'
    expect(page).to have_content 'Success'
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

這邊的 `scenario` 等同於之前我們使用的 `it`， 但語意更明確一些像是創造一個情境的概念。

接著我們看到第 9 行的程式碼，拜訪 `/sessins/new` 這個網站，但這邊的寫法不太好，其實我們可以用 rails 的 `routes prefix` 來造訪網站喔！

```ruby=
visit new_user_session # users/sessions#new
```

而看到第 10 行 `within('#session')` 是找到這個表格，而 `#session` 這就是我們熟悉的 CSS 選擇器找 ID 的用法。

進入表格後呢，找到 `Email` 以及 `Password` 這兩個欄位並且填入我們事先創建好的 User 的帳號密碼！

在到 第 14 行，按下登入按鈕，並且期待頁面會有 `Success` 的文字出現！

這樣算是完成了一個簡易的 feature test，這邊的示範也呼應了我提到的第二個優點，Capybara 使用超級簡單的英文，讓呼叫 API 的過程變得流暢和舒服！

所以我們這次的測試流程是：

```ruby=
# 拜訪登入網站 > 找到登入的表格 > 填寫帳號密碼 > 按下登入按鈕 > 期待看到成功訊息！
```

因為是自動化的測試程序，所以機器其實比我們想像的還要笨，你必要要給予他很瑣碎的指令，你可以往上拉看到，以人類的我們來說想像的測試流程非常簡便。

是因為人類的腦袋幫助我們節省了很多習以為常的事情，但對於寫程式來說，這些都是要考慮進去的喔！

# 常見問題

**1. 我想看到瀏覽器彈出來的樣子！**

我們只需要在 `spec_helper.rb` 裡面加入：

```ruby=
Capybara.javascript_driver = :selenium_chrome
```

他就會在執行 feature test 的時候，從無頭模式切換到瀏覽器模式喔！

第一次看到的時候真的覺得超酷的，他會自動的填寫帳號密碼等等，真的像一個真實的使用者！

**2. 為什麼我的頁面找不到按鈕了？**

常常會遇到在 `click_button` 的時候找不到按鈕的情形。

最常遇到的有三種可能：

**第一種**：你的按鈕是跟隨著資料出現的，例如：編輯這顆按鈕，在你沒有資料的情境下，畫面上就真的沒有，所以要記得在 `before do` 的時候先建立好相關的資料，讓測試的情境更寫實喔！

**第二種**：你的按鈕在 `mobile` 的情形下會消失，有些專案可能會需要在 `mobile` 的情境時 hidden 某些按鈕，那就會造成 capybara 找不到它，所以我們也可以設定在測試時的視窗大小喔！

一樣在 `spec_helper.rb` 加入：

```ruby=
config.before(:each, js: true) do
  Capybara.page.current_window.resize_to(1080, 1920)
end
```

這樣在每次測試前，都會以這樣的視窗太小在進行測試。

**第三種**：你的按鈕是伴隨著事件出現的，例如你點了某個按鈕後，會利用 Ajax 到後端並且 render 出新的按鈕，但對於 capybara 來說，一切都太快了，他還沒等到按鈕出現，就已經在 click 了，自然而然就找不到。

所以我們也可以在 `spec_helper.rb` 加入：

```ruby=
Capybara.default_max_wait_time = 5
```

其實原先的 default 是兩秒，但我們可以提高秒數，來試試看會不會通過測試！

**3. 怎麼找我要的元素？**

capybara 有提供 css selector 以及 xpath 兩種方式來找到網頁上的元素，當你找不到的時候，可以檢查是不是資料在測試前寫入的順序不對，或是關聯沒建立好等等。

# 小結

今天的 capybara 就介紹到這邊，關於他的使用方法實在是太多了，但在官方文件都有詳細的說明，各個方法要如何使用。

但大部分都脫離不了以使用者角度出發的，拜訪網站、點擊按鈕、填寫等等

也有一些有趣的小功能，說不定會在你需要的時候使用到喔！



