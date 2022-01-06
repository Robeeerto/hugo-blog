---
title: "Day 16 Matcher 介紹 (上)"
date: 2021-09-30T19:15:41+08:00
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

介紹 Matcher 的上集
<!--more-->
{{< toc >}}
今天進入介紹 `Matcher` 的第一天！

會介紹一些常用，也有可能不常用的東西～

首先我想先介紹一個超基本的用法 `not_to`，基本上就是把我們常用的 `to` 替代掉而已，至於功能是什麼，我想字面上應該是非常的清楚！

但我們還是用一個小小的範例來補充一下：

{{< tabbed-codeblock "Example" >}}
<!-- tab ruby -->
RSpec.describe "not_to" do
  it "will be pass" do
    expect(1+1).not_to eq(3)
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

就非常簡單的反義，實用點數五顆星！

# Comparison matchers

比較用的 `Mathcer` 這也是使用度非常高的一個，使用的方法也非常簡單。

就是大於、小於、這類很直覺的東西！

我們用可以用很多的範例來讓看看，順便練習我們之前提過 subject 的 `一行流`寫法！

{{< tabbed-codeblock "One Line Syntax" >}}
<!-- tab ruby -->
RSpec.describe "Comparison matchers" do
  describe 50 do
    it { is_expected.to be > 40 }
    it { is_expected.to be >= 50 }
    it { is_expected.to be < 500 }
    it { is_expected.to be <= 50 }
    it { is_expected.not_to be > 55 }
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

他可以接受所有 Ruby 的比較符號，那這個範例中，也有用到 subject 的概念，因為我們 `describe` 後面接的不是字串，是整數！

所以他會在 block 裡面生成：

```ruby=
let(:subject) { 50 }
```

而下面的 一行流 寫法也是 RSpec 提供給我們的語法！

# Predicate matchers

predicate method 在 Ruby 的世界中，都泛指回傳 boolean 的方法！

而在 Ruby 的慣例中，predicate method 通常都會用 `?` 做結尾。

我覺得這點真的很貼心，也很好識別！

用一些範例來讓大家看看！

{{< tabbed-codeblock "predicate method" >}}
<!-- tab ruby -->
> p 0.zero?
> true
> p 15.odd?
> true
> p [].empty?
> true
<!-- endtab -->
{{</ tabbed-codeblock>}}

我的破英文都看得很有感了，相信一般人看到也會覺得好貼心啊！！！

所以接下來要介紹的是在 RSpec 中的 predicate matcher，其實大同小異，但還是做點範例～

有些人會把 predicate method 寫在 expect 裡面，像這樣：

{{< tabbed-codeblock "RSpec predicate" >}}
<!-- tab ruby -->
RSpec.describe "predicate method" do
  it "use ruby method" do
    expect(16.even?).to eq(true)
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

這樣也沒有什麼問題，但就是少了 RSpec 的浪漫，所以我們要用他幫我們準備好的方法來改寫看看

{{< tabbed-codeblock "RSpec predicate" >}}
<!-- tab ruby -->
RSpec.describe "predicate matchers" do
  it "use rspec matcher" do
    expect(16).to be_even
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

這樣要測試的效果其實和上面一樣！

所有 Ruby 的 predicate method 都可以直接無痛的轉移成 Rspec 的 matcher！

{{< tabbed-codeblock "Ruby to RSpec" >}}
<!-- tab ruby -->
> odd? -> be_odd
> empty? -> be_empty
> zero? -> be_zero
<!-- endtab -->
{{</ tabbed-codeblock>}}

就是把問號拔掉，前面用 `be_` 串接上去，就能做到一樣的效果！而且閱讀起來又更 RSpec 一點～

甚至你也可以用更像英文的寫法來寫， `be_a_` & `be_an_` 都是可以的用法喔！看看範例：

{{< tabbed-codeblock "start_with_be" >}}
<!-- tab ruby -->
RSpec.describe "read more naturally" do
  it "test a string" do
    expect("string").to be_an_instance_of(String)
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

效果等同於：

```ruby=
"string".instance_of?(String)
# true
```

但要記得，寫 Rails 時，若是自己定義的 predicate method 是 private 的話，就沒辦法使用喔！會造成 error ～

# 小結

今天先介紹了兩種非常實用的 matcher，明天也會接著繼續介紹！





