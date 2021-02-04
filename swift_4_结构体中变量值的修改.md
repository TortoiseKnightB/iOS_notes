# swift 结构体的一些注意事项

&ensp;&ensp;&ensp;&ensp;Swift 中函数中定义的参数一般默认是参量 let 类型，如果需要修改参数的值，需要添加关键字`inout`，如：
```swift
func changeInt(_ num: inout Int) {
    num += 1
}

var num: Int = 1
changeInt(&num)
print(num)		// 2
```

&ensp;&ensp;&ensp;&ensp;Swift 的结构体中可以定义函数，如果该函数实现了对自身的修改，那么需要在函数定义语句前加上 `mutating`关键字
```swift
struct Stu {
    var name: String
    mutating func changeName(_ newName: String) {
        self.name = newName
    }
}

var stu1: Stu = Stu(name: "张三", isInSchool: false)
print(stu1.name)
stu1.changeName("李四")
print(stu1.name)

// 张三
// 李四
```
&ensp;&ensp;&ensp;&ensp;下面再来谈论一下深拷贝与浅拷贝
&ensp;&ensp;&ensp;&ensp;在 Swift 中，struct 类型是深拷贝，将原结构体中所有变量都复制一份
```swift
var stu1: Stu = Stu(name: "张三")
var stu2 = stu1
print(stu1.name)
print(stu2.name)
stu1.name = "李四"
print(stu1.name)
print(stu2.name)
// 张三
// 张三
// 李四
// 张三
// stu1 的改变不影响 stu2
```

&ensp;&ensp;&ensp;&ensp;class 类型是浅拷贝，只拷贝了原对象的内存地址，两个对象指向同一个内容
```swift
class Book {
    var name: String
    init(_ name: String) {
        self.name = name
    }
}

var book1: Book = Book("三国演义")
var book2 = book1
print(book1.name)
print(book2.name)
book1.name = "西游记"
print(book1.name)
print(book2.name)
// 三国演义
// 三国演义
// 西游记
// 西游记
// book1 的改变引起了	book2 的改变
```



&ensp;&ensp;&ensp;&ensp;再结合斯坦福 Swift 课程的代码来看一下如何实现卡片翻转
```swift
//  MemoryGame.swift
import Foundation

struct MemoryGame<CardContent> {
    var cards: Array<Card>
    
    mutating func choose(card: Card){       // 该函数改变了自身的值，应该加上 mutating 关键字
        print("card chosen: \(card)")
//        card.isFaceUp = !card.isFaceUp    // 无法直接修改
        // 所以我们创建一个指向数组 cards 的链接
        let chosenIndex: Int = self.index(of: card)
				// 卡片翻转
        self.cards[chosenIndex].isFaceUp = !self.cards[chosenIndex].isFaceUp
    }
    
    // 查找该卡片的在数组中的下标，因为 Card 是 Identifiable 类型，所以直接返回 id 即可
    func index(of card: Card) -> Int {
        for index in 0..<self.cards.count{
            if self.cards[index].id == card.id{
                return index
            }
        }
        return 0 // TODO: bogus ! 
    }
    
    // init 默认加上 mutating
    init(numberOfPairsOfCards: Int, cardContentFactory: (Int) -> CardContent) {
        cards = Array<Card>()
        for pairIndex in 0..<numberOfPairsOfCards{
            let content = cardContentFactory(pairIndex)
            cards.append(Card(content: content, id: pairIndex*2))
            cards.append(Card(content: content, id: pairIndex*2+1))
        }
    }
    
    struct Card: Identifiable {
        var isFaceUp: Bool = true
        var isMatched: Bool = false
        var content: CardContent
        var id: Int
    }
}
```

&ensp;&ensp;&ensp;&ensp;先创建一个函数，通过匹配卡片的 id 找到用户点击的卡片的下标`chosenIndex`，再通过语句`self.cards[chosenIndex].isFaceUp = !self.cards[chosenIndex].isFaceUp`实现卡片翻转

