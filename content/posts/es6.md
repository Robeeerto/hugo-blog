---
title: "寫 React 時常常使用的語法"
date: 2021-08-22T22:38:11+08:00
metaAlignment: center
coverImage: "https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202108231629312.jpg" 
coverMeta: out
coverSize: partial
categories:
- JavaScript
tags:
- Syntax
---

近期開始寫 React，有好多寫法好陌生，記錄一下！
<!--more-->

{{< toc >}}


# 箭頭函式

下方是初學者一開始會學到的基礎函式！
{{< tabbed-codeblock JS範例 >}}
<!-- tab js -->
function showMeTheMoney(money){
  return("Yo! I got $" + money + "dollars")
}
console.log(showMeTheMoney(1000))
// Yo! I got $1000 dollars
<!-- endtab -->
{{</ tabbed-codeblock >}}

而下面這個呢？ 登登登！ 就是箭頭函式啦

故名思義它擁有一個箭頭， ES6 讓我們用更簡短的方式來定義函式

Tips！如果是單一一行的話會直接 `return` 結果，我們就不用特別定義

還有啊！若是單一參數可以不需要加上小括號唷

{{< tabbed-codeblock JS範例 >}}
<!-- tab js -->
const showMeTheMoney = money => `Yo! I got $${money} dollars`
console.log(showMeTheMoney(1000))
//  Yo! I got $1000 dollars
<!-- endtab -->
{{</ tabbed-codeblock >}}

---
# 字串樣版
剛剛在上方的示範中會看到我們在箭頭函式裡面回傳的字串中穿插了變數！

奇怪，以前的寫法明明就是要像下面這樣啊

我們要分隔字串和變數在做相加不是嗎？
{{< tabbed-codeblock JS範例 >}}
<!-- tab js -->
const a = 5
const b = 10
console.log('我有' + a + '元，你有' + 10 + '元');
//我有 5 元，你有 10 元
<!-- endtab -->
{{</ tabbed-codeblock >}}

現在 ES6 提供給我們更強大的穿插變數的方式！

{{< tabbed-codeblock JS範例 >}}
<!-- tab js -->
const a = 5
const b = 10
console.log(`我有${a}元，你有${b}元`)
//我有 5 元，你有 10 元
<!-- endtab -->
{{</ tabbed-codeblock >}}
也就是在字串中把變數放入`${}`的符號裡面，就能夠正常的顯示你要的內容

是不是非常方便呢！

---
# 物件屬性名稱縮寫

下面是舊版的物件屬性名稱寫法：
{{< tabbed-codeblock JS範例 >}}
<!-- tab js -->
const likeFood = 'apple'
const likeDrink = 'beer'
const likeShow = 'MLB'
const Me = {
  likeFood: likeFood,
  likeDrink: likeDrink,
  likeShow: likeShow
}
<!-- endtab -->
{{</ tabbed-codeblock>}}
而下面這個呢，就是 ES6 新的物件屬性名稱縮寫！
{{< tabbed-codeblock JS範例 >}}
<!-- tab js -->
const likeFood = 'apple'
const likeDrink = 'beer'
const likeShow = 'MLB'
const Me = {
  likeFood,
  likeDrink,
  likeShow
}
//Me.likeFood = 'apple'
<!-- endtab -->
{{< /tabbed-codeblock >}}
有看出什麼差別嗎？ 最大的不同就在於 `key & value` 若是相同命名就可以節省成一個喔

---
# 解構賦值
這個是我一開始最難理解也最看不懂的，但懂了之後真的很方便

若是以前我們要提取某個物件的值：
{{< tabbed-codeblock JS範例 >}}
<!-- tab js -->
const props = {
  name: 'Robert',
  phoneNumber: '123456'
}
const name = props.name
const phoneNumber = props.phoneNumber
console.log(name) // Robert
console.log(phoneNumber) // 123456
<!-- endtab -->
{{</ tabbed-codeblock>}}

下面是 ES6 提供的解構賦值：
{{< tabbed-codeblock JS範例 >}}
<!-- tab js-->
const props = {
  name: 'Robert',
  phoneNumber: '123456'
}
const { name, phoneNumber } = props
console.log(name) // Robert
console.log(phoneNumber) // 123456
<!-- endtab -->
{{</ tabbed-codeblock>}}
看出差別了嗎？我們可以把兩行的定義，變成一行，若是你要定義 5 個不同的變數

這會是你最好的朋友！

應用在函式裡面也非常厲害，把一個物件直接傳進去，擷取我們需要的值就可以了！

---
# 展開符號

這個新的符號也非常厲害和實用，不儘可以展開物件或是陣列

同時還保有原有的值，等於是複製一份過去的概念，超級無敵實用！
{{< tabbed-codeblock JS範例 >}}
<!-- tab js -->
const robert = {
  name: 'Robert',
	phone: '123456'
}
const newRobert = {
  ...robert,
  hair: 'short',
  skin: 'black'
}
console.log(newRobert)
// {name: 'Robert', phone: '123456', hair: 'short', skin: 'black'}
<!-- endtab -->
{{</ tabbed-codeblock>}}

---
# 其餘符號

和展開符號長得一樣，一樣的好處是可以讓我們不影響傳進來的參數

進而指提取我們需要的部分，甚至拿來切割陣列分別做操作也可以，實用推推
{{< tabbed-codeblock JS範例 >}}
<!-- tab js -->
const arrray = [1,2,3,4,5,6]
const [one, two, ...others] = arrray
console.log(one) = 1
console.log(two) = 2
console.log(others) = [3,4,5,6]
<!-- endtab -->
{{</ tabbed-codeblock>}}



