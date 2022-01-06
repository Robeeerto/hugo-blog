---
title: "Day 25 用 WebMock + VCR 來實作測試"
date: 2021-10-09T15:12:30+08:00
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

假的 HTTP 請求！
<!--more-->
{{< toc >}}
WebMock 以及 VCR 是拿來實作關於網站請求的工具，在這篇文章中也會解釋兩者分別是什麼，並且做一個簡單的教學！

# 為什麼需要 WebMock & VCR ?

測試的原則之一就是測試的確定性，一個測試的通過以及失敗都應該是應用程式的程式碼內容來決定，而不是其他的因素，所有的測試都理應要通過，無論他是用什麼方式運行，在什麼時間運行，或是其他的因素。

由於這個原因，我們就必須在封裝的世界中運行測試，如果我們想要測試的確定性以及穩定性，我們就不該讓測試和網路對話，因為網路有很多的變因可能改變測試的結果。

想像一下程式裡面和第三方 API 的交流，在想像一下每次的測試都會去觸碰這個 API，在想像一下，某一次的測試運行中，第三方的 API 碰巧壞掉了，導致測試的失敗，但這次的失敗明顯是不合邏輯的，因為測試要做的目的是要告訴我們的程式碼有問題，但這次不是，是外部的干擾導致而成！

如果我們的測試不想和真實的網路對話，那我們就需要一種方法來模擬和網路的互動，讓我們的應用程式在一個穩定的環境中進行測試，這就是 VCR & WebMock 為什麼會存在的原因！

我們也不希望測試的過程中，真實的對資料有任何的變動，假設我們寫了一個刪除用戶的測試，總不希望測試的過程中真的影響了資料吧？因此，VCR & WebMock 也給了我們不必接觸真實資料的功能！

# 兩者之間的不同

VCR 是一個拿來紀錄你的應用程式和 HTTP 交流過程的工具，並且在之後進行回放，而且不需要寫什麼程式碼，VCR 欺騙你的應用程式，讓他以為他正在接受來自網路的回覆，但實際上你的專案只是在接受預錄的 VCR 數據而已！

WebMock 則不像 VCR 那樣可以紀錄和回放，儘管 HTTP 的交流是可以被偽造的，但那和 VCR 還是有所不同，WebMock 需要寫更多的程式碼，而且也有諸多的限制，在這次的文章中，會有很基本的介紹和了解！

# 該怎麼做？

這篇文章將會做一個簡單的例子，說明如何使用 VCR & WebMock 來滿足需求，執行對於網路的請求，但不實際的發送請求出去！

至於這篇文章的基礎 Rails Setup 就不展示了，主要專注在工具的使用上面！

**情境**

我們要寫一個小型的搜尋功能，去請求第三方的 API，所以在測試中，我們也該發送出 API 來完成測試

**WebMock**

一但測試完成的差不多了，我們就會安裝和設定 WebMock，這時候會禁止任何的網路請求，因此，我們的測試會停止運作。

**VCR**

最後，我們會安裝和設定 VCR，VCR 能夠和 WebMock 做互動，也因為這樣，VCR & WebMock 可以一起合作，在某些條件下，我們的測試可以進入網路，VCR 將紀錄測試過程的 HTTP 請求及回覆，接著在之後的測試中，都會使用 VCR 的回放，而不是運作都發送新的請求！

# 功能實作

> 因為臨時想不到有什麼第三方的資源可以去打，就用想像的方式來進行～

今天這篇文章實作的功能是一個搜尋某些第三方資料的功能，使用者可以輸入姓名，點擊搜尋，然後會有資料出現！

下面是 Controller 的程式碼：

{{< tabbed-codeblock "Controller" >}}
<!-- tab ruby -->
# app/controllers/data_searches_controller.rb
ENDPOINT_URL = "https://xxxx/api"

