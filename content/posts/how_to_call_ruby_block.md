---
title: "如何呼叫 Ruby 的 block"
date: 2022-03-15T00:05:07+08:00
metaAlignment: center
thumbnailImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202203140228710.png'
coverImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202203150006741.jpg'
coverMeta: out
coverSize: partial
categories:
- Ruby
tags:
- Syntax
summary: 呼叫 block 的方式
keywords:
- Ruby
- yield
- call
- block
---

{{< toc >}}

Ruby 的 block 可以搞得很複雜，其中要完全理解 block 的障礙就在於它超過了一種的方式去呼叫。

今天的文章會介紹兩種最常用來呼叫 block 的方式，分別是：`block.call` & `yield`。

也有其他的方式可以呼叫 block，像是 `instance_exec`，但因為應用的方式比較動態，怕會離題，改天在寫一篇來研究一下！

所以開始進入正題來介紹最常用來呼叫 block 的兩種方式吧！

# 第一種方式 block.call

下面是一個方法，接受的參數是一個 block

{{< tabbed-codeblock call block >}}
<!-- tab ruby -->
def hi(&block)
  block.call 
end

hi { p 'hi!' }
<!-- endtab -->
{{</ tabbed-codeblock>}}

如果你執行上面這個方法，會在終端機中看到 hi! 被印出！

你一定會想知道在 `&block` 前面的 `&` 是什麼！

簡單來說這個 `&` 就是把 `block` 轉換成 `Proc` 物件，因為 block 不能直接被 `.call` 方法呼叫，必須要先被轉換成 `Proc` 物件，才能使用 `.call` 方法！

之後也會補上關於 `&block` 詳細的文章以及 `Proc` 物件的說明！

# 第二種方式 yield

下面這個方法其實和上面寫的一模一樣，只差在 `.call` 這個呼叫的方式，而是改用 `yield`

{{< tabbed-codeblock yield >}}
<!-- tab ruby -->
def hi(&block)
  yield
end

hi { p 'hi!' }
<!-- endtab -->
{{</ tabbed-codeblock>}}

你可能會想，啊都已經有一個 `.call` 方法可以使用，Ruby 幹嘛來要提供第二個完全不同的方式來呼叫 block？ 

其中有一個原因就是 `yield` 給了一個能力是 `block.call` 沒有的，在下面的例子裡，我們定義了一個方法然後傳了 block 進去，但我們卻沒有明確地指示這個方法放入 block 的參數。

{{< tabbed-codeblock no_argument_version >}}
<!-- tab ruby -->
def hi
  yield
end

hi { p 'hi!' }
<!-- endtab -->
{{</ tabbed-codeblock>}}

如你所見，`yield` 給了特殊能力，讓我們去呼叫 block 卻不需要明確地指示或接收 block ( 小提示： Ruby 的所有方法都可以被傳入 block，即使方法本身沒有指示 )

這邊又有一個有趣的問題來了，那我們幹嘛不永遠都使用 `yield` 就好了呢？

答案是，當你使用 `block.call`，你可以把 block 傳到你想用的方法裡面，但這是用 yield 辦不到的

當把 `&block` 傳入一個方法的 `signature` 時，我們可以對這個 block 做更多不同的事，而不是單純的用 `block.call` 而已，舉個例子，我們可以選擇不在這個方法呼叫他，而是在被傳入的另一個方法中呼叫，有點像是一層一層傳遞 block 的概念。

# 總結

1. 大部分用來呼叫 Ruby block 的方式是 `yield` 和 `block.call`
2. 不像是 `block.call`，`yield` 可以讓我們的方法不需要特別地指示和接收 block
3. 使用 `block.call` 的好處在於，我們明確地指示 block 的位置，讓他可以自由地穿梭在各個方法之間！
4. 留了三個坑 `instance_exec` 以及 `Proc Object` 和 `& 在 block 前面的作用`，改天在寫～





