# swift闭包，简化代码

&ensp;&ensp;本人最近学习2020斯坦福 swift 语言教程（在 bilibili或 youtube 搜索 CS193p 即可），Lecture 2 中初步展示了如何用闭包来简化 swift 的代码，使代码看起来更加精炼。这里对课程中出现的代码简化稍作总结：

&ensp;&ensp;这是 MVVM 模型中的 model 部分，负责实现卡片的内容。注意到 MemoryGame 的初始化函数需要两个参数，其中一个为整型 numberOfPiarsOfCards，另一个为函数 cardContentFactory（需要一个 Int 型参数，返回 CardContent 型的结果）

```swift
struct MemoryGame<CardContent> {
	  // 卡片数组
    var cards: Array<Card>

    func choose(card: Card) {
        print("card chosen:\(card)")
    }

  	// 初始化卡片数组
    init(numberOfPiarsOfCards: Int, cardContentFactory: (Int) -> CardContent) {
        cards = Array<Card>()
        for pairIndex in 0..<numberOfPiarsOfCards {
            let content = cardContentFactory(pairIndex)
            cards.append(Card(content: content, id: pairIndex * 2))
            cards.append(Card(content: content, id: pairIndex * 2 + 1))
        }
    }

  	// 卡片
    struct Card: Identifiable {
        var isFaceUp: Bool = true
        var isMatched: Bool = true
        var content: CardContent
        var id: Int
    }
}
```

&ensp;&ensp;这是MVVM模型中VM部分的代码块，负责将数据传入 model 部分。EmojiMemoryGame 类中定义了变量 model，类型为 MemoryGame\<String\>。第一个传入的参数假设为 2，第二个参数为我们自定义的外部函数 createCardContent

```swift
// 外部函数，第二个参数
func createCardContent(pairIndex: Int) -> String {
    return "👻"
}

class EmojiMemoryGame {
    private var model: MemoryGame<String> = MemoryGame<String>(numberOfPiarsOfCards: 2, cardContentFactory: createCardContent)

    // Access to Model
    var cards: Array<MemoryGame<String>.Card> {
        model.cards
    }

    // Intent(s)
    func choose(card: MemoryGame<String>.Card) {
        model.choose(card: card)
    }
}
```

&ensp;&ensp;第二个参数 createCardContent 可以直接写到变量 model 的定义语句中，如下

```swift
class EmojiMemoryGame {
  	// 外部函数作为参数，内联到定义语句内
    private var model: MemoryGame<String> =
            MemoryGame<String>(numberOfPiarsOfCards: 2, cardContentFactory: { (pairIndex: Int) -> String in
                return "👻"
            })
    
    // Access to Model
    var cards: Array<MemoryGame<String>.Card> {
        model.cards
    }

    // Intent(s)
    func choose(card: MemoryGame<String>.Card) {
        model.choose(card: card)
    }
}
```

&ensp;&ensp;因为作为第二个参数的函数只有一个返回语句，所以 return 可以省略。而且最后一个参数可以不写参数名，可以直接用 {} 调用语句，修改后如下

```swift
private var model: MemoryGame<String> =
        MemoryGame<String>(numberOfPiarsOfCards: 2) { (pairIndex: Int) -> String in
            "👻"
        }
```

&ensp;&ensp;由于第二个参数函数的传入类型 Int 和返回值类型 String 可以从 MemoryGame 推测出来，因此可以进一步省略，修改如下

```swift
private var model: MemoryGame<String> =
        MemoryGame<String>(numberOfPiarsOfCards: 2) { pairIndex in "👻"}
```

&ensp;&ensp;到这里代码已经足够精简了。如果参数函数没有用到 pairIndex，甚至可以进一步将 “pairIndex” 改成 “_”