class DataSearchesController < ApplicationController
  def new
    @results = []
    return unless params[:first_name].present? || params[:last_name].present?

    query_string = {
      first_name: params[:first_name],
      last_name: params[:last_name]
    }.to_query

    uri = URI("#{ENDPOINT_URL}/?#{query_string}")
    response = Net::HTTP.get_response(uri)
    @results = JSON.parse(response.body)["results"]
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

我們也應該要有 View 來 render 畫面：

{{< tabbed-codeblock "View" >}}
<!-- tab ruby -->
<%= form_tag xxx_search_path, method: :get do %>
  <%= text_field_tag :first_name, params[:first_name] %>
  <%= text_field_tag :last_name, params[:last_name] %>
  <%= submit_tag "搜尋" %>
<% end %>

<% @results.each do |result| %>
  <div>
    <%= result["basic"]["first_name"] %>
    <%= result["basic"]["last_name"] %>
    <%= result["number"] %>
  </div>
<% end %>
<!-- endtab -->
{{</ tabbed-codeblock>}}

# 寫測試

簡單的寫一個 feature test，在姓名的欄位分別填入 "Robert" & "Chang"，點擊搜尋，然後期待 Robert Chang 的某個識別證號碼會出現在畫面上！

{{< tabbed-codeblock "Test" >}}
<!-- tab ruby -->
# spec/feature/data_search_spec.rb

require "rails_helper"

RSpec.feature "Data Search", type: :feature do
  scenario "show the identify number" do
    visit data_search_path
    fill_in "first_name", with: "Robert"
    fill_in "last_name", with: "Chang"
    click_on "搜尋"

    # 這段是 RobertChang 的識別碼
    expect(page).to have_content("118583921")
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

OK，這樣會通過，沒有問題！

# 安裝以及設定 WebMock

像剛剛前面提過的，我們不希望測試時也真的送出第三方的 API 請求，所以我們需要假造一段請求。

我們先把 WebMock 裝入 Gemfile：

```ruby=
group :development, :test do
  gem 'webmock', '~> 3.14'
end
```

第二步，我們建立一個檔案在 `spec/support/webmock.rb`，並且加入下面的程式碼：

```ruby=
# spec/support/webmock.rb

# This line makes it so WebMock and RSpec know how to talk to each other.
require "webmock/rspec"

# This line disables HTTP requests, with the exception of HTTP requests
# to localhost.
WebMock.disable_net_connect!(allow_localhost: true)
```

記得到 `spec/rails_helper.rb` 裡面去 uncomment 下面那一行，不然 `spec/support` 的文件不會被 Load 進來用。

```ruby=
# spec/rails_helper.rb

# Make sure to uncomment this line
Dir[Rails.root.join('spec', 'support', '**', '*.rb')].sort.each { |f| require f }
```

第三步：看測試壞掉！

安裝之後再跑了一次測試，他會壞掉，然後說 `Real HTTP connections are disabled`，還給我們一些提示，叫我們可以去 Stub 這個請求，但我們不這麼做，我們要用 VCR 來替代！

```ruby=
Failures:

  1) Data Search show the indentify number
   Failure/Error: response = Net::HTTP.get_response(uri)

   WebMock::NetConnectNotAllowedError:
     Real HTTP connections are disabled. Unregistered request: GET https://xxx/api/?first_name=Robert&last_name=Chang with headers {'Accept'=>'*/*', 'Accept-Encoding'=>'gzip;q=1.0,deflate;q=0.6,identity;q=0.3', 'Host'=>'xxx', 'User-Agent'=>'Ruby'}

     You can stub this request with the following snippet:

     stub_request(:get, "https://xxx/api/?first_name=Robert&last_name=Chang").
       with(
         headers: {
           'Accept'=>'*/*',
           'Accept-Encoding'=>'gzip;q=1.0,deflate;q=0.6,identity;q=0.3',
           'Host'=>'xxx',
           'User-Agent'=>'Ruby'
           }).
         to_return(status: 200, body: "", headers: {})

       ============================================================
```

