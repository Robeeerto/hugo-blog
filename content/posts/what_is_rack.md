---
title: "什麼是 Rack ?"
date: 2022-03-15T17:14:44+08:00
metaAlignment: center
thumbnailImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202203151716337.png'
coverImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202203151717903.jpg'
coverMeta: out
coverSize: partial
categories:
- Ruby
tags:
- Rack
summary: 介紹 Rack 以及如何使用
keywords:
- Rack
- Http
---

{{< toc >}}

Rack 是 Ruby 所有的網路框架背後最底層的技術，Rack 可以是：

１。 一個架構：Rack 定義了一個非常簡單的結構，任何符合這個結構的程式碼都可以在 Rack 的應用程式中被應用，這讓我們在建構小型、集中、可重複使用的程式碼段落變得簡單，然後組合這些 Rack 變成一個更大的應用程式。

２。一個 Ruby Gem：以 Ruby Gem 的形式發布，並且提供了組合程式碼所需要的 glue code。

# Rack Interface

Rack 定義了一個很簡單的 interface 以及接口，符合 Rack 標準的程式碼包含下面三點：

１。他必須對 `call` 做出回應，`call` 方法必須接受一個參數，通常是 `env` 或是 `environment`，因為他會匯集所有關於這次請求的資料，就像是這次請求送過來的所有參數。

２。`call` 方法必須回應一個由 `HTTP狀態碼` & `Headers` 以及 `body`(內容) 所組成的陣列

３。關於 `call` 有個很棒的點就是 `Proc` 以及 `lambda` 可以被當作 Rack 的物件做使用

# Rack Hello World

接下來的例子是一個符合 Rack 標準的 app，簡單地回應 Hello World

{{< tabbed-codeblock Hello_World >}}
<!-- tab ruby -->
require 'rack'

class HelloWorld
  def call(env)
    [ 200, { "Content-Type" => "text/plain" }, ["Hello World"] ]
  end
end

run HelloWorld.new
<!-- endtab -->
{{</ tabbed-codeblock>}}

在上面的例子裡面，我們實踐了一個類別搭配一個實體方法 `call`，並且加入環境變數，然後總是回傳 HTTP Status 200、一個 `Content-type` 的 headers，最後是內容 `Hello World`。

還有一個需要注意的地方，Rack 期待 body 回傳的部分是能夠套用 `each` 方法，所以單純的字串需要被放到陣列裡面才能夠正確的回應，但 body 大部分都是其他類型的 IO 物件，而不是單純的字串，只要不是單純字串，都可以不需要套入陣列之中。

# 探索 env 物件

{{< tabbed-codeblock ENV_Object >}}
<!-- tab ruby -->
require 'rack'

class HelloWorld
  def call(env)
    [ 200, { "Content-Type" => "text/plain" }, [env.inspect] ]
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

將原本的程式碼改成上面的樣子，就可以看到 env 物件本身是一個 hash，然後內部集合了所有關於這次 request 和 response 的內容。

裡面包含了使用者使用的裝置，cookies 以及 request 的路徑等等，全部的值都是簡單的字串，而不是像 rails 一樣複雜結構的 hash，但是所有的東西都在這包 hash 內，也供我們能夠檢查和組成我們的伺服器端的回應！

# 使用 lambda

這個示範只是好玩，並證明上面提到可以使用 lambda 或是 Proc 來執行 rack！

{{< tabbed-codeblock lambda_object >}}
<!-- tab ruby -->
require 'rack'

app = -> (env) do
  [200, { 'Content-type' => 'text/plain'}, [env.inspect]]
end

run app
<!-- endtab -->
{{</ tabbed-codeblock>}}

# Middleware

Middleware 是用 rack 模式所組成的，通常會是大型應用程式的小積木，每一個 middleware 都是一個和 rack 相容的程式，而最終的應用程式就是透過這些 middleware 組合再一起而成。

# Logging Middleware 實作

不像一般的 rack 程式，middleware 必須是類別，因為需要 `initialize` 來存放 app 並且傳送給執行鏈上的下一個 middleware 或是 rack。

像是下面的 middleware 示範，我們在 request 以及 response 中間加入了 logs 來計算回應所花費的時間。

但在一開始，我們需要讓 Rack app 先沈睡三秒，這樣我們才有東西可以計算，來看看怎麼寫吧！

{{< tabbed-codeblock middleware >}}
<!-- tab ruby -->
require 'rack'

app = ->(env) do
  sleep 3
  [200, {'Content-type' => 'text/plain'}, ['Hello, World']]
end

