---
title: "MVC 架構是不是就夠用了？"
date: 2021-10-04T03:57:01+08:00
metaAlignment: center
coverImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202110040405929.webp'
coverMeta: out
coverSize: partial
categories:
- "Ruby on Rails"
tags:
- OrganizeCode
---

通通寫在 Model 就好了？
<!--more-->
{{< toc >}}
# MVC 架構是不是就夠用了？

最近工作的時候，收到了這樣的問題，確實讓我想了蠻多的，也透過很多的文章來讓自己去思考，究竟 Rails 的 MVC 架構是不是就能夠處理你整個應用程式呢？

我認為答案是：可以

但這樣真的是符合物件導向的設計模式嗎？

下面這段話是 Wiki 中對於物件導向設計的描述：

> 物件導向設計被理解為一種將程式分解為封裝資料及相關操作的模組而進行的程式設計方式

以前剛開始在學 Rails 的時候，會習慣把 Code 全部塞在 Controller 裡面，像這樣... ( 我知道這段 Code 很鬧 )

但其實對於新手來說，這非常直覺，也好像很容易 debug，但我們真的有讓 MVC 各司其職嗎？

其實並沒有...

{{< tabbed-codeblock "Bad Code" >}}
<!-- tab ruby -->
def random_create
    params[:ticketCount].each do |per_ticket|
      for_sale_ticket =  Ticket.find(per_ticket[0].to_i).seats.where(status: 'for_sale')

      num = (for_sale_ticket.ids.count / 2).ceil

      id_array = for_sale_ticket.ids.sort

      per_ticket[1].to_i.times do
        seat = for_sale_ticket.find(id_array[num])
        ticket = Ticket.find(per_ticket[0].to_i)
        line_item = current_cart.line_items.new(seat_id: seat.id)
        line_item.save
        num += 1
      end
    end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

所以之後開始被教育，`fat model skinny controller`，這時候又開始把所有的邏輯都塞進去 Model 裡面。

基本上只要主詞是 Model 的類別或是實體變數，我就會寫一個 Model 的方法。

但變超肥的 Model 真的沒關係嗎？

其實道理一樣，都可以沒關係，但這樣有讓物件互動起來嗎？好像就失去了靈魂了，也失去了好擴充以及好維護的初衷！

於是 Rails 有 View Helper 以及 Concern 這兩個東西來幫助我們整理邏輯。

# View Helper 

光看名字就能夠猜到，這個 Helper 主要的功能是在頁面的呈現上給予幫助。

假設我們要根據一個漢堡的不同種類的起司分別給予不一樣的 icon 呢？


{{< tabbed-codeblock "View Helper" >}}
<!-- tab ruby -->
# Helper
module BurgerHelper
  def cheese_icon(type)
    case type
    when ricotta
      tag.i(...)
    when parmesan
      tag.i(...)
    when feta
      tag.i(...)
    end
  end
end

# View
<%= cheese_icon(@burger.cheese) %>
<!-- endtab -->
{{</ tabbed-codeblock>}}

這樣子 Model 是不是又瘦身一波了呢？而且這些關於畫面呈現的 Code 真的適合放在 Model 裡面嗎？

我想明顯的是非常的不適合，如果 Rails 有提供這樣方便的功能，其實真的能夠嘗試著移動自己塞在 Model 的程式碼試試看喔！

# Concern

在看文章的過程中，我看到了一段 DHH 對於 Concern 的描述：

> Concerns are also a helpful way of extracting a slice of model that doesn’t seem part of its essence。

其實就是，Concern 可以幫助你提取那些不屬於 Model 本質的片段～

有些時候某些邏輯其實不一定隸屬於某個特定的 Model，他可能是，通知，管理權限等等。

就拿權限來說吧，某些網頁能不能夠被特定的使用者給瀏覽，這難道是 User 這個 Model 本身的責任嗎？

能不能做些什麼，應該要拉出另一個邏輯來進行判斷，例如：


{{< tabbed-codeblock "Concern" >}}
<!-- tab ruby -->
# concern
module Authorized
  extend ActiveSupport::Concern
  
  def authorize(role)
    # 能不能瀏覽的邏輯放在這邊...
  end
  
  module ClassMethods
  end
end

# Controller
class PagesController < ApplicationController
  include Authorized
  
  def index
    authorize(current_user)
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

當然拆分邏輯的方式有很多種，但讓每個東西各司其職是一個在未來開發的過程中能夠道路暢通的方式！

當某個 Model 或是 Controller 承擔了太多的責任，那當發生錯誤的時候，就會需要大量的時間瀏覽原始碼，甚至改動會牽動到各處。

至於什麼東西該屬於 Model 這件事情，其實可以經過團隊的討論再來決定，因為有時候界線真的很模糊，能夠找到共識是更重要的事情！

# Plain Old Ruby Object

這也是一種減少肥胖 Model 的方式，利用 Ruby 本身的類別以及模組的特型來完成任務，因為有些邏輯本身就不屬於 Model 的範疇，甚至連關聯都扯不上邊。

例如：計算、讀檔、通知、串接金流等等

這些功能，其實都是應該要有自己的家，而不是通通寄生在某個 Model 身上～

其實我覺得和前面提過的 View Helper 以及 Concern 有異曲同工之妙，但 Concern 以及 View Helper 本來就是和我們本身應用程式的邏輯有所關聯，而 PORO 這件事所提供的更像是某種外部的共通服務！


{{< tabbed-codeblock "PORO" >}}
<!-- tab ruby -->
# Service Object 像是提供某種服務
class "某種服務的名稱"
  def initialize(...)
    # 其實不一定需要 new 的方法
  end
  
  def send
     ....
  end
end

# 如果想要有 Model 的特殊效果，驗證，可以 include ActiveModel::Model
class "某種服務的名稱"
  include ActiveModel::Model

  def initialize(...)
    # 其實不一定需要 new 的方法
  end
  
  def send
     ....
  end
end

# 甚至你的服務上面還可以有個統稱？
module TopLevelService
  class "某種服務的名稱"
  include ActiveModel::Model

  def initialize(...)
    # 其實不一定需要 new 的方法
  end
  
  def send
     ....
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

# 小結

關於 MVC 究竟夠不夠用呢？我還是會說：夠！

若是你只是要這個應用程式可以動起來，那真的很夠！

但若是考慮到長期的維護和擴充，我想特殊的服務和沒那麼符合 Model 本質的邏輯應該需要特別的切割出來才對！

Model 更多的還是和資料之間的互動，而不是寫一些實體方法來做別的事情！

這也是我近期在專案開發中學習到的事！

原來讓一個 Model 瘦下來不是一件簡單的事情，是從以前到現在的 Rails 開發者都會碰到，且官方也提供了許多方式來提高程式碼的維護和擴充性！

但更多時候，巨大的專案可能就需要使用到 PORO 的方式來整理服務內容！