# 安裝以及設定 VCR

一樣先加入 Gemfile：

```ruby=
# Gemfile

group :development, :test do
  gem 'vcr', '~> 6.0'
end
```

接著我會加入這個設定檔，我有加入了自己的註解，這樣可以更快理解！

```ruby=
# spec/support/vcr.rb

VCR.configure do |c|
  # 這個資料夾是存放 VCR 的地方，就是存放和 HTTP 紀錄的過程
  c.cassette_library_dir = "spec/cassettes"

  # 這行讓 VCR 和 WebMock 知道如何和彼此溝通！
  c.hook_into :webmock

  # 這行讓 VCR 忽略了對 localhost 的請求，這是必要的儘管 WebMock allow_localhost 是 true
  c.ignore_localhost = true

  # ChromeDriver 會向 chromedriver.storage.googleapis.com 發送 update 的請求。
  # 這些請求會在我們的 cassette 中發出噪音，除非我們告訴 VCR 去忽略他
  c.ignore_hosts "chromedriver.storage.googleapis.com"
end
```

**加入 VCR 囉！**

現在我們可以加入 `VCR.use_cassette "data_search"` 的 block 進入測試中，`data_search` 是可以任意取的名字，只是讓 VCR 可以辨識。

{{< tabbed-codeblock "Test" >}}
<!-- tab ruby -->
# spec/feature/data_search_spec.rb

require "rails_helper"

RSpec.feature "Data Search", type: :feature do
  scenario "show the identify number" do
    VCR.use_cassette "data_search" do # <----- 加這行
      visit data_search_path
      fill_in "first_name", with: "Robert"
      fill_in "last_name", with: "Chang"
      click_on "搜尋"

      # 這段是 RobertChang 的識別碼
      expect(page).to have_content("118583921")
    end
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

上次在只有安裝 WebMock 的時候噴錯了，因為 WebMock 阻止了 HTTP 的請求，但現在執行這個測試會通過，因為 VCR 和 WebMock 一起讓 HTTP 請求發生。

結束會去看看 `spec/cassettes` 資料夾，會發現有一個叫做 `data_search.yml` 的檔案，內容如下：

```ruby=
---
http_interactions:
- request:
    method: get
    uri: https://xxx/api/?first_name=Robert&last_name=Chang
    body:
      encoding: US-ASCII
      string: ''
    headers:
      Accept-Encoding:
      - gzip;q=1.0,deflate;q=0.6,identity;q=0.3
      Accept:
      - "*/*"
      User-Agent:
      - Ruby
      Host:
      - xxx
  response:
    status:
      code: 200
      message: OK
    headers:
      Date:
      - Fri, 18 Oct 2021 03:02:41 GMT
      Content-Type:
      - application/json
      Strict-Transport-Security:
      - max-age=31536000; includeSubDomains
      Set-Cookie:
      - TS017b4e40=01acfeb9489bd3c233ef0e8a55b458849e619bdc886c02193c4772ba662379fa1f8493887950c06233f28bbbaac373afba8b58b00f;
        Path=/; Domain=xxx
      Transfer-Encoding:
      - chunked
    body:
      encoding: UTF-8
      string: '{ 回傳的資料... }'
  recorded_at: Fri, 18 Oct 2021 03:02:41 GMT
recorded_with: VCR 6.0.0
```

接著在之後的每次測試，VCR 都會問，有沒有一個叫做 `data_search` 的錄影帶，沒有的話就允許去執行一次 HTTP 的請求，有的話就使用它！

# 小結

今天花了蠻多的時間寫這個的，內容都是根據之前自己玩的 side project 改編內容而成～

所以 VCR 中的回傳內容應該會是一些 json 格式的資料，但我相信理解概念就可以了，和實際運作的 Code 沒有太大的關係！

感謝收看～