class LoggingMiddleware
  def initialize(app)
    @app = app
  end

  def call(env)
    before = Time.now.to_i
    status, headers, body = @app.call(env)
    after = Time.now.to_i
    log_messages = "App Work #{after - before} second."

    [status, headers, [log_messages.inspect]]
  end
end

run LoggingMiddleware.new(app)
<!-- endtab -->
{{</ tabbed-codeblock>}}

這裡我們可以看到，我們交給 `LoggingMiddleware` 的 `app` 是最基礎的應用程式，被我們的 middleware 給包覆住了。

在 middleware 的 `call` 方法中，我們擷取了時間戳記，並把請求轉送給執行鏈上的下一個 `app`，每一個 middleware 都只知道下一個是誰，所以在最終的結果出來之前，可能有超級多層的 middleware。

在我們 `call` 下一個 app 的那一行，我們解構了 app 回傳的陣列到 `status, headers, body`，這樣可以讓我們針對回傳的結果做檢查或是更改。

在這個 `LoggingMiddleware` 的情況下，我們把時間戳記加入到了回應的 `body` 內，並且讓其顯示在畫面中。

# Middleware 的實際用法

感謝 Rack interface 的簡單性和靈活性，middleware 被用在大量不同的功能。

有一個很酷的東西叫做 `rack-honeypot` 可以拿來偵測和應用程式互動的垃圾郵件或是惡意機器人，他在所有的 response 中加入了一個隱藏的欄位，是只有機器人才有可能填寫的，然後他會解析這個所有的請求，並取消掉有填了隱藏欄位的請求。

# Rack::Builder

雖然 Rack 和使用 middleware 是一個很好把應用程式組合起來的方式，但他可能會變得有點冗長，而 `Rack::Builder` 是一個模式，可以讓我們更簡潔地定義我們的應用程式和 middleware

我們可以在 `config.ru` 中定義，這是一個 Rack 專用的文件 ( `.ru` 代表 `rackup` )

{{< tabbed-codeblock rack_builder >}}
<!-- tab ruby -->
app = ->(env) do
  sleep 3
  [200, {'Content-type' => 'text/plain'}, ['Hello, World']]
end

class LoggingMiddleware
  def initialize(app)
    @app = app
  end

  def call(env)
    before = Time.now.to_i
    status, headers, body = @app.call(env)
    after = Time.now.to_i
    log_messages = "App Work #{after - before} second."

    [status, headers, [log_messages.inspect]]
  end
end

use LoggingMiddleware
run app
<!-- endtab -->
{{</ tabbed-codeblock>}}

主要的變化就是可以透過 `run` & `use` 來建構我們的應用程序，可以一目瞭然地知道使用了什麼樣子的 middleware

# The middleware stack

上面的 `Rack::Builder` 方法可以讓我們更輕鬆地建構應用程式的 `middleware stack`，middleware 將會已特定地順序排序，並以特定地順序執行。

這邊就是所謂的 FILO ( First In Last Out )，所以執行的順序會先是 `run` 的那個主程式，因為他是最後放進 stack 裡面的，接著再往上執行( 就是往程式碼的上放繼續執行下去 )

一個 request 將會穿越這個 middleware stack 來到最終的 app，然後所有的 response 都會透過執行鏈向上傳遞。

{{< tabbed-codeblock middleware_stack >}}
<!-- tab ruby -->
use FirstMiddleware #第四個執行
use SecondMiddleware #第三個執行
use ThirdMiddleware #第二個執行

run app #第一個執行
<!-- endtab -->
{{</ tabbed-codeblock>}}

# 用 rackup 跑起來！

當我們有了 `config.ru` 來組織我們的 middleware，我們就可以利用 `rack gem` 提供的 `rackup` 來執行我們的 rack app

`rackup` 執行的時候，將會自動載入 `rack`，並且 `use` & `run` 方法將會被使用。

# 我們可以用 Middleware 做些什麼？

Middleware 很適合拿來用在非應用程式的特定邏輯。像是設置暫存的 headers，logging，解析 request 物件等等...

上述提到的幾個用法，都是在 Rails 裡面很常見的 middleware！

# 結語

Rack 本身提供了強大又簡潔的抽象概念，讓我們可以去分離應用程式的核心邏輯和外圍的關注點 ( ex: 解析 HTTP headers )

他幾乎是所有 Ruby 有關的網路框架拿來運行和 server 之間的 interface，真的是非常好用！

會想寫關於 Rack 的東西是因為，今天被有被問到關於 Rack 的問題，但覺得自己解釋的並沒有很好，所以才趕快來研究並且紀錄